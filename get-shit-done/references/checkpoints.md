<overview>
Plans execute autonomously. Checkpoints formalize interaction points where human verification or decisions are needed.

**Core principle:** Claude automates everything with CLI/API. Checkpoints are for verification and decisions, not manual work.

**Golden rules:**
1. **If Claude can run it, Claude runs it** - Never ask user to execute CLI commands, start servers, or run builds
2. **Claude sets up the verification environment** - Build targets, launch Simulator, configure schemes and xcconfig
3. **User only does what requires human judgment** - Visual checks, UX evaluation, "does this feel right?"
4. **Secrets come from user, automation comes from Claude** - Ask for API keys, then Claude uses them via CLI
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify (Most Common - 90%)

**When:** Claude completed automated work, human confirms it works correctly.

**Use for:**
- Visual UI checks (layout, styling, responsiveness)
- Interactive flows (click through wizard, test user flows)
- Functional verification (feature works as expected)
- Audio/video playback quality
- Animation smoothness
- Accessibility testing

**Structure:**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[What Claude automated and deployed/built]</what-built>
  <how-to-verify>
    [Exact steps to test - URLs, commands, expected behavior]
  </how-to-verify>
  <resume-signal>[How to continue - "approved", "yes", or describe issues]</resume-signal>
</task>
```

**Example: UI Component (shows key pattern: Claude builds and launches BEFORE checkpoint)**
```xml
<task type="auto">
  <n>Build adaptive dashboard layout</n>
  <files>Sources/Views/DashboardView.swift, Sources/Views/SidebarView.swift</files>
  <action>Create dashboard with NavigationSplitView for sidebar, header, and content area. Use ViewThatFits and horizontalSizeClass for adaptive layout.</action>
  <verify>xcodebuild build succeeds, no compiler errors</verify>
  <done>Dashboard view builds without errors</done>
</task>

<task type="auto">
  <n>Launch Simulator for verification</n>
  <action>Run `xcrun simctl boot "iPhone 16 Pro"` and build-and-run with `xcodebuild -destination 'platform=iOS Simulator'`</action>
  <verify>App launches in Simulator without crash</verify>
  <done>App running in iPhone 16 Pro Simulator</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Adaptive dashboard layout - app running in Simulator</what-built>
  <how-to-verify>
    Check the app in Simulator and verify:
    1. iPad (regular width): Sidebar visible, content fills detail area
    2. iPhone landscape: Sidebar collapses, swipe to reveal
    3. iPhone portrait: Tab-based navigation, no sidebar
    4. Rotate device: Layout adapts smoothly, no clipping
  </how-to-verify>
  <resume-signal>Type "approved" or describe layout issues</resume-signal>
</task>
```

**Example: Xcode Build**
```xml
<task type="auto">
  <name>Build macOS app with Xcode</name>
  <files>App.xcodeproj, Sources/</files>
  <action>Run `xcodebuild -project App.xcodeproj -scheme App build`. Check for compilation errors in output.</action>
  <verify>Build output contains "BUILD SUCCEEDED", no errors</verify>
  <done>App builds successfully</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Built macOS app at DerivedData/Build/Products/Debug/App.app</what-built>
  <how-to-verify>
    Open App.app and test:
    - App launches without crashes
    - Menu bar icon appears
    - Preferences window opens correctly
    - No visual glitches or layout issues
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```
</type>

<type name="decision">
## checkpoint:decision (9%)

**When:** Human must make choice that affects implementation direction.

**Use for:**
- Technology selection (which auth provider, which database)
- Architecture decisions (monorepo vs separate repos)
- Design choices (color scheme, layout approach)
- Feature prioritization (which variant to build)
- Data model decisions (schema structure)

**Structure:**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>[What's being decided]</decision>
  <context>[Why this decision matters]</context>
  <options>
    <option id="option-a">
      <name>[Option name]</name>
      <pros>[Benefits]</pros>
      <cons>[Tradeoffs]</cons>
    </option>
    <option id="option-b">
      <name>[Option name]</name>
      <pros>[Benefits]</pros>
      <cons>[Tradeoffs]</cons>
    </option>
  </options>
  <resume-signal>[How to indicate choice]</resume-signal>
</task>
```

**Example: Auth Provider Selection**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Select authentication provider</decision>
  <context>
    Need user authentication for the app. Three solid options with different tradeoffs.
  </context>
  <options>
    <option id="apple">
      <n>Sign in with Apple</n>
      <pros>Required for App Store if offering social login, native integration, privacy-focused, free</pros>
      <cons>Apple-only, limited user info returned, requires Apple Developer account</cons>
    </option>
    <option id="firebase">
      <n>Firebase Auth</n>
      <pros>Multi-provider (Apple, Google, email), good free tier, cross-platform</pros>
      <cons>Google dependency, SDK size, limited customization of flows</cons>
    </option>
    <option id="auth0">
      <n>Auth0</n>
      <pros>Maximum flexibility, enterprise features, excellent docs, custom domains</pros>
      <cons>Paid after 7.5k MAU, vendor lock-in, adds external dependency</cons>
    </option>
  </options>
  <resume-signal>Select: apple, firebase, or auth0</resume-signal>
</task>
```

**Example: Persistence Selection**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Select persistence strategy for user data</decision>
  <context>
    App needs persistent storage for users, sessions, and user-generated content.
    Expected scale: offline-first with cloud sync for cross-device access.
  </context>
  <options>
    <option id="swiftdata">
      <n>SwiftData (local + CloudKit sync)</n>
      <pros>Native Apple framework, automatic iCloud sync, Swift-native API, no server costs</pros>
      <cons>Apple-only, CloudKit sync can be opaque, limited query flexibility</cons>
    </option>
    <option id="cloudkit">
      <n>CloudKit (direct)</n>
      <pros>Full control over sync logic, public/private databases, generous free tier</pros>
      <cons>More boilerplate, CloudKit-specific API, Apple ecosystem only</cons>
    </option>
    <option id="firebase">
      <n>Firebase Firestore</n>
      <pros>Real-time sync, cross-platform, offline support, good free tier</pros>
      <cons>Google dependency, SDK size, NoSQL only, vendor lock-in</cons>
    </option>
  </options>
  <resume-signal>Select: swiftdata, cloudkit, or firebase</resume-signal>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action (1% - Rare)

**When:** Action has NO CLI/API and requires human-only interaction, OR Claude hit an authentication gate during automation.

**Use ONLY for:**
- **Authentication gates** - Claude tried CLI/API but needs credentials (this is NOT a failure)
- Email verification links (clicking email)
- SMS 2FA codes (phone verification)
- Manual account approvals (platform requires human review)
- Credit card 3D Secure flows (web-based payment authorization)
- OAuth app approvals (web-based approval)

**Do NOT use for pre-planned manual work:**
- Deploying (use CLI - auth gate if needed)
- Creating webhooks/databases (use API/CLI - auth gate if needed)
- Running builds/tests (use Bash tool)
- Creating files (use Write tool)

**Structure:**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[What human must do - Claude already did everything automatable]</action>
  <instructions>
    [What Claude already automated]
    [The ONE thing requiring human action]
  </instructions>
  <verification>[What Claude can check afterward]</verification>
  <resume-signal>[How to continue]</resume-signal>
</task>
```

**Example: Push Notification Certificate**
```xml
<task type="auto">
  <n>Configure push notification entitlement</n>
  <action>Add Push Notifications capability to the Xcode project entitlements file. Update Info.plist with required background modes.</action>
  <verify>xcodebuild build succeeds with push entitlement, no signing errors</verify>
  <done>Push entitlement configured in project</done>
</task>

<task type="checkpoint:human-action" gate="blocking">
  <action>Create APNs key in Apple Developer portal</action>
  <instructions>
    I configured the project for push notifications.
    Create an APNs authentication key:
    1. Go to developer.apple.com → Certificates, IDs & Profiles → Keys
    2. Create a new key with "Apple Push Notifications service (APNs)" enabled
    3. Download the .p8 file and note the Key ID
  </instructions>
  <verification>I'll configure the server with the .p8 key and send a test push</verification>
  <resume-signal>Provide the .p8 file path and Key ID</resume-signal>
</task>
```

**Example: Authentication Gate (Dynamic Checkpoint)**
```xml
<task type="auto">
  <n>Archive and upload to TestFlight</n>
  <files>App.xcodeproj, Sources/</files>
  <action>Run `xcodebuild archive` then `xcodebuild -exportArchive` and upload with `xcrun altool --upload-app`</action>
  <verify>altool output shows "No errors uploading"</verify>
</task>

<!-- If altool returns authentication error, Claude creates checkpoint on the fly -->

<task type="checkpoint:human-action" gate="blocking">
  <action>Provide App Store Connect credentials for upload</action>
  <instructions>
    I tried to upload but got authentication error.
    I need an app-specific password for altool:
    1. Go to appleid.apple.com → Sign-In and Security → App-Specific Passwords
    2. Generate a new password for "altool"
    Provide your Apple ID email and the app-specific password.
  </instructions>
  <verification>xcrun altool --validate-app succeeds with provided credentials</verification>
  <resume-signal>Provide Apple ID and app-specific password</resume-signal>
</task>

<!-- After authentication, Claude retries the upload -->

<task type="auto">
  <n>Retry TestFlight upload</n>
  <action>Run `xcrun altool --upload-app` with provided credentials</action>
  <verify>Upload succeeds, build appears in App Store Connect</verify>
</task>
```

**Key distinction:** Auth gates are created dynamically when Claude encounters auth errors. NOT pre-planned — Claude automates first, asks for credentials only when blocked.
</type>
</checkpoint_types>

<execution_protocol>

When Claude encounters `type="checkpoint:*"`:

1. **Stop immediately** - do not proceed to next task
2. **Display checkpoint clearly** using the format below
3. **Wait for user response** - do not hallucinate completion
4. **Verify if possible** - check files, run tests, whatever is specified
5. **Resume execution** - continue to next task only after confirmation

**For checkpoint:human-verify:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Verification Required                    ║
╚═══════════════════════════════════════════════════════╝

Progress: 5/8 tasks complete
Task: Adaptive dashboard layout

Built: Adaptive dashboard with NavigationSplitView

How to verify:
  1. Check app running in Simulator (iPhone 16 Pro)
  2. iPad (regular width): Sidebar visible, detail fills remaining space
  3. iPhone landscape: Sidebar collapses, swipe from edge to reveal
  4. iPhone portrait: Tab-based layout, no sidebar

────────────────────────────────────────────────────────
→ YOUR ACTION: Type "approved" or describe issues
────────────────────────────────────────────────────────
```

**For checkpoint:decision:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Decision Required                        ║
╚═══════════════════════════════════════════════════════╝

Progress: 2/6 tasks complete
Task: Select authentication provider

Decision: Which auth provider should we use?

Context: Need user authentication. Three options with different tradeoffs.

Options:
  1. apple - Native Sign in with Apple, privacy-focused
     Pros: Required for App Store (if offering social login), native integration
     Cons: Apple-only, limited user info returned

  2. firebase - Multi-provider, cross-platform
     Pros: Apple + Google + email login, good free tier
     Cons: Google dependency, SDK size

  3. auth0 - Maximum flexibility, enterprise features
     Pros: Custom domains, extensive provider support
     Cons: Paid after 7.5k MAU, external dependency

────────────────────────────────────────────────────────
→ YOUR ACTION: Select apple, firebase, or auth0
────────────────────────────────────────────────────────
```

**For checkpoint:human-action:**
```
╔═══════════════════════════════════════════════════════╗
║  CHECKPOINT: Action Required                          ║
╚═══════════════════════════════════════════════════════╝

Progress: 3/8 tasks complete
Task: Upload to TestFlight

Attempted: xcrun altool --upload-app
Error: Authentication failed. Need app-specific password.

What you need to do:
  1. Go to appleid.apple.com → App-Specific Passwords
  2. Generate a password for "altool"
  3. Provide your Apple ID and the generated password

I'll verify: xcrun altool --validate-app succeeds

────────────────────────────────────────────────────────
→ YOUR ACTION: Provide Apple ID and app-specific password
────────────────────────────────────────────────────────
```
</execution_protocol>

<authentication_gates>

**Auth gate = Claude tried CLI/API, got auth error.** Not a failure — a gate requiring human input to unblock.

**Pattern:** Claude tries automation → auth error → creates checkpoint:human-action → user authenticates → Claude retries → continues

**Gate protocol:**
1. Recognize it's not a failure - missing auth is expected
2. Stop current task - don't retry repeatedly
3. Create checkpoint:human-action dynamically
4. Provide exact authentication steps
5. Verify authentication works
6. Retry the original task
7. Continue normally

**Key distinction:**
- Pre-planned checkpoint: "I need you to do X" (wrong - Claude should automate)
- Auth gate: "I tried to automate X but need credentials" (correct - unblocks automation)

</authentication_gates>

<automation_reference>

**The rule:** If it has CLI/API, Claude does it. Never ask human to perform automatable work.

## Service CLI Reference

| Service | CLI/API | Key Commands | Auth Gate |
|---------|---------|--------------|-----------|
| Xcode | `xcodebuild` | `build`, `test`, `archive`, `-exportArchive` | N/A |
| Simulator | `xcrun simctl` | `boot`, `install`, `launch`, `screenshot` | N/A |
| Swift PM | `swift` | `build`, `test`, `package resolve` | N/A |
| altool | `xcrun altool` | `--upload-app`, `--validate-app` | App-specific password |
| notarytool | `xcrun notarytool` | `submit`, `log`, `history` | App-specific password |
| Fastlane | `fastlane` | `build`, `test`, `pilot upload` | `fastlane init` |
| CocoaPods | `pod` | `install`, `update`, `repo update` | N/A |
| SwiftLint | `swiftlint` | `lint`, `analyze`, `--fix` | N/A |
| GitHub | `gh` | `repo create`, `pr create`, `secret set` | `gh auth login` |
| Firebase | `firebase` | `deploy`, `emulators:start`, `auth:export` | `firebase login` |
| TestFlight | App Store Connect API | Upload via altool, manage via API | API key (.p8) |

## Build Configuration Automation

**xcconfig files:** Use Write/Edit tools. Never ask human to create .xcconfig manually.

**Configuration methods:**

| Method | Use Case | Example |
|--------|----------|---------|
| .xcconfig | Build-time settings per scheme | `API_BASE_URL = https:$()/$()/api.example.com` |
| Info.plist | Runtime config, bundle metadata | `$(API_BASE_URL)` references xcconfig |
| Xcode scheme env | Runtime environment variables | Set via `xcodebuild -scheme ... ENV_VAR=value` |
| Keychain | Secrets at runtime | Store via Security framework, never in source |
| .env + build script | Pre-build secret injection | Build phase script reads .env into xcconfig |

**Secret collection pattern:**
```xml
<!-- WRONG: Asking user to add API key in Xcode UI -->
<task type="checkpoint:human-action">
  <action>Add API_KEY to Xcode scheme environment variables</action>
  <instructions>Go to Product → Scheme → Edit Scheme → Run → Arguments → Environment Variables</instructions>
</task>

<!-- RIGHT: Claude asks for value, then configures via xcconfig -->
<task type="checkpoint:human-action">
  <action>Provide your API key for the backend service</action>
  <instructions>
    I need your API key for the backend integration.
    Get it from your service dashboard.
    Paste the key value.
  </instructions>
  <verification>I'll add it to the .xcconfig and verify the build resolves it</verification>
  <resume-signal>Paste your API key</resume-signal>
</task>

<task type="auto">
  <n>Configure API key in build settings</n>
  <action>Write key to Debug.xcconfig as `API_KEY = {user-provided-key}`. Add Info.plist entry referencing `$(API_KEY)`.</action>
  <verify>xcodebuild build succeeds, `Bundle.main.infoDictionary["API_KEY"]` resolves</verify>
</task>
```

## Build & Run Automation

| Action | Command | Success Signal | Notes |
|--------|---------|----------------|-------|
| Build (debug) | `xcodebuild -scheme App -configuration Debug build` | "BUILD SUCCEEDED" | Fast incremental builds |
| Build (release) | `xcodebuild -scheme App -configuration Release build` | "BUILD SUCCEEDED" | Optimized, no debug symbols |
| Test | `xcodebuild test -scheme App -destination 'platform=iOS Simulator,name=iPhone 16 Pro'` | "Test Suite passed" | Runs XCTest targets |
| Archive | `xcodebuild archive -scheme App -archivePath App.xcarchive` | "ARCHIVE SUCCEEDED" | For distribution |
| Boot Simulator | `xcrun simctl boot "iPhone 16 Pro"` | Device state: Booted | Needed before install |
| Install on Sim | `xcrun simctl install booted App.app` | No error output | After build |
| Launch on Sim | `xcrun simctl launch booted com.app.bundleid` | PID returned | App starts |

**Simulator lifecycle:**
```bash
# Boot simulator
xcrun simctl boot "iPhone 16 Pro" 2>/dev/null || true

# Build and get app path
xcodebuild -scheme App -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
  -derivedDataPath build/ build 2>&1 | tail -1

# Install and launch
xcrun simctl install booted build/Build/Products/Debug-iphonesimulator/App.app
xcrun simctl launch booted com.app.bundleid
```

**Simulator conflicts:** Shut down stale simulators (`xcrun simctl shutdown all`) or use a specific device UDID.

**Simulator stays running** through checkpoints. Only shutdown when plan complete, switching devices, or testing fresh install.

## CLI Installation Handling

| CLI | Auto-install? | Command |
|-----|---------------|---------|
| xcodebuild | No - requires Xcode | User installs Xcode from App Store |
| xcrun simctl | No - comes with Xcode | Part of Xcode Command Line Tools |
| swift | No - comes with Xcode | Part of Xcode toolchain |
| swiftlint | Yes | `brew install swiftlint` |
| swiftformat | Yes | `brew install swiftformat` |
| fastlane | Yes | `brew install fastlane` or `gem install fastlane` |
| gh (GitHub) | Yes | `brew install gh` |
| pod (CocoaPods) | Yes | `gem install cocoapods` or `brew install cocoapods` |
| firebase | Yes | `brew install firebase-cli` |
| mint | Yes | `brew install mint` (Swift tool manager) |

**Protocol:** Try command → "command not found" → auto-installable? → yes: install silently, retry → no: checkpoint asking user to install (especially Xcode itself).

## Pre-Checkpoint Automation Failures

| Failure | Response |
|---------|----------|
| Build fails | Check compiler errors, fix issue, rebuild (don't proceed to checkpoint) |
| Simulator won't boot | Shutdown all simulators, retry boot, or use different device |
| Missing dependency | Run `swift package resolve` or `pod install`, retry |
| Signing error | Check provisioning profile, fix entitlements (may need auth gate) |
| Auth error (upload) | Create auth gate checkpoint for credentials |
| Test target fails | Fix test configuration first (scheme, destination, host app) |

**Never present a checkpoint with broken verification environment.** If `xcodebuild build` fails, don't ask user to "check the app in Simulator".

```xml
<!-- WRONG: Checkpoint with broken environment -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard view (build failed)</what-built>
  <how-to-verify>Check the app in Simulator...</how-to-verify>
</task>

<!-- RIGHT: Fix first, then checkpoint -->
<task type="auto">
  <n>Fix build issue</n>
  <action>Investigate compiler error, fix root cause, rebuild</action>
  <verify>xcodebuild build succeeds with "BUILD SUCCEEDED"</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Dashboard view - app running in Simulator</what-built>
  <how-to-verify>Check the dashboard tab in Simulator...</how-to-verify>
</task>
```

## Automatable Quick Reference

| Action | Automatable? | Claude does it? |
|--------|--------------|-----------------|
| Build project | Yes (`xcodebuild build`) | YES |
| Run tests | Yes (`xcodebuild test`) | YES |
| Boot Simulator | Yes (`xcrun simctl boot`) | YES |
| Install on Simulator | Yes (`xcrun simctl install`) | YES |
| Archive for distribution | Yes (`xcodebuild archive`) | YES |
| Write .xcconfig file | Yes (Write tool) | YES |
| Resolve dependencies | Yes (`swift package resolve`) | YES |
| Run SwiftLint | Yes (`swiftlint lint`) | YES |
| Upload to TestFlight | Yes (`xcrun altool`) | YES (with credentials) |
| Approve TestFlight build for testing | No (App Store Connect web) | NO |
| Complete Apple Developer enrollment | No | NO |
| Visually verify UI in Simulator | No | NO |
| Test interactive user flows | No | NO |
| Evaluate animation smoothness | No | NO |

</automation_reference>

<writing_guidelines>

**DO:**
- Automate everything with CLI/API before checkpoint
- Be specific: "Check the Dashboard tab in Simulator" not "check the app"
- Number verification steps
- State expected outcomes: "You should see X"
- Provide context: why this checkpoint exists

**DON'T:**
- Ask human to do work Claude can automate ❌
- Assume knowledge: "Configure the usual settings" ❌
- Skip steps: "Set up database" (too vague) ❌
- Mix multiple verifications in one checkpoint ❌

**Placement:**
- **After automation completes** - not before Claude does the work
- **After UI buildout** - before declaring phase complete
- **Before dependent work** - decisions before implementation
- **At integration points** - after configuring external services

**Bad placement:** Before automation ❌ | Too frequent ❌ | Too late (dependent tasks already needed the result) ❌
</writing_guidelines>

<examples>

### Example 1: Persistence Setup (No Checkpoint Needed)

```xml
<task type="auto">
  <n>Configure SwiftData model container</n>
  <files>Sources/Models/User.swift, Sources/App/AppModel.swift</files>
  <action>
    1. Define @Model classes (User, Bookmark) with relationships
    2. Create ModelContainer configuration in App entry point
    3. Add .modelContainer() modifier to root view
    4. Verify schema compiles and container initializes
  </action>
  <verify>
    - xcodebuild build succeeds with SwiftData models
    - Unit test creates in-memory container and inserts test data
    - No runtime crash on container initialization
  </verify>
  <done>SwiftData persistence layer configured and verified</done>
</task>

<!-- NO CHECKPOINT NEEDED - Claude automated everything and verified programmatically -->
```

### Example 2: Full Auth Flow (Single checkpoint at end)

```xml
<task type="auto">
  <n>Create user model</n>
  <files>Sources/Models/User.swift</files>
  <action>Define @Model User with profile fields and session tracking via SwiftData</action>
  <verify>xcodebuild build succeeds with model</verify>
</task>

<task type="auto">
  <n>Create auth service and view model</n>
  <files>Sources/Services/AuthService.swift, Sources/ViewModels/AuthViewModel.swift</files>
  <action>Set up AuthService with Sign in with Apple using AuthenticationServices. Create @Observable AuthViewModel managing auth state.</action>
  <verify>Build succeeds, unit tests pass for AuthViewModel state transitions</verify>
</task>

<task type="auto">
  <n>Create login UI</n>
  <files>Sources/Views/LoginView.swift, Sources/Views/Components/SignInWithAppleButton.swift</files>
  <action>Create LoginView with SignInWithAppleButton and auth state observation</action>
  <verify>xcodebuild build succeeds</verify>
</task>

<task type="auto">
  <n>Build and launch in Simulator for auth testing</n>
  <action>Build and install app on booted Simulator</action>
  <verify>App launches without crash, LoginView appears</verify>
  <done>App running in Simulator showing login screen</done>
</task>

<!-- ONE checkpoint at end verifies the complete flow -->
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Complete authentication flow - app running in Simulator</what-built>
  <how-to-verify>
    1. App shows LoginView with "Sign in with Apple" button
    2. Tap "Sign in with Apple" and complete the auth sheet
    3. Verify: App navigates to DashboardView, user name displayed
    4. Kill and relaunch app: Session persists (user still logged in)
    5. Tap logout: Returns to LoginView, session cleared
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```
</examples>

<anti_patterns>

### ❌ BAD: Asking user to build and run

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard view</what-built>
  <how-to-verify>
    1. Build and run in Xcode (Cmd+R)
    2. Navigate to Dashboard tab
    3. Check layout is correct
  </how-to-verify>
</task>
```

**Why bad:** Claude can run `xcodebuild` and launch in Simulator. User should only interact with the running app, not build it.

### ✅ GOOD: Claude builds and launches, user checks

```xml
<task type="auto">
  <n>Build and launch in Simulator</n>
  <action>Run `xcodebuild` and install on booted Simulator</action>
  <verify>App launches without crash</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Dashboard view - app running in Simulator</what-built>
  <how-to-verify>
    Check the Dashboard tab in Simulator and verify:
    1. Layout matches design
    2. No visual glitches or clipping
  </how-to-verify>
</task>
```

### ❌ BAD: Asking human to upload / ✅ GOOD: Claude automates

```xml
<!-- BAD: Asking user to upload via Xcode Organizer -->
<task type="checkpoint:human-action" gate="blocking">
  <action>Upload to TestFlight</action>
  <instructions>Open Xcode → Product → Archive → Distribute App → App Store Connect → Upload</instructions>
</task>

<!-- GOOD: Claude archives and uploads, user verifies -->
<task type="auto">
  <n>Archive and upload to TestFlight</n>
  <action>Run `xcodebuild archive` then `xcrun altool --upload-app` with stored credentials</action>
  <verify>altool output shows "No errors uploading"</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Build uploaded to TestFlight (check App Store Connect)</what-built>
  <how-to-verify>Open TestFlight app, verify new build appears and installs</how-to-verify>
  <resume-signal>Type "approved"</resume-signal>
</task>
```

### ❌ BAD: Too many checkpoints / ✅ GOOD: Single checkpoint

```xml
<!-- BAD: Checkpoint after every task -->
<task type="auto">Create model</task>
<task type="checkpoint:human-verify">Check model</task>
<task type="auto">Create service</task>
<task type="checkpoint:human-verify">Check service</task>
<task type="auto">Create view</task>
<task type="checkpoint:human-verify">Check view</task>

<!-- GOOD: One checkpoint at end -->
<task type="auto">Create model</task>
<task type="auto">Create service + ViewModel</task>
<task type="auto">Create view</task>

<task type="checkpoint:human-verify">
  <what-built>Complete auth flow (model + service + ViewModel + view)</what-built>
  <how-to-verify>Test full flow: sign in, view dashboard, sign out</how-to-verify>
  <resume-signal>Type "approved"</resume-signal>
</task>
```

### ❌ BAD: Vague verification / ✅ GOOD: Specific steps

```xml
<!-- BAD -->
<task type="checkpoint:human-verify">
  <what-built>Dashboard</what-built>
  <how-to-verify>Check it works</how-to-verify>
</task>

<!-- GOOD -->
<task type="checkpoint:human-verify">
  <what-built>Adaptive dashboard - app running in iPhone 16 Pro Simulator</what-built>
  <how-to-verify>
    Check the dashboard in Simulator and verify:
    1. iPad (regular width): Sidebar visible, detail area fills remaining space
    2. iPhone landscape: Sidebar collapses, swipe from edge to reveal
    3. iPhone portrait: Tab bar navigation, sidebar hidden
    4. Rotate device: Smooth transition, no clipping or layout breaks
  </how-to-verify>
  <resume-signal>Type "approved" or describe layout issues</resume-signal>
</task>
```

### ❌ BAD: Asking user to run CLI commands

```xml
<task type="checkpoint:human-action">
  <action>Resolve dependencies and rebuild</action>
  <instructions>Run: swift package resolve && xcodebuild build</instructions>
</task>
```

**Why bad:** Claude can run these commands. User should never execute CLI commands.

### ❌ BAD: Asking user to configure services manually

```xml
<task type="checkpoint:human-action">
  <action>Configure push notification key in Firebase console</action>
  <instructions>Upload APNs key → Firebase Console → Project Settings → Cloud Messaging → Upload</instructions>
</task>
```

**Why bad:** Firebase has a CLI. Claude should configure via `firebase` CLI and write config to xcconfig directly.

</anti_patterns>

<summary>

Checkpoints formalize human-in-the-loop points for verification and decisions, not manual work.

**The golden rule:** If Claude CAN automate it, Claude MUST automate it.

**Checkpoint priority:**
1. **checkpoint:human-verify** (90%) - Claude automated everything, human confirms visual/functional correctness
2. **checkpoint:decision** (9%) - Human makes architectural/technology choices
3. **checkpoint:human-action** (1%) - Truly unavoidable manual steps with no API/CLI

**When NOT to use checkpoints:**
- Things Claude can verify programmatically (tests, builds)
- File operations (Claude can read files)
- Code correctness (tests and static analysis)
- Anything automatable via CLI/API
</summary>
