---
name: accessibility
description: VoiceOver, Dynamic Type, contrast, assistive tech, WCAG AA, Assistive Access, testing, enforcement
---

<essential_principles>
## How This Skill Works

1. **Accessibility is mandatory, not optional.** Every interactive view MUST support VoiceOver, Dynamic Type, and high contrast. Fail verification if missing.
2. **User-centric, not checklist-driven.** Checklists help, but the goal is a great experience for real users with real needs.
3. **Shift left.** Implement accessibility as you build, not after. It's part of the definition of done.
4. **Test with assistive tech.** Automated checks catch ~30% of issues. Manual VoiceOver/Voice Control testing catches the rest.
5. **Non-deterministic.** There's no universal "correct" solution — propose options with trade-offs, respecting the user's context.
</essential_principles>

<intake>
## What do you need?

1. Add accessibility to a view (labels, traits, grouping)
2. Audit a screen for accessibility issues
3. Fix a VoiceOver problem
4. Support Dynamic Type and large text
5. Implement Assistive Access (iOS 17+)
6. Test accessibility (manual or automated)
7. Review enforcement checklist
</intake>

<routing>
| Response | Reference |
|----------|-----------|
| 1, "label", "trait", "VoiceOver", "group", "action" | `references/voiceover.md` |
| 2, "audit", "check", "review", "issues" | `references/enforcement.md` + `references/testing.md` |
| 3, "VoiceOver", "reads wrong", "focus", "announcement" | `references/voiceover.md` |
| 4, "Dynamic Type", "large text", "scale", "@ScaledMetric" | `references/dynamic-type.md` |
| 5, "Assistive Access", "cognitive", "simplified" | `references/anti-patterns.md` (Assistive Access section) |
| 6, "test", "manual test", "automated", "Inspector" | `references/testing.md` |
| 7, "checklist", "enforcement", "WCAG", "contrast" | `references/enforcement.md` |
</routing>

<reference_index>
## References

| File | When to Read |
|------|-------------|
| references/voiceover.md | Labels, traits, hints, grouping, custom actions, focus management, announcements |
| references/dynamic-type.md | Text styles, @ScaledMetric, layout adaptation, Large Content Viewer |
| references/testing.md | Manual VoiceOver testing, automated audit, Accessibility Inspector, CI patterns |
| references/anti-patterns.md | Common mistakes, quick fixes, Assistive Access (iOS 17+) |
| references/enforcement.md | GSD verification checklist, WCAG AA ratios, mandatory requirements |
</reference_index>

<canonical_terminology>
## Terminology

- **VoiceOver** (not: screen reader generically)
- **Dynamic Type** (not: font scaling, text zoom)
- **@ScaledMetric** (not: manual font size calculation)
- **accessibility label** (not: alt text — that's web)
- **WCAG AA** (not: WCAG AAA unless explicitly required)
- **Assistive Access** (not: simplified mode, easy mode)
- **trait** (not: role — traits describe behavior, not semantic role)
</canonical_terminology>
