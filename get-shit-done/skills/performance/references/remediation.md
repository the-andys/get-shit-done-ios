<overview>
Fix patterns for SwiftUI performance issues: narrow state scope, stabilize identity, precompute, downsample images, lazy loading, @ObservationIgnored, equatable views, and advanced memory optimization (InlineArray, Span). Related: code-review.md (identify issues), instruments.md (measure impact).
</overview>

## Narrow State Scope

Pass only what each view needs:

```swift
// BEFORE — entire store dependency
struct ItemRow: View {
    @Environment(Store.self) private var store
    let index: Int
    var body: some View { Text(store.items[index].name) }
}

// AFTER — specific value
struct ItemRow: View {
    let name: String
    var body: some View { Text(name) }
}
```

## Stabilize Identity

```swift
// BEFORE — \.self is unstable for mutable types
ForEach(items, id: \.self) { ... }

// AFTER — stable identifier
ForEach(items) { item in ... }  // Uses Identifiable.id
```

Ensure `id` doesn't change when content changes. UUID generated at creation time is stable. UUID generated per render is not.

## Precompute Outside body

```swift
// BEFORE — sorts every recompute
var body: some View {
    ForEach(items.sorted(by: { $0.date > $1.date })) { ... }
}

// AFTER — sorted in ViewModel
@Observable class ListViewModel {
    var sortedItems: [Item] = []

    func load() async {
        let raw = try await repo.fetchAll()
        sortedItems = raw.sorted(by: { $0.date > $1.date })
    }
}
```

## Cache Formatters

```swift
// Static formatter
private static let priceFormatter: NumberFormatter = {
    let f = NumberFormatter()
    f.numberStyle = .currency
    return f
}()

// Or use FormatStyle (auto-cached)
Text(price, format: .currency(code: "USD"))
```

## Downsample Images

Don't decode full-resolution images for small thumbnails:

```swift
func downsample(imageAt url: URL, to size: CGSize, scale: CGFloat) -> UIImage? {
    let options: [CFString: Any] = [
        kCGImageSourceShouldCache: false
    ]
    guard let source = CGImageSourceCreateWithURL(url as CFURL, options as CFDictionary) else { return nil }

    let maxDimension = max(size.width, size.height) * scale
    let downsampleOptions: [CFString: Any] = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimension
    ]

    guard let cgImage = CGImageSourceCreateThumbnailAtIndex(source, 0, downsampleOptions as CFDictionary) else { return nil }
    return UIImage(cgImage: cgImage)
}
```

## Lazy Loading

```swift
ScrollView {
    LazyVStack(spacing: 12) {
        ForEach(items) { item in
            ItemRow(item: item)  // Created on demand, recycled
        }
    }
}
```

Use `LazyVStack` / `LazyHStack` for lists over ~20 items. Regular stacks create all views upfront.

iOS 26+: Nested lazy stacks defer loading automatically.

## @ObservationIgnored

Exclude properties from observation tracking:

```swift
@Observable class ViewModel {
    var displayItems: [Item] = []  // Tracked — triggers UI updates
    @ObservationIgnored var cache: [String: Data] = [:]  // Not tracked — no UI updates
    @ObservationIgnored var dateFormatter = DateFormatter()  // Not tracked
}
```

## Equatable Views

Custom equality for coarse-grained skip:

```swift
struct ExpensiveView: View, Equatable {
    let title: String
    let metadata: LargeObject

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.title == rhs.title  // Skip body if only title matters
    }

    var body: some View { /* expensive rendering */ }
}
```

## Off-Main-Thread Closures

These run off main thread — capture values, not bindings:

```swift
.visualEffect { [pulse] content, geometry in
    content.blur(radius: pulse ? 5 : 0)
}
```

Applies to: `Shape.path()`, `visualEffect`, `Layout`, `onGeometryChange`.

## InlineArray (iOS 26+ / Swift 6.2)

Fixed-size array with inline storage — no heap allocation:

```swift
var buffer = InlineArray<16, UInt8>(repeating: 0)
buffer[0] = 42
```

Use for: performance-critical fixed-size buffers, avoiding heap allocation and reference counting. Not for growable collections.

## Span (iOS 26+ / Swift 6.2)

Safe read-only access to contiguous memory:

```swift
func process(_ span: Span<UInt8>) {
    for byte in span { /* zero-copy access */ }
}
```

Cannot escape scope — compile-time safety without runtime overhead. Use for high-performance algorithms, binary parsing, large data processing.

## Priority Order

When fixing performance issues:

1. **Narrow state scope** (cheapest, highest impact)
2. **Stabilize identity** (fixes most list issues)
3. **Precompute / cache formatters** (removes body overhead)
4. **Use lazy stacks** (fixes large list issues)
5. **Downsample images** (fixes memory + rendering)
6. **@ObservationIgnored** (reduces false invalidations)
7. **Equatable views** (last resort for complex views)
