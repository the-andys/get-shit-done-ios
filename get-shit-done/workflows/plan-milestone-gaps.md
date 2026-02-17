<purpose>
Create all phases necessary to close gaps identified by `/gsd:audit-milestone`. Reads MILESTONE-AUDIT.md, groups gaps into logical phases, creates phase entries in ROADMAP.md, and offers to plan each phase. One command creates all fix phases — no manual `/gsd:add-phase` per gap.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

## 1. Load Audit Results

```bash
# Find the most recent audit file
ls -t .planning/v*-MILESTONE-AUDIT.md 2>/dev/null | head -1
```

Parse YAML frontmatter to extract structured gaps:
- `gaps.requirements` — unsatisfied requirements
- `gaps.integration` — missing cross-phase connections
- `gaps.flows` — broken E2E flows

If no audit file exists or has no gaps, error:
```
No audit gaps found. Run `/gsd:audit-milestone` first.
```

## 2. Prioritize Gaps

Group gaps by priority from REQUIREMENTS.md:

| Priority | Action |
|----------|--------|
| `must` | Create phase, blocks milestone |
| `should` | Create phase, recommended |
| `nice` | Ask user: include or defer? |

For integration/flow gaps, infer priority from affected requirements.

## 3. Group Gaps into Phases

Cluster related gaps into logical phases:

**Grouping rules:**
- Same affected phase → combine into one fix phase
- Same subsystem (auth, API, UI) → combine
- Dependency order (fix stubs before wiring)
- Keep phases focused: 2-4 tasks each

**Example grouping:**
```
Gap: DASH-01 unsatisfied (Dashboard doesn't load data)
Gap: Integration Phase 1→3 (Auth token not passed to ViewModel)
Gap: Flow "View dashboard" broken at data load

→ Phase 6: "Wire Dashboard ViewModel to Service"
  - Implement DashboardViewModel.loadData() with service call
  - Pass auth token via Environment to service layer
  - Store response in @Observable state, handle errors
  - Render data in DashboardView via ForEach
```

## 4. Determine Phase Numbers

Find highest existing phase:
```bash
# Get sorted phase list, extract last one
PHASES=$(node ~/.claude/get-shit-done/bin/gsd-tools.cjs phases list)
HIGHEST=$(echo "$PHASES" | jq -r '.directories[-1]')
```

New phases continue from there:
- If Phase 5 is highest, gaps become Phase 6, 7, 8...

## 5. Present Gap Closure Plan

```markdown
## Gap Closure Plan

**Milestone:** {version}
**Gaps to close:** {N} requirements, {M} integration, {K} flows

### Proposed Phases

**Phase {N}: {Name}**
Closes:
- {REQ-ID}: {description}
- Integration: {from} → {to}
Tasks: {count}

**Phase {N+1}: {Name}**
Closes:
- {REQ-ID}: {description}
- Flow: {flow name}
Tasks: {count}

{If nice-to-have gaps exist:}

### Deferred (nice-to-have)

These gaps are optional. Include them?
- {gap description}
- {gap description}

---

Create these {X} phases? (yes / adjust / defer all optional)
```

Wait for user confirmation.

## 6. Update ROADMAP.md

Add new phases to current milestone:

```markdown
### Phase {N}: {Name}
**Goal:** {derived from gaps being closed}
**Requirements:** {REQ-IDs being satisfied}
**Gap Closure:** Closes gaps from audit

### Phase {N+1}: {Name}
...
```

## 7. Create Phase Directories

```bash
mkdir -p ".planning/phases/{NN}-{name}"
```

## 8. Commit Roadmap Update

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.cjs commit "docs(roadmap): add gap closure phases {N}-{M}" --files .planning/ROADMAP.md
```

## 9. Offer Next Steps

```markdown
## ✓ Gap Closure Phases Created

**Phases added:** {N} - {M}
**Gaps addressed:** {count} requirements, {count} integration, {count} flows

---

## ▶ Next Up

**Plan first gap closure phase**

`/gsd:plan-phase {N}`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `/gsd:execute-phase {N}` — if plans already exist
- `cat .planning/ROADMAP.md` — see updated roadmap

---

**After all gap phases complete:**

`/gsd:audit-milestone` — re-audit to verify gaps closed
`/gsd:complete-milestone {version}` — archive when audit passes
```

</process>

<gap_to_phase_mapping>

## How Gaps Become Tasks

**Requirement gap → Tasks:**
```yaml
gap:
  id: DASH-01
  description: "User sees their data"
  reason: "Dashboard exists but ViewModel doesn't fetch from service"
  missing:
    - ".task modifier calling viewModel.loadData()"
    - "ViewModel state for userData, isLoading, errorMessage"
    - "ForEach rendering of userData in View body"

becomes:

phase: "Wire Dashboard Data"
tasks:
  - name: "Add data fetching"
    files: [Sources/ViewModels/DashboardViewModel.swift]
    action: "Implement loadData() with try await service.fetchUserData()"

  - name: "Add state management"
    files: [Sources/ViewModels/DashboardViewModel.swift]
    action: "Add userData, isLoading, errorMessage properties to @Observable class"

  - name: "Render user data"
    files: [Sources/Views/Dashboard/DashboardView.swift]
    action: "Replace placeholder Text with ForEach over viewModel.userData, add .task { await viewModel.loadData() }"
```

**Integration gap → Tasks:**
```yaml
gap:
  from_phase: 1
  to_phase: 3
  connection: "Auth token → Service layer"
  reason: "Dashboard service calls don't include auth token"
  missing:
    - "Auth token passed to service via Environment or init"
    - "Token refresh on 401 response"

becomes:

phase: "Add Auth to Dashboard Service"
tasks:
  - name: "Pass auth token to service"
    files: [Sources/ViewModels/DashboardViewModel.swift, Sources/Services/APIClient.swift]
    action: "Inject AuthService via Environment, include bearer token in URLRequest headers"

  - name: "Handle 401 responses"
    files: [Sources/Services/APIClient.swift]
    action: "Check HTTPURLResponse statusCode, trigger token refresh or navigate to login on 401"
```

**Flow gap → Tasks:**
```yaml
gap:
  name: "User views dashboard after login"
  broken_at: "Dashboard data load"
  reason: "No fetch call"
  missing:
    - "Fetch user data on mount"
    - "Display loading state"
    - "Render user data"

becomes:

# Usually same phase as requirement/integration gap
# Flow gaps often overlap with other gap types
```

</gap_to_phase_mapping>

<success_criteria>
- [ ] MILESTONE-AUDIT.md loaded and gaps parsed
- [ ] Gaps prioritized (must/should/nice)
- [ ] Gaps grouped into logical phases
- [ ] User confirmed phase plan
- [ ] ROADMAP.md updated with new phases
- [ ] Phase directories created
- [ ] Changes committed
- [ ] User knows to run `/gsd:plan-phase` next
</success_criteria>
