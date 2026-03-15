<overview>
Protocol-based mocking pattern for dependency injection in tests. Covers mock network clients, repositories, UserDefaults, and cache. This is UNIQUE fork content — the community skill assumes protocol mocking but doesn't provide templates. Read when setting up test infrastructure. Related: viewmodel-testing.md (testing with mocks), async-testing.md (networking mocks).
</overview>

## The Pattern: Protocol + Mock + DI

### 1. Define Protocol

```swift
protocol TaskRepositoryProtocol: Sendable {
    func fetchAll() async throws -> [TaskItem]
    func save(_ item: TaskItem) async throws
    func delete(id: UUID) async throws
}
```

### 2. Production Implementation

```swift
final class TaskRepository: TaskRepositoryProtocol {
    private let context: ModelContext

    init(context: ModelContext) {
        self.context = context
    }

    func fetchAll() async throws -> [TaskItem] {
        let descriptor = FetchDescriptor<TaskItem>(sortBy: [SortDescriptor(\.createdAt)])
        return try context.fetch(descriptor)
    }

    func save(_ item: TaskItem) async throws {
        context.insert(item)
        try context.save()
    }

    func delete(id: UUID) async throws {
        let descriptor = FetchDescriptor<TaskItem>(predicate: #Predicate { $0.id == id })
        if let item = try context.fetch(descriptor).first {
            context.delete(item)
            try context.save()
        }
    }
}
```

### 3. Mock Implementation

```swift
final class MockTaskRepository: TaskRepositoryProtocol, @unchecked Sendable {
    var stubbedTasks: [TaskItem] = []
    var shouldThrow = false
    var savedItems: [TaskItem] = []
    var deletedIDs: [UUID] = []

    func fetchAll() async throws -> [TaskItem] {
        if shouldThrow { throw RepositoryError.fetchFailed }
        return stubbedTasks
    }

    func save(_ item: TaskItem) async throws {
        if shouldThrow { throw RepositoryError.saveFailed }
        savedItems.append(item)
    }

    func delete(id: UUID) async throws {
        if shouldThrow { throw RepositoryError.deleteFailed }
        deletedIDs.append(id)
    }
}
```

### 4. Inject via Init

```swift
@Observable @MainActor
class TaskListViewModel {
    var tasks: [TaskItem] = []
    var isLoading = false
    private let repository: TaskRepositoryProtocol

    init(repository: TaskRepositoryProtocol) {
        self.repository = repository
    }

    func loadTasks() async {
        isLoading = true
        defer { isLoading = false }
        do {
            tasks = try await repository.fetchAll()
        } catch {
            // handle error
        }
    }
}
```

## Mock Network Client

```swift
protocol APIClientProtocol: Sendable {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

struct MockAPIClient: APIClientProtocol {
    var result: Any?
    var error: Error?

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        if let error { throw error }
        guard let result = result as? T else {
            fatalError("Mock not configured for type \(T.self)")
        }
        return result
    }
}
```

## Mock UserDefaults

Use isolated instances — never touch `UserDefaults.standard` in tests:

```swift
@Test func userPreferencesSaved() {
    let defaults = UserDefaults(suiteName: "test-\(UUID().uuidString)")!
    defer { defaults.removePersistentDomain(forName: defaults.suiteName!) }

    let settings = SettingsManager(defaults: defaults)
    settings.setTheme(.dark)
    #expect(settings.theme == .dark)
}
```

## Mock Cache

```swift
protocol CacheProtocol: Sendable {
    func get(_ key: String) -> Data?
    func set(_ key: String, data: Data)
    func clear()
}

final class MockCache: CacheProtocol, @unchecked Sendable {
    private var storage: [String: Data] = [:]

    func get(_ key: String) -> Data? { storage[key] }
    func set(_ key: String, data: Data) { storage[key] = data }
    func clear() { storage.removeAll() }
}
```

## Environment Key for DI in SwiftUI

```swift
private struct RepositoryKey: EnvironmentKey {
    static let defaultValue: any TaskRepositoryProtocol = TaskRepository(context: .default)
}

extension EnvironmentValues {
    var taskRepository: any TaskRepositoryProtocol {
        get { self[RepositoryKey.self] }
        set { self[RepositoryKey.self] = newValue }
    }
}

// In tests/previews
ContentView()
    .environment(\.taskRepository, MockTaskRepository())
```

## Rules

- **One protocol per dependency** — not one mega-protocol
- **Mock tracks calls** — `savedItems`, `deletedIDs` for verification
- **`shouldThrow` flag** — toggles error simulation
- **`@unchecked Sendable`** on mocks — acceptable in test targets
- **Isolated UserDefaults** — never share state between tests
