<overview>
VoiceOver implementation in SwiftUI: labels, traits, hints, grouping, custom actions, adjustable controls, focus management, announcements, and Magic Tap. Read when adding VoiceOver support to views. Related: dynamic-type.md (text scaling), testing.md (VoiceOver testing), enforcement.md (mandatory rules).
</overview>

## Labels

```swift
// Icon-only buttons MUST have labels
Button(action: play) { Image(systemName: "play.fill") }
    .accessibilityLabel("Play")

// Images that convey meaning
Image("companyLogo")
    .accessibilityLabel("Company name")

// Decorative images — hide from VoiceOver
Image("decorativeDivider")
    .accessibilityHidden(true)
```

**Rule:** Don't put trait names in labels. Say "Close", not "Close button" — VoiceOver adds "button" automatically.

### Badge Pattern

Stable label + dynamic value:

```swift
Button { showCart.toggle() } label: {
    ZStack(alignment: .topTrailing) {
        Image(systemName: "cart.fill")
        if orderCount > 0 { Text("\(orderCount)").background(.red) }
    }
}
.accessibilityLabel("Cart")
.accessibilityValue("\(orderCount) items")
```

## Hints

Only when the action isn't obvious from the label:

```swift
Toggle(isOn: $isEnabled) { Text("Notifications") }
    .accessibilityHint("Double tap to toggle notifications on or off")
```

## Grouping

### .combine — Merge child labels

```swift
HStack {
    Image(systemName: "star.fill")
    Text("Favorites")
    Text("(\(count))")
}
.accessibilityElement(children: .combine)
// VoiceOver reads: "star, Favorites, (5)"
```

### .ignore — Single element with custom label

```swift
VStack {
    Text(product.name)
    Text(product.price.formatted())
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("\(product.name), \(product.price.formatted())")
```

### .contain — Semantic group, children accessible individually

```swift
VStack { /* children */ }
.accessibilityElement(children: .contain)
```

## Custom Actions

For secondary actions hidden behind swipes:

```swift
NavigationLink { DrinkDetail() } label: { Text(drink.name) }
    .accessibilityAction(named: "Add to cart") { basket.add(drink) }
```

## Adjustable Controls

For increment/decrement:

```swift
HStack { /* rating stars */ }
    .accessibilityElement(children: .ignore)
    .accessibilityLabel("Rating")
    .accessibilityValue("\(rating) of 5")
    .accessibilityAdjustableAction { direction in
        switch direction {
        case .increment: if rating < 5 { rating += 1 }
        case .decrement: if rating > 0 { rating -= 1 }
        @unknown default: break
        }
    }
```

## Accessibility Representation (iOS 16+)

Complex custom UI seen as native control:

```swift
CustomRatingView(rating: $rating)
    .accessibilityRepresentation {
        Stepper("Rating", value: $rating, in: 1...5)
    }
```

## Traits

```swift
Text("Settings").accessibilityAddTraits(.isHeader)

// Heading levels (iOS 17+)
Text("Main Title").accessibilityHeading(.h1)
Text("Subsection").accessibilityHeading(.h2)

// Frequently updating values
Text(timer).accessibilityAddTraits(.updatesFrequently)
```

Use `.disabled()` modifier instead of `.notEnabled` trait.

## Focus Management (iOS 15+)

```swift
@AccessibilityFocusState private var isFocused: Bool

TextField("Name", text: $name)
    .accessibilityFocused($isFocused)

Button("Focus Field") { isFocused = true }
```

## Announcements

```swift
if #available(iOS 17, *) {
    var announcement = AttributedString(message)
    announcement.accessibilitySpeechAnnouncementPriority = .high
    AccessibilityNotification.Announcement(announcement).post()
} else {
    UIAccessibility.post(notification: .announcement, argument: message)
}
```

Use for dynamic content changes (toasts, loading completion, errors).

## Magic Tap & Escape

```swift
.accessibilityAction(.magicTap) { isPlaying.toggle() }
.accessibilityAction(.escape) { dismiss() }
```

## Input Labels (Voice Control)

```swift
Button("Remove User") { remove() }
    .accessibilityInputLabels(["Remove User", "Remove", "Delete"])
```
