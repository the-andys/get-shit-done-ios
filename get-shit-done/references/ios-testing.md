# iOS Testing Patterns

Reference for testing iOS applications. Swift Testing is the primary framework for unit tests. XCTest is used for UI tests (XCUITest) and legacy compatibility.

<swift_testing>

## Swift Testing Framework (Primary for Unit Tests)

Swift Testing is Apple's modern testing framework, available from Xcode 16+ and Swift 6+. It replaces XCTest for unit testing with a cleaner, more expressive API.

### Test Organization with @Suite

Use `@Suite` to group related tests. Suites can be nested and support display names.

```swift
import Testing

@Suite("Authentication Service")
struct AuthenticationServiceTests {
    let sut: AuthenticationService

    init() {
        sut = AuthenticationService(
            repository: MockUserRepository(),
            tokenStore: MockTokenStore()
        )
    }

    @Test("Successful login returns user token")
    func successfulLogin() async throws {
        let token = try await sut.login(email: "user@example.com", password: "valid-pass")

        #expect(token.isValid)
        #expect(!token.value.isEmpty)
    }

    @Test("Invalid credentials throw authentication error")
    func invalidCredentials() async throws {
        await #expect(throws: AuthError.invalidCredentials) {
            try await sut.login(email: "user@example.com", password: "wrong")
        }
    }

    @Suite("Password Validation")
    struct PasswordValidation {
        @Test("Rejects passwords shorter than 8 characters")
        func shortPassword() {
            let result = PasswordValidator.validate("abc")
            #expect(result == .tooShort)
        }

        @Test("Accepts valid passwords")
        func validPassword() {
            let result = PasswordValidator.validate("SecureP@ss123")
            #expect(result == .valid)
        }
    }
}
```

### Assertions with #expect and #require

`#expect` records a test failure but continues execution. `#require` stops the test immediately if the condition fails — use it for unwrapping optionals and preconditions.

```swift
import Testing

@Suite("User Profile")
struct UserProfileTests {

    @Test("Display name formats correctly")
    func displayNameFormatting() {
        let user = User(firstName: "John", lastName: "Doe")

        // Basic assertion — test continues if this fails
        #expect(user.displayName == "John Doe")
        #expect(user.initials == "JD")
    }

    @Test("Parses user from valid JSON")
    func parseValidJSON() throws {
        let json = """
        {"id": "123", "name": "John Doe", "email": "john@example.com"}
        """.data(using: .utf8)!

        // #require unwraps or fails the test immediately
        let user = try #require(try? JSONDecoder().decode(User.self, from: json))

        #expect(user.id == "123")
        #expect(user.name == "John Doe")
        #expect(user.email == "john@example.com")
    }

    @Test("Handles optional profile image URL")
    func optionalProfileImage() throws {
        let user = User(id: "1", name: "Test", profileImageURL: "https://example.com/photo.jpg")

        // #require unwraps the optional or fails the test
        let url = try #require(user.profileImageURL)
        #expect(url.contains("example.com"))
    }

    @Test("Collection assertions")
    func collectionChecks() {
        let items = ["apple", "banana", "cherry"]

        #expect(items.count == 3)
        #expect(items.contains("banana"))
        #expect(!items.isEmpty)
    }
}
```

### Parameterized Tests

Run the same test logic with multiple inputs using `@Test(arguments:)`.

```swift
import Testing

@Suite("Email Validation")
struct EmailValidationTests {

    @Test("Valid emails are accepted", arguments: [
        "user@example.com",
        "name.surname@company.co.uk",
        "test+tag@gmail.com",
        "user123@domain.org"
    ])
    func validEmails(email: String) {
        #expect(EmailValidator.isValid(email))
    }

    @Test("Invalid emails are rejected", arguments: [
        "",
        "not-an-email",
        "@no-local-part.com",
        "no-domain@",
        "spaces in@email.com"
    ])
    func invalidEmails(email: String) {
        #expect(!EmailValidator.isValid(email))
    }

    @Test("Currency formatting", arguments: [
        (1000, "en_US", "$1,000.00"),
        (1000, "pt_BR", "R$ 1.000,00"),
        (0, "en_US", "$0.00"),
    ])
    func currencyFormatting(amount: Int, localeId: String, expected: String) {
        let formatter = CurrencyFormatter(locale: Locale(identifier: localeId))
        #expect(formatter.format(cents: amount) == expected)
    }
}
```

### Async/Throws Support

Swift Testing has native support for `async` and `throws` — no special setup needed.

```swift
import Testing

@Suite("Network Service")
struct NetworkServiceTests {
    let sut: NetworkService

    init() {
        sut = NetworkService(session: MockURLSession())
    }

    @Test("Fetches user list successfully")
    func fetchUsers() async throws {
        let users = try await sut.fetchUsers()

        #expect(!users.isEmpty)
        #expect(users.first?.name == "John Doe")
    }

    @Test("Throws network error on timeout")
    func networkTimeout() async {
        let slowService = NetworkService(session: MockURLSession(delay: .seconds(30)))

        await #expect(throws: NetworkError.timeout) {
            try await slowService.fetchUsers()
        }
    }

    @Test("Retries failed request up to 3 times")
    func retryBehavior() async throws {
        let failingSession = MockURLSession(failCount: 2)
        let service = NetworkService(session: failingSession)

        let result = try await service.fetchWithRetry(url: URL(string: "https://api.example.com")!)

        #expect(failingSession.requestCount == 3)
        #expect(result != nil)
    }
}
```

### Tags for Categorization

Tags let you filter and organize tests across suites.

```swift
import Testing

extension Tag {
    @Tag static var networking: Self
    @Tag static var persistence: Self
    @Tag static var authentication: Self
    @Tag static var slow: Self
}

@Suite("Data Sync Service")
struct DataSyncServiceTests {

    @Test("Syncs local changes to server", .tags(.networking, .persistence))
    func syncLocalChanges() async throws {
        // ...
    }

    @Test("Handles conflict resolution", .tags(.persistence))
    func conflictResolution() async throws {
        // ...
    }

    @Test("Full sync with large dataset", .tags(.networking, .slow))
    func fullSync() async throws {
        // ...
    }
}
```

### Disabling and Conditional Tests

```swift
import Testing

@Suite("Feature Flags")
struct FeatureFlagTests {

    @Test("New onboarding flow", .disabled("Feature not yet implemented"))
    func newOnboarding() {
        // Will not run, but shows as disabled in test results
    }

    @Test("iCloud sync", .enabled(if: ProcessInfo.processInfo.environment["CI"] == nil,
                                   "Requires iCloud account — skip on CI"))
    func iCloudSync() async throws {
        // Only runs locally, not on CI
    }
}
```

</swift_testing>

<xctest_legacy>

## XCTest (Legacy and UI Tests)

Use XCTest when:
- Writing UI tests (XCUITest) — Swift Testing does not support UI testing
- Maintaining existing XCTest test suites
- Targeting iOS 16 or earlier (Swift Testing requires iOS 17+/Xcode 16+)
- Using performance testing APIs (`measure {}`)

### XCTestCase Pattern

```swift
import XCTest
@testable import MyApp

final class UserServiceXCTests: XCTestCase {

    private var sut: UserService!
    private var mockRepository: MockUserRepository!

    override func setUp() {
        super.setUp()
        mockRepository = MockUserRepository()
        sut = UserService(repository: mockRepository)
    }

    override func tearDown() {
        sut = nil
        mockRepository = nil
        super.tearDown()
    }

    func test_fetchUser_withValidID_returnsUser() async throws {
        // Arrange
        mockRepository.stubbedUser = User(id: "123", name: "John")

        // Act
        let user = try await sut.fetchUser(id: "123")

        // Assert
        XCTAssertEqual(user.name, "John")
        XCTAssertEqual(mockRepository.fetchCalledWithID, "123")
    }

    func test_fetchUser_withInvalidID_throwsNotFound() async {
        // Arrange
        mockRepository.shouldThrow = true

        // Act & Assert
        do {
            _ = try await sut.fetchUser(id: "invalid")
            XCTFail("Expected error to be thrown")
        } catch {
            XCTAssertEqual(error as? UserError, .notFound)
        }
    }

    func test_performance_largeDatasetProcessing() {
        let items = (0..<10000).map { Item(id: "\($0)") }

        measure {
            _ = sut.processItems(items)
        }
    }
}
```

### XCTest Assertion Reference

```swift
// Equality
XCTAssertEqual(actual, expected)
XCTAssertNotEqual(actual, expected)

// Boolean
XCTAssertTrue(condition)
XCTAssertFalse(condition)

// Nil
XCTAssertNil(value)
XCTAssertNotNil(value)

// Optionals (unwrap or fail)
let unwrapped = try XCTUnwrap(optionalValue)

// Throwing
XCTAssertThrowsError(try expression()) { error in
    XCTAssertEqual(error as? MyError, .specific)
}
XCTAssertNoThrow(try expression())

// Comparison
XCTAssertGreaterThan(a, b)
XCTAssertLessThanOrEqual(a, b)

// Floating point
XCTAssertEqual(1.0, 1.001, accuracy: 0.01)
```

</xctest_legacy>

<xcuitest>

## XCUITest (UI Testing)

XCUITest uses XCTest and runs in a separate process from the app. It interacts with the app through accessibility elements.

### Basic UI Test Structure

```swift
import XCTest

final class LoginUITests: XCTestCase {

    private var app: XCUIApplication!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launchEnvironment = ["DISABLE_ANIMATIONS": "1"]
        app.launch()
    }

    override func tearDown() {
        app = nil
        super.tearDown()
    }

    func test_loginFlow_withValidCredentials_showsHomeScreen() {
        // Find and interact with elements using accessibility identifiers
        let emailField = app.textFields["login_email_field"]
        emailField.tap()
        emailField.typeText("user@example.com")

        let passwordField = app.secureTextFields["login_password_field"]
        passwordField.tap()
        passwordField.typeText("password123")

        app.buttons["login_submit_button"].tap()

        // Wait for navigation to complete
        let homeTitle = app.staticTexts["home_title"]
        XCTAssertTrue(homeTitle.waitForExistence(timeout: 5))
    }

    func test_loginFlow_withEmptyEmail_showsValidationError() {
        app.buttons["login_submit_button"].tap()

        let errorLabel = app.staticTexts["login_error_label"]
        XCTAssertTrue(errorLabel.waitForExistence(timeout: 2))
        XCTAssertEqual(errorLabel.label, "Email is required")
    }
}
```

### Setting Accessibility Identifiers in SwiftUI

Accessibility identifiers are the bridge between your UI code and UI tests.

```swift
// In your SwiftUI View
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var errorMessage: String?

    var body: some View {
        VStack(spacing: 16) {
            TextField("Email", text: $email)
                .accessibilityIdentifier("login_email_field")

            SecureField("Password", text: $password)
                .accessibilityIdentifier("login_password_field")

            if let errorMessage {
                Text(errorMessage)
                    .foregroundStyle(.red)
                    .accessibilityIdentifier("login_error_label")
            }

            Button("Log In") {
                performLogin()
            }
            .accessibilityIdentifier("login_submit_button")
        }
    }
}
```

### Element Queries

```swift
// By type
app.buttons["identifier"]
app.textFields["identifier"]
app.secureTextFields["identifier"]
app.staticTexts["identifier"]
app.images["identifier"]
app.switches["identifier"]
app.sliders["identifier"]
app.navigationBars["Title"]
app.tabBars.buttons["Tab Name"]
app.alerts.buttons["OK"]
app.sheets.buttons["Cancel"]

// By predicate
let predicate = NSPredicate(format: "label CONTAINS 'Welcome'")
let element = app.staticTexts.matching(predicate).firstMatch

// Existence checks
XCTAssertTrue(app.buttons["save"].exists)
XCTAssertTrue(app.buttons["save"].isHittable)
XCTAssertTrue(app.buttons["save"].waitForExistence(timeout: 5))
```

### Gestures

```swift
// Tap
app.buttons["action"].tap()
app.buttons["action"].doubleTap()
app.buttons["action"].press(forDuration: 2.0)

// Swipe
app.cells.firstMatch.swipeLeft()
app.cells.firstMatch.swipeRight()
app.scrollViews.firstMatch.swipeUp()
app.scrollViews.firstMatch.swipeDown()

// Scroll to element
let lastCell = app.cells["cell_99"]
while !lastCell.isHittable {
    app.swipeUp()
}

// Drag
let source = app.cells["item_1"]
let destination = app.cells["item_3"]
source.press(forDuration: 0.5, thenDragTo: destination)
```

### Waiting for Asynchronous State

```swift
func test_loadingIndicator_disappearsAfterDataLoads() {
    app.buttons["refresh_button"].tap()

    // Wait for loading to appear
    let spinner = app.activityIndicators["loading_indicator"]
    XCTAssertTrue(spinner.waitForExistence(timeout: 2))

    // Wait for loading to disappear using expectation
    let disappeared = expectation(
        for: NSPredicate(format: "exists == false"),
        evaluatedWith: spinner
    )
    wait(for: [disappeared], timeout: 10)

    // Verify data loaded
    XCTAssertTrue(app.cells.count > 0)
}

func test_pullToRefresh() {
    let firstCell = app.cells.firstMatch
    let start = firstCell.coordinate(withNormalizedOffset: CGVector(dx: 0.5, dy: 0))
    let end = firstCell.coordinate(withNormalizedOffset: CGVector(dx: 0.5, dy: 5))
    start.press(forDuration: 0, thenDragTo: end)

    // Wait for refresh to complete
    let newContent = app.staticTexts["updated_label"]
    XCTAssertTrue(newContent.waitForExistence(timeout: 5))
}
```

### Launch Arguments for Test Configuration

```swift
// In UI Test setUp:
override func setUp() {
    super.setUp()
    app = XCUIApplication()

    // Pass flags to the app
    app.launchArguments = [
        "--uitesting",           // App checks this to use mock data
        "--reset-state",         // Clear user defaults / keychain
        "--skip-onboarding"      // Skip first-launch flow
    ]

    app.launchEnvironment = [
        "API_BASE_URL": "http://localhost:8080",
        "DISABLE_ANIMATIONS": "1"
    ]

    app.launch()
}

// In the app (e.g., AppDelegate or @main App):
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            if CommandLine.arguments.contains("--uitesting") {
                ContentView(service: MockDataService())
            } else {
                ContentView(service: LiveDataService())
            }
        }
    }
}
```

</xcuitest>

<tdd_ios>

## TDD Patterns for iOS

### Red-Green-Refactor with Swift Testing

The cycle for iOS with Swift Testing:

**RED:** Write a failing test using `@Test` and `#expect`.
**GREEN:** Write the minimum code to make the test pass.
**REFACTOR:** Clean up without changing behavior — tests must still pass.

```swift
// === RED: Write this test first. It will not compile. ===

import Testing
@testable import MyApp

@Suite("Shopping Cart")
struct ShoppingCartTests {

    @Test("Adding item increases total")
    func addItemIncreasesTotal() {
        let cart = ShoppingCart()
        let item = CartItem(name: "Widget", price: 9.99, quantity: 1)

        cart.add(item)

        #expect(cart.total == 9.99)
        #expect(cart.itemCount == 1)
    }
}

// === GREEN: Implement the minimum to pass ===

struct CartItem {
    let name: String
    let price: Double
    let quantity: Int
}

class ShoppingCart {
    private(set) var items: [CartItem] = []

    var total: Double {
        items.reduce(0) { $0 + $1.price * Double($1.quantity) }
    }

    var itemCount: Int { items.count }

    func add(_ item: CartItem) {
        items.append(item)
    }
}

// === REFACTOR: Improve if needed (e.g., use Decimal for money) ===
```

### Arrange-Act-Assert with Async/Throws

```swift
import Testing
@testable import MyApp

@Suite("Task Repository")
struct TaskRepositoryTests {
    let sut: TaskRepository
    let mockStore: MockPersistenceStore

    init() {
        mockStore = MockPersistenceStore()
        sut = TaskRepository(store: mockStore)
    }

    @Test("Saves task and retrieves it by ID")
    func saveAndRetrieve() async throws {
        // Arrange
        let task = TaskItem(id: UUID(), title: "Buy groceries", isCompleted: false)

        // Act
        try await sut.save(task)
        let retrieved = try await sut.fetch(by: task.id)

        // Assert
        let result = try #require(retrieved)
        #expect(result.title == "Buy groceries")
        #expect(result.isCompleted == false)
    }

    @Test("Deleting non-existent task throws notFound")
    func deleteNonExistent() async {
        // Arrange
        let fakeID = UUID()

        // Act & Assert
        await #expect(throws: RepositoryError.notFound) {
            try await sut.delete(by: fakeID)
        }
    }
}
```

### ViewModel Testing Pattern

ViewModels are the primary testing target in MVVM iOS apps. They contain business logic, state management, and data transformations — all highly testable.

```swift
// === ViewModel ===

import SwiftUI

@Observable
final class TaskListViewModel {
    private let repository: TaskRepositoryProtocol

    var tasks: [TaskItem] = []
    var isLoading = false
    var errorMessage: String?

    var completedCount: Int {
        tasks.filter(\.isCompleted).count
    }

    var pendingTasks: [TaskItem] {
        tasks.filter { !$0.isCompleted }
    }

    init(repository: TaskRepositoryProtocol) {
        self.repository = repository
    }

    func loadTasks() async {
        isLoading = true
        errorMessage = nil

        do {
            tasks = try await repository.fetchAll()
        } catch {
            errorMessage = "Failed to load tasks. Please try again."
        }

        isLoading = false
    }

    func toggleCompletion(for task: TaskItem) async {
        guard let index = tasks.firstIndex(where: { $0.id == task.id }) else { return }

        tasks[index].isCompleted.toggle()

        do {
            try await repository.save(tasks[index])
        } catch {
            tasks[index].isCompleted.toggle() // Revert on failure
            errorMessage = "Failed to update task."
        }
    }

    func deleteTask(_ task: TaskItem) async {
        let originalTasks = tasks
        tasks.removeAll { $0.id == task.id }

        do {
            try await repository.delete(by: task.id)
        } catch {
            tasks = originalTasks // Revert on failure
            errorMessage = "Failed to delete task."
        }
    }
}

// === ViewModel Tests ===

import Testing
@testable import MyApp

@Suite("TaskListViewModel")
struct TaskListViewModelTests {
    let sut: TaskListViewModel
    let mockRepo: MockTaskRepository

    init() {
        mockRepo = MockTaskRepository()
        sut = TaskListViewModel(repository: mockRepo)
    }

    @Test("Load tasks updates tasks array")
    func loadTasks() async {
        mockRepo.stubbedTasks = [
            TaskItem(id: UUID(), title: "Task 1", isCompleted: false),
            TaskItem(id: UUID(), title: "Task 2", isCompleted: true),
        ]

        await sut.loadTasks()

        #expect(sut.tasks.count == 2)
        #expect(sut.isLoading == false)
        #expect(sut.errorMessage == nil)
    }

    @Test("Load tasks sets error message on failure")
    func loadTasksFailure() async {
        mockRepo.shouldThrow = true

        await sut.loadTasks()

        #expect(sut.tasks.isEmpty)
        #expect(sut.isLoading == false)
        #expect(sut.errorMessage != nil)
    }

    @Test("Completed count reflects completed tasks")
    func completedCount() async {
        mockRepo.stubbedTasks = [
            TaskItem(id: UUID(), title: "Done 1", isCompleted: true),
            TaskItem(id: UUID(), title: "Done 2", isCompleted: true),
            TaskItem(id: UUID(), title: "Pending", isCompleted: false),
        ]

        await sut.loadTasks()

        #expect(sut.completedCount == 2)
        #expect(sut.pendingTasks.count == 1)
    }

    @Test("Toggle completion reverts on save failure")
    func toggleCompletionRevertsOnFailure() async {
        let task = TaskItem(id: UUID(), title: "Test", isCompleted: false)
        mockRepo.stubbedTasks = [task]
        await sut.loadTasks()

        mockRepo.shouldThrow = true
        await sut.toggleCompletion(for: task)

        // Should revert to original state
        #expect(sut.tasks.first?.isCompleted == false)
        #expect(sut.errorMessage != nil)
    }

    @Test("Delete task reverts on failure")
    func deleteRevertsOnFailure() async {
        let task = TaskItem(id: UUID(), title: "Test", isCompleted: false)
        mockRepo.stubbedTasks = [task]
        await sut.loadTasks()

        mockRepo.shouldThrow = true
        await sut.deleteTask(task)

        // Should revert — task still in list
        #expect(sut.tasks.count == 1)
        #expect(sut.errorMessage != nil)
    }
}
```

### Service / Repository Testing with Protocols

Define protocols for dependencies. Test the real implementation with mock collaborators.

```swift
// === Protocol ===

protocol TaskRepositoryProtocol: Sendable {
    func fetchAll() async throws -> [TaskItem]
    func fetch(by id: UUID) async throws -> TaskItem?
    func save(_ task: TaskItem) async throws
    func delete(by id: UUID) async throws
}

// === Real Implementation ===

final class TaskRepository: TaskRepositoryProtocol {
    private let networkClient: NetworkClientProtocol
    private let cache: CacheProtocol

    init(networkClient: NetworkClientProtocol, cache: CacheProtocol) {
        self.networkClient = networkClient
        self.cache = cache
    }

    func fetchAll() async throws -> [TaskItem] {
        if let cached: [TaskItem] = cache.get(key: "tasks") {
            return cached
        }
        let tasks: [TaskItem] = try await networkClient.get("/tasks")
        cache.set(key: "tasks", value: tasks)
        return tasks
    }

    func save(_ task: TaskItem) async throws {
        try await networkClient.put("/tasks/\(task.id)", body: task)
        cache.invalidate(key: "tasks")
    }

    // ... other methods
}

// === Tests for Real Implementation using Mock Collaborators ===

import Testing
@testable import MyApp

@Suite("TaskRepository")
struct TaskRepositoryTests {
    let sut: TaskRepository
    let mockNetwork: MockNetworkClient
    let mockCache: MockCache

    init() {
        mockNetwork = MockNetworkClient()
        mockCache = MockCache()
        sut = TaskRepository(networkClient: mockNetwork, cache: mockCache)
    }

    @Test("FetchAll returns cached data when available")
    func fetchAllFromCache() async throws {
        let cachedTasks = [TaskItem(id: UUID(), title: "Cached", isCompleted: false)]
        mockCache.store["tasks"] = cachedTasks

        let result = try await sut.fetchAll()

        #expect(result.count == 1)
        #expect(result.first?.title == "Cached")
        #expect(mockNetwork.getCallCount == 0) // Did not hit network
    }

    @Test("FetchAll hits network when cache is empty")
    func fetchAllFromNetwork() async throws {
        let networkTasks = [TaskItem(id: UUID(), title: "Remote", isCompleted: false)]
        mockNetwork.stubbedGetResponse = networkTasks

        let result = try await sut.fetchAll()

        #expect(result.first?.title == "Remote")
        #expect(mockNetwork.getCallCount == 1)
        #expect(mockCache.setCallCount == 1) // Cached the result
    }

    @Test("Save invalidates cache")
    func saveInvalidatesCache() async throws {
        let task = TaskItem(id: UUID(), title: "Test", isCompleted: false)

        try await sut.save(task)

        #expect(mockCache.invalidatedKeys.contains("tasks"))
    }
}
```

</tdd_ios>

<what_to_test>

## What to Test vs What NOT to Test

### Test These (High Value)

| Target | Why | Example |
|--------|-----|---------|
| **ViewModels** | Business logic, state management, data transformations | `TaskListViewModel`, `LoginViewModel` |
| **Services** | Core app behavior, API integration logic | `AuthService`, `SyncService` |
| **Repositories** | Data access logic, caching, error handling | `TaskRepository`, `UserRepository` |
| **Data Models** | Encoding/decoding, computed properties, validation | `User.init(from: decoder)`, `Order.total` |
| **Utilities** | Pure functions, formatters, validators | `DateFormatter`, `EmailValidator` |
| **State Machines** | Transitions, edge cases | `OnboardingFlow`, `PaymentState` |
| **Algorithms** | Sorting, filtering, searching, calculations | `SearchRanker`, `PriceCalculator` |

### Do NOT Test These (Low Value / Wrong Tool)

| Target | Why | What to Do Instead |
|--------|-----|--------------------|
| **SwiftUI View `body`** | Declarative UI — Apple handles rendering | Use Xcode Previews for visual checks |
| **Apple framework internals** | URLSession, CoreData, UserDefaults work correctly | Test your wrappers, not Apple's code |
| **Simple property wrappers** | `@State`, `@Binding`, `@AppStorage` are Apple's responsibility | Test the logic that reads/writes state |
| **Trivial getters/setters** | No logic to test | Skip unless they have computed behavior |
| **Static layout** | Colors, fonts, padding | Use Previews and snapshot tests if needed |
| **Third-party library internals** | Not your code | Test your integration layer |

### Snapshot Testing for Visual Validation

For complex custom components where visual correctness matters (custom charts, branded UI, multi-state components), use **swift-snapshot-testing** by Point-Free. It captures reference images of views and fails the test if the rendered output changes.

```swift
import SnapshotTesting
import XCTest
@testable import MyApp

final class ProfileCardSnapshotTests: XCTestCase {
    func test_profileCard_defaultState() {
        let view = ProfileCardView(user: .preview)
        assertSnapshot(of: view, as: .image(layout: .device(config: .iPhone13)))
    }

    func test_profileCard_darkMode() {
        let view = ProfileCardView(user: .preview)
            .environment(\.colorScheme, .dark)
        assertSnapshot(of: view, as: .image(layout: .device(config: .iPhone13)))
    }
}
```

Useful for detecting visual regressions in CI. Not a substitute for unit tests — use it for UI components where pixel accuracy matters.

### Automated Accessibility Auditing

Xcode 15+ includes built-in accessibility auditing in UI tests. Use it to catch common accessibility issues automatically.

```swift
// In a UI test (XCTestCase)
func test_homeScreen_passesAccessibilityAudit() throws {
    let app = XCUIApplication()
    app.launch()

    // Runs a comprehensive accessibility audit on the current screen
    try app.performAccessibilityAudit()

    // Or audit for specific categories
    try app.performAccessibilityAudit(for: [.dynamicType, .contrast, .hitRegion])
}
```

This catches missing labels, insufficient contrast, small tap targets, and Dynamic Type issues. Run on every major screen as part of the UI test suite.

### Decision Heuristic

```
Does this code contain logic I wrote?
  YES → Does it make a decision, transform data, or manage state?
    YES → Write a test
    NO  → Probably skip (glue code, configuration)
  NO  → Don't test it (framework/library code)
```

</what_to_test>

<mocking>

## Mocking Patterns

No third-party mocking frameworks needed. Use protocol-based dependency injection and hand-written mocks.

### Protocol + Mock Pattern

```swift
// === Protocol ===

protocol NetworkClientProtocol: Sendable {
    func get<T: Decodable>(_ path: String) async throws -> T
    func post<T: Encodable>(_ path: String, body: T) async throws
    func put<T: Encodable>(_ path: String, body: T) async throws
    func delete(_ path: String) async throws
}

// === Mock ===

// @unchecked Sendable: Mocks are only used in test context where concurrent
// access is controlled by the test runner. This avoids adding thread-safety
// overhead (locks, actors) to simple test doubles that don't need it.
final class MockNetworkClient: NetworkClientProtocol, @unchecked Sendable {
    var stubbedGetResponse: Any?
    var shouldThrow = false
    var getCallCount = 0
    var lastGetPath: String?
    var postBodies: [Any] = []

    func get<T: Decodable>(_ path: String) async throws -> T {
        getCallCount += 1
        lastGetPath = path
        if shouldThrow { throw NetworkError.serverError(500) }
        guard let response = stubbedGetResponse as? T else {
            throw NetworkError.decodingFailed
        }
        return response
    }

    func post<T: Encodable>(_ path: String, body: T) async throws {
        if shouldThrow { throw NetworkError.serverError(500) }
        postBodies.append(body)
    }

    func put<T: Encodable>(_ path: String, body: T) async throws {
        if shouldThrow { throw NetworkError.serverError(500) }
    }

    func delete(_ path: String) async throws {
        if shouldThrow { throw NetworkError.serverError(500) }
    }
}
```

### Mock for Cache

```swift
protocol CacheProtocol {
    func get<T>(key: String) -> T?
    func set<T>(key: String, value: T)
    func invalidate(key: String)
}

final class MockCache: CacheProtocol {
    var store: [String: Any] = [:]
    var setCallCount = 0
    var invalidatedKeys: [String] = []

    func get<T>(key: String) -> T? {
        store[key] as? T
    }

    func set<T>(key: String, value: T) {
        setCallCount += 1
        store[key] = value
    }

    func invalidate(key: String) {
        invalidatedKeys.append(key)
        store.removeValue(forKey: key)
    }
}
```

### Mock for UserDefaults

```swift
protocol UserDefaultsProtocol {
    func bool(forKey key: String) -> Bool
    func set(_ value: Bool, forKey key: String)
    func string(forKey key: String) -> String?
    func set(_ value: String?, forKey key: String)
}

// Real UserDefaults conforms easily:
extension UserDefaults: UserDefaultsProtocol {}

// Mock:
final class MockUserDefaults: UserDefaultsProtocol {
    var boolStore: [String: Bool] = [:]
    var stringStore: [String: String?] = [:]

    func bool(forKey key: String) -> Bool { boolStore[key] ?? false }
    func set(_ value: Bool, forKey key: String) { boolStore[key] = value }
    func string(forKey key: String) -> String? { stringStore[key] ?? nil }
    func set(_ value: String?, forKey key: String) { stringStore[key] = value }
}
```

### Injecting Mocks into ViewModels

```swift
// The ViewModel accepts a protocol, not a concrete type:
@Observable
final class SettingsViewModel {
    private let defaults: UserDefaultsProtocol

    var isDarkMode: Bool {
        didSet { defaults.set(isDarkMode, forKey: "dark_mode") }
    }

    init(defaults: UserDefaultsProtocol = UserDefaults.standard) {
        self.defaults = defaults
        self.isDarkMode = defaults.bool(forKey: "dark_mode")
    }
}

// In tests:
@Suite("SettingsViewModel")
struct SettingsViewModelTests {

    @Test("Reads dark mode preference on init")
    func readsDarkMode() {
        let mockDefaults = MockUserDefaults()
        mockDefaults.boolStore["dark_mode"] = true

        let sut = SettingsViewModel(defaults: mockDefaults)

        #expect(sut.isDarkMode == true)
    }

    @Test("Persists dark mode toggle")
    func persistsDarkMode() {
        let mockDefaults = MockUserDefaults()
        let sut = SettingsViewModel(defaults: mockDefaults)

        sut.isDarkMode = true

        #expect(mockDefaults.boolStore["dark_mode"] == true)
    }
}
```

</mocking>

<build_and_test_commands>

## Build and Test Commands

### Xcode Project — Build

```bash
# Build for iOS Simulator
xcodebuild build \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest' \
  -configuration Debug \
  | xcpretty

# Build with specific SDK
xcodebuild build \
  -scheme "MyApp" \
  -sdk iphonesimulator \
  | xcpretty

# Build workspace (with SPM or CocoaPods)
xcodebuild build \
  -workspace "MyApp.xcworkspace" \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  | xcpretty
```

### Xcode Project — Run Unit Tests

```bash
# Run all unit tests
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16,OS=latest' \
  | xcpretty

# Run a specific test suite
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:"MyAppTests/TaskListViewModelTests" \
  | xcpretty

# Run a specific test method
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:"MyAppTests/TaskListViewModelTests/loadTasks" \
  | xcpretty

# Run tests without building (if already built)
xcodebuild test-without-building \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  | xcpretty
```

### Xcode Project — Run UI Tests

```bash
# Run all UI tests
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:"MyAppUITests" \
  | xcpretty

# Run specific UI test class
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:"MyAppUITests/LoginUITests" \
  | xcpretty
```

### Swift Package Manager

```bash
# Build the package
swift build

# Run all tests
swift test

# Run tests with verbose output
swift test --verbose

# Run a specific test
swift test --filter "TaskRepositoryTests"

# Run tests matching a pattern
swift test --filter "TaskRepositoryTests/fetchAllFromCache"

# Build and test in release mode
swift build -c release
swift test -c release
```

### Common Destinations

```bash
# List available simulators
xcrun simctl list devices available

# Show valid destinations for a scheme (more useful for xcodebuild)
xcodebuild -showdestinations -scheme "MyApp"

# Common destinations
-destination 'platform=iOS Simulator,name=iPhone 16'
-destination 'platform=iOS Simulator,name=iPhone 16 Pro'
-destination 'platform=iOS Simulator,name=iPhone 16 Pro Max'
-destination 'platform=iOS Simulator,name=iPad Pro 13-inch (M4)'
-destination 'platform=iOS Simulator,name=iPhone SE (3rd generation)'

# Generic destination (any compatible simulator)
-destination 'generic/platform=iOS Simulator'
```

### CI-Friendly Commands

```bash
# Build and test with result bundle for CI
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -resultBundlePath ./TestResults.xcresult \
  -enableCodeCoverage YES \
  CODE_SIGN_IDENTITY="" \
  CODE_SIGNING_REQUIRED=NO

# Clean build folder before testing
xcodebuild clean test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16'

# Parallel testing on multiple simulators
xcodebuild test \
  -scheme "MyApp" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -destination 'platform=iOS Simulator,name=iPad Pro 13-inch (M4)' \
  -parallel-testing-enabled YES
```

### xcpretty — Readable Output

```bash
# Install xcpretty (one-time)
gem install xcpretty

# Pipe xcodebuild output through xcpretty for clean results
xcodebuild test -scheme "MyApp" -destination '...' | xcpretty

# Generate JUnit XML report (for CI)
xcodebuild test -scheme "MyApp" -destination '...' | xcpretty --report junit

# Generate HTML report
xcodebuild test -scheme "MyApp" -destination '...' | xcpretty --report html
```

</build_and_test_commands>

<test_file_organization>

## Test File Organization

### Recommended Structure

```
MyApp/
├── MyApp/                          # Main app target
│   ├── Models/
│   │   └── TaskItem.swift
│   ├── ViewModels/
│   │   └── TaskListViewModel.swift
│   ├── Services/
│   │   └── TaskRepository.swift
│   └── Views/
│       └── TaskListView.swift
├── MyAppTests/                     # Unit test target (Swift Testing)
│   ├── ViewModels/
│   │   └── TaskListViewModelTests.swift
│   ├── Services/
│   │   └── TaskRepositoryTests.swift
│   ├── Models/
│   │   └── TaskItemTests.swift
│   └── Mocks/
│       ├── MockTaskRepository.swift
│       ├── MockNetworkClient.swift
│       └── MockCache.swift
└── MyAppUITests/                   # UI test target (XCTest/XCUITest)
    ├── Flows/
    │   ├── LoginUITests.swift
    │   └── TaskManagementUITests.swift
    └── Helpers/
        └── XCUIApplication+Extensions.swift
```

### Naming Conventions

| Convention | Example |
|-----------|---------|
| Test file name | `{SourceFile}Tests.swift` |
| Test suite (Swift Testing) | `@Suite("{ClassName}")` struct `{ClassName}Tests` |
| Test class (XCTest) | `final class {ClassName}Tests: XCTestCase` |
| Test method (Swift Testing) | `@Test("description") func descriptiveName()` |
| Test method (XCTest) | `func test_{method}_{condition}_{expected}()` |
| Mock class | `Mock{ProtocolName}` |
| Accessibility ID | `{screen}_{element}_{type}` (e.g., `login_email_field`) |

</test_file_organization>

<framework_setup>

## Test Framework Setup for iOS

### Swift Testing (New Projects — Xcode 16+)

Swift Testing is automatically available in Xcode 16+. When creating a new test target:

1. In Xcode: File > New > Target > Unit Testing Bundle
2. Select "Swift Testing" as the testing framework
3. Import `Testing` in your test files

No additional configuration needed. Swift Testing works alongside XCTest in the same target.

### Adding Test Targets to Existing Projects

```bash
# If the project doesn't have a test target, add one in Xcode:
# File > New > Target > Unit Testing Bundle (for unit tests)
# File > New > Target > UI Testing Bundle (for UI tests)
```

### SPM Package — Package.swift Test Configuration

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyLibrary",
    platforms: [.iOS(.v17)],
    products: [
        .library(name: "MyLibrary", targets: ["MyLibrary"]),
    ],
    targets: [
        .target(name: "MyLibrary"),
        .testTarget(
            name: "MyLibraryTests",
            dependencies: ["MyLibrary"]
        ),
    ]
)
```

### Mixing Swift Testing and XCTest

Both frameworks can coexist in the same test target. Use Swift Testing for new unit tests and keep XCTest for UI tests and existing test suites.

```swift
// This file uses Swift Testing
import Testing
@testable import MyApp

@Suite("Calculator")
struct CalculatorTests {
    @Test("Addition works")
    func addition() {
        #expect(Calculator.add(2, 3) == 5)
    }
}

// This file in the same target uses XCTest (for UI tests)
import XCTest

final class CalculatorUITests: XCTestCase {
    func test_addButton_showsResult() {
        let app = XCUIApplication()
        app.launch()
        // ...
    }
}
```

</framework_setup>

---

## MCP Testing Tools

When Xcode MCP tools are available, prefer them over `xcodebuild test` CLI commands for faster feedback and richer output.

### Apple Xcode MCP (`xcode-tools`)

| Tool | Purpose |
|------|---------|
| **`RunAllTests`** | Execute the full test suite for the active scheme |
| **`RunSomeTests`** | Run specific test targets, suites, or individual test methods |
| **`GetTestList`** | List available test targets and test methods without running them |

Use `GetTestList` to discover available tests before running. Use `RunSomeTests` during development to run only the tests affected by your changes. Reserve `RunAllTests` for pre-commit verification.

### XcodeBuildMCP (Extended)

| Tool | Purpose |
|------|---------|
| **`test_sim`** | Run tests on a specific simulator with detailed output |
| **`test_device`** | Run tests on a connected physical device |
| **`list_sims`** | List available simulators with their state |
| **`boot_sim`** | Boot a specific simulator before testing |
| **`screenshot`** | Capture a screenshot from a running simulator after tests |

### When to Use Which

| Scenario | Tool |
|----------|------|
| Quick check after editing one test | `RunSomeTests` (Apple MCP) |
| Run full suite before commit | `RunAllTests` (Apple MCP) |
| Test on specific device/OS combo | `test_sim` or `test_device` (XcodeBuildMCP) |
| Discover what tests exist | `GetTestList` (Apple MCP) |
| Visual regression check | `screenshot` after test run (XcodeBuildMCP) |
| No MCP available | `xcodebuild test` CLI fallback |

### CLI Fallback

When MCP tools are not available:

```bash
# Run all tests
xcodebuild test -scheme <Scheme> -destination 'platform=iOS Simulator,name=iPhone 16'

# Run specific test target
xcodebuild test -scheme <Scheme> -only-testing:<TestTarget>

# Run specific test method
xcodebuild test -scheme <Scheme> -only-testing:<TestTarget>/<Suite>/<testMethod>

# List simulators
xcrun simctl list devices available
```
