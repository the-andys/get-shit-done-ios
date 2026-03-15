<overview>
Dynamic Type support: text styles, @ScaledMetric, layout adaptation at accessibility sizes, Large Content Viewer, and line limit strategies. Read when implementing scalable text and layouts. Related: voiceover.md (labels), enforcement.md (mandatory rules).
</overview>

## Automatic Scaling

SwiftUI text with text styles scales automatically:

```swift
Text("Title").font(.title)
Text("Body").font(.body)
Text("Caption").font(.caption)
```

Custom fonts with scaling:

```swift
Text("Custom").font(.custom("Avenir-Medium", size: 17, relativeTo: .body))
```

**WRONG — fixed size, does NOT scale:**
```swift
Text("Broken").font(.system(size: 17))
```

## Detecting Accessibility Sizes

```swift
@Environment(\.dynamicTypeSize) private var dynamicTypeSize

var isAccessibilitySize: Bool {
    dynamicTypeSize.isAccessibilitySize
}
```

## Layout Adaptation

At accessibility sizes, switch horizontal layouts to vertical:

### ViewThatFits (Simplest)

```swift
ViewThatFits {
    HStack { content }  // Preferred
    VStack { content }  // Fallback when horizontal doesn't fit
}
```

### AnyLayout (iOS 16+)

```swift
let layout = dynamicTypeSize.isAccessibilitySize
    ? AnyLayout(VStackLayout())
    : AnyLayout(HStackLayout())

layout {
    Image(systemName: "star.fill")
    Text("Favorites")
}
```

### Reusable AdaptiveStack

```swift
struct AdaptiveStack<Content: View>: View {
    @Environment(\.dynamicTypeSize) private var dynamicTypeSize
    let spacing: CGFloat
    @ViewBuilder let content: () -> Content

    var body: some View {
        if dynamicTypeSize.isAccessibilitySize {
            VStack(spacing: spacing) { content() }
        } else {
            HStack(spacing: spacing) { content() }
        }
    }
}
```

## @ScaledMetric

Scale non-text values with Dynamic Type:

```swift
@ScaledMetric private var avatarSize: CGFloat = 60
@ScaledMetric(relativeTo: .caption) private var iconSize: CGFloat = 16

Image(systemName: "star.fill")
    .frame(width: iconSize, height: iconSize)
```

## Line Limits for Large Sizes

```swift
@Environment(\.dynamicTypeSize) private var dynamicTypeSize

private var titleLineLimit: Int {
    if dynamicTypeSize >= .accessibility3 { return 6 }
    if dynamicTypeSize.isAccessibilitySize { return 4 }
    return 2
}

Text(title).lineLimit(titleLineLimit)
```

## Large Content Viewer

For bar items and icons that don't scale (navigation bars, toolbars):

```swift
Button { showCart.toggle() } label: {
    Image(systemName: "cart.fill")
}
.accessibilityShowsLargeContentViewer {
    Image(systemName: "cart.fill")
    Text("Cart, \(orderCount) items")
}
```

User long-presses to see enlarged version. Don't scale chrome controls — use this instead.

## Testing Dynamic Type

- **Xcode shortcut:** Option + Command + +/- to adjust text size in simulator
- **Environment Overrides:** Debug Area toolbar in Xcode
- **Preview:** `.dynamicTypeSize(.accessibility3)` in `#Preview`
- **Double-length pseudo:** Edit Scheme > Options > App Language

```swift
#Preview("Accessibility Size") {
    ProfileView(user: .preview)
        .dynamicTypeSize(.accessibility3)
}
```
