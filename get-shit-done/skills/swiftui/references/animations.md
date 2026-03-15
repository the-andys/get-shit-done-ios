<overview>
SwiftUI animations: implicit/explicit, transitions, phase/keyframe, matchedGeometryEffect, and Liquid Glass (iOS 26+). Read when adding animations, transitions, or implementing Liquid Glass design. Related: performance.md (animation performance), latest-apis.md (new animation APIs).
</overview>

## Implicit vs Explicit Animations

### Implicit (State-Tied)

```swift
// CORRECT — with value parameter
Rectangle()
    .frame(width: isExpanded ? 200 : 100)
    .animation(.spring, value: isExpanded)
```

**NEVER use `.animation(_:)` without value parameter — it is deprecated.**

### Explicit (Event-Driven)

```swift
Button("Toggle") {
    withAnimation(.spring) {
        isExpanded.toggle()
    }
}
```

### When to Use Which

| Scenario | Use |
|----------|-----|
| State change should always animate | Implicit `.animation(_:value:)` |
| Animation tied to user action | Explicit `withAnimation` |
| Different properties need different timing | Multiple `.animation` modifiers |

## Animation Placement

Apply AFTER the properties they animate:

```swift
Rectangle()
    .frame(width: isExpanded ? 200 : 100)  // Property to animate
    .animation(.spring, value: isExpanded)   // Animation for above
    .foregroundStyle(.blue)                  // Not animated
```

## Timing Curves

| Curve | When |
|-------|------|
| `.spring` | Default for UI interactions |
| `.bouncy` (iOS 17+) | Playful, energetic elements |
| `.easeInOut` | Standard transitions |
| `.linear` | Progress indicators only |

**Prefer `.spring`** — it feels natural and handles interruption gracefully.

## Transitions

```swift
if showDetail {
    DetailView()
        .transition(.slide)
}
```

Common transitions: `.opacity`, `.slide`, `.scale`, `.move(edge:)`, `.push(from:)`

### Combined Transitions

```swift
.transition(.asymmetric(
    insertion: .scale.combined(with: .opacity),
    removal: .opacity
))
```

## matchedGeometryEffect

Smooth transitions between views with shared identity:

```swift
@Namespace private var animation

if isExpanded {
    ExpandedCard()
        .matchedGeometryEffect(id: "card", in: animation)
} else {
    CompactCard()
        .matchedGeometryEffect(id: "card", in: animation)
}
```

## Phase Animator (iOS 17+)

```swift
PhaseAnimator([false, true]) { phase in
    Image(systemName: "heart.fill")
        .scaleEffect(phase ? 1.2 : 1.0)
        .foregroundStyle(phase ? .red : .pink)
}
```

## Keyframe Animator (iOS 17+)

```swift
KeyframeAnimator(initialValue: AnimationValues()) { values in
    content
        .scaleEffect(values.scale)
        .rotationEffect(values.rotation)
} keyframes: { _ in
    KeyframeTrack(\.scale) {
        SpringKeyframe(1.2, duration: 0.2)
        SpringKeyframe(1.0, duration: 0.2)
    }
    KeyframeTrack(\.rotation) {
        LinearKeyframe(.degrees(10), duration: 0.1)
        LinearKeyframe(.degrees(-10), duration: 0.1)
        LinearKeyframe(.zero, duration: 0.1)
    }
}
```

## Reduce Motion Support

```swift
@Environment(\.accessibilityReduceMotion) private var reduceMotion

content
    .animation(reduceMotion ? nil : .spring(), value: isExpanded)
    .transition(reduceMotion ? .opacity : .slide)
```

**Mandatory** — always respect the user's Reduce Motion preference.

## Liquid Glass (iOS 26+)

### Basic Usage

```swift
if #available(iOS 26, *) {
    content.glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
} else {
    content.background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16))
}
```

### GlassEffectContainer (Required for Grouped Glass)

Glass cannot sample other glass — wrap in container:

```swift
GlassEffectContainer(spacing: 24) {
    HStack(spacing: 24) {
        Button("One") { }.glassEffect()
        Button("Two") { }.glassEffect()
    }
}
```

### Morphing Transitions

```swift
@Namespace private var glassAnimation

GlassEffectContainer {
    if isExpanded {
        ExpandedCard()
            .glassEffect()
            .glassEffectID("card", in: glassAnimation)
    } else {
        CompactCard()
            .glassEffect()
            .glassEffectID("card", in: glassAnimation)
    }
}
```

### Glass Button Styles

```swift
Button("Action") { }
    .buttonStyle(.glass)

Button("Primary") { }
    .buttonStyle(.glassProminent)
```

### Glass Rules

- Apply `.glassEffect()` LAST — after all layout and visual modifiers
- Always provide material fallback for pre-iOS 26
- `.interactive()` only on tappable elements
- Toolbar icons use monochrome rendering by default with glass
- Sheets auto-use glass background in iOS 26
