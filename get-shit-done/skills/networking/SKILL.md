---
name: networking
description: URLSession API clients, async networking, error handling, WebView/WebPage integration, offline detection
---

<essential_principles>
## How This Skill Works

1. **URLSession native only.** Do NOT use Alamofire. URLSession with async/await covers all REST/GraphQL needs.
2. **Thin client pattern.** Protocol-based API client with a single `request<T>()` method. Easy to mock, easy to test.
3. **Verify HTTP status before decoding.** Always check response codes. Never decode blindly.
4. **Always handle offline.** Use `NWPathMonitor` for connectivity, show appropriate UI, retry with backoff.
5. **Never block main thread.** All network calls via async/await. ViewModels are `@MainActor @Observable`.
</essential_principles>

<intake>
## What do you need?

1. Build an API client
2. Handle network errors and retries
3. Add WebView/WebPage integration
4. Debug network issues
5. Add offline support or connectivity monitoring
</intake>

<routing>
| Response | Reference |
|----------|-----------|
| 1, "API client", "URLSession", "REST", "JSON", "Codable", "download" | `references/api-client.md` |
| 2, "error", "retry", "timeout", "offline", "status code" | `references/error-handling.md` |
| 3, "WebView", "WebPage", "HTML", "JavaScript", "web content" | `references/webview-integration.md` |
| 4, "debug", "network", "request", "response" | `references/error-handling.md` |
| 5, "offline", "connectivity", "NWPathMonitor" | `references/error-handling.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/api-client.md | URLSession patterns, async data/bytes/upload, Codable, thin client design, background downloads |
| references/error-handling.md | HTTP status codes, typed NetworkError, retry strategies, offline detection, timeouts |
| references/webview-integration.md | WebView/WebPage SwiftUI integration, JavaScript bridge, URL schemes, navigation, snapshots |
</reference_index>

<canonical_terminology>
## Terminology

- **URLSession** (not: Alamofire, NSURLConnection)
- **async/await** (not: completion handlers, Combine publishers for network calls)
- **thin client** (not: fat networking layer, manager class)
- **NWPathMonitor** (not: Reachability, SCNetworkReachability)
- **WebView** (not: WKWebView in SwiftUI context — use the SwiftUI WebView struct)
</canonical_terminology>
