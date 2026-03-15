<overview>
WebView/WebPage SwiftUI integration (iOS 26+): embedding web content, JavaScript bridge, URL scheme handlers, navigation management, content worlds, snapshots, and PDF export. Based on Apple's SwiftUI-WebKit-Integration documentation. Read when adding web content to a SwiftUI app. Related: api-client.md (native networking).
</overview>

## WebView Basics

```swift
import SwiftUI
import WebKit

struct ContentView: View {
    var body: some View {
        WebView(url: URL(string: "https://www.apple.com"))
            .frame(height: 400)
    }
}
```

## WebView with WebPage Controller

`WebPage` is an `@Observable` class that controls web content behavior:

```swift
struct BrowserView: View {
    @State private var page = WebPage()

    var body: some View {
        NavigationStack {
            WebView(page)
                .navigationTitle(page.title ?? "")
        }
        .onAppear {
            if let url = URL(string: "https://www.apple.com") {
                let _ = page.load(URLRequest(url: url))
            }
        }
    }
}
```

## WebPage Configuration

```swift
var configuration = WebPage.Configuration()
configuration.loadsSubresources = true
configuration.defaultNavigationPreferences.allowsContentJavaScript = true
configuration.websiteDataStore = .default()  // or .nonPersistent()

let page = WebPage(configuration: configuration)
page.customUserAgent = "MyApp/1.0"
```

## Navigation Management

### Loading Content

```swift
// From URL
let _ = page.load(URLRequest(url: url))

// From HTML string
let _ = page.load(html: "<h1>Hello</h1>", baseURL: URL(string: "https://example.com")!)

// From data
let _ = page.load(data, mimeType: "text/html", characterEncoding: .utf8, baseURL: baseURL)
```

### Navigation Controls

```swift
let canGoBack = !page.backForwardList.backList.isEmpty
let canGoForward = !page.backForwardList.forwardList.isEmpty

// Navigate back
if let backItem = page.backForwardList.backItem {
    let _ = page.load(backItem)
}
```

### Observing Navigation Events

```swift
.onChange(of: page.currentNavigationEvent) { _, newEvent in
    if let event = newEvent {
        switch event.state {
        case .started: isLoading = true
        case .finished, .failed: isLoading = false
        default: break
        }
    }
}
```

### Custom Navigation Policies

```swift
struct MyNavigationDecider: WebPage.NavigationDeciding {
    func decidePolicyFor(navigationAction: WebPage.NavigationAction) async -> WebPage.NavigationPreferences? {
        // Return nil to cancel navigation
        if navigationAction.request.url?.host == "blocked.com" {
            return nil
        }
        var prefs = WebPage.NavigationPreferences()
        prefs.allowsContentJavaScript = true
        return prefs
    }

    func decidePolicyFor(navigationResponse: WebPage.NavigationResponse) async -> Bool {
        if let http = navigationResponse.response as? HTTPURLResponse {
            return http.statusCode == 200
        }
        return true
    }
}
```

## JavaScript Bridge

### Execute JavaScript

```swift
let title = try await page.callJavaScript("document.title")
```

### With Arguments

```swift
let script = """
function findElement(selector) {
    return document.querySelector(selector)?.textContent;
}
return findElement(selector);
"""
let result = try await page.callJavaScript(script, arguments: ["selector": ".heading"])
```

### Content Worlds (Isolated Execution)

```swift
let result = try await page.callJavaScript("document.title", contentWorld: .page)
// .page = shares with page scripts
// .defaultClient = isolated from page scripts
```

## Custom URL Scheme Handler

```swift
struct MySchemeHandler: URLSchemeHandler {
    func start(task: URLSchemeTask) {
        guard let url = task.request.url, url.scheme == "myapp" else {
            task.didFailWithError(URLError(.badURL))
            return
        }
        let html = "<html><body><h1>Custom Content</h1></body></html>"
        let response = URLResponse(url: url, mimeType: "text/html",
                                   expectedContentLength: -1, textEncodingName: "utf-8")
        task.didReceive(response)
        task.didReceive(Data(html.utf8))
        task.didFinish()
    }

    func stop(task: URLSchemeTask) { }
}

// Register
var config = WebPage.Configuration()
config.setURLSchemeHandler(MySchemeHandler(), forURLScheme: "myapp")
```

## View Modifiers

```swift
WebView(url: url)
    .webViewBackForwardNavigationGestures(.disabled)
    .webViewMagnificationGestures(.enabled)
    .webViewLinkPreviews(.disabled)
    .webViewTextSelection(.enabled)
    .webViewContentBackground(.color(.systemBackground))
    .findNavigator(isPresented: $searchVisible)
```

## Snapshot and PDF Export

### Screenshot

```swift
let config = WKSnapshotConfiguration()
config.rect = CGRect(x: 0, y: 0, width: 1024, height: 768)
let image = try await page.snapshot(config)
```

### PDF Generation

```swift
let pdfData = try await page.pdf(configuration: WKPDFConfiguration())
```

### Web Archive

```swift
let archiveData = try await page.webArchiveData()
```
