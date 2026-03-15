<overview>
CloudKit-specific constraints for SwiftData. These rules ONLY apply when the project uses SwiftData with CloudKit sync. Read when adding iCloud sync or reviewing code that uses cloudKitDatabase configuration. Related: core-rules.md (general SwiftData rules), advanced.md (indexing supports CloudKit).
</overview>

## Configuration

Enable CloudKit sync in the container:

```swift
let config = ModelConfiguration(cloudKitDatabase: .automatic)
let container = try ModelContainer(for: TaskItem.self, configurations: config)
```

Or with `.modelContainer`:

```swift
WindowGroup { ContentView() }
    .modelContainer(for: TaskItem.self, inMemory: false, isAutosaveEnabled: true)
```

For CloudKit, you also need:
- CloudKit capability in Signing & Capabilities
- iCloud container configured in the developer portal
- Background Modes → Remote Notifications enabled

## Strict Constraints

### No Unique Constraints

```swift
// WRONG — @Attribute(.unique) is NOT supported with CloudKit
@Model class User {
    @Attribute(.unique) var email: String  // Will break CloudKit sync
}

// CORRECT — handle uniqueness in app logic
@Model class User {
    var email: String
    // Deduplicate on sync: check for existing before insert
}
```

`#Unique` is also NOT supported with CloudKit.

### All Properties Must Have Defaults or Be Optional

```swift
// WRONG — non-optional without default
@Model class Item {
    var title: String  // No default, not optional — CloudKit will fail
}

// CORRECT
@Model class Item {
    var title: String = ""  // Default value
    var subtitle: String?   // Or optional
}
```

### All Relationships Must Be Optional

```swift
// WRONG — non-optional relationship
@Model class Item {
    var folder: Folder  // Must be optional for CloudKit
}

// CORRECT
@Model class Item {
    var folder: Folder?
}
```

### Array Relationships Are Acceptable

Array relationships (to-many) are supported since they default to empty:

```swift
@Relationship(deleteRule: .cascade) var items: [Item] = []  // OK
```

## Eventual Consistency

CloudKit is designed for **eventual consistency**. Code must function when data hasn't synced yet.

### Design Principles

- Never assume data is immediately available after creation on another device
- Handle missing relationships gracefully (they may not have synced yet)
- UI should indicate sync status when relevant
- Conflict resolution: last-write-wins is CloudKit's default

### Common Patterns

```swift
// Show placeholder while waiting for sync
if items.isEmpty {
    ContentUnavailableView("Syncing...", systemImage: "icloud.and.arrow.down")
}

// Don't crash on missing relationships
if let folder = item.folder {
    Text(folder.name)
} else {
    Text("Uncategorized")  // Handle not-yet-synced
}
```

## Supported Features with CloudKit

| Feature | Status |
|---------|--------|
| @Model classes | Supported |
| Relationships (optional) | Supported |
| @Attribute(.externalStorage) | Supported |
| Indexing (iOS 18+) | Supported |
| Class inheritance (iOS 26+) | Supported |
| @Attribute(.unique) / #Unique | **NOT supported** |
| Non-optional properties without defaults | **NOT supported** |
| Non-optional relationships | **NOT supported** |

## Testing CloudKit Sync

Testing CloudKit requires a real iCloud account and network. For unit tests:

```swift
// Use in-memory store for unit tests (no CloudKit)
let config = ModelConfiguration(isStoredInMemoryOnly: true)
let container = try ModelContainer(for: TaskItem.self, configurations: config)
```

For integration testing with CloudKit, use a dedicated iCloud container and test on physical devices.
