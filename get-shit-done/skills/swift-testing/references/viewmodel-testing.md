<overview>
Testing @Observable ViewModels — the most valuable fork-unique pattern. Covers direct ViewModel testing, protocol-injected repositories, optimistic update with revert, and snapshot testing. Read when testing business logic in ViewModels. Related: mocking.md (mock setup), fundamentals.md (assertion patterns).
</overview>

## Why Test ViewModels, Not Views

- SwiftUI View `body` is unpredictable and framework-managed — don't test it
- `@Observable` ViewModels are plain Swift classes — fully testable
- Test the **logic and state**, verify the **UI visually** (Previews/screenshots)

## Basic ViewModel Test

```swift
import Testing
@testable import MyApp

@Suite("TaskListViewModel")
struct TaskListViewModelTests {
    let mockRepo: MockTaskRepository
    let sut: TaskListViewModel

    init() {
        mockRepo = MockTaskRepository()
        mockRepo.stubbedTasks = [
            TaskItem(title: "Buy groceries"),
            TaskItem(title: "Clean house"),
        ]
        sut = TaskListViewModel(repository: mockRepo)
    }

    @Test @MainActor
    func loadTasks() async {
        await sut.loadTasks()
        #expect(sut.tasks.count == 2)
        #expect(sut.isLoading == false)
    }

    @Test @MainActor
    func loadTasksShowsLoading() async {
        // Before load
        #expect(sut.isLoading == false)

        // During load (we can't easily test mid-async, so test before/after)
        await sut.loadTasks()
        #expect(sut.isLoading == false)
    }

    @Test @MainActor
    func loadTasksHandlesError() async {
        mockRepo.shouldThrow = true
        await sut.loadTasks()
        #expect(sut.tasks.isEmpty)
        #expect(sut.errorMessage != nil)
    }
}
```

## Testing with Protocol-Injected Repository

The ViewModel accepts a protocol, not a concrete type:

```swift
@Observable @MainActor
class ProfileViewModel {
    var profile: UserProfile?
    var errorMessage: String?
    private let repository: ProfileRepositoryProtocol

    init(repository: ProfileRepositoryProtocol) {
        self.repository = repository
    }

    func load() async {
        do {
            profile = try await repository.fetchProfile()
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

Test with mock:

```swift
@Test @MainActor
func profileLoadsSuccessfully() async {
    let mock = MockProfileRepository()
    mock.stubbedProfile = UserProfile(name: "Ada", email: "ada@example.com")

    let vm = ProfileViewModel(repository: mock)
    await vm.load()

    #expect(vm.profile?.name == "Ada")
    #expect(vm.errorMessage == nil)
}
```

## Optimistic Update with Revert

Pattern: update UI immediately, revert if server call fails.

```swift
@Observable @MainActor
class TaskListViewModel {
    var tasks: [TaskItem] = []
    private let repository: TaskRepositoryProtocol

    func toggleComplete(_ task: TaskItem) async {
        // Optimistic update
        let originalState = task.isComplete
        task.isComplete.toggle()

        do {
            try await repository.save(task)
        } catch {
            // Revert on failure
            task.isComplete = originalState
        }
    }
}
```

Test the revert:

```swift
@Test @MainActor
func toggleRevertsOnFailure() async {
    let mock = MockTaskRepository()
    mock.shouldThrow = true

    let task = TaskItem(title: "Test", isComplete: false)
    let vm = TaskListViewModel(repository: mock)
    vm.tasks = [task]

    await vm.toggleComplete(task)

    // Should revert to original state
    #expect(task.isComplete == false)
}
```

## Testing State Transitions

```swift
@Test @MainActor
func deleteRemovesFromList() async {
    let mock = MockTaskRepository()
    let vm = TaskListViewModel(repository: mock)
    let task = TaskItem(title: "Delete me")
    vm.tasks = [task]

    await vm.delete(task)

    #expect(vm.tasks.isEmpty)
    #expect(mock.deletedIDs.contains(task.id))
}
```

## Snapshot Testing (Optional)

For views where pixel accuracy matters, use swift-snapshot-testing:

```swift
import SnapshotTesting

@Test func profileCardRendersCorrectly() {
    let view = ProfileCard(user: .preview)
    assertSnapshot(of: view, as: .image(layout: .device(config: .iPhone13)))
}
```

Use sparingly — snapshot tests are brittle and slow. Prefer ViewModel unit tests + manual Preview verification.

## Testing Patterns Checklist

| Pattern | Test |
|---------|------|
| Happy path | Load data successfully |
| Error handling | Repository throws → ViewModel shows error |
| Empty state | No data → ViewModel reflects empty |
| Optimistic update | Success → state persists; Failure → state reverts |
| Loading state | isLoading toggles correctly |
| Delete/Add | List count changes, mock tracks call |
