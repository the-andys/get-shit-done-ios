<overview>
GSD mandatory accessibility requirements: WCAG AA ratios, minimum tap targets, verification checklist, and environment settings. These are enforcement rules — every screen MUST pass. Related: voiceover.md (how to implement), testing.md (how to verify), anti-patterns.md (common violations).
</overview>

## WCAG AA Contrast Ratios (Mandatory)

| Element | Minimum Ratio |
|---------|--------------|
| Normal text | 4.5:1 |
| Large text (≥18pt or ≥14pt bold) | 3:1 |
| UI components and graphical objects | 3:1 |

```swift
// Use semantic colors — system handles contrast automatically
Text("Primary").foregroundStyle(.primary)
Text("Secondary").foregroundStyle(.secondary)

// Custom colors MUST have light AND dark variants in asset catalog
// Test with Accessibility Inspector > Color Contrast Calculator
```

## Minimum Tap Targets (Mandatory)

- All interactive elements: **≥44×44pt**
- Spacing between targets: **≥32pt**

```swift
// Expand hit area without changing appearance
Button(action: dismiss) {
    Image(systemName: "xmark")
        .padding(12)
}
.contentShape(Rectangle())
```

## Verification Checklist

Run for EVERY screen before declaring task complete.

### VoiceOver

- [ ] All buttons/controls have `.accessibilityLabel`
- [ ] Decorative images use `.accessibilityHidden(true)`
- [ ] Related elements grouped with `.accessibilityElement(children:)`
- [ ] Section headings marked with `.accessibilityAddTraits(.isHeader)`
- [ ] Reading order is logical (top-to-bottom, left-to-right)
- [ ] Custom controls use `.accessibilityRepresentation` or adjustable actions

### Dynamic Type

- [ ] All text uses semantic font styles (`.body`, `.title`) or `relativeTo:`
- [ ] No fixed font sizes for user-facing text
- [ ] `@ScaledMetric` used for non-text dimensions (icons, spacing)
- [ ] Layout adapts at accessibility sizes (horizontal → vertical)
- [ ] Line limits increase at larger sizes

### Color & Contrast

- [ ] Text contrast ≥4.5:1 (normal) or ≥3:1 (large)
- [ ] UI element contrast ≥3:1
- [ ] All colors have light AND dark mode variants
- [ ] Respects Increase Contrast setting

### Motion

- [ ] Animations respect `accessibilityReduceMotion`
- [ ] No auto-playing animations without Reduce Motion check

### Touch

- [ ] All tap targets ≥44×44pt
- [ ] Targets ≥32pt apart

### System Settings

Check these environment values when relevant:

```swift
@Environment(\.accessibilityReduceMotion) private var reduceMotion
@Environment(\.accessibilityReduceTransparency) private var reduceTransparency
@Environment(\.colorSchemeContrast) private var contrast
@Environment(\.legibilityWeight) private var legibilityWeight
@Environment(\.accessibilityShowButtonShapes) private var showButtonShapes
```

| Setting | Action |
|---------|--------|
| Reduce Motion | Disable/simplify animations |
| Reduce Transparency | Make backgrounds opaque |
| Increase Contrast | Use higher contrast colors |
| Bold Text | Respect `legibilityWeight` |
| Button Shapes | Show visual button borders |

## Programmatic Contrast Check

```swift
extension UIColor {
    func contrastRatio(with other: UIColor) -> CGFloat {
        let l1 = relativeLuminance
        let l2 = other.relativeLuminance
        return (max(l1, l2) + 0.05) / (min(l1, l2) + 0.05)
    }
}
```

## Smart Invert

Prevent images from inverting in Smart Invert mode:

```swift
Image(imageName)
    .accessibilityIgnoresInvertColors()
```

## Haptic Feedback

Provide haptic feedback for important actions:

```swift
UINotificationFeedbackGenerator().notificationOccurred(.success)
UIImpactFeedbackGenerator(style: .medium).impactOccurred()

// iOS 17+
Button("Tap") { }
    .sensoryFeedback(.impact(weight: .medium), trigger: tapCount)
```
