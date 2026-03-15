<overview>
Navigation patterns: NavigationStack with type-safe links, enum-based sheet management, NavigationSplitView, Inspector, deep links, and programmatic navigation. Read when building navigation flows or managing modal presentations. Related: state-management.md (navigation state ownership), components.md (TabView).
</overview>

## NavigationStack

```swift
struct AppRootView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            HomeView()
                .navigationDestination(for: UserProfile.self) { user in
                    ProfileView(user: user)
                }
                .navigationDestination(for: TaskItem.self) { task in
                    TaskDetailView(task: task)
                }
        }
    }
}

// Push programmatically
Button("View Profile") { path.append(user) }

// Pop to root
path.removeLast(path.count)
```

## Sheets — Item-Driven (Preferred)

```swift
@State private var selectedItem: Item?

content
    .sheet(item: $selectedItem) { item in
        ItemDetailSheet(item: item)
    }
```

### Enum-Based Sheet Management

```swift
enum Sheet: Identifiable {
    case add
    case edit(Article)
    case categories

    var id: String {
        switch self {
        case .add: "add"
        case .edit(let a): "edit-\(a.id)"
        case .categories: "categories"
        }
    }
}

@State private var presentedSheet: Sheet?

content
    .sheet(item: $presentedSheet) { sheet in
        switch sheet {
        case .add: AddArticleView()
        case .edit(let article): EditArticleView(article: article)
        case .categories: CategoriesView()
        }
    }
```

### Sheet Ownership Principle

Sheets own their actions and dismiss internally. Avoid callback prop-drilling.

```swift
struct EditItemSheet: View {
    @Environment(\.dismiss) private var dismiss
    @Environment(Store.self) private var store
    let item: Item

    var body: some View {
        Button("Save") {
            Task {
                await store.save(item)
                dismiss()
            }
        }
    }
}
```

## NavigationSplitView

For sidebar-based multi-column layouts:

```swift
NavigationSplitView {
    SidebarView(selection: $selectedCategory)
} content: {
    CategoryListView(category: selectedCategory)
} detail: {
    DetailView(item: selectedItem)
}
```

Adapts automatically: three columns on iPad landscape, collapses on iPhone.

## Inspector (iOS 17+)

Trailing-edge supplementary panel that adapts to sheet on compact:

```swift
content
    .inspector(isPresented: $showInspector) {
        InspectorContent(item: selectedItem)
            .inspectorColumnWidth(min: 200, ideal: 300, max: 400)
    }
```

## Alerts and Confirmation Dialogs

```swift
.alert("Delete Item?", isPresented: $showDeleteAlert) {
    Button("Delete", role: .destructive) { deleteItem() }
    Button("Cancel", role: .cancel) { }
} message: {
    Text("This action cannot be undone.")
}

.confirmationDialog("Options", isPresented: $showOptions, titleVisibility: .visible) {
    Button("Share") { share() }
    Button("Delete", role: .destructive) { delete() }
}
```

## FullScreenCover

```swift
.fullScreenCover(isPresented: $showOnboarding) {
    OnboardingFlow()
}
```

## Deep Links

```swift
.onOpenURL { url in
    guard let components = URLComponents(url: url, resolvingAgainstBaseURL: false) else { return }
    switch components.host {
    case "profile": path.append(Route.profile)
    case "task": if let id = components.queryItems?.first?.value {
        path.append(Route.task(id: id))
    }
    default: break
    }
}
```

## Platform Adaptation

| Pattern | iPhone | iPad Regular | iPad Compact | macOS |
|---------|--------|-------------|-------------|-------|
| NavigationStack | Full screen | Detail pane | Full screen | Window content |
| Sheet | Bottom card | Centered modal | Bottom card | Window sheet |
| Inspector | Sheet | Trailing column | Sheet | Trailing column |
| SplitView | Collapsed | 2-3 columns | Collapsed | 2-3 columns |
