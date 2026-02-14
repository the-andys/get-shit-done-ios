# iOS Swift Guidelines

Cornerstone reference for all GSD agents working on iOS projects. Every agent MUST follow these conventions. When in doubt, this file is the source of truth.

<naming_conventions>

## Naming Conventions

### Types (PascalCase)

All type declarations use PascalCase — structs, classes, enums, protocols, actors, typealiases.

```swift
struct UserProfile { }
class NetworkManager { }
enum PaymentStatus { }
protocol DataProviding { }
actor ImageDownloader { }
typealias UserID = String
```

**Protocols:** Use `-ing`, `-able`, or `-Providing` suffixes for capability protocols. Use noun form for "is-a" protocols.

```swift
// Capability protocols
protocol Loadable { }
protocol DataProviding { }
protocol Configurable { }

// Identity protocols
protocol Animal { }
protocol Vehicle { }
```

### Properties, Methods, Variables (camelCase)

```swift
let userName: String
var isLoading = false
func fetchUserProfile() async throws -> UserProfile { }
func didTapSubmitButton() { }
```

### Constants

Module-level constants use camelCase with `let`. No `k` prefix. No `SCREAMING_CASE`.

```swift
let maxRetryCount = 3
let defaultTimeout: TimeInterval = 30
let apiBaseURL = URL(string: "https://api.example.com")!
```

### Enum Cases (camelCase)

```swift
enum PaymentStatus {
    case pending
    case completed
    case failedWithError(Error)
    case refunded(amount: Decimal)
}
```

### Property Wrappers

Property wrappers keep the `@` prefix and use their standard names:

```swift
// SwiftUI state
@State private var isPresented = false
@Binding var userName: String
@Environment(\.dismiss) private var dismiss
@Environment(\.modelContext) private var modelContext

// Observable
@Observable
class UserStore { }

// AppStorage
@AppStorage("hasCompletedOnboarding") private var hasCompletedOnboarding = false
```

### File Naming

| Type | Convention | Example |
|------|-----------|---------|
| SwiftUI View | `{Name}View.swift` | `ProfileView.swift` |
| ViewModel | `{Name}ViewModel.swift` | `ProfileViewModel.swift` |
| Model | `{Name}.swift` | `UserProfile.swift` |
| Protocol | `{Name}.swift` or `{Name}Protocol.swift` | `DataProviding.swift` |
| Extension | `{Type}+{Purpose}.swift` | `String+Validation.swift` |
| Tests | `{Name}Tests.swift` | `ProfileViewModelTests.swift` |

</naming_conventions>

<code_structure>

## Code Structure Patterns

### Import Organization

Group imports in this order, separated by blank lines:

```swift
import SwiftUI

import SwiftData
import Observation

import MyFeatureModule
import MyNetworkingModule
```

Order: Apple frameworks first, then third-party/internal modules. Within groups, alphabetical.

### View Body Structure

Every SwiftUI View follows this internal structure:

```swift
struct ProfileView: View {
    // MARK: - Environment
    @Environment(\.dismiss) private var dismiss
    @Environment(\.modelContext) private var modelContext

    // MARK: - State
    @State private var isEditing = false
    @State private var errorMessage: String?

    // MARK: - Properties
    let user: UserProfile
    @Binding var selectedTab: Tab

    // MARK: - Body
    var body: some View {
        NavigationStack {
            content
                .navigationTitle("Profile")
                .toolbar { toolbarContent }
                .alert("Error", isPresented: hasError) { alertActions }
                .task { await loadData() }
        }
    }

    // MARK: - Subviews
    private var content: some View {
        List {
            headerSection
            detailsSection
        }
    }

    private var headerSection: some View {
        Section("Header") {
            Text(user.name)
        }
    }

    private var detailsSection: some View {
        Section("Details") {
            Text(user.email)
        }
    }

    // MARK: - Toolbar
    @ToolbarContentBuilder
    private var toolbarContent: some ToolbarContent {
        ToolbarItem(placement: .primaryAction) {
            Button("Edit") { isEditing = true }
        }
    }

    // MARK: - Actions
    private func loadData() async {
        // ...
    }
}
```

### Preview Macro (REQUIRED)

**Always use `#Preview`.** Never use the deprecated `PreviewProvider` protocol.

```swift
// Correct
#Preview {
    ProfileView(user: .preview)
}

#Preview("Editing Mode") {
    ProfileView(user: .preview, isEditing: true)
}

// With traits
#Preview(traits: .landscapeLeft) {
    ProfileView(user: .preview)
}

// WRONG - Never do this
struct ProfileView_Previews: PreviewProvider {  // DEPRECATED
    static var previews: some View { ... }
}
```

### File Organization

Recommended order within a file:

1. Imports
2. Type declaration (struct/class/enum)
3. Stored properties (environment, state, bindings, lets, vars)
4. `body` (for Views)
5. Computed properties / subviews
6. Methods
7. Nested types
8. Extensions (protocol conformances in separate extensions)

```swift
struct TaskListView: View {
    @Environment(\.modelContext) private var modelContext
    @State private var tasks: [TaskItem] = []

    var body: some View { /* ... */ }

    private var emptyStateView: some View { /* ... */ }

    private func addTask() { /* ... */ }
}

// MARK: - Preview Data
extension TaskItem {
    static var preview: TaskItem {
        TaskItem(title: "Sample Task", isComplete: false)
    }
}
```

</code_structure>

<preference_hierarchy>

## Preference Hierarchy

When multiple technologies can solve the same problem, prefer them in this order. Justification is included so agents can explain choices in plans.

### Language

**Swift > Objective-C > C/C++**

| Choice | When | Justification |
|--------|------|---------------|
| Swift | Always (default) | Type safety, modern concurrency, SwiftUI interop |
| Objective-C | Legacy interop, runtime features | Only when wrapping existing ObjC libraries |
| C/C++ | Performance-critical, system-level | Audio processing, graphics, cryptography |

### UI Framework

**SwiftUI > UIKit**

| Choice | When | Justification |
|--------|------|---------------|
| SwiftUI | Always (default) | Declarative, less code, built-in accessibility, previews |
| UIKit | Complex gestures, custom layouts, camera/video | When SwiftUI lacks the API (use `UIViewRepresentable`) |

When UIKit is necessary, wrap it:

```swift
struct CameraPreview: UIViewRepresentable {
    func makeUIView(context: Context) -> UIView {
        let view = UIView()
        // Camera setup
        return view
    }

    func updateUIView(_ uiView: UIView, context: Context) { }
}
```

### Concurrency

**async/await > Combine > Dispatch**

| Choice | When | Justification |
|--------|------|---------------|
| async/await | Always (default) | Native, readable, structured concurrency |
| Combine | Reactive streams (e.g., debouncing text input) | When you need operators like `debounce`, `combineLatest` |
| Dispatch | Never in new code | Legacy only; use `Task` and actors instead |

```swift
// CORRECT: async/await
func fetchUser() async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// ACCEPTABLE: Combine for reactive streams
textField.publisher(for: \.text)
    .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
    .sink { query in search(query) }

// WRONG: Dispatch in new code
DispatchQueue.main.async { self.updateUI() }  // Use @MainActor instead
```

### Persistence

**SwiftData > Core Data > UserDefaults**

| Choice | When | Justification |
|--------|------|---------------|
| SwiftData | Structured data, relationships, queries | Native Swift, macro-based, SwiftUI integration |
| Core Data | iOS 16 compatibility, CloudKit sync (legacy) | Only if deployment target requires it |
| UserDefaults | Simple key-value pairs (preferences, flags) | Settings, feature flags, small values only |
| Keychain | Secrets, tokens, credentials | Security-sensitive data (use KeychainAccess or native) |

```swift
// SwiftData model
@Model
final class TaskItem {
    var title: String
    var isComplete: Bool
    var createdAt: Date

    init(title: String, isComplete: Bool = false) {
        self.title = title
        self.isComplete = isComplete
        self.createdAt = .now
    }
}

// UserDefaults for preferences
@AppStorage("selectedTheme") private var selectedTheme = "system"
```

### Testing

**Swift Testing > XCTest**

| Choice | When | Justification |
|--------|------|---------------|
| Swift Testing | Unit tests, parameterized tests (default) | Modern, expressive, `@Test`/`@Suite`/`#expect` |
| XCTest | UI tests (XCUITest), performance tests | XCUITest has no Swift Testing equivalent yet |

See [Testing](#testing) section for full details.

### Networking

**URLSession > Third-party**

| Choice | When | Justification |
|--------|------|---------------|
| URLSession | Always (default) | Native, async/await support, zero dependencies |
| Alamofire / etc. | Never (unless project already uses it) | Adds dependency for no real benefit |

```swift
// Native URLSession — all you need
func fetchUser(id: String) async throws -> User {
    var request = URLRequest(url: baseURL.appending(path: "users/\(id)"))
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

    let (data, response) = try await URLSession.shared.data(for: request)

    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw APIError.invalidResponse
    }

    return try JSONDecoder().decode(User.self, from: data)
}
```

### General Rule

**Native frameworks > Third-party libraries — always.**

Before adding any dependency, ask: "Does Apple provide this?" If yes, use the native solution. SPM dependencies add build time, binary size, and maintenance burden.

</preference_hierarchy>

<testing>

## Testing

### Swift Testing (Default for Unit Tests)

Swift Testing is the default for all new unit tests. Uses `@Suite`, `@Test`, and `#expect`.

```swift
import Testing
@testable import MyApp

@Suite("UserProfile Validation")
struct UserProfileTests {

    @Test("Valid email passes validation")
    func validEmail() {
        let profile = UserProfile(email: "user@example.com")
        #expect(profile.isEmailValid)
    }

    @Test("Empty name fails validation")
    func emptyName() {
        let profile = UserProfile(name: "", email: "user@example.com")
        #expect(!profile.isNameValid)
    }

    @Test("Rejects invalid email formats", arguments: [
        "invalid",
        "@missing.local",
        "no-at-sign.com",
        "",
    ])
    func invalidEmails(email: String) {
        let profile = UserProfile(email: email)
        #expect(!profile.isEmailValid)
    }

    @Test("Throws on network failure")
    func networkFailure() async {
        let service = UserService(session: .mock(error: URLError(.notConnectedToInternet)))
        await #expect(throws: UserServiceError.networkUnavailable) {
            try await service.fetchProfile()
        }
    }
}
```

**Key Swift Testing patterns:**

| Pattern | Syntax |
|---------|--------|
| Test function | `@Test func name()` |
| Test with display name | `@Test("Description") func name()` |
| Test suite | `@Suite struct NameTests { }` |
| Assertion (true) | `#expect(condition)` |
| Assertion (equality) | `#expect(a == b)` |
| Assertion (throws) | `#expect(throws: ErrorType.self) { try expr }` |
| Parameterized test | `@Test(arguments: [...]) func name(arg: Type)` |
| Skip test | `@Test(.disabled("reason")) func name()` |
| Tag test | `@Test(.tags(.networking)) func name()` |
| Async test | `@Test func name() async throws` |

### XCTest (UI Tests and Legacy)

Use XCTest for UI tests (XCUITest) and when you need compatibility with iOS 16 targets.

```swift
import XCTest

final class OnboardingUITests: XCTestCase {

    private var app: XCUIApplication!

    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }

    func testOnboardingFlowCompletes() throws {
        // Navigate through onboarding
        let getStartedButton = app.buttons["Get Started"]
        XCTAssertTrue(getStartedButton.waitForExistence(timeout: 5))
        getStartedButton.tap()

        // Verify completion
        let homeTitle = app.staticTexts["Home"]
        XCTAssertTrue(homeTitle.waitForExistence(timeout: 5))
    }
}
```

### Test File Organization

```
MyApp/
  Sources/
    Models/
      UserProfile.swift
    ViewModels/
      ProfileViewModel.swift
    Views/
      ProfileView.swift
  Tests/
    MyAppTests/                   # Swift Testing (unit tests)
      Models/
        UserProfileTests.swift
      ViewModels/
        ProfileViewModelTests.swift
    MyAppUITests/                 # XCTest (UI tests)
      OnboardingUITests.swift
      ProfileUITests.swift
```

### Running Tests

```bash
# All tests
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16'

# Specific test suite
xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16' -only-testing:MyAppTests/UserProfileTests

# Swift CLI (Swift Testing only, for SPM packages)
swift test
```

</testing>

<accessibility>

## Accessibility (MANDATORY)

Accessibility is NOT optional. Every interactive View MUST support VoiceOver, Dynamic Type, and high-contrast modes. Fail a verification check if accessibility is missing.

### VoiceOver Labels on Every Interactive View

Every button, link, toggle, slider, and custom interactive element MUST have an accessibility label.

```swift
// Buttons with icons need labels
Button(action: addItem) {
    Image(systemName: "plus")
}
.accessibilityLabel("Add new item")

// Custom controls
Toggle(isOn: $isEnabled) {
    Text("Notifications")
}
.accessibilityHint("Double tap to toggle notifications on or off")

// Images that convey meaning
Image("companyLogo")
    .accessibilityLabel("Company name")

// Decorative images — hide from VoiceOver
Image("decorativeDivider")
    .accessibilityHidden(true)

// Group related elements
VStack {
    Text(product.name)
    Text(product.price.formatted(.currency(code: "USD")))
}
.accessibilityElement(children: .combine)
```

### Dynamic Type Support

All text MUST scale with the user's preferred text size. Never use fixed font sizes for user-facing text.

```swift
// CORRECT: Uses Dynamic Type automatically
Text("Welcome")
    .font(.title)

Text("Details")
    .font(.body)

// CORRECT: Custom font with Dynamic Type scaling
Text("Custom")
    .font(.custom("Avenir-Medium", size: 17, relativeTo: .body))

// WRONG: Fixed size that won't scale
Text("Broken")
    .font(.system(size: 17))  // Does not respond to Dynamic Type

// Layout that adapts to large text
@ScaledMetric(relativeTo: .body) private var iconSize: CGFloat = 24

Image(systemName: "star.fill")
    .frame(width: iconSize, height: iconSize)
```

### WCAG AA Contrast Ratios

Minimum contrast ratios:
- **Normal text:** 4.5:1
- **Large text (18pt+ or 14pt+ bold):** 3:1
- **UI components and graphical objects:** 3:1

```swift
// Use semantic colors that adapt to light/dark mode
Text("Primary content")
    .foregroundStyle(.primary)    // Always high contrast

Text("Secondary content")
    .foregroundStyle(.secondary)  // System handles contrast

// Custom colors MUST be defined with light AND dark variants
// in the asset catalog — never use a single hardcoded color.
```

### Bold Text Support

Respect the user's Bold Text accessibility setting:

```swift
// System fonts handle Bold Text automatically.
// For custom fonts, provide bold variants:
@Environment(\.legibilityWeight) private var legibilityWeight

var body: some View {
    Text("Important info")
        .font(legibilityWeight == .bold
            ? .custom("Avenir-Heavy", size: 17, relativeTo: .body)
            : .custom("Avenir-Medium", size: 17, relativeTo: .body))
}
```

### Reduce Motion Support

Respect the user's Reduce Motion preference:

```swift
@Environment(\.accessibilityReduceMotion) private var reduceMotion

var body: some View {
    content
        .animation(reduceMotion ? nil : .spring(), value: isExpanded)
}

// For transitions
.transition(reduceMotion ? .opacity : .slide)

// For auto-playing animations
TimelineView(.animation(paused: reduceMotion)) { context in
    AnimatedGradient(date: context.date)
}
```

### Accessibility Verification Checklist

For every screen, verify:
- [ ] All buttons/controls have `.accessibilityLabel`
- [ ] Decorative images use `.accessibilityHidden(true)`
- [ ] Text uses semantic `Font` styles (`.body`, `.title`, etc.) or `relativeTo:`
- [ ] No hardcoded font sizes for user-facing text
- [ ] Custom colors have light + dark variants in asset catalog
- [ ] Animations respect `accessibilityReduceMotion`
- [ ] Interactive elements have minimum 44x44pt tap targets
- [ ] Reading order makes sense with VoiceOver (test by swiping)

</accessibility>

<localization>

## Localization (MANDATORY)

No user-facing string should be hardcoded. Every text the user sees must be localized, even if the app initially ships in a single language. This ensures localization readiness from day one.

### String(localized:) — The Default

```swift
// Simple string
Text(String(localized: "welcome_title"))

// With interpolation
Text(String(localized: "greeting \(userName)"))

// With comment for translators
Text(String(localized: "delete_confirmation",
            defaultValue: "Are you sure you want to delete this item?",
            comment: "Shown when user taps delete on a task"))
```

### Localizable.xcstrings (String Catalog)

Xcode 15+ uses `.xcstrings` files (JSON-based string catalogs) instead of the old `.strings` / `.stringsdict` files. The string catalog:
- Auto-discovers `String(localized:)` calls during build
- Supports pluralization and device variations natively
- Provides a visual editor in Xcode for translators

Place `Localizable.xcstrings` in `Resources/`.

### What to Localize

- All `Text()` content
- Button labels
- Alert titles and messages
- Navigation titles
- Tab bar labels
- Error messages shown to users
- Accessibility labels (`.accessibilityLabel`)

### What NOT to Localize

- Accessibility identifiers (`.accessibilityIdentifier`)
- Log messages
- Analytics event names
- API parameter keys
- Internal enum raw values

</localization>

<error_handling>

## Error Handling Patterns

### Typed Errors with Enums

Define domain-specific error types. Always conform to `LocalizedError` for user-facing messages.

```swift
enum NetworkError: LocalizedError {
    case invalidURL
    case unauthorized
    case serverError(statusCode: Int)
    case decodingFailed(underlying: Error)
    case noConnection

    var errorDescription: String? {
        switch self {
        case .invalidURL:
            "The request URL is invalid."
        case .unauthorized:
            "Your session has expired. Please sign in again."
        case .serverError(let code):
            "Server error (\(code)). Please try again later."
        case .decodingFailed:
            "Unable to process the server response."
        case .noConnection:
            "No internet connection. Check your network and try again."
        }
    }
}
```

### try-catch with async/await

```swift
func loadProfile() async {
    isLoading = true
    defer { isLoading = false }

    do {
        let profile = try await apiClient.fetchProfile()
        self.profile = profile
    } catch let error as NetworkError {
        self.errorMessage = error.localizedDescription
    } catch is CancellationError {
        // Task was cancelled — no action needed
    } catch {
        self.errorMessage = "An unexpected error occurred."
    }
}
```

### guard for Early Exit

Use `guard` to validate preconditions at the top of a function. Keeps the happy path un-indented.

```swift
func processPayment(for order: Order?) async throws -> Receipt {
    guard let order else {
        throw PaymentError.noOrder
    }
    guard !order.items.isEmpty else {
        throw PaymentError.emptyCart
    }
    guard order.total > 0 else {
        throw PaymentError.invalidTotal
    }

    // Happy path continues here, un-indented
    let receipt = try await paymentService.charge(order)
    return receipt
}
```

### Optional Chaining and nil-Coalescing

```swift
// Optional chaining — safely traverse optional values
let cityName = user.address?.city?.name

// nil-coalescing — provide defaults
let displayName = user.nickname ?? user.fullName ?? "Anonymous"

// Optional map — transform only if non-nil
let avatarImage = user.avatarURL.map { AsyncImage(url: $0) }
```

### Result Type (for Callbacks)

Use `Result` only when you need to pass success/failure through closures. Prefer async/await for new code.

```swift
// Use Result only for callback-based APIs
func legacyFetch(completion: @escaping (Result<User, NetworkError>) -> Void) {
    // ...
}

// Convert to async/await when wrapping
func fetch() async throws -> User {
    try await withCheckedThrowingContinuation { continuation in
        legacyFetch { result in
            continuation.resume(with: result)
        }
    }
}
```

### Displaying Errors in SwiftUI

```swift
struct ContentView: View {
    @State private var error: (any LocalizedError)?

    private var hasError: Binding<Bool> {
        Binding(
            get: { error != nil },
            set: { if !$0 { error = nil } }
        )
    }

    var body: some View {
        content
            .alert("Something went wrong", isPresented: hasError) {
                Button("OK", role: .cancel) { }
                if error is NetworkError {
                    Button("Retry") { Task { await loadData() } }
                }
            } message: {
                if let error {
                    Text(error.localizedDescription)
                }
            }
    }
}
```

</error_handling>

<swiftui_patterns>

## SwiftUI-Specific Patterns

### State Management

| Wrapper | Purpose | Ownership |
|---------|---------|-----------|
| `@State` | View-local mutable state | View owns it |
| `@Binding` | Two-way reference to parent's state | Parent owns it |
| `@Observable` (class) | Shared observable model object | External ownership |
| `@Environment` | System or injected values | System/ancestor owns it |
| `@AppStorage` | UserDefaults-backed state | Persisted |
| `@SceneStorage` | Scene-level state restoration | System |

```swift
// @Observable replaces ObservableObject (iOS 17+)
@Observable
class TaskStore {
    var tasks: [TaskItem] = []
    var isLoading = false

    func fetch() async throws {
        isLoading = true
        defer { isLoading = false }
        tasks = try await api.fetchTasks()
    }
}

// Inject into environment
@main
struct MyApp: App {
    @State private var store = TaskStore()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(store)
        }
    }
}

// Consume from environment
struct TaskListView: View {
    @Environment(TaskStore.self) private var store

    var body: some View {
        List(store.tasks) { task in
            TaskRow(task: task)
        }
    }
}
```

### NavigationStack

```swift
struct AppRootView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            HomeView()
                .navigationDestination(for: UserProfile.self) { user in
                    ProfileView(user: user)
                }
                .navigationDestination(for: TaskItem.self) { task in
                    TaskDetailView(task: task)
                }
        }
    }
}

// Push programmatically
Button("View Profile") {
    path.append(user)
}
```

### Sheets, Alerts, and Confirmations

```swift
struct ExampleView: View {
    @State private var showSettings = false
    @State private var showDeleteConfirmation = false
    @State private var selectedItem: Item?

    var body: some View {
        content
            // Sheet
            .sheet(isPresented: $showSettings) {
                SettingsView()
            }
            // Item-based sheet
            .sheet(item: $selectedItem) { item in
                ItemDetailView(item: item)
            }
            // Confirmation dialog
            .confirmationDialog(
                "Delete this item?",
                isPresented: $showDeleteConfirmation,
                titleVisibility: .visible
            ) {
                Button("Delete", role: .destructive) { deleteItem() }
                Button("Cancel", role: .cancel) { }
            }
    }
}
```

### Task Modifier for Async Work

Use `.task` to start async work tied to a View's lifecycle. It is automatically cancelled when the View disappears.

```swift
struct UserListView: View {
    @State private var users: [User] = []
    @State private var isLoading = true

    var body: some View {
        List(users) { user in
            UserRow(user: user)
        }
        .overlay {
            if isLoading {
                ProgressView()
            }
        }
        .task {
            // Automatically cancelled if view disappears
            await loadUsers()
        }
        .refreshable {
            await loadUsers()
        }
    }

    private func loadUsers() async {
        isLoading = true
        defer { isLoading = false }
        do {
            users = try await api.fetchUsers()
        } catch {
            // handle error
        }
    }
}

// Task with ID — restarts when ID changes
.task(id: selectedCategory) {
    await loadItems(for: selectedCategory)
}
```

### @MainActor for UI Updates

```swift
// Mark entire ViewModel for main thread
@MainActor
@Observable
class ProfileViewModel {
    var profile: UserProfile?
    var errorMessage: String?

    func load() async {
        do {
            profile = try await api.fetchProfile()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// Or mark individual functions
@MainActor
func updateUI(with data: [Item]) {
    self.items = data
}
```

### View Modifiers for Reusable Behavior

```swift
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(.regularMaterial)
            .clipShape(RoundedRectangle(cornerRadius: 12))
            .shadow(radius: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardModifier())
    }
}

// Usage
Text("Hello")
    .cardStyle()
```

</swiftui_patterns>

<deployment_targets>

## Deployment Targets

The base deployment target is **iOS 17+**. Use conditional compilation and availability checks for newer APIs.

### iOS 17 (Base)

Everything in this document works on iOS 17. This is the minimum bar.

Available: SwiftData, `@Observable`, `#Preview`, NavigationStack, Swift Testing, `contentMargins`, `scrollTargetBehavior`.

### iOS 18 (Conditional — Significant Leap)

iOS 18 is a major capability jump. Agents should know these APIs exist even if the project targets iOS 17 as base.

Use `if #available(iOS 18, *)` for:
- `MeshGradient` — complex multi-point gradient rendering
- `@Previewable` macro — use `@State` and other property wrappers directly inside `#Preview` blocks without boilerplate wrapper views
- Enhanced `TabView` with `Tab` type and `.sidebarAdaptable` style
- SwiftData improvements: history tracking (`ModelContext.fetchHistory`), custom data stores (`DataStoreConfiguration`), `#Expression` macro for type-safe predicates
- `AccessoryWidgetGroup` — compose multiple small widgets on Apple Watch / Lock Screen
- `CustomAnimation` protocol — create reusable, type-safe custom animations conforming to a formal protocol
- `TextRenderer` — custom text rendering pipeline for advanced typographic effects

```swift
// @Previewable eliminates wrapper views in previews
#Preview {
    @Previewable @State var isOn = false
    Toggle("Setting", isOn: $isOn)
}

// MeshGradient
if #available(iOS 18, *) {
    MeshGradient(/* ... */)
} else {
    LinearGradient(/* fallback */)
}
```

### iOS 26 (Conditional — Liquid Glass Era)

Use `if #available(iOS 26, *)` for:
- **Liquid Glass** design language (automatic for NavigationStack, TabView, sheets)
- `FoundationModels` framework (on-device LLM)
- `@Codable` macro (synthesized from `@Model`)
- Enhanced widget interactivity
- `RealityKit` improvements

```swift
#if canImport(FoundationModels)
import FoundationModels

@available(iOS 26, *)
func summarize(text: String) async throws -> String {
    let session = LanguageModelSession()
    let response = try await session.respond(to: "Summarize: \(text)")
    return response.content
}
#endif
```

### Availability Check Patterns

```swift
// Runtime check
if #available(iOS 18, *) {
    // Use iOS 18 API
} else {
    // Fallback
}

// Compile-time check for optional frameworks
#if canImport(FoundationModels)
// iOS 26+ only code
#endif

// Mark entire declarations
@available(iOS 18, *)
struct MeshBackgroundView: View { /* ... */ }
```

### Deployment Target in Xcode

Always set in xcconfig, not in Xcode GUI:

```
// Base.xcconfig
IPHONEOS_DEPLOYMENT_TARGET = 17.0
```

</deployment_targets>

<anti_patterns>

## Anti-Patterns (NEVER DO THESE)

### Force Unwraps Without Justification

```swift
// WRONG
let url = URL(string: someUserInput)!          // Crash if invalid
let cell = tableView.dequeueReusableCell(...)! // Crash if registration missing

// CORRECT
guard let url = URL(string: someUserInput) else {
    throw ValidationError.invalidURL
}

// ACCEPTABLE: Force unwrap only for compile-time-known values
let url = URL(string: "https://api.example.com")!  // Constant, always valid — add comment
```

### Combine in New Code

```swift
// WRONG: Using Combine for simple async operations
cancellable = URLSession.shared.dataTaskPublisher(for: url)
    .map(\.data)
    .decode(type: User.self, decoder: JSONDecoder())
    .receive(on: DispatchQueue.main)
    .sink(receiveCompletion: { _ in }, receiveValue: { user in
        self.user = user
    })

// CORRECT: async/await
func fetchUser() async throws -> User {
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}
```

**Exception:** Combine is acceptable for reactive streams that need operators like `debounce`, `throttle`, `combineLatest`.

### PreviewProvider (Deprecated)

```swift
// WRONG — Deprecated protocol
struct MyView_Previews: PreviewProvider {
    static var previews: some View {
        MyView()
    }
}

// CORRECT — #Preview macro
#Preview {
    MyView()
}
```

### DispatchQueue.main.async for UI Updates

```swift
// WRONG
DispatchQueue.main.async {
    self.isLoading = false
    self.data = newData
}

// CORRECT: Use @MainActor
@MainActor
func updateData(_ newData: [Item]) {
    self.isLoading = false
    self.data = newData
}

// Or mark the whole type
@MainActor
@Observable
class ViewModel { /* ... */ }
```

### Massive View Bodies

```swift
// WRONG: 100+ line body
var body: some View {
    VStack {
        // ... 50 lines of header ...
        // ... 30 lines of content ...
        // ... 20 lines of footer ...
    }
}

// CORRECT: Extract subviews
var body: some View {
    VStack {
        headerSection
        contentSection
        footerSection
    }
}

private var headerSection: some View {
    // ...
}
```

**Rule of thumb:** If `body` exceeds ~30 lines, extract subviews.

### God ViewModels

```swift
// WRONG: One ViewModel managing everything
class AppViewModel: ObservableObject {
    @Published var user: User?
    @Published var tasks: [Task]
    @Published var settings: Settings
    @Published var notifications: [Notification]
    // 500 lines of methods covering all features...
}

// CORRECT: Focused ViewModels per feature
@Observable class ProfileViewModel {
    var user: User?
    func load() async { /* ... */ }
}

@Observable class TaskListViewModel {
    var tasks: [TaskItem] = []
    func fetch() async { /* ... */ }
}
```

### Hardcoded User-Facing Strings

```swift
// WRONG — hardcoded string
Text("Welcome back!")
Button("Delete") { deleteItem() }
Label("Settings", systemImage: "gear")

// CORRECT — localized
Text(String(localized: "welcome_back"))
Button(String(localized: "delete_action")) { deleteItem() }
Label(String(localized: "settings_title"), systemImage: "gear")
```

No user-facing string should be a raw string literal. Use `String(localized:)` for all text visible to users. Store translations in `Localizable.xcstrings` (Xcode 15+ string catalog format).

**Exception:** Accessibility identifiers (`.accessibilityIdentifier("login_button")`) are NOT user-facing and should remain as plain strings.

### Other Anti-Patterns

| Anti-Pattern | Do Instead |
|-------------|-----------|
| Hardcoded user-facing strings | Use `String(localized:)` or `Localizable.xcstrings` |
| Singletons for everything | Use dependency injection via `@Environment` |
| `Any` / `AnyObject` abuse | Use generics or protocols with associated types |
| Stringly-typed APIs | Use enums, strongly-typed identifiers |
| Ignoring errors with `try?` silently | Handle the error or log it explicitly |
| `@ObservedObject` in iOS 17+ | Use `@Observable` + `@Environment` or `@Bindable` |
| `NavigationView` | Use `NavigationStack` (or `NavigationSplitView`) |
| `onChange(of:perform:)` (one-param) | Use `onChange(of:) { oldValue, newValue in }` |

</anti_patterns>

<project_architecture>

## Project Architecture (MVVM with SwiftUI)

### Recommended Folder Structure

```
MyApp/
  Sources/
    App/
      MyApp.swift                  # @main App entry point
      AppDelegate.swift            # Only if needed (push notifications, etc.)
    Models/
      UserProfile.swift
      TaskItem.swift
    ViewModels/
      ProfileViewModel.swift
      TaskListViewModel.swift
    Views/
      Components/                  # Reusable UI components
        AvatarView.swift
        LoadingOverlay.swift
      Screens/
        Home/
          HomeView.swift
        Profile/
          ProfileView.swift
          ProfileEditView.swift
        Tasks/
          TaskListView.swift
          TaskDetailView.swift
    Services/
      APIClient.swift
      AuthService.swift
    Utilities/
      Extensions/
        String+Validation.swift
        Date+Formatting.swift
      Modifiers/
        CardModifier.swift
    Resources/
      Assets.xcassets
      Localizable.xcstrings
  Tests/
    MyAppTests/
    MyAppUITests/
```

### App Entry Point

```swift
@main
struct MyApp: App {
    @State private var taskStore = TaskStore()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(taskStore)
                .modelContainer(for: [TaskItem.self])
        }
    }
}
```

### Dependency Injection via Environment

```swift
// Define environment key
private struct APIClientKey: EnvironmentKey {
    static let defaultValue: APIClient = .live
}

extension EnvironmentValues {
    var apiClient: APIClient {
        get { self[APIClientKey.self] }
        set { self[APIClientKey.self] = newValue }
    }
}

// Inject
ContentView()
    .environment(\.apiClient, .live)

// Consume
struct ProfileView: View {
    @Environment(\.apiClient) private var apiClient

    var body: some View {
        // use apiClient
    }
}

// For testing: inject mock
ProfileView()
    .environment(\.apiClient, .mock)
```

### Modularization with Local Swift Packages

For larger projects (10+ screens, multiple developers, or shared components across targets), use local Swift Packages as feature modules. This is the modern modularization pattern in iOS.

```
MyApp/
  MyApp/                          # Main app target (thin — wires modules together)
  Packages/
    FeatureAuth/                  # Local SPM package
      Sources/FeatureAuth/
      Tests/FeatureAuthTests/
      Package.swift
    FeatureProfile/
    CoreNetworking/               # Shared infrastructure
    CoreDesignSystem/             # Shared UI components
```

When to modularize:
- **10+ screens** — feature boundaries become natural package boundaries
- **Multiple developers** — packages reduce merge conflicts and enable independent work
- **Shared components** — a design system or networking layer reused across features or app targets (main app + widgets + extensions)
- **Build time** — SPM packages enable incremental compilation, speeding up builds

Keep it simple: start as a monolith, extract packages when pain (build time, conflicts, reuse) justifies it. Don't modularize prematurely.

</project_architecture>

<concurrency_patterns>

## Concurrency Patterns

### Structured Concurrency with TaskGroup

```swift
func fetchDashboard() async throws -> Dashboard {
    async let profile = api.fetchProfile()
    async let tasks = api.fetchTasks()
    async let notifications = api.fetchNotifications()

    return try await Dashboard(
        profile: profile,
        tasks: tasks,
        notifications: notifications
    )
}
```

### Actors for Thread-Safe State

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]

    func image(for url: URL) -> UIImage? {
        cache[url]
    }

    func store(_ image: UIImage, for url: URL) {
        cache[url] = image
    }
}
```

### Cancellation

```swift
func search(query: String) async throws -> [Result] {
    // Check for cancellation before expensive work
    try Task.checkCancellation()

    let results = try await api.search(query)

    // Check again before processing
    try Task.checkCancellation()

    return results.sorted()
}
```

</concurrency_patterns>

<validation_checklist>

## Validation Checklist

Run this checklist before declaring any task complete. Any unchecked item is a blocker.

### Code Quality
- [ ] No compiler warnings
- [ ] No force unwraps without justification comment
- [ ] No `DispatchQueue.main.async` — using `@MainActor` instead
- [ ] No `PreviewProvider` — using `#Preview` macro
- [ ] No `NavigationView` — using `NavigationStack`
- [ ] No Combine for simple async operations — using async/await
- [ ] All `@State` properties are `private`
- [ ] View `body` is under ~30 lines (subviews extracted if longer)

### Accessibility
- [ ] All interactive elements have `.accessibilityLabel`
- [ ] Decorative images use `.accessibilityHidden(true)`
- [ ] Text uses semantic font styles or `relativeTo:` for Dynamic Type
- [ ] Animations respect `accessibilityReduceMotion`
- [ ] Tap targets are at least 44x44pt
- [ ] Colors have light and dark mode variants

### Architecture
- [ ] One ViewModel per feature/screen (no God ViewModels)
- [ ] ViewModels use `@Observable` (not `ObservableObject`)
- [ ] Dependencies injected via `@Environment` (not singletons)
- [ ] Models are value types (`struct`) unless shared state requires `class`
- [ ] Error types conform to `LocalizedError`

### Testing
- [ ] New business logic has Swift Testing tests (`@Test`, `#expect`)
- [ ] Tests are in the correct target (`MyAppTests` for unit, `MyAppUITests` for UI)
- [ ] Tests run and pass: `xcodebuild test -scheme MyApp -destination 'platform=iOS Simulator,name=iPhone 16'`

### iOS-Specific
- [ ] Base deployment target is iOS 17+
- [ ] iOS 18+ / iOS 26+ APIs are behind `#available` / `#if canImport` checks
- [ ] Native frameworks used over third-party equivalents
- [ ] SwiftData used for persistence (not Core Data, unless legacy)
- [ ] `#Preview` provided for every new View

### Localization
- [ ] No hardcoded user-facing strings — using `String(localized:)` or `Localizable.xcstrings`

### Before Committing
- [ ] Build succeeds with no errors
- [ ] All existing tests still pass
- [ ] No secrets, API keys, or tokens in source files
- [ ] No `TODO` or `FIXME` left without a tracking issue

</validation_checklist>
