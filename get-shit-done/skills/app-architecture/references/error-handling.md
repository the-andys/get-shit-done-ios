<overview>
Error handling patterns for Swift and SwiftUI: typed errors with enums, try-catch with async/await, guard for early exit, optional chaining, Result for callbacks, and displaying errors in SwiftUI views. Read when implementing error handling, reviewing error patterns, or adding error UI. Related: code-structure.md (ViewModel patterns), anti-patterns.md (silent error swallowing).
</overview>

## Typed Errors with Enums

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

### Error Type Design Rules

- **One enum per domain** — `NetworkError`, `ValidationError`, `StorageError`
- **Always `LocalizedError`** — enables `.localizedDescription` for UI display
- **Associated values** for context — status codes, underlying errors, field names
- **Avoid catch-all cases** — each case should be actionable

## try-catch with async/await

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

### Catch Order

1. **Specific typed errors first** — `catch let error as NetworkError`
2. **CancellationError** — always handle (no-op, don't show error)
3. **Generic catch** — fallback with user-friendly message

## guard for Early Exit

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

**Use `guard` when:** The rest of the function depends on the condition being true.
**Use `if` when:** Both branches have meaningful work to do.

## Optional Chaining and nil-Coalescing

```swift
// Optional chaining — safely traverse optional values
let cityName = user.address?.city?.name

// nil-coalescing — provide defaults
let displayName = user.nickname ?? user.fullName ?? "Anonymous"

// Optional map — transform only if non-nil
let avatarImage = user.avatarURL.map { AsyncImage(url: $0) }
```

## Result Type (for Callbacks)

Use `Result` only when passing success/failure through closures. Prefer async/await for new code.

```swift
// Use Result only for callback-based APIs
func legacyFetch(completion: @escaping (Result<User, NetworkError>) -> Void) { }

// Convert to async/await when wrapping
func fetch() async throws -> User {
    try await withCheckedThrowingContinuation { continuation in
        legacyFetch { result in
            continuation.resume(with: result)
        }
    }
}
```

## Displaying Errors in SwiftUI

### Alert Pattern

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

### Inline Error Display

```swift
struct ProfileView: View {
    @State private var viewModel = ProfileViewModel()

    var body: some View {
        Group {
            if let errorMessage = viewModel.errorMessage {
                ContentUnavailableView {
                    Label("Error", systemImage: "exclamationmark.triangle")
                } description: {
                    Text(errorMessage)
                } actions: {
                    Button("Retry") { Task { await viewModel.load() } }
                }
            } else if let profile = viewModel.profile {
                ProfileContent(profile: profile)
            } else {
                ProgressView()
            }
        }
        .task { await viewModel.load() }
    }
}
```

### Error State in ViewModel

```swift
@MainActor
@Observable
class ProfileViewModel {
    var profile: UserProfile?
    var errorMessage: String?
    var isLoading = false

    func load() async {
        isLoading = true
        errorMessage = nil
        defer { isLoading = false }

        do {
            profile = try await apiClient.fetchProfile()
        } catch is CancellationError {
            // Ignore
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

## Anti-Patterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| `try?` with no handling | Silently swallows errors | Log or display the error |
| Generic `catch { }` empty | Errors disappear | At minimum log the error |
| Throwing in View body | Crashes the view | Handle in ViewModel or `.task` |
| `fatalError()` for recoverable errors | App crashes | Use typed errors and UI feedback |
