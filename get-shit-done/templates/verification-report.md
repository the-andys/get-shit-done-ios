# Verification Report Template

Template for `.planning/phases/XX-name/{phase}-VERIFICATION.md` — phase goal verification results.

---

## File Template

```markdown
---
phase: XX-name
verified: YYYY-MM-DDTHH:MM:SSZ
status: passed | gaps_found | human_needed
score: N/M must-haves verified
---

# Phase {X}: {Name} Verification Report

**Phase Goal:** {goal from ROADMAP.md}
**Verified:** {timestamp}
**Status:** {passed | gaps_found | human_needed}

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | {truth from must_haves} | ✓ VERIFIED | {what confirmed it} |
| 2 | {truth from must_haves} | ✗ FAILED | {what's wrong} |
| 3 | {truth from must_haves} | ? UNCERTAIN | {why can't verify} |

**Score:** {N}/{M} truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `Sources/Views/Chat/ChatListView.swift` | Message list view | ✓ EXISTS + SUBSTANTIVE | Conforms to View, renders ForEach over messages, no stubs |
| `Sources/ViewModels/ChatViewModel.swift` | Message data provider | ✗ STUB | File exists but loadMessages() returns hardcoded [] |
| `Sources/Models/ChatMessage.swift` | Message model | ✓ EXISTS + SUBSTANTIVE | @Model with all fields (id, content, sender, createdAt) |

**Artifacts:** {N}/{M} verified

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|----|--------|---------|
| ChatListView | ChatViewModel | @Environment in body | ✓ WIRED | Line 12: `@Environment(ChatViewModel.self)` with ForEach over viewModel.messages |
| ChatInputView | ChatViewModel | sendMessage() call | ✗ NOT WIRED | Send button action only calls print() |
| ChatViewModel | ChatMessage | modelContext.insert | ✗ NOT WIRED | Returns hardcoded response, no SwiftData insert |

**Wiring:** {N}/{M} connections verified

## Requirements Coverage

| Requirement | Status | Blocking Issue |
|-------------|--------|----------------|
| {REQ-01}: {description} | ✓ SATISFIED | - |
| {REQ-02}: {description} | ✗ BLOCKED | API route is stub |
| {REQ-03}: {description} | ? NEEDS HUMAN | Can't verify WebSocket programmatically |

**Coverage:** {N}/{M} requirements satisfied

## Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| Sources/ViewModels/ChatViewModel.swift | 12 | `// TODO: implement` | ⚠️ Warning | Indicates incomplete |
| Sources/Views/Chat/ChatListView.swift | 15 | `Text("Chat will be here")` | 🛑 Blocker | Renders placeholder, not message data |
| Sources/Services/ChatService.swift | - | File missing | 🛑 Blocker | Expected service doesn't exist |

**Anti-patterns:** {N} found ({blockers} blockers, {warnings} warnings)

## Human Verification Required

{If no human verification needed:}
None — all verifiable items checked programmatically.

{If human verification needed:}

### 1. {Test Name}
**Test:** {What to do}
**Expected:** {What should happen}
**Why human:** {Why can't verify programmatically}

### 2. {Test Name}
**Test:** {What to do}
**Expected:** {What should happen}
**Why human:** {Why can't verify programmatically}

## Gaps Summary

{If no gaps:}
**No gaps found.** Phase goal achieved. Ready to proceed.

{If gaps found:}

### Critical Gaps (Block Progress)

1. **{Gap name}**
   - Missing: {what's missing}
   - Impact: {why this blocks the goal}
   - Fix: {what needs to happen}

2. **{Gap name}**
   - Missing: {what's missing}
   - Impact: {why this blocks the goal}
   - Fix: {what needs to happen}

### Non-Critical Gaps (Can Defer)

1. **{Gap name}**
   - Issue: {what's wrong}
   - Impact: {limited impact because...}
   - Recommendation: {fix now or defer}

## Recommended Fix Plans

{If gaps found, generate fix plan recommendations:}

### {phase}-{next}-PLAN.md: {Fix Name}

**Objective:** {What this fixes}

**Tasks:**
1. {Task to fix gap 1}
2. {Task to fix gap 2}
3. {Verification task}

**Estimated scope:** {Small / Medium}

---

### {phase}-{next+1}-PLAN.md: {Fix Name}

**Objective:** {What this fixes}

**Tasks:**
1. {Task}
2. {Task}

**Estimated scope:** {Small / Medium}

---

## Verification Metadata

**Verification approach:** Goal-backward (derived from phase goal)
**Must-haves source:** {PLAN.md frontmatter | derived from ROADMAP.md goal}
**Automated checks:** {N} passed, {M} failed
**Human checks required:** {N}
**Total verification time:** {duration}

---
*Verified: {timestamp}*
*Verifier: Claude (subagent)*
```

---

## Guidelines

**Status values:**
- `passed` — All must-haves verified, no blockers
- `gaps_found` — One or more critical gaps found
- `human_needed` — Automated checks pass but human verification required

**Evidence types:**
- For EXISTS: "File at path, exports X"
- For SUBSTANTIVE: "N lines, has patterns X, Y, Z"
- For WIRED: "Line N: code that connects A to B"
- For FAILED: "Missing because X" or "Stub because Y"

**Severity levels:**
- 🛑 Blocker: Prevents goal achievement, must fix
- ⚠️ Warning: Indicates incomplete but doesn't block
- ℹ️ Info: Notable but not problematic

**Fix plan generation:**
- Only generate if gaps_found
- Group related fixes into single plans
- Keep to 2-3 tasks per plan
- Include verification task in each plan

---

## Example

```markdown
---
phase: 03-chat
verified: 2025-01-15T14:30:00Z
status: gaps_found
score: 2/5 must-haves verified
---

# Phase 3: Chat Interface Verification Report

**Phase Goal:** Working chat interface where users can send and receive messages
**Verified:** 2025-01-15T14:30:00Z
**Status:** gaps_found

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | User can see existing messages | ✗ FAILED | View body returns Text("Chat will be here"), not message data |
| 2 | User can type a message | ✓ VERIFIED | TextField exists with @State binding for input text |
| 3 | User can send a message | ✗ FAILED | Send button action only calls print(), no ViewModel method |
| 4 | Sent message appears in list | ✗ FAILED | No state update after send |
| 5 | Messages persist across app launch | ? UNCERTAIN | Can't verify — send doesn't work |

**Score:** 1/5 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `Sources/Views/Chat/ChatListView.swift` | Message list view | ✗ STUB | body returns `Text("Chat will be here")` |
| `Sources/Views/Chat/ChatInputView.swift` | Message input | ✓ EXISTS + SUBSTANTIVE | TextField with @State binding, Send button with action |
| `Sources/ViewModels/ChatViewModel.swift` | Message data provider | ✗ STUB | loadMessages() is empty async func, messages always [] |
| `Sources/Models/ChatMessage.swift` | Message model | ✓ EXISTS + SUBSTANTIVE | @Model with id, content, sender, createdAt fields |

**Artifacts:** 2/4 verified

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|----|--------|---------|
| ChatListView | ChatViewModel | @Environment | ✗ NOT WIRED | ViewModel declared but not read in body |
| ChatInputView | ChatViewModel | sendMessage() | ✗ NOT WIRED | Button action only calls print("send tapped") |
| ChatViewModel | ChatMessage | modelContext.fetch | ✗ NOT WIRED | loadMessages() body is empty |
| ChatViewModel | ChatMessage | modelContext.insert | ✗ NOT WIRED | sendMessage() body is empty |

**Wiring:** 0/4 connections verified

## Requirements Coverage

| Requirement | Status | Blocking Issue |
|-------------|--------|----------------|
| CHAT-01: User can send message | ✗ BLOCKED | ViewModel sendMessage() is stub |
| CHAT-02: User can view messages | ✗ BLOCKED | View body is placeholder |
| CHAT-03: Messages persist | ✗ BLOCKED | No SwiftData integration |

**Coverage:** 0/3 requirements satisfied

## Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| Sources/Views/Chat/ChatListView.swift | 8 | `Text("Chat will be here")` | 🛑 Blocker | Placeholder, no real content |
| Sources/ViewModels/ChatViewModel.swift | 15 | `func loadMessages() async { }` | 🛑 Blocker | Empty async function |
| Sources/ViewModels/ChatViewModel.swift | 22 | `// TODO: save to SwiftData` | ⚠️ Warning | Incomplete |
| Sources/Views/Chat/ChatInputView.swift | 18 | `print("send tapped")` | ⚠️ Warning | Print-only action |

**Anti-patterns:** 4 found (2 blockers, 2 warnings)

## Human Verification Required

None needed until automated gaps are fixed.

## Gaps Summary

### Critical Gaps (Block Progress)

1. **Chat list view is placeholder**
   - Missing: Actual message list rendering with ForEach over ViewModel data
   - Impact: Users see "Chat will be here" instead of messages
   - Fix: Implement ChatListView to read ChatViewModel.messages and render via ForEach

2. **ViewModel methods are stubs**
   - Missing: SwiftData integration in loadMessages() and sendMessage()
   - Impact: No data loading or persistence
   - Fix: Wire modelContext.fetch and modelContext.insert in ViewModel methods

3. **No wiring between View and ViewModel**
   - Missing: @Environment usage in View body, method calls in .task and button actions
   - Impact: Even if ViewModel worked, View wouldn't call it
   - Fix: Add .task { await viewModel.loadMessages() } and wire send button to viewModel.sendMessage()

## Recommended Fix Plans

### 03-04-PLAN.md: Implement Chat ViewModel

**Objective:** Wire ViewModel to SwiftData

**Tasks:**
1. Implement loadMessages() with modelContext.fetch(FetchDescriptor<ChatMessage>())
2. Implement sendMessage() with modelContext.insert(ChatMessage(...))
3. Verify: ViewModel populates messages array, new messages persist

**Estimated scope:** Small

---

### 03-05-PLAN.md: Implement Chat Views

**Objective:** Wire Chat views to ViewModel

**Tasks:**
1. Implement ChatListView with @Environment(ChatViewModel.self) and ForEach rendering
2. Wire ChatInputView send button to viewModel.sendMessage()
3. Verify: Messages display on load, new messages appear after send

**Estimated scope:** Small

---

## Verification Metadata

**Verification approach:** Goal-backward (derived from phase goal)
**Must-haves source:** 03-01-PLAN.md frontmatter
**Automated checks:** 2 passed, 8 failed
**Human checks required:** 0 (blocked by automated failures)
**Total verification time:** 2 min

---
*Verified: 2025-01-15T14:30:00Z*
*Verifier: Claude (subagent)*
```
