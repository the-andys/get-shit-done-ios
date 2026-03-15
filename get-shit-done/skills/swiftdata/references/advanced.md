<overview>
Advanced SwiftData features: indexing (iOS 18+), class inheritance (iOS 26+), custom data stores, and schema migration patterns. Read when working with iOS 18+ or iOS 26+ features, optimizing query performance, or designing complex model hierarchies. Related: core-rules.md (base patterns), predicates.md (predicate safety — especially for inheritance filtering).
</overview>

## Indexing (iOS 18+)

Indexes speed up queries at a small write cost. Use when data is read often and updated less frequently.

### Single Property Index

```swift
@Model class Article {
    #Index<Article>([\.type], [\.author])

    var type: String
    var author: String
    var publishDate: Date

    init(type: String, author: String, publishDate: Date) {
        self.type = type
        self.author = author
        self.publishDate = publishDate
    }
}
```

### Compound Index (Properties Used Together)

```swift
#Index<Article>([\.type], [\.type, \.author])
```

Compound indexes are more efficient when you frequently filter on both properties together.

### When NOT to Index

- Data written frequently, read rarely (logging, analytics)
- Small datasets where scan is fast enough
- Properties rarely used in predicates

## Class Inheritance (iOS 26+)

SwiftData supports model subclassing starting in iOS 26. Both base and subclass MUST have `@Model`.

### Basic Pattern

```swift
@Model class Trip {
    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date

    init(name: String, destination: String, startDate: Date, endDate: Date) {
        self.name = name
        self.destination = destination
        self.startDate = startDate
        self.endDate = endDate
    }
}

@available(iOS 26, *)
@Model class BusinessTrip: Trip {
    var purpose: String
    var expenseCode: String

    @Relationship(deleteRule: .cascade, inverse: \BusinessMeal.trip)
    var businessMeals: [BusinessMeal] = []

    init(name: String, destination: String, startDate: Date, endDate: Date,
         purpose: String, expenseCode: String) {
        self.purpose = purpose
        self.expenseCode = expenseCode
        super.init(name: name, destination: destination, startDate: startDate, endDate: endDate)
    }
}

@available(iOS 26, *)
@Model class PersonalTrip: Trip {
    var reason: Reason
    var notes: String?

    enum Reason: String, CaseIterable, Codable {
        case family, vacation, wellness, other
    }

    init(name: String, destination: String, startDate: Date, endDate: Date,
         reason: Reason, notes: String? = nil) {
        self.reason = reason
        self.notes = notes
        super.init(name: name, destination: destination, startDate: startDate, endDate: endDate)
    }
}
```

### Critical Rules

1. **`@available` is REQUIRED** on subclasses, even when iOS 26 is the deployment target
2. **List both parent and subclasses** in ModelContainer schemas — SwiftData cannot infer connections
3. **Keep hierarchies shallow** (1-2 levels) — deep hierarchies complicate migrations
4. **Relationships are polymorphic** — a `[Trip]` relationship may contain `BusinessTrip` or `PersonalTrip`

### Container Setup

```swift
let container = try ModelContainer(for: Trip.self, BusinessTrip.self, PersonalTrip.self)
```

### Querying with Inheritance

```swift
// All trips (includes subclasses automatically)
@Query var allTrips: [Trip]

// Only business trips
@Query(filter: #Predicate<Trip> { $0 is BusinessTrip }) var businessTrips: [Trip]

// Filter by subclass properties using type casting
@Query(filter: #Predicate<Trip> { trip in
    if let personal = trip as? PersonalTrip {
        personal.reason == .vacation
    } else {
        false
    }
}) var vacations: [Trip]
```

**Note:** The result array type is the base class. Use `as?` to access subclass properties.

### When to Use Inheritance vs Alternatives

| Use Inheritance When | Use Enum/Protocol Instead |
|---------------------|--------------------------|
| True IS-A relationship | Only a few differing fields |
| Need to query across all types AND by specific type | Only query by specific type |
| Subclasses have significantly different properties | Types share 90%+ of properties |
| Polymorphic relationships are needed | Simple type distinction |

### Enum Alternative

```swift
// Often simpler than inheritance
@Model class Trip {
    var name: String
    var tripType: TripType

    enum TripType: String, Codable {
        case business, personal, vacation
    }
}
```

## Custom Data Stores (iOS 18+)

iOS 18 introduced `DataStoreConfiguration` for custom backing stores:

```swift
let customConfig = DataStoreConfiguration(/* custom store */)
let container = try ModelContainer(for: Item.self, configurations: customConfig)
```

Use when you need a non-SQLite backing store (e.g., JSON files, custom binary format). Most apps don't need this.

## Schema Migration

### Lightweight (Automatic)

SwiftData handles these automatically:
- Adding properties with default values
- Removing properties
- Renaming with `@Attribute(originalName: "oldName")`

### Versioned Migration

```swift
enum SchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)
    static var models: [any PersistentModel.Type] { [ItemV1.self] }
}

enum SchemaV2: VersionedSchema {
    static var versionIdentifier = Schema.Version(2, 0, 0)
    static var models: [any PersistentModel.Type] { [ItemV2.self] }
}

let plan = SchemaMigrationPlan(schemas: [SchemaV1.self, SchemaV2.self])
```

### Best Practice

Always have an explicit migration schema, even for lightweight migrations. It documents the evolution of your data model and prevents surprises.

## History Tracking (iOS 18+)

```swift
// Fetch history of changes
let history = try context.fetchHistory(after: lastSyncToken)
```

Useful for incremental sync, undo, and audit trails.
