<overview>
Accessibility in SwiftUI views: labels, traits, hints, Dynamic Type, @ScaledMetric, grouping, and VoiceOver focus management. This covers accessibility AS PART OF building views. For full accessibility depth (auditing, testing, Assistive Access), load the accessibility skill. Related: performance.md (accessibility doesn't hurt performance), components.md (controls with built-in a11y).
</overview>

## Core Principle

Use `Button` over `onTapGesture`. Semantic controls (Button, Toggle, Slider, Picker) get VoiceOver support automatically. `onTapGesture` is invisible to assistive technology.

## Labels

```swift
// Icon-only buttons MUST have labels
Button(action: addItem) {
    Image(systemName: "plus")
}
.accessibilityLabel("Add new item")

// Images that convey meaning
Image("companyLogo")
    .accessibilityLabel("Company name")

// Decorative images — hide from VoiceOver
Image("decorativeDivider")
    .accessibilityHidden(true)
```

## Hints

```swift
Toggle(isOn: $isEnabled) {
    Text("Notifications")
}
.accessibilityHint("Double tap to toggle notifications on or off")
```

Use hints when the action isn't obvious from the label alone.

## Dynamic Type

All text MUST scale with the user's preferred size.

```swift
// CORRECT: Semantic font styles scale automatically
Text("Title").font(.title)
Text("Body").font(.body)

// CORRECT: Custom font with scaling
Text("Custom").font(.custom("Avenir-Medium", size: 17, relativeTo: .body))

// WRONG: Fixed size — does not scale
Text("Broken").font(.system(size: 17))
```

## @ScaledMetric

Scale custom values with Dynamic Type:

```swift
@ScaledMetric private var avatarSize: CGFloat = 60
@ScaledMetric(relativeTo: .caption) private var iconSize: CGFloat = 16

Image(systemName: "star.fill")
    .frame(width: iconSize, height: iconSize)
```

## Element Grouping

```swift
// Combine: Joins child labels into one VoiceOver element
HStack {
    Image(systemName: "star.fill")
    Text("Favorites")
    Text("(\(count))")
}
.accessibilityElement(children: .combine)
// VoiceOver reads: "star, Favorites, (5)"

// Ignore: Replaces children with custom label
VStack {
    Text(product.name)
    Text(product.price.formatted(.currency(code: "USD")))
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("\(product.name), \(product.price.formatted(.currency(code: "USD")))")

// Contain: Semantic grouping (VoiceOver traverses children but treats as group)
VStack { /* children */ }
.accessibilityElement(children: .contain)
```

## Adjustable Action

For increment/decrement controls:

```swift
struct RatingControl: View {
    @Binding var rating: Int

    var body: some View {
        HStack { /* stars */ }
            .accessibilityElement()
            .accessibilityLabel("Rating")
            .accessibilityValue("\(rating) of 5")
            .accessibilityAdjustableAction { direction in
                switch direction {
                case .increment: if rating < 5 { rating += 1 }
                case .decrement: if rating > 0 { rating -= 1 }
                @unknown default: break
                }
            }
    }
}
```

## Bold Text Support

```swift
@Environment(\.legibilityWeight) private var legibilityWeight

Text("Important")
    .font(legibilityWeight == .bold
        ? .custom("Avenir-Heavy", size: 17, relativeTo: .body)
        : .custom("Avenir-Medium", size: 17, relativeTo: .body))
```

System fonts handle Bold Text automatically.

## Reduce Motion

```swift
@Environment(\.accessibilityReduceMotion) private var reduceMotion

content
    .animation(reduceMotion ? nil : .spring(), value: isExpanded)
    .transition(reduceMotion ? .opacity : .slide)
```

## Minimum Tap Targets

All interactive elements must be at least 44x44pt:

```swift
Button("Small Label") { }
    .frame(minWidth: 44, minHeight: 44)
```

## Checklist

- [ ] All buttons/controls have `.accessibilityLabel`
- [ ] Decorative images use `.accessibilityHidden(true)`
- [ ] Text uses semantic font styles or `relativeTo:`
- [ ] Animations respect `accessibilityReduceMotion`
- [ ] Tap targets at least 44x44pt
- [ ] Colors have light and dark mode variants
- [ ] Related elements grouped with `.accessibilityElement(children:)`
