<overview>
File organization, import ordering, MARK patterns, view body structure, MVVM folder structure, dependency injection via Environment, modularization with local Swift Packages, and localization patterns. Read when starting a new project, reviewing file organization, or setting up DI. Related: naming-conventions.md (naming rules), framework-selection.md (which framework to pick).
</overview>

## Import Organization

Group imports separated by blank lines. Apple frameworks first, then third-party/internal.

```swift
import SwiftUI

import SwiftData
import Observation

import MyFeatureModule
import MyNetworkingModule
```

Within groups, alphabetical order.

## View Body Structure

Every SwiftUI View follows this internal structure:

```swift
struct ProfileView: View {
    // MARK: - Environment
    @Environment(\.dismiss) private var dismiss
    @Environment(\.modelContext) private var modelContext

    // MARK: - State
    @State private var isEditing = false
    @State private var errorMessage: String?

    // MARK: - Properties
    let user: UserProfile
    @Binding var selectedTab: Tab

    // MARK: - Body
    var body: some View {
        NavigationStack {
            content
                .navigationTitle("Profile")
                .toolbar { toolbarContent }
                .alert("Error", isPresented: hasError) { alertActions }
                .task { await loadData() }
        }
    }

    // MARK: - Subviews
    private var content: some View {
        List {
            headerSection
            detailsSection
        }
    }

    // MARK: - Toolbar
    @ToolbarContentBuilder
    private var toolbarContent: some ToolbarContent {
        ToolbarItem(placement: .primaryAction) {
            Button("Edit") { isEditing = true }
        }
    }

    // MARK: - Actions
    private func loadData() async { /* ... */ }
}
```

**Rule:** Keep `body` under ~30 lines. Extract subviews as computed properties when it grows.

## Preview Macro (REQUIRED)

Always use `#Preview`. Never use the deprecated `PreviewProvider` protocol.

```swift
#Preview {
    ProfileView(user: .preview)
}

#Preview("Editing Mode") {
    ProfileView(user: .preview, isEditing: true)
}

#Preview(traits: .landscapeLeft) {
    ProfileView(user: .preview)
}
```

Provide meaningful preview data via static factory methods:

```swift
extension UserProfile {
    static var preview: UserProfile {
        UserProfile(name: "Jane Doe", email: "jane@example.com")
    }
}
```

## File Organization

Recommended order within a file:

1. Imports
2. Type declaration (struct/class/enum)
3. Stored properties (environment, state, bindings, lets, vars)
4. `body` (for Views)
5. Computed properties / subviews
6. Methods
7. Nested types
8. Extensions (protocol conformances in separate extensions)

## MVVM Folder Structure

```
MyApp/
  Sources/
    App/
      MyApp.swift                  # @main App entry point
      AppDelegate.swift            # Only if needed (push notifications)
    Models/
      UserProfile.swift
      TaskItem.swift
    ViewModels/
      ProfileViewModel.swift
      TaskListViewModel.swift
    Views/
      Components/                  # Reusable UI components
        AvatarView.swift
        LoadingOverlay.swift
      Screens/
        Home/
          HomeView.swift
        Profile/
          ProfileView.swift
          ProfileEditView.swift
    Services/
      APIClient.swift
      AuthService.swift
    Utilities/
      Extensions/
        String+Validation.swift
      Modifiers/
        CardModifier.swift
    Resources/
      Assets.xcassets
      Localizable.xcstrings
  Tests/
    MyAppTests/
    MyAppUITests/
```

### App Entry Point

```swift
@main
struct MyApp: App {
    @State private var taskStore = TaskStore()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(taskStore)
                .modelContainer(for: [TaskItem.self])
        }
    }
}
```

## Dependency Injection via Environment

The standard DI pattern in SwiftUI. No singletons, no service locators.

```swift
// 1. Define environment key
private struct APIClientKey: EnvironmentKey {
    static let defaultValue: APIClient = .live
}

extension EnvironmentValues {
    var apiClient: APIClient {
        get { self[APIClientKey.self] }
        set { self[APIClientKey.self] = newValue }
    }
}

// 2. Inject at root
ContentView()
    .environment(\.apiClient, .live)

// 3. Consume in views
struct ProfileView: View {
    @Environment(\.apiClient) private var apiClient
    var body: some View { /* use apiClient */ }
}

// 4. Inject mock for testing/previews
ProfileView()
    .environment(\.apiClient, .mock)
```

For `@Observable` objects, inject directly:

```swift
ContentView().environment(taskStore)
// consume: @Environment(TaskStore.self) private var store
```

## Modularization with Local Swift Packages

For larger projects (10+ screens, multiple developers, shared components):

```
MyApp/
  MyApp/                          # Thin shell — wires modules
  Packages/
    FeatureAuth/
      Sources/FeatureAuth/
      Tests/FeatureAuthTests/
      Package.swift
    FeatureProfile/
    CoreNetworking/               # Shared infrastructure
    CoreDesignSystem/             # Shared UI components
```

**When to modularize:**
- 10+ screens — feature boundaries become natural package boundaries
- Multiple developers — reduces merge conflicts
- Shared components — design system or networking reused across targets
- Build time — SPM enables incremental compilation

**Start as monolith.** Extract packages when pain (build time, conflicts, reuse) justifies it. Don't modularize prematurely.

## Localization Patterns

All user-facing text MUST use `String(localized:)`. No exceptions.

```swift
// Simple string
Text(String(localized: "welcome_title"))

// With interpolation
Text(String(localized: "greeting \(userName)"))

// With translator comment
Text(String(localized: "delete_confirmation",
            defaultValue: "Are you sure you want to delete this item?",
            comment: "Shown when user taps delete on a task"))
```

Store translations in `Localizable.xcstrings` (Xcode 15+ string catalog). Place in `Resources/`.

**Localize:** All `Text()`, button labels, alert titles, navigation titles, tab labels, error messages, accessibility labels.

**Do NOT localize:** Accessibility identifiers, log messages, analytics events, API keys, enum raw values.
