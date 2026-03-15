<overview>
URLSession patterns for iOS: async data/bytes/upload, request construction, Codable decoding, thin client architecture, background downloads, and WebSocket basics. Read when building an API client or making network calls. Related: error-handling.md (HTTP errors, retries), webview-integration.md (web content).
</overview>

## URLSession Async Patterns

### Basic Data Request

```swift
func fetchUser(id: String) async throws -> User {
    let url = baseURL.appending(path: "users/\(id)")
    var request = URLRequest(url: url)
    request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")

    let (data, response) = try await URLSession.shared.data(for: request)

    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.invalidResponse
    }

    return try JSONDecoder().decode(User.self, from: data)
}
```

### Bytes Streaming

```swift
func downloadLargeFile(from url: URL) async throws {
    let (bytes, response) = try await URLSession.shared.bytes(for: URLRequest(url: url))

    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw NetworkError.invalidResponse
    }

    for try await byte in bytes {
        // Process streaming data
    }
}
```

### Upload

```swift
func uploadImage(_ data: Data) async throws -> UploadResponse {
    var request = URLRequest(url: baseURL.appending(path: "upload"))
    request.httpMethod = "POST"
    request.setValue("image/jpeg", forHTTPHeaderField: "Content-Type")

    let (responseData, response) = try await URLSession.shared.upload(for: request, from: data)

    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw NetworkError.uploadFailed
    }

    return try JSONDecoder().decode(UploadResponse.self, from: responseData)
}
```

## Thin Client Architecture

Protocol-based design for easy mocking and testing:

```swift
protocol APIClient: Sendable {
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T
}

struct Endpoint {
    let path: String
    let method: HTTPMethod
    let queryItems: [URLQueryItem]?
    let body: Data?
    let headers: [String: String]

    enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case delete = "DELETE"
    }
}

@MainActor
final class ProductionAPIClient: APIClient {
    private let baseURL: URL
    private let session: URLSession
    private let decoder: JSONDecoder

    nonisolated init(baseURL: URL, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
        self.decoder = JSONDecoder()
        self.decoder.dateDecodingStrategy = .iso8601
    }

    nonisolated func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var url = baseURL.appending(path: endpoint.path)
        if let queryItems = endpoint.queryItems {
            url.append(queryItems: queryItems)
        }

        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.httpBody = endpoint.body
        for (key, value) in endpoint.headers {
            request.setValue(value, forHTTPHeaderField: key)
        }
        request.setValue("application/json", forHTTPHeaderField: "Accept")

        let (data, response) = try await session.data(for: request)

        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }

        guard (200...299).contains(httpResponse.statusCode) else {
            throw NetworkError.httpError(statusCode: httpResponse.statusCode, data: data)
        }

        return try decoder.decode(T.self, from: data)
    }
}
```

### Mock Client for Testing

```swift
struct MockAPIClient: APIClient {
    var result: Any?
    var error: Error?

    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        if let error { throw error }
        guard let result = result as? T else {
            throw NetworkError.decodingFailed
        }
        return result
    }
}
```

### Environment Injection

```swift
private struct APIClientKey: EnvironmentKey {
    static let defaultValue: any APIClient = ProductionAPIClient(baseURL: URL(string: "https://api.example.com")!)
}

extension EnvironmentValues {
    var apiClient: any APIClient {
        get { self[APIClientKey.self] }
        set { self[APIClientKey.self] = newValue }
    }
}
```

## Loading States with .task

```swift
struct UserListView: View {
    @State private var users: [User] = []
    @State private var isLoading = true
    @State private var errorMessage: String?

    var body: some View {
        List(users) { UserRow(user: $0) }
            .overlay { if isLoading { ProgressView() } }
            .task { await loadUsers() }
            .refreshable { await loadUsers() }
    }

    private func loadUsers() async {
        isLoading = true
        defer { isLoading = false }
        do {
            users = try await apiClient.fetchUsers()
        } catch is CancellationError {
            // View disappeared — ignore
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

## Background Downloads

```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.app.downloads")
let backgroundSession = URLSession(configuration: config, delegate: downloadDelegate, delegateQueue: nil)

let task = backgroundSession.downloadTask(with: url)
task.resume()
```

Background downloads continue even when the app is suspended.

## WebSocket

```swift
let task = URLSession.shared.webSocketTask(with: url)
task.resume()

// Send
try await task.send(.string("Hello"))

// Receive
let message = try await task.receive()
switch message {
case .string(let text): print(text)
case .data(let data): print(data)
}
```
