# Verification Patterns

How to verify different types of iOS artifacts are real implementations, not stubs or placeholders.

<core_principle>
**Existence ≠ Implementation**

A file existing does not mean the feature works. Verification must check:
1. **Exists** — File is present at expected path
2. **Substantive** — Content is real implementation, not placeholder
3. **Wired** — Connected to the rest of the system
4. **Functional** — Actually works when invoked

Levels 1-3 can be checked programmatically. Level 4 often requires human verification (Xcode Preview, Simulator, device).
</core_principle>

<stub_detection>

## Universal Stub Patterns

These patterns indicate placeholder code regardless of file type:

**Comment-based stubs:**
```bash
# Grep patterns for stub comments
grep -E "(TODO|FIXME|XXX|HACK|PLACEHOLDER)" "$file"
grep -E "implement|add later|coming soon|will be" "$file" -i
grep -E "// \.\.\.|/\* \.\.\. \*/" "$file"
```

**Placeholder text in output:**
```bash
# UI placeholder patterns
grep -E "placeholder|lorem ipsum|coming soon|under construction" "$file" -i
grep -E "Hello, World|Hello, world|ContentView" "$file"  # Default Xcode template text
```

**Empty or trivial implementations:**
```bash
# Functions that do nothing
grep -E "func .+\{[[:space:]]*\}$" "$file"
grep -E "fatalError|return nil|return \[\]|return \.init\(\)" "$file"
grep -E "// no-op|// empty" "$file" -i
```

**Hardcoded values where dynamic expected:**
```bash
# Hardcoded user-facing strings (should use String(localized:))
grep -E '(Text|Label|Button|Toggle|\.navigationTitle)\("[A-Z]' "$file" | grep -v 'String(localized:\|#Preview\|accessibilityIdentifier'
# Hardcoded data that should come from a model or service
grep -E "let .+ = \[.*\".*\".*\]" "$file"  # Hardcoded array literals in view code
```

</stub_detection>

<swiftui_views>

## SwiftUI Views

**Existence check:**
```bash
# File exists and declares a View
[ -f "$view_path" ] && grep -E "struct \w+View: View" "$view_path"
```

**Substantive check:**
```bash
# Has a body with real content (not placeholder)
grep -E "var body: some View" "$view_path"

# Returns meaningful content, not empty/placeholder
grep -A 5 "var body: some View" "$view_path" | grep -v "EmptyView()\|Text(\"Hello\|Text(\"Placeholder\|Text(\"TODO\|Color\.clear$\|Spacer()$"

# Has meaningful UI elements
grep -E "List\|ForEach\|NavigationStack\|Form\|ScrollView\|LazyVStack\|LazyVGrid\|TabView" "$view_path"

# Uses state or model data (not fully static)
grep -E "@State\|@Binding\|@Environment\|@Query\|@Observable\|@Bindable\|\.task\|\.onAppear" "$view_path"
```

**Stub patterns specific to SwiftUI Views:**
```swift
// RED FLAGS — placeholder views:
var body: some View {
    Text("Hello, World!")          // Default Xcode template
}

var body: some View {
    Text("TODO")                   // Placeholder text
}

var body: some View {
    EmptyView()                    // Empty body
}

var body: some View {
    Color.clear                    // Invisible placeholder
}

// Empty action closures:
Button("Save") { }                // Button does nothing
.onTapGesture { }                 // Gesture does nothing
.task { }                         // Empty async task
.onAppear { }                     // Empty onAppear
.refreshable { }                  // Pull-to-refresh does nothing

// NavigationLink to placeholder:
NavigationLink("Details") {
    Text("Coming soon")            // Destination is stub
}
```

**Wiring check:**
```bash
# View is referenced by another view or navigation destination
grep -r "$(basename ${view_path%.swift})()" Sources/ --include="*.swift" | grep -v "$view_path"

# Or used as a navigationDestination
grep -r "$(basename ${view_path%.swift})" Sources/ --include="*.swift" | grep -E "navigationDestination|NavigationLink|sheet|fullScreenCover" | grep -v "$view_path"

# View uses its expected ViewModel or model
grep -E "@Environment\(.*\)|@State.*ViewModel\|@Bindable" "$view_path"
```

**Functional verification (human required):**
- Does `#Preview` render meaningful content (not blank or error)?
- Do interactive elements respond to taps?
- Does data appear when view loads (`.task` triggers)?
- Does navigation push to a real destination?

</swiftui_views>

<viewmodels>

## ViewModels (@Observable Classes)

**Existence check:**
```bash
# File exists and declares an @Observable class
[ -f "$vm_path" ] && grep -E "@Observable|class \w+ViewModel" "$vm_path"
```

**Substantive check:**
```bash
# Has published properties (state the view reads)
grep -E "var \w+:" "$vm_path" | grep -v "private let\|private(set) let\|init("

# Has async methods that do real work
grep -E "func \w+.*async" "$vm_path"

# Calls a service, repository, or API (not hardcoded data)
grep -E "try await\|URLSession\|modelContext\|repository\.\|service\.\|api\.\|client\." "$vm_path"

# Has error handling
grep -E "do \{|catch\|errorMessage\|error:" "$vm_path"

# More than trivial length
[ $(wc -l < "$vm_path") -gt 15 ]
```

**Stub patterns specific to ViewModels:**
```swift
// RED FLAGS — empty or hardcoded ViewModels:
@Observable
class ChatViewModel {
    var messages: [Message] = []
    // ... no methods to populate messages
}

@Observable
class ProfileViewModel {
    func loadProfile() async {
        // TODO: implement
    }
}

// Always returns empty/hardcoded data:
func fetchItems() async -> [Item] {
    return []                      // No API/DB call
}

func fetchUser() async -> User {
    return User(name: "Test User") // Hardcoded, not from service
}
```

**Wiring check:**
```bash
# ViewModel is used by a View
grep -r "$(basename ${vm_path%.swift})" Sources/ --include="*.swift" | grep -E "@Environment|@State.*=.*\(|@Bindable" | grep -v "$vm_path"

# ViewModel methods are called from a View
VM_NAME=$(grep -oE "class \w+ViewModel" "$vm_path" | sed 's/class //')
grep -r "$VM_NAME" Sources/ --include="*.swift" | grep -E "\.\w+(" | grep -v "$vm_path" | grep -v "init("
```

**Functional verification (human required):**
- Does the ViewModel populate its state when its async methods are called?
- Does the View update when ViewModel state changes?

</viewmodels>

<swiftdata_models>

## SwiftData Models

**Existence check:**
```bash
# File exists and declares a @Model
[ -f "$model_path" ] && grep -E "@Model" "$model_path"
```

**Substantive check:**
```bash
# Has meaningful stored properties (not just id)
grep -A 20 "@Model" "$model_path" | grep -E "^\s+var \w+:" | wc -l
# Should be > 1 (more than just an id field)

# Has relationships if expected
grep -E "@Relationship\|var .*: \[.*\]\|var .*:.*?" "$model_path"

# Has appropriate types (not all String)
grep -A 20 "@Model" "$model_path" | grep -E "Int\|Double\|Date\|Bool\|Decimal\|Data\|UUID\|URL"

# Has initializer
grep -E "init(" "$model_path"
```

**Stub patterns specific to models:**
```swift
// RED FLAGS — incomplete models:
@Model
final class ChatMessage {
    var id: UUID
    // TODO: add fields
}

@Model
final class Order {
    var id: UUID
    var title: String  // Only one real field — missing items, total, status, createdAt
}

// Computed properties that return placeholder:
var displayName: String {
    "Unknown"                      // Hardcoded, not derived from properties
}
```

**Wiring check:**
```bash
# Model registered in modelContainer
grep -r "modelContainer\|ModelContainer" Sources/ --include="*.swift" | grep "$(basename ${model_path%.swift})"

# Model queried somewhere (@Query or modelContext.fetch)
MODEL_NAME=$(grep -oE "class \w+" "$model_path" | head -1 | sed 's/class //')
grep -r "$MODEL_NAME" Sources/ --include="*.swift" | grep -E "@Query\|modelContext\.fetch\|modelContext\.delete\|modelContext\.insert\|FetchDescriptor" | grep -v "$model_path"
```

**Functional verification (human required):**
- Does the model persist data across app launches?
- Do relationships load correctly?

</swiftdata_models>

<services_utilities>

## Services, Repositories, and Utilities

**Existence check:**
```bash
# File exists and exports a class, struct, or protocol
[ -f "$service_path" ] && grep -E "class \w+|struct \w+|protocol \w+" "$service_path"
```

**Substantive check:**
```bash
# Has real implementations (async methods that do work)
grep -E "func \w+.*async.*throws" "$service_path"

# Makes actual network calls or data operations
grep -E "URLSession\|URLRequest\|JSONDecoder\|JSONEncoder\|modelContext\|FileManager\|Keychain" "$service_path"

# Has error handling
grep -E "do \{|catch\|throw " "$service_path"

# More than trivial length
[ $(wc -l < "$service_path") -gt 15 ]
```

**Stub patterns specific to services:**
```swift
// RED FLAGS — services that do nothing:
class AuthService {
    func login(email: String, password: String) async throws -> Token {
        return Token(value: "fake-token")  // Hardcoded, no real auth
    }
}

class APIClient {
    func fetch<T: Decodable>(_ path: String) async throws -> T {
        fatalError("Not implemented")
    }
}

struct NetworkService {
    func request(_ url: URL) async throws -> Data {
        return Data()                      // Empty data, no network call
    }
}

// Protocol declared but no conforming type:
protocol TaskRepositoryProtocol {
    func fetchAll() async throws -> [TaskItem]
}
// ... no class/struct conforms to it
```

**Wiring check:**
```bash
# Service is injected somewhere (Environment, init parameter, or property)
SERVICE_NAME=$(grep -oE "(class|struct) \w+(Service|Client|Repository|Manager)" "$service_path" | head -1 | awk '{print $2}')
grep -r "$SERVICE_NAME" Sources/ --include="*.swift" | grep -E "@Environment\|init.*$SERVICE_NAME\|let.*:.*$SERVICE_NAME\|var.*:.*$SERVICE_NAME" | grep -v "$service_path"

# Service methods are called
grep -r "$SERVICE_NAME" Sources/ --include="*.swift" | grep -E "\.\w+\(" | grep -v "$service_path" | grep -v "init("
```

</services_utilities>

<project_config>

## Project Configuration

**Info.plist / Privacy keys:**
```bash
# Info.plist exists (may be in project root or target folder)
find . -name "Info.plist" -not -path "*/Pods/*" -not -path "*/.build/*" | head -1

# Required privacy keys present (for features that need them)
# Camera
grep -q "NSCameraUsageDescription" "$INFO_PLIST"
# Location
grep -q "NSLocationWhenInUseUsageDescription" "$INFO_PLIST"
# Photos
grep -q "NSPhotoLibraryUsageDescription" "$INFO_PLIST"
```

**Xcode project / Package.swift:**
```bash
# Project file exists
[ -d *.xcodeproj ] || [ -d *.xcworkspace ] || [ -f Package.swift ]

# Deployment target is set correctly
grep -r "IPHONEOS_DEPLOYMENT_TARGET" *.xcodeproj/project.pbxproj 2>/dev/null | head -1
# Or in Package.swift:
grep "\.iOS(" Package.swift 2>/dev/null
```

**Asset catalog:**
```bash
# Assets.xcassets exists
[ -d "$(find . -name 'Assets.xcassets' -not -path '*/.build/*' | head -1)" ]

# App icon is present (not empty)
find . -name "AppIcon.appiconset" -not -path "*/.build/*" | head -1 | xargs ls 2>/dev/null | grep -v "Contents.json"

# AccentColor is defined
[ -d "$(find . -name 'AccentColor.colorset' -not -path '*/.build/*' | head -1)" ]
```

**Localization:**
```bash
# String catalog exists
find . -name "Localizable.xcstrings" -not -path "*/.build/*" | head -1

# Or legacy .strings files
find . -name "Localizable.strings" -not -path "*/.build/*" | head -1
```

**Entitlements (if required):**
```bash
# Entitlements file exists when features need it
find . -name "*.entitlements" -not -path "*/.build/*" | head -1

# Check specific capabilities
grep -q "aps-environment" "$ENTITLEMENTS"         # Push notifications
grep -q "com.apple.developer.icloud" "$ENTITLEMENTS"  # iCloud
grep -q "keychain-access-groups" "$ENTITLEMENTS"      # Keychain sharing
```

**Stub patterns specific to configuration:**
```bash
# RED FLAGS:
# Info.plist with placeholder descriptions
grep -E "PRODUCT_NAME|TODO|change this|replace" "$INFO_PLIST" -i

# Empty entitlements (file exists but no capabilities)
[ $(grep -c "key" "$ENTITLEMENTS" 2>/dev/null) -eq 0 ]

# Missing required privacy key for a feature that uses it
# (e.g., code calls CLLocationManager but NSLocationWhenInUseUsageDescription is missing)
```

</project_config>

<wiring_verification>

## Wiring Verification Patterns

Wiring verification checks that components actually communicate. This is where most stubs hide.

### Pattern: View → ViewModel

**Check:** Does the View actually use a ViewModel's state and call its methods?

```bash
# View declares the ViewModel
grep -E "@Environment.*ViewModel\|@State.*var.*viewModel\|@Bindable.*var.*viewModel" "$view_path"

# View reads ViewModel properties in body
VM_NAME=$(grep -oE "\w+ViewModel" "$view_path" | head -1)
grep -E "$VM_NAME\.\w+" "$view_path" | grep -v "import\|//\|init("

# View calls ViewModel methods (in .task, .onAppear, button actions, etc.)
grep -E "\.$VM_NAME\.\w+(" "$view_path" 2>/dev/null || grep -E "viewModel\.\w+(" "$view_path"
```

**Red flags:**
```swift
// ViewModel declared but never referenced in body:
@State private var viewModel = ChatViewModel()
var body: some View {
    Text("Chat")                   // viewModel not used at all
}

// ViewModel state never read:
@State private var viewModel = TaskListViewModel()
var body: some View {
    List {
        Text("No tasks")           // Shows static text, ignores viewModel.tasks
    }
}

// ViewModel methods never called:
.task {
    // viewModel.loadTasks() is missing — data never loads
}
```

### Pattern: ViewModel → Service / Repository

**Check:** Does the ViewModel call a service to get or mutate data?

```bash
# ViewModel holds reference to service
grep -E "let \w+:.*Protocol\|let \w+:.*Service\|let \w+:.*Repository\|let \w+:.*Client" "$vm_path"

# ViewModel calls service methods
grep -E "try await.*\.\w+(\|service\.\|repository\.\|client\.\|api\." "$vm_path"

# Service response is stored in ViewModel state
grep -E "self\.\w+ = try await\|self\.\w+ = .*result" "$vm_path"
```

**Red flags:**
```swift
// Service declared but never called:
private let repository: TaskRepositoryProtocol
func loadTasks() async {
    isLoading = true
    // repository.fetchAll() is missing
    isLoading = false
}

// Service called but result discarded:
func loadTasks() async {
    _ = try? await repository.fetchAll()  // Result thrown away
}

// Service called but response not stored in state:
func loadTasks() async throws {
    let tasks = try await repository.fetchAll()
    // self.tasks = tasks is missing — View never sees the data
}
```

### Pattern: Navigation → Destination

**Check:** Does navigation actually push to a real View?

```bash
# NavigationStack with destinations
grep -E "\.navigationDestination\(for:" "$view_path"

# NavigationLink with real destinations (not Text placeholders)
grep -A 3 "NavigationLink" "$view_path" | grep -v "Text(\"Coming soon\|Text(\"TODO\|Text(\"Placeholder\|EmptyView()"

# Sheet presentations with real content
grep -A 3 "\.sheet\|\.fullScreenCover" "$view_path" | grep -v "Text(\"Coming soon\|EmptyView()"

# Programmatic navigation via path.append
grep -E "path\.append\|\.path\.append" "$view_path"
```

**Red flags:**
```swift
// NavigationLink to placeholder:
NavigationLink("Profile") {
    Text("Coming soon")            // Destination is stub
}

// .navigationDestination declared but type never pushed:
.navigationDestination(for: UserProfile.self) { user in
    ProfileView(user: user)
}
// ... but nowhere does path.append(UserProfile(...)) occur

// Sheet with empty content:
.sheet(isPresented: $showSettings) {
    EmptyView()                    // Sheet shows nothing
}
```

### Pattern: State → Render

**Check:** Does the View render dynamic state, not hardcoded content?

```bash
# @State or ViewModel properties used in body
grep -E "\{.*\.\w+.*\}|ForEach.*\.\w+|if .*\.\w+" "$view_path" | grep -v "//"

# List/ForEach renders model data
grep -E "ForEach\|List.*\{" "$view_path"

# Conditional rendering based on state
grep -E "if .*isLoading\|if .*isEmpty\|if let .*error\|if .*== nil" "$view_path"
```

**Red flags:**
```swift
// State exists but not rendered:
@State private var tasks: [TaskItem] = []
var body: some View {
    Text("No tasks")              // Always shows static text
}

// @Query declared but results ignored:
@Query var items: [Item]
var body: some View {
    List {
        Text("Item 1")            // Hardcoded, not using items
        Text("Item 2")
    }
}

// ForEach over empty constant:
let items: [String] = []
ForEach(items, id: \.self) { item in
    Text(item)                     // Will never render anything
}
```

### Pattern: Model → Persistence Container

**Check:** Is the SwiftData model registered and queryable?

```bash
# Model registered in app's modelContainer
grep -r "modelContainer\|ModelContainer" Sources/ --include="*.swift" | grep -E "for:.*\[.*$(basename ${model_path%.swift})"

# Model inserted somewhere
grep -r "modelContext\.insert" Sources/ --include="*.swift" | grep -v "Tests/"

# Model queried somewhere
grep -r "@Query\|modelContext\.fetch\|FetchDescriptor" Sources/ --include="*.swift" | grep -v "Tests/"
```

**Red flags:**
```swift
// @Model defined but never registered in modelContainer:
// App.swift has:
.modelContainer(for: [TaskItem.self])
// ... but ChatMessage.self is missing — @Query for ChatMessage will crash or return empty

// modelContext.insert() never called — model is defined but nothing creates records
```

</wiring_verification>

<verification_checklist>

## Quick Verification Checklist

For each artifact type, run through this checklist:

### View Checklist
- [ ] File exists at expected path (`Sources/Views/` or `Sources/Screens/`)
- [ ] Conforms to `View` protocol with `body` property
- [ ] `body` returns real content (not `Text("Hello, World!")`, `EmptyView()`, or `Color.clear`)
- [ ] No placeholder text in rendered output
- [ ] Uses state, bindings, or model data (not fully static)
- [ ] Interactive elements have real action closures (not empty `{ }`)
- [ ] Has `#Preview` macro (not deprecated `PreviewProvider`)
- [ ] Referenced by another View, NavigationLink, or sheet

### ViewModel Checklist
- [ ] File exists at expected path (`Sources/ViewModels/`)
- [ ] Marked with `@Observable` (not `ObservableObject` on iOS 17+)
- [ ] Has properties that hold meaningful state
- [ ] Has async methods that call services/repositories (not hardcoded returns)
- [ ] Error handling exists (do/catch with error state)
- [ ] Used by at least one View via `@Environment`, `@State`, or `@Bindable`
- [ ] Methods are called from View (in `.task`, actions, etc.)

### Model Checklist
- [ ] File exists at expected path (`Sources/Models/`)
- [ ] Decorated with `@Model` (SwiftData)
- [ ] Has all expected stored properties (not just `id`)
- [ ] Properties have appropriate types (not all `String`)
- [ ] Has initializer
- [ ] Registered in `modelContainer(for:)` in App entry point
- [ ] Queried or inserted somewhere (via `@Query` or `modelContext`)

### Service / Repository Checklist
- [ ] File exists at expected path (`Sources/Services/`)
- [ ] Protocol defined for testability
- [ ] Has real implementations (async methods with actual work)
- [ ] Makes network calls, database operations, or meaningful computation
- [ ] Error handling present
- [ ] Injected into at least one ViewModel or View
- [ ] Methods are called (not just declared)

### Configuration Checklist
- [ ] Xcode project or Package.swift exists
- [ ] Deployment target is set (iOS 17+)
- [ ] Assets.xcassets present with AppIcon
- [ ] Info.plist has required privacy usage descriptions for used APIs
- [ ] Entitlements file present if needed (push notifications, iCloud, etc.)
- [ ] Localizable.xcstrings present if app has user-facing text

### Wiring Checklist
- [ ] View → ViewModel: View declares and uses ViewModel state and methods
- [ ] ViewModel → Service: ViewModel calls service, stores results in state
- [ ] Navigation → Destination: NavigationLink/sheet/destination points to real View
- [ ] State → Render: View body renders state variables, not hardcoded text
- [ ] Model → Container: @Model registered in modelContainer, queried/inserted somewhere

</verification_checklist>

<automated_verification_script>

## Automated Verification Approach

For the verification subagent, use this pattern:

```bash
# 1. Check existence
check_exists() {
  [ -f "$1" ] && echo "EXISTS: $1" || echo "MISSING: $1"
}

# 2. Check for stub patterns
check_stubs() {
  local file="$1"
  local stubs=$(grep -c -E "TODO|FIXME|placeholder|not implemented|Hello, World|EmptyView()" "$file" 2>/dev/null || echo 0)
  [ "$stubs" -gt 0 ] && echo "STUB_PATTERNS: $stubs in $file"
}

# 3. Check wiring (view references viewmodel)
check_view_wiring() {
  local view_file="$1"
  local vm_name="$2"
  grep -q "$vm_name" "$view_file" && echo "WIRED: $view_file → $vm_name" || echo "NOT_WIRED: $view_file → $vm_name"
}

# 4. Check substantive (more than N lines, has expected patterns)
check_substantive() {
  local file="$1"
  local min_lines="$2"
  local pattern="$3"
  local lines=$(wc -l < "$file" 2>/dev/null || echo 0)
  local has_pattern=$(grep -c -E "$pattern" "$file" 2>/dev/null || echo 0)
  [ "$lines" -ge "$min_lines" ] && [ "$has_pattern" -gt 0 ] && echo "SUBSTANTIVE: $file" || echo "THIN: $file ($lines lines, $has_pattern matches)"
}

# 5. Check view is not a stub
check_view_substantive() {
  local file="$1"
  check_exists "$file"
  check_stubs "$file"
  check_substantive "$file" 15 "List|ForEach|NavigationStack|Form|ScrollView|LazyVStack|@State|@Binding|@Environment|@Query|\.task|\.onAppear"
}

# 6. Check model is registered in container
check_model_registered() {
  local model_name="$1"
  local app_files=$(find Sources/ -name "*.swift" -exec grep -l "modelContainer" {} \;)
  if [ -n "$app_files" ]; then
    grep -q "$model_name" $app_files && echo "REGISTERED: $model_name in modelContainer" || echo "NOT_REGISTERED: $model_name missing from modelContainer"
  else
    echo "NO_CONTAINER: No modelContainer found in project"
  fi
}
```

Run these checks against each must-have artifact. Aggregate results into VERIFICATION.md.

</automated_verification_script>

<human_verification_triggers>

## When to Require Human Verification

Some things can't be verified programmatically. Flag these for human testing:

**Always human:**
- Visual appearance (does the layout look correct in Preview / Simulator?)
- User flow completion (can you actually complete the task end-to-end?)
- SwiftUI Preview rendering (does `#Preview` show expected content?)
- Animations and transitions (do they feel right? do they respect Reduce Motion?)
- Real-time behavior (WebSocket, push notifications, background refresh)
- External service integration (Sign in with Apple, StoreKit, CloudKit)
- Performance on physical device (scrolling smoothness, launch time)
- VoiceOver navigation (does swipe order make sense? are all elements announced?)
- Dark Mode appearance (do custom colors adapt correctly?)
- Dynamic Type at largest sizes (does layout hold at accessibility sizes?)
- Different device sizes (iPhone SE, Pro Max, iPad if supported)

**Human if uncertain:**
- Complex navigation flows that grep can't trace
- Dynamic state behavior depending on multiple conditions
- Edge cases (empty state, error state, offline state)
- Haptic feedback and sound effects

**Format for human verification request:**
```markdown
### 1. {Feature} — Visual Check

**Test:** {What to do in Simulator or on device}
**Expected:** {What should appear or happen}
**Check:** {Specific thing to confirm}
**Why human:** {Why grep/bash can't verify this}
```

### Example Human Verification Items

```markdown
### 1. Task List — Data Loading

**Test:** Launch app, navigate to Tasks tab
**Expected:** Loading indicator appears briefly, then task list populates from SwiftData
**Check:** Pull-to-refresh triggers reload. Empty state shows when no tasks exist.
**Why human:** Requires visual confirmation of loading states and data rendering

### 2. Profile Edit — Form Submission

**Test:** Navigate to Profile, tap Edit, change name, tap Save
**Expected:** Name updates immediately in the UI, persists after app restart
**Check:** Does the save actually persist? Does navigation pop back?
**Why human:** Multi-step flow with persistence — can't verify end-to-end via grep

### 3. Accessibility — VoiceOver Navigation

**Test:** Enable VoiceOver, navigate through the main screen by swiping right
**Expected:** Every interactive element is announced with a meaningful label
**Check:** Are buttons announced as "button"? Do images have descriptions?
**Why human:** VoiceOver behavior requires actual device/Simulator interaction

### 4. Dark Mode — Custom Colors

**Test:** Switch to Dark Mode in Settings, check all screens
**Expected:** All text is readable, custom colors adapt, no white-on-white
**Check:** Focus on cards, badges, and any custom-colored elements
**Why human:** Color contrast in context requires visual evaluation
```

</human_verification_triggers>

<checkpoint_automation_reference>

## Pre-Checkpoint Automation

For automation-first checkpoint patterns and error recovery protocols, see:

**@~/.claude/get-shit-done/references/checkpoints.md** → `<automation_reference>` section

Key principles:
- Claude sets up verification environment BEFORE presenting checkpoints
- Users visit Xcode Previews or Simulator — not CLI commands
- Build verification: `xcodebuild build` before checkpoint
- Test verification: `xcodebuild test` before checkpoint
- Error handling: fix build errors before checkpoint, never present checkpoint with broken build

### iOS-Specific Verification Commands

```bash
# Verify project builds
xcodebuild build \
  -scheme "AppName" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  | tail -5

# Verify tests pass
xcodebuild test \
  -scheme "AppName" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  | tail -10

# Verify no compiler warnings (optional, stricter)
xcodebuild build \
  -scheme "AppName" \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  2>&1 | grep -E "warning:" | wc -l
```

</checkpoint_automation_reference>
