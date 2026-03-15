<overview>
10 interface text pattern categories with templates, before/after examples, and checklists. Read when writing specific UI copy. Related: voice-tone.md (how to apply voice to these patterns).
</overview>

## 1. Alerts & Dialogs

**Principle:** Justify the interruption. Answer: What happened? Why? What now?

```
Title: [What happened — short noun phrase]
Body: [Why it matters — one sentence max]
Actions: [Specific verb] / [Cancel]
```

**Before:** "Are you sure?" / OK / Cancel
**After:** "Remove 'Groceries' list?" / "This will delete all 12 items." / Remove / Cancel

- Use specific verbs for actions, not "OK" or "Yes"
- Destructive actions use `.destructive` role
- Title is a statement or question, not an exclamation

## 2. Error Messages

**Principle:** Say what happened, explain why, tell the next step. No jargon, no blame.

```
Title: [What went wrong — plain language]
Body: [Why + what to do next]
Action: [Specific recovery action]
```

**Before:** "Error 403: Forbidden"
**After:** "Can't load your photos" / "Check your internet connection and try again." / Try Again

- Never show error codes to users
- Never blame the user ("You entered an invalid...")
- Never use exclamation marks in errors
- Always provide a next step

## 3. Destructive Actions

**Principle:** Name the specific thing. Make consequences explicit.

**Before:** "Delete?" / Yes / No
**After:** "Delete 'Beach Vacation' album?" / "24 photos will be permanently removed." / Delete Album / Keep

- Label buttons with the actual action, not "Yes"/"No"
- State what will be lost
- Use `.destructive` styling on the action button

## 4. Empty States

**Principle:** Tell what will appear here + how to make it happen.

```
[Icon or illustration]
[What will appear here]
[How to make it happen — one action]
```

**Before:** "No data"
**After:** "No trips yet" / "Plan your first trip to see it here." / [+ New Trip]

- Match tone to context (first-time use = welcoming, search = helpful)
- Include an action when possible
- Don't apologize ("Sorry, nothing here")

## 5. Onboarding

**Principle:** One idea per screen. Lead with why, not how.

- Define the purpose of each screen before writing
- Lead with the benefit: "See your week at a glance" not "This screen shows your calendar"
- Be honest about permissions: "We use your location to show nearby restaurants"
- Keep it scannable — title + one sentence + action

## 6. Notifications

**Principle:** Lead with benefit. Respect attention.

**Before:** "Hey! Don't forget to check your tasks today! 📋"
**After:** "3 tasks due today"

- One idea per notification
- No exclamation marks, emojis, or chattiness
- Be specific: "Flight AA123 delayed 45 min" not "Travel update"

## 7. Accessibility Labels

**Principle:** Describe intent, not appearance.

**Before:** `.accessibilityLabel("Red circle icon")`
**After:** `.accessibilityLabel("Error indicator")`

- Be succinct — VoiceOver reads every word
- Don't include trait names ("button", "image") — VoiceOver adds them
- Update labels when state changes
- Match label richness to content importance

## 8. Buttons

**Principle:** Use specific verbs. Make both options clear.

| Bad | Good |
|-----|------|
| OK | Save |
| Submit | Send Message |
| Yes / No | Delete / Keep |
| Continue | Start Free Trial |

- Verb + noun when action isn't obvious from context
- Match button label to surrounding copy
- Paired choices: both should be clear independently

## 9. Instructional Copy

**Principle:** Lead with benefit, not mechanics.

**Before:** "Tap the + button to add a new item"
**After:** "Add items to track your progress"

- Place where the user is looking (near the action)
- One instruction at a time
- Be direct — no "You can..." or "Simply..."

## 10. Settings & Permissions

**Principle:** Clear labels, honest descriptions.

- Setting labels: practical, not technical ("Notifications" not "Push Notification Configuration")
- Descriptions: one line explaining what changes
- Permissions: explain WHY before asking, not just WHAT
- Link to relevant settings when appropriate

```swift
// Permission request
"Allow Notifications"
"Get reminders about upcoming tasks and due dates."
[Allow] / [Not Now]
```

## PACE Checklist

For every piece of interface text:

- [ ] **Purpose** — What is the single most important thing right now?
- [ ] **Anticipation** — What does the user need next?
- [ ] **Context** — What is the user's physical/emotional state?
- [ ] **Empathy** — Is this respectful, inclusive, and accessible?
