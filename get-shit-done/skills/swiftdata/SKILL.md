---
name: swiftdata
description: SwiftData persistence — @Model, predicates, CloudKit, migrations, class inheritance, FetchDescriptor
---

<essential_principles>
## How This Skill Works

1. **@Model for all persistent types.** SwiftData uses the `@Model` macro. All stored properties are automatically persisted.
2. **Predicates must be safe.** Some predicates compile but crash at runtime. Always verify predicate safety before writing.
3. **CloudKit has strict constraints.** No `@Attribute(.unique)`, all properties need defaults or be optional, relationships must be optional.
4. **Explicit delete rules.** Always specify `@Relationship(deleteRule:)` — the default `.nullify` can orphan objects or crash on non-optionals.
5. **Class inheritance is iOS 26+.** Powerful but not always the right tool — consider enums/protocols first.
</essential_principles>

<intake>
## What do you need?

1. Define a new model or relationship
2. Review existing SwiftData code for issues
3. Write or fix a predicate
4. Add CloudKit sync
5. Set up class inheritance (iOS 26+)
6. Migrate schema or add indexing (iOS 18+)
</intake>

<routing>
| Response | Reference |
|----------|-----------|
| 1, "model", "relationship", "@Model", "container", "CRUD" | `references/core-rules.md` |
| 2, "review", "check", "issues", "fix" | `references/core-rules.md` + `references/predicates.md` |
| 3, "predicate", "#Predicate", "query", "filter", "crash" | `references/predicates.md` |
| 4, "CloudKit", "iCloud", "sync" | `references/cloudkit.md` |
| 5, "inheritance", "subclass", "iOS 26" | `references/advanced.md` |
| 6, "migration", "schema", "index", "iOS 18" | `references/advanced.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/core-rules.md | @Model, autosaving, relationships, delete rules, FetchDescriptor, ModelContainer, @Query |
| references/predicates.md | Safe operations, CRASH patterns, unsupported methods, string matching |
| references/cloudkit.md | Uniqueness constraints, optionality, eventual consistency, sync rules |
| references/advanced.md | Indexing (iOS 18+), class inheritance (iOS 26+), custom data stores, schema migration |
</reference_index>

<canonical_terminology>
## Terminology

- **@Model** (not: NSManagedObject, Entity)
- **ModelContainer** (not: NSPersistentContainer)
- **ModelContext** (not: NSManagedObjectContext)
- **#Predicate** (not: NSPredicate, NSCompoundPredicate)
- **FetchDescriptor** (not: NSFetchRequest)
- **@Query** (not: @FetchRequest)
- **@Attribute** (not: @NSManaged)
</canonical_terminology>
