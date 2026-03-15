<overview>
Naming conventions for all Swift types, properties, methods, files, and enums. Read when writing new code, reviewing names, or resolving naming inconsistencies. Related: code-structure.md (file organization), anti-patterns.md (naming-related smells).
</overview>

## Types (PascalCase)

All type declarations use PascalCase — structs, classes, enums, protocols, actors, typealiases.

```swift
struct UserProfile { }
class NetworkManager { }
enum PaymentStatus { }
protocol DataProviding { }
actor ImageDownloader { }
typealias UserID = String
```

### Protocol Naming

Use `-ing`, `-able`, or `-Providing` suffixes for capability protocols. Use noun form for "is-a" protocols.

```swift
// Capability protocols (what it can do)
protocol Loadable { }
protocol DataProviding { }
protocol Configurable { }

// Identity protocols (what it is)
protocol Animal { }
protocol Vehicle { }
```

## Properties, Methods, Variables (camelCase)

```swift
let userName: String
var isLoading = false
func fetchUserProfile() async throws -> UserProfile { }
func didTapSubmitButton() { }
```

### Boolean Naming

Booleans read as assertions: `is`, `has`, `should`, `can`.

```swift
var isLoading = false
var hasUnreadMessages: Bool { !messages.filter(\.isUnread).isEmpty }
var shouldShowOnboarding: Bool { !hasCompletedSetup }
var canSubmit: Bool { !title.isEmpty && isValid }
```

## Constants

Module-level constants use camelCase with `let`. No `k` prefix. No `SCREAMING_CASE`.

```swift
let maxRetryCount = 3
let defaultTimeout: TimeInterval = 30
let apiBaseURL = URL(string: "https://api.example.com")!
```

## Enum Cases (camelCase)

```swift
enum PaymentStatus {
    case pending
    case completed
    case failedWithError(Error)
    case refunded(amount: Decimal)
}
```

## Property Wrappers

Property wrappers keep the `@` prefix and use their standard names:

```swift
@State private var isPresented = false
@Binding var userName: String
@Environment(\.dismiss) private var dismiss
@Environment(\.modelContext) private var modelContext
@Observable class UserStore { }
@AppStorage("hasCompletedOnboarding") private var hasCompletedOnboarding = false
```

## File Naming

| Type | Convention | Example |
|------|-----------|---------|
| SwiftUI View | `{Name}View.swift` | `ProfileView.swift` |
| ViewModel | `{Name}ViewModel.swift` | `ProfileViewModel.swift` |
| Model | `{Name}.swift` | `UserProfile.swift` |
| Protocol | `{Name}.swift` or `{Name}Protocol.swift` | `DataProviding.swift` |
| Extension | `{Type}+{Purpose}.swift` | `String+Validation.swift` |
| Tests | `{Name}Tests.swift` | `ProfileViewModelTests.swift` |
| Modifier | `{Name}Modifier.swift` | `CardModifier.swift` |

## Argument Labels

Follow Swift API Design Guidelines for fluent call sites:

```swift
// Read naturally at the call site
func move(from source: Point, to destination: Point)
func insert(_ element: Element, at index: Int)
func removeItems(matching predicate: (Item) -> Bool)

// Omit first label when method name describes the role
func contains(_ element: Element) -> Bool
func append(_ newElement: Element)

// Use prepositions for disambiguation
func fade(to opacity: Double, duration: TimeInterval)
```

### Label Fluency Rules

- **Omit first label** when the method name makes the argument's role clear: `append(_:)`, `contains(_:)`
- **Use prepositions** (`to`, `from`, `at`, `with`, `by`) when they clarify relationships: `move(from:to:)`
- **Use `of`/`for`** for lookup operations: `value(forKey:)`, `index(of:)`
- **Avoid meaningless labels** like `and`, `with` when they add no information

## MARK Structure

Standard ordering within a type:

```swift
struct ProfileView: View {
    // MARK: - Environment
    // MARK: - State
    // MARK: - Properties
    // MARK: - Body
    // MARK: - Subviews
    // MARK: - Toolbar
    // MARK: - Actions
}
```

For non-View types:

```swift
class ProfileViewModel {
    // MARK: - Properties
    // MARK: - Init
    // MARK: - Public Methods
    // MARK: - Private Methods
}
```
