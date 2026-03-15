<overview>
Network error handling: HTTP status codes, typed NetworkError enum, retry strategies with exponential backoff, offline detection with NWPathMonitor, timeout configuration, and certificate pinning overview. Read when implementing error handling for network calls. Related: api-client.md (client patterns), app-architecture error-handling.md (general error patterns).
</overview>

## Typed NetworkError

```swift
enum NetworkError: LocalizedError {
    case invalidURL
    case invalidResponse
    case httpError(statusCode: Int, data: Data)
    case decodingFailed(underlying: Error)
    case noConnection
    case timeout
    case unauthorized
    case serverError(statusCode: Int)

    var errorDescription: String? {
        switch self {
        case .invalidURL: "Invalid URL."
        case .invalidResponse: "Invalid server response."
        case .httpError(let code, _): "HTTP error \(code)."
        case .decodingFailed: "Unable to process the response."
        case .noConnection: "No internet connection."
        case .timeout: "Request timed out."
        case .unauthorized: "Session expired. Please sign in again."
        case .serverError(let code): "Server error (\(code))."
        }
    }
}
```

## HTTP Status Code Handling

```swift
func handleResponse(_ response: HTTPURLResponse, data: Data) throws {
    switch response.statusCode {
    case 200...299:
        return // Success
    case 401:
        throw NetworkError.unauthorized
    case 429:
        throw NetworkError.httpError(statusCode: 429, data: data) // Rate limited
    case 500...599:
        throw NetworkError.serverError(statusCode: response.statusCode)
    default:
        throw NetworkError.httpError(statusCode: response.statusCode, data: data)
    }
}
```

## Retry with Exponential Backoff

```swift
func fetchWithRetry<T>(
    maxRetries: Int = 3,
    operation: () async throws -> T
) async throws -> T {
    var lastError: Error?

    for attempt in 0..<maxRetries {
        do {
            return try await operation()
        } catch {
            lastError = error
            // Don't retry cancellation
            if error is CancellationError { throw error }
            // Don't retry client errors (4xx except 429)
            if let networkError = error as? NetworkError,
               case .httpError(let code, _) = networkError,
               (400...499).contains(code), code != 429 {
                throw error
            }
            // Backoff: 1s, 2s, 4s
            if attempt < maxRetries - 1 {
                let delay = pow(2.0, Double(attempt))
                try await Task.sleep(for: .seconds(delay))
                try Task.checkCancellation()
            }
        }
    }
    throw lastError!
}
```

## Offline Detection with NWPathMonitor

```swift
import Network

@Observable
@MainActor
class ConnectivityMonitor {
    private(set) var isConnected = true
    private(set) var connectionType: ConnectionType = .unknown
    private let monitor = NWPathMonitor()

    enum ConnectionType {
        case wifi, cellular, wired, unknown
    }

    func start() {
        monitor.pathUpdateHandler = { [weak self] path in
            Task { @MainActor in
                self?.isConnected = path.status == .satisfied
                self?.connectionType = self?.type(from: path) ?? .unknown
            }
        }
        monitor.start(queue: DispatchQueue(label: "connectivity"))
    }

    func stop() {
        monitor.cancel()
    }

    private func type(from path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) { return .wifi }
        if path.usesInterfaceType(.cellular) { return .cellular }
        if path.usesInterfaceType(.wiredEthernet) { return .wired }
        return .unknown
    }
}
```

### Usage in Views

```swift
struct ContentView: View {
    @State private var connectivity = ConnectivityMonitor()

    var body: some View {
        content
            .overlay {
                if !connectivity.isConnected {
                    offlineBanner
                }
            }
            .task { connectivity.start() }
    }
}
```

## Timeout Configuration

```swift
var request = URLRequest(url: url)
request.timeoutInterval = 30  // 30 seconds

// Or configure at session level
let config = URLSessionConfiguration.default
config.timeoutIntervalForRequest = 30
config.timeoutIntervalForResource = 300  // 5 minutes for large downloads
let session = URLSession(configuration: config)
```

## Certificate Pinning (Overview)

For apps with strict security requirements:

```swift
class PinningDelegate: NSObject, URLSessionDelegate {
    func urlSession(
        _ session: URLSession,
        didReceive challenge: URLAuthenticationChallenge
    ) async -> (URLSession.AuthChallengeDisposition, URLCredential?) {
        guard let serverTrust = challenge.protectionSpace.serverTrust else {
            return (.cancelAuthenticationChallenge, nil)
        }
        // Validate against pinned certificate
        // ...
        return (.useCredential, URLCredential(trust: serverTrust))
    }
}
```

Use certificate pinning only for high-security apps (banking, health data). For most apps, ATS provides sufficient security.

## Error Display in SwiftUI

```swift
struct NetworkAwareView<Content: View>: View {
    let state: LoadingState<some Any>
    let retry: () async -> Void
    @ViewBuilder let content: () -> Content

    var body: some View {
        switch state {
        case .idle: Color.clear
        case .loading: ProgressView()
        case .loaded: content()
        case .failed(let error):
            ContentUnavailableView {
                Label("Connection Error", systemImage: "wifi.exclamationmark")
            } description: {
                Text(error.localizedDescription)
            } actions: {
                Button("Retry") { Task { await retry() } }
            }
        }
    }
}
```
