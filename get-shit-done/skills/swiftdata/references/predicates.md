<overview>
SwiftData predicate safety guide: safe operations, CRASH patterns that compile but fail at runtime, unsupported methods, and string matching rules. This is critical knowledge — predicate bugs are the #1 source of SwiftData runtime crashes. Read when writing or reviewing any #Predicate code. Related: core-rules.md (model patterns), advanced.md (predicate filtering with class inheritance).
</overview>

## Safe String Matching

Always use `localizedStandardContains()` for string matching in predicates:

```swift
@Query(filter: #Predicate<Movie> {
    $0.name.localizedStandardContains("titanic")
}) private var movies: [Movie]
```

Do NOT use `lowercased().contains()` — `lowercased()` is not supported in predicates.

### hasPrefix/hasSuffix

`hasPrefix()` and `hasSuffix()` are NOT supported. Use `starts(with:)` instead:

```swift
@Query(filter: #Predicate<Website> {
    $0.url.starts(with: "https://apple.com")
}) private var appleLinks: [Website]
```

## CRASH Patterns (Compile But Fail at Runtime)

These predicates compile cleanly but CRASH at runtime. Catch them in code review.

### isEmpty == false

```swift
// CRASHES at runtime
@Query(filter: #Predicate<Movie> { $0.cast.isEmpty == false })

// CORRECT — use !
@Query(filter: #Predicate<Movie> { !$0.cast.isEmpty })
```

### Computed Properties in Predicates

```swift
// CRASHES — computed properties not backed by database
#Predicate<Task> { $0.ageInDays > 30 }  // ageInDays is computed

// CORRECT — use persisted properties only
#Predicate<Task> { $0.createdAt < threshold }
```

### @Transient Properties in Predicates

```swift
// CRASHES — @Transient is not in the database
#Predicate<Task> { $0.isEditing }  // @Transient var isEditing

// CORRECT — use only stored properties
```

### Regular Expressions

```swift
// CRASHES — regex not supported in predicates
@Query(filter: #Predicate<Movie> {
    $0.name.contains(/Titanic/)
})

// CORRECT — use localizedStandardContains
@Query(filter: #Predicate<Movie> {
    $0.name.localizedStandardContains("Titanic")
})
```

### Custom Codable Struct Data

```swift
// CRASHES — custom Codable structs can't be queried
#Predicate<User> { $0.address.city == "NYC" }  // address is Codable struct

// CORRECT — flatten to stored properties or use separate @Model
```

## Unsupported Methods (Won't Compile)

These are caught at compile time:

- `String.hasSuffix()`
- `String.lowercased()`
- `Sequence.map()`
- `Sequence.reduce()`
- `Sequence.count(where:)`
- `Collection.first`
- Custom operators

## Safe Predicate Operations

| Operation | Status | Example |
|-----------|--------|---------|
| `==`, `!=` | Safe | `$0.name == "Test"` |
| `<`, `>`, `<=`, `>=` | Safe | `$0.count > 5` |
| `&&`, `\|\|`, `!` | Safe | `!$0.isComplete && $0.priority > 0` |
| `.contains()` (on String) | Safe | `$0.name.contains("test")` |
| `.localizedStandardContains()` | Safe (preferred) | `$0.name.localizedStandardContains("test")` |
| `.starts(with:)` | Safe | `$0.name.starts(with: "A")` |
| `.isEmpty` / `!.isEmpty` | Safe | `!$0.items.isEmpty` |
| `.isEmpty == false` | **CRASHES** | Use `!.isEmpty` instead |
| Computed property | **CRASHES** | Use stored properties only |
| `@Transient` property | **CRASHES** | Use stored properties only |
| Regex | **CRASHES** | Use `localizedStandardContains` |

## Testing Predicates

Before deploying a predicate, verify it works:

1. Write the predicate
2. Test with a small dataset in a `#Preview` or test
3. Verify it returns expected results
4. Check that it doesn't crash with edge cases (empty collections, nil optionals)

```swift
@Test("Predicate filters active items")
func activeItemsPredicate() async throws {
    let container = try ModelContainer(for: TaskItem.self, configurations: .init(isStoredInMemoryOnly: true))
    let context = container.mainContext

    context.insert(TaskItem(title: "Active", isComplete: false))
    context.insert(TaskItem(title: "Done", isComplete: true))
    try context.save()

    let descriptor = FetchDescriptor<TaskItem>(predicate: #Predicate { !$0.isComplete })
    let results = try context.fetch(descriptor)
    #expect(results.count == 1)
}
```
