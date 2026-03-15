<overview>
Anti-patterns that GSD agents must catch and fix. Each entry includes the rule, why it matters, the correct alternative, and exceptions. Read when reviewing code quality, fixing code smells, or onboarding to project conventions. Related: code-structure.md (correct patterns), framework-selection.md (correct framework choices).
</overview>

## Force Unwraps Without Justification

```swift
// WRONG
let url = URL(string: someUserInput)!          // Crash if invalid
let cell = tableView.dequeueReusableCell(...)! // Crash if registration missing

// CORRECT
guard let url = URL(string: someUserInput) else {
    throw ValidationError.invalidURL
}
```

**Exception:** Force unwrap is acceptable for compile-time-known constant values with a comment:
```swift
let url = URL(string: "https://api.example.com")!  // Constant, always valid
```

## Combine in New Code

```swift
// WRONG: Combine for simple async operations
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

**Exception:** Combine is acceptable for reactive streams needing `debounce`, `throttle`, `combineLatest`.

## PreviewProvider (Deprecated)

```swift
// WRONG
struct MyView_Previews: PreviewProvider {
    static var previews: some View { MyView() }
}

// CORRECT
#Preview { MyView() }
```

No exceptions. `#Preview` is always the right choice.

## DispatchQueue.main.async for UI Updates

```swift
// WRONG
DispatchQueue.main.async {
    self.isLoading = false
    self.data = newData
}

// CORRECT
@MainActor
func updateData(_ newData: [Item]) {
    self.isLoading = false
    self.data = newData
}

// Or mark the whole type
@MainActor @Observable
class ViewModel { /* ... */ }
```

## Massive View Bodies

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

private var headerSection: some View { /* ... */ }
```

**Rule of thumb:** If `body` exceeds ~30 lines, extract subviews.

## God ViewModels

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

## Hardcoded User-Facing Strings

```swift
// WRONG
Text("Welcome back!")
Button("Delete") { deleteItem() }

// CORRECT
Text(String(localized: "welcome_back"))
Button(String(localized: "delete_action")) { deleteItem() }
```

**Exception:** Accessibility identifiers (`.accessibilityIdentifier("login_button")`) are NOT user-facing and remain as plain strings.

## Summary Table

| Anti-Pattern | Correct Alternative | Exception |
|---|---|---|
| Force unwrap on runtime values | `guard let` / `if let` / throw | Compile-time constants with comment |
| Combine for simple async | async/await | Reactive streams (debounce, throttle) |
| `PreviewProvider` | `#Preview` macro | None |
| `DispatchQueue.main.async` | `@MainActor` | None |
| View body 100+ lines | Extract subviews (~30 line rule) | None |
| God ViewModel | One ViewModel per feature | None |
| Hardcoded user-facing strings | `String(localized:)` | Accessibility identifiers |
| `ObservableObject` + `@Published` | `@Observable` (iOS 17+) | iOS 16 compatibility |
| `NavigationView` | `NavigationStack` | None |
| `onChange(of:perform:)` one-param | `onChange(of:) { old, new in }` | None |
| Singletons for DI | `@Environment` injection | None |
| `Any` / `AnyObject` abuse | Generics or protocols with associated types | None |
| Stringly-typed APIs | Enums, strongly-typed identifiers | None |
| `try?` silently | Handle or log the error | None |
