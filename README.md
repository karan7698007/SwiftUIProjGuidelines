# SwiftUI Project Guidelines

A comprehensive set of architectural and coding standards for SwiftUI projects to ensure consistency, maintainability, scalability, and performance.

---

## Table of Contents

1. [Architecture](#architecture)
2. [Platform Support](#platform-support)
3. [Reusable UI Components](#reusable-ui-components)
4. [Appearance](#appearance)
5. [Splash Screen](#splash-screen)
6. [Data Storage](#data-storage)
7. [Manager Classes](#manager-classes)
8. [Swift Version & Syntax](#swift-version--syntax)
9. [Combine Usage](#combine-usage)
10. [Performance Optimization](#performance-optimization)
11. [Dynamic Font Sizing](#dynamic-font-sizing)
12. [Centralized Configuration](#centralized-configuration)
13. [SOLID Principles](#solid-principles)
14. [Navigation](#navigation)
15. [Reusable System Components](#reusable-system-components)
16. [Networking](#networking)
17. [Concurrency](#concurrency)
18. [Async Image Loading](#async-image-loading)
19. [Package Management](#package-management)
20. [Logging](#logging)
21. [Permission Handling](#permission-handling)
22. [Code Quality](#code-quality)
23. [Documentation](#documentation)
24. [Data Passing Between Modules](#data-passing-between-modules)

---

## Architecture

- The project must follow the **MVVM (Model-View-ViewModel)** architectural pattern.
- The **Model**, **View**, and **ViewModel** for each module must be placed inside a **single dedicated folder** for that module.

```
Features/
└── Login/
    ├── LoginModel.swift
    ├── LoginView.swift
    └── LoginViewModel.swift
```

---

## Platform Support

- The project must support **iPhone**, **iPad**, and **Mac Catalyst**.
- UI layouts and interactions must be adapted appropriately for each platform.

---

## Reusable UI Components

- Common UI elements must be wrapped into **reusable custom components**.
- Examples include (but are not limited to):

| Component | Reusable Wrapper |
|---|---|
| `Button` | `AppButton` |
| `Text` | `AppText` |
| `Segment` | `AppSegment` |
| `TextEditor` | `AppTextEditor` |
| `TextField` | `AppTextField` |

- These components should expose configurable properties (style, font, color, etc.) to remain flexible.

---

## Appearance

- The project must fully support both **Light Mode** and **Dark Mode**.
- All colors must be defined using **Color Assets** or a centralized color configuration (see [Centralized Configuration](#centralized-configuration)) so they respond correctly to the system appearance.

---

## Splash Screen

- A **Splash Screen** must be created for both **iPhone** and **iPad**.
- The splash screen should be implemented in a way that supports platform-specific layouts if needed.

---

## Data Storage

- For storing simple key-value data, create a **`UserData` singleton** (or another appropriately named class/struct) backed by `UserDefaults`.
- All stored properties must be accessed via explicit **`get` and `set` methods** to ensure encapsulation.

```swift
final class UserData {
    static let shared = UserData()
    private init() {}

    var authToken: String? {
        get { UserDefaults.standard.string(forKey: "authToken") }
        set { UserDefaults.standard.set(newValue, forKey: "authToken") }
    }
}
```

---

## Manager Classes

- Any feature or logic that needs to be **triggered or handled from multiple places** must be centralized in a dedicated **Manager** class, struct, or actor.
- Examples: `NetworkManager`, `PermissionManager`, `NotificationManager`.
- Managers must follow the Single Responsibility Principle (see [SOLID Principles](#solid-principles)).

---

## Swift Version & Syntax

- The project must use **Swift 6 or later**.
- Outdated, deprecated, or downgraded syntax must **not** be used.
- Always prefer modern Swift idioms (e.g., `if let`, `guard let`, structured concurrency, `@Observable`, etc.).

---

## Combine Usage

- Use **`PassthroughSubject`** and **`CurrentValueSubject`** where they add clear value — for example, event streams, state broadcasting, or one-time signals.
- Do not overuse Combine where simpler solutions (e.g., `@State`, `@Published`) suffice.

```swift
let errorSubject = PassthroughSubject<Error, Never>()
let isLoadingSubject = CurrentValueSubject<Bool, Never>(false)
```

---

## Performance Optimization

- Heavy computations must **never run on the main thread**. Offload work using `Task`, `async/await`, or background actors.
- Avoid unnecessary view re-renders by using `@State`, `@StateObject`, and `equatable` view optimizations appropriately.
- Use lazy loading (`LazyVStack`, `LazyHStack`) for large lists.

---

## Dynamic Font Sizing

- Dynamic font sizing must be properly supported throughout the app.
- Never use hardcoded, fixed font sizes without accommodating Dynamic Type.
- All custom font usage must go through a dedicated **`Font` extension** — do not reference font names or sizes directly in views.
- All methods in the `Font` extension must be prefixed with **`app`** (e.g., `appRegular`, `appBold`) to clearly distinguish them from native SwiftUI system fonts.
- The `Font` extension is the single source of truth for all font definitions and must return fonts that scale with the user's accessibility font size settings using `Font.custom(_:size:relativeTo:)`.

Font sizes must be fully dynamic — they must automatically scale with the user's accessibility settings using `@ScaledMetric`. Base sizes are defined once in the `Font` extension itself, and `@ScaledMetric` is used inside each view to get the scaled value at runtime:

```swift
/// Centralized extension for all custom font usage.
/// Prefix `app` distinguishes these from native SwiftUI system fonts.
/// Base sizes are defined here; views use @ScaledMetric to scale them dynamically.
extension Font {
    enum AppFontSize {
        static let small: CGFloat = 12
        static let body: CGFloat = 14
        static let medium: CGFloat = 16
        static let large: CGFloat = 18
        static let title: CGFloat = 22
    }

    static func appRegular(size: CGFloat, relativeTo style: Font.TextStyle = .body) -> Font {
        .custom("YourFont-Regular", size: size, relativeTo: style)
    }

    static func appBold(size: CGFloat, relativeTo style: Font.TextStyle = .body) -> Font {
        .custom("YourFont-Bold", size: size, relativeTo: style)
    }

    static func appSemiBold(size: CGFloat, relativeTo style: Font.TextStyle = .body) -> Font {
        .custom("YourFont-SemiBold", size: size, relativeTo: style)
    }
}
```

Usage in views — `@ScaledMetric` scales the base size dynamically at runtime according to the user's accessibility font size preference:

```swift
struct SomeView: View {
    @ScaledMetric(relativeTo: .body) private var fontSize = Font.AppFontSize.medium

    var body: some View {
        Text("Hello")
            .font(.appRegular(size: fontSize))
    }
}
```

---

## Centralized Configuration

- **App colors**, **font names**, **spacing constants**, and other static values must be defined in a single centralized location**.
- Recommended structure:

```
Core/
└── Config/
    ├── AppColors.swift
    ├── AppFonts.swift
    └── AppConstants.swift
```

```swift
// AppColors.swift
extension Color {
    static let primaryBackground = Color("PrimaryBackground")
    static let accentGreen = Color("AccentGreen")
}
```

---

## SOLID Principles

The project must adhere to all five SOLID principles:

| Principle | Description |
|---|---|
| **S** – Single Responsibility | Each class/struct/view has one clear responsibility. |
| **O** – Open/Closed | Components are open for extension, closed for modification. |
| **L** – Liskov Substitution | Subtypes must be substitutable for their base types. |
| **I** – Interface Segregation | Prefer small, focused protocols over large, general ones. |
| **D** – Dependency Inversion | Depend on abstractions (protocols), not concrete implementations. |

---

## Navigation

### NavigationStack

- The project must use **`NavigationStack`** for all navigation.
- **`NavigationView` must not be used** anywhere in the codebase.

### Custom Navigation Bar

- A **custom navigation bar component** must be created with the following configurable properties:
  - **Left button** (e.g., back, close, custom action)
  - **Right button** (e.g., save, settings, custom action)
  - **Center title** (text or custom view)

```swift
AppNavigationBar(
    title: "Profile",
    leftButton: .back { router.pop() },
    rightButton: .text("Save") { viewModel.save() }
)
```

### Centralized Routing

- All **navigation routes must be defined in a single centralized file** (e.g., `AppRouter.swift` or `Routes.swift`).
- Use an `enum` conforming to `Hashable` to represent all possible destinations.

```swift
enum AppRoute: Hashable {
    case home
    case profile
    case playerPersonalInfo(PersonalInfoDataPasser)
}
```

- Deprecated navigation APIs must **not** be used.

---

## Reusable System Components

The following UI components must be created as **reusable views**:

| Component | Description |
|---|---|
| **Loader** | Full-screen or inline activity indicator. |
| **Alert** | Configurable alert dialog supporting title, message, and actions. |
| **Empty State View** | Displayed when a list or content area has no data. |
| **Error View** | Displayed when a network or data error occurs, with a retry option. |

---

## Networking

- All API calls must use **`async/await`** instead of completion handlers.
- Use **[Alamofire](https://github.com/Alamofire/Alamofire)** as the networking library.
- Network requests must **not** run on the main thread.
- A centralized `NetworkManager` or `APIClient` must handle all requests.

```swift
func fetchUser(id: String) async throws -> User {
    try await networkManager.request(endpoint: .user(id: id))
}
```

---

## Concurrency

- The project must fully support **Swift 6 concurrency**.
- Use `async/await`, `Task`, `actor`, and `@MainActor` appropriately.
- Annotate UI-related updates with `@MainActor` to ensure they run on the main thread.
- Heavy or background work must be explicitly dispatched off the main thread.

```swift
@MainActor
func updateUI(with data: [Item]) {
    self.items = data
}
```

---

## Async Image Loading

- For loading remote images asynchronously, use **[SDWebImageSwiftUI](https://github.com/SDWebImage/SDWebImageSwiftUI)**.
- Do not use `AsyncImage` where SDWebImageSwiftUI provides a more capable alternative (caching, placeholders, transitions).

```swift
WebImage(url: URL(string: imageURL)) { image in
    image.resizable()
} placeholder: {
    ProgressView()
}
.scaledToFill()
```

---

## Package Management

- All third-party libraries must be integrated using **Swift Package Manager (SPM)**.
- Do **not** use CocoaPods or Carthage.

---

## Logging

- Debug logging must be **centralized** in a dedicated logger utility (e.g., `AppLogger`).
- Use log levels (e.g., `debug`, `info`, `warning`, `error`) to categorize output.
- All logging calls should go through `AppLogger` — avoid raw `print()` statements across the codebase.

```swift
AppLogger.debug("User fetched: \(userId)")
AppLogger.error("Network failure: \(error.localizedDescription)")
```

---

## Permission Handling

- All system permission requests (camera, location, notifications, etc.) must be managed through a centralized **`PermissionManager`**.
- The manager should expose clean async APIs for requesting and checking permissions.

```swift
let granted = await PermissionManager.shared.requestCamera()
```

---

## Code Quality

- **Force unwrap (`!`) must not be used** anywhere in the codebase. Use `guard let`, `if let`, or provide safe fallbacks.
- **Duplicate code must be avoided** — extract shared logic into reusable utilities, extensions, or components.
- Code must be clean, readable, and maintainable.

---

## Documentation

- Proper **documentation comments** (`///` or `/** */`) must be added to:
  - All public types (classes, structs, enums, protocols)
  - All public or non-obvious methods and properties
  - Complex logic blocks that benefit from explanation

```swift
/// Fetches the user profile for the given user ID.
/// - Parameter id: The unique identifier of the user.
/// - Returns: A fully populated `User` object.
/// - Throws: `NetworkError` if the request fails.
func fetchUser(id: String) async throws -> User
```

### Documentation Policy

**Do NOT create any `.md` files** for documentation purposes (aside from this `README.md`). No per-feature READMEs, no per-folder documentation files, no NETWORK.md, no CONFIG.md, no PROJECT_STRUCTURE.md, etc.

All documentation must be done through **code comments** (`///`) in the actual Swift files where needed.

### Project Folder Structure

The project must follow this folder structure:

```
ProjectName/
├── App/
│   ├── ProjectNameApp.swift           # @main entry point
│   └── ContentView.swift              # Root view
├── Core/
│   ├── Network/
│   │   ├── NetworkManager.swift       # Alamofire-based API client
│   │   ├── APIEndpoint.swift          # Endpoint definitions
│   │   └── NetworkError.swift         # Custom network errors
│   ├── Config/
│   │   ├── AppColors.swift            # Color extension
│   │   ├── AppFonts.swift             # Font extension with @ScaledMetric support
│   │   └── AppConstants.swift         # All static constants
│   ├── Router/
│   │   ├── AppRoute.swift             # Route enum (Hashable)
│   │   └── Router.swift               # Navigation router class
│   ├── Managers/
│   │   ├── PermissionManager.swift    # Centralized permission handling
│   │   ├── AppLogger.swift            # Centralized logging
│   │   └── UserData.swift             # UserDefaults singleton
│   └── Extensions/
│       ├── View+Extensions.swift
│       ├── String+Extensions.swift
│       └── ...
├── Components/
│   ├── AppButton.swift                # Reusable button
│   ├── AppTextField.swift             # Reusable text field
│   ├── AppTextEditor.swift            # Reusable text editor
│   ├── AppText.swift                  # Reusable text
│   ├── AppSegment.swift               # Reusable segment control
│   ├── AppNavigationBar.swift         # Custom navigation bar
│   ├── Loader.swift                   # Loading indicator
│   ├── AlertView.swift                # Custom alert
│   ├── EmptyStateView.swift           # Empty state
│   └── ErrorView.swift                # Error state with retry
├── Features/
│   ├── Login/
│   │   ├── LoginModel.swift           # Data models
│   │   ├── LoginView.swift            # UI only
│   │   └── LoginViewModel.swift       # Business logic
│   ├── Home/
│   │   ├── HomeModel.swift
│   │   ├── HomeView.swift
│   │   └── HomeViewModel.swift
│   └── Profile/
│       ├── ProfileModel.swift
│       ├── ProfileView.swift
│       └── ProfileViewModel.swift
└── Resources/
    ├── Assets.xcassets            # Images, colors
    ├── Fonts/                     # Custom font files (.ttf, .otf)
    └── Localizable.strings        # Localization
```

**Dependency Flow:**

```
Features → Components → Core → Resources
```

- Features can use Components and Core.
- Components can use Core.
- Core and Components must never import Features.
- Each layer is independent and reusable.

---

## Data Passing Between Modules

When passing data from one module/screen to another, use the following pattern:

1. Define a **`DataPasser` struct** conforming to `Hashable` that carries the required data along with the originating screen context.
2. The **ViewModel** accepts the `DataPasser` in its initializer.
3. The **View** is initialized with a `@StateObject` ViewModel.
4. The **router** maps the route enum case to the corresponding View, injecting the `DataPasser`.

### Example

```swift
// 1. DataPasser
struct PersonalInfoDataPasser: Hashable {
    var fromScreen: ScreenName = .none
    var userId: String?
}

// 2. ViewModel
class PersonalInfoViewModel: ObservableObject {
    var dataPasser: PersonalInfoDataPasser

    init(dataPasser: PersonalInfoDataPasser) {
        self.dataPasser = dataPasser
        setup()
    }
}

// 3. View
struct PersonalInfoView: View {
    @StateObject var viewModel: PersonalInfoViewModel
}

// 4. Router mapping
case .playerPersonalInfo(let data):
    PersonalInfoView(viewModel: PersonalInfoViewModel(dataPasser: data))

// 5. Pushing to a new screen
case .personalInfo:
    let dataPasser = PersonalInfoDataPasser(
        fromScreen: viewModel.dataPasser.fromScreen,
        userId: viewModel.dataPasser.userId
    )
    router.push(.playerPersonalInfo(dataPasser))
```

---

*These guidelines are intended to be followed strictly throughout the project lifecycle to maintain a high-quality, scalable, and consistent codebase.*
