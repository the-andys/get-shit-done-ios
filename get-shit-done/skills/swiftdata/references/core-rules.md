<overview>
Core SwiftData rules: @Model macro, autosaving behavior, relationships, delete rules, FetchDescriptor patterns, ModelContainer configuration, @Query usage. Read when writing or reviewing SwiftData models and persistence code. Related: predicates.md (predicate safety), cloudkit.md (sync constraints), advanced.md (indexing, inheritance).
</overview>

## @Model Macro

The `@Model` macro transforms a Swift class into a persistent model. All stored properties are automatically persisted.

```swift
@Model
class TaskItem {
    var title: String
    var isComplete: Bool
    var createdAt: Date

    init(title: String, isComplete: Bool = false) {
        self.title = title
        self.isComplete = isComplete
        self.createdAt = .now
    }
}
```

### Supported Property Types

- Basic: `String`, `Int`, `Double`, `Bool`, `Date`, `UUID`, `URL`, `Data`
- `Codable` types (stored as JSON) — enums with associated values work fine
- Collections: `[String]`, `[Int]`, etc.
- Relationships to other `@Model` types
- Optionals of any above type

### Property Rules

- Do NOT use the property name `description` — it is explicitly disallowed
- Do NOT add property observers — they will be silently ignored
- `@Attribute(.externalStorage)` is a *suggestion*, not requirement — only for `Data`
- `@Transient` properties reset to default on fetch — use computed properties for derived values

## Autosaving

SwiftData autosaves at unpredictable intervals. For correctness-critical operations, call `save()` explicitly. No need to check `hasChanges` first.

```swift
context.insert(newItem)
try context.save()  // Explicit save for correctness
```

## Relationships

### Delete Rules (ALWAYS specify)

```swift
@Relationship(deleteRule: .cascade) var items: [Item] = []
```

| Rule | Behavior | When |
|------|----------|------|
| `.cascade` | Delete related objects | Parent owns children |
| `.nullify` | Set reference to nil (default) | Shared objects |
| `.deny` | Prevent deletion if related | Foreign key integrity |
| `.noAction` | Do nothing | Use with caution |

**The default `.nullify` can orphan objects or crash if the property is non-optional.** Always specify explicitly.

### Inverse Relationships

SwiftData frequently gets inverse relationships wrong. Always be explicit:

```swift
@Relationship(deleteRule: .cascade, inverse: \Item.folder) var items: [Item] = []
```

Place `@Relationship` on ONE side only. Both sides causes circular reference.

### Uniqueness

```swift
@Model class User {
    #Unique<User>([\.email], [\.username])  // One #Unique per model
    var email: String
    var username: String
}
```

Do NOT write `#Unique` more than once per model. Pass multiple key path arrays in a single declaration.

## Persistent Identifiers

Persistent identifiers are **temporary** before first save. Temporary IDs start with lowercase "t". Always save before relying on an object's ID.

```swift
context.insert(item)
try context.save()  // ID is now permanent
let stableID = item.persistentModelID  // Safe to store/send
```

`ModelContainer` and `PersistentIdentifier` are `Sendable`. Model instances are NOT. To transfer across actors, send the identifier and re-fetch.

## ModelContainer Setup

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [TaskItem.self, Folder.self])
    }
}
```

### Custom Configuration

```swift
let config = ModelConfiguration(
    schema: Schema([TaskItem.self]),
    url: URL.documentsDirectory.appending(path: "MyApp.store"),
    cloudKitDatabase: .automatic
)
let container = try ModelContainer(for: TaskItem.self, configurations: config)
```

## @Query (SwiftUI Views Only)

```swift
@Query var items: [TaskItem]
@Query(sort: \TaskItem.createdAt, order: .reverse) var items: [TaskItem]
@Query(filter: #Predicate<TaskItem> { !$0.isComplete }) var activeItems: [TaskItem]
```

**@Query works ONLY inside SwiftUI views.** For ViewModels or services, use `FetchDescriptor`.

### Dynamic Queries

```swift
struct ItemList: View {
    @Query var items: [TaskItem]

    init(searchText: String) {
        let predicate = #Predicate<TaskItem> { item in
            searchText.isEmpty || item.title.localizedStandardContains(searchText)
        }
        _items = Query(filter: predicate, sort: \.createdAt)
    }
}
```

## FetchDescriptor (Outside Views)

```swift
let descriptor = FetchDescriptor<TaskItem>(
    predicate: #Predicate { $0.isComplete },
    sortBy: [SortDescriptor(\.createdAt)]
)
let completed = try context.fetch(descriptor)
```

### Optimization

- `fetchCount()` for counts (doesn't live-update unless combined with @Query)
- `propertiesToFetch` to limit loaded properties
- `relationshipKeyPathsForPrefetching` for known relationship access

## CRUD Operations

```swift
// Create
let item = TaskItem(title: "New")
context.insert(item)

// Update (auto-tracked)
item.title = "Updated"

// Delete
context.delete(item)

// Save explicitly
try context.save()
```

## Schema Migration

Always have a migration schema, even for lightweight migrations:

```swift
enum SchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)
    static var models: [any PersistentModel.Type] { [TaskItem.self] }
}
```

Lightweight migration handles: adding properties with defaults, removing properties, renaming with `@Attribute(originalName:)`.
