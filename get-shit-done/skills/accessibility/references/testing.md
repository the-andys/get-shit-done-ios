<overview>
Accessibility testing: manual VoiceOver flow, Accessibility Inspector, automated audits, CI integration, and testing checklist. Automated checks catch ~30% of issues — manual testing with assistive tech catches the rest. Related: voiceover.md (what to verify), enforcement.md (requirements).
</overview>

## Manual VoiceOver Testing

### Setup

- **Accessibility Shortcut:** Settings > Accessibility > Accessibility Shortcut (triple-click side button)
- **Siri:** "Turn on VoiceOver"
- **Control Center:** Add Accessibility Shortcuts

### Core Gestures

| Gesture | Action |
|---------|--------|
| Tap | Select element |
| Double-tap | Activate |
| Swipe right | Next element |
| Swipe left | Previous element |
| Two-finger rotate | Change rotor setting |
| Swipe up/down | Navigate by rotor |
| Two-finger double-tap | Magic Tap (play/pause) |
| Two-finger scrub (Z shape) | Escape / go back |
| Three-finger triple-tap | Screen Curtain (blank screen) |

### What to Verify

- [ ] Every element is reachable by swiping
- [ ] Labels are clear and descriptive
- [ ] Traits are correct (button, header, link)
- [ ] Reading order is logical
- [ ] Focus moves after state changes (navigation, errors)
- [ ] Errors are announced
- [ ] Custom actions are discoverable

### Screen Curtain Test

Triple-tap with three fingers to blank the screen. Try completing a task using only VoiceOver audio. This is how blind users experience your app.

## Accessibility Inspector

Xcode > Open Developer Tool > Accessibility Inspector

### Tabs

| Tab | Purpose |
|-----|---------|
| Inspection | View properties of any element (label, traits, value) |
| Audit | Run automated checks (missing labels, contrast, hit targets) |
| Settings | Change Dynamic Type, Reduce Motion, etc. without rebuilding |

### Automated Audit

```swift
// In XCUITest
try app.performAccessibilityAudit()

// Specific checks
try app.performAccessibilityAudit(for: [.dynamicType, .contrast, .hitRegion])
```

### Common Inspector Warnings

| Warning | Fix |
|---------|-----|
| "Element has no label" | Add `.accessibilityLabel()` |
| "Text doesn't support Dynamic Type" | Use `.font(.body)` not fixed sizes |
| "Contrast ratio below 4.5:1" | Use semantic colors (`.primary`, `.label`) |
| "Touch target < 44x44" | Add padding or `.contentShape(Rectangle())` |

## Voice Control Testing

Settings > Accessibility > Voice Control

1. Say "Show names" to overlay element labels
2. Try speaking button labels to activate them
3. Say "Show numbers" for elements without names

## Full Keyboard Access

Settings > Accessibility > Keyboards > Full Keyboard Access

- Tab/Shift+Tab to navigate
- Space to activate
- Escape to dismiss

## CI Integration

```bash
xcodebuild test \
    -scheme MyApp \
    -destination 'platform=iOS Simulator,name=iPhone 16' \
    -testPlan AccessibilityTests
```

### Test with Large Dynamic Type

```swift
let app = XCUIApplication()
app.launchArguments += ["-UIPreferredContentSizeCategoryName", "UICTContentSizeCategoryAccessibilityExtraExtraLarge"]
app.launch()
```

## Testing Checklist

- [ ] VoiceOver: all elements reachable, labels clear, traits correct
- [ ] Dynamic Type: text scales, layout adapts at accessibility sizes
- [ ] Reduce Motion: animations disabled/simplified
- [ ] Increase Contrast: UI adapts to high contrast
- [ ] Voice Control: all buttons activatable by voice
- [ ] Tap targets: ≥44×44pt, ≥32pt apart
- [ ] Contrast: ≥4.5:1 text, ≥3:1 UI elements
