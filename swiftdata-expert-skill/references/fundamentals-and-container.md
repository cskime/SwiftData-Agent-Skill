# SwiftData Fundamentals and Container Reference

## Scope
Define the persistence boundary with `ModelContainer` and `ModelConfiguration`. Keep container ownership explicit and stable so data behavior is predictable across app launches, previews, and tests.

## Do
- Create the primary container once at app composition boundary
- Decide early between single-configuration and multi-configuration topology
- Keep schema registration explicit in one place
- Inject container via `.modelContainer(...)` at the correct scene/root level

## Don't
- Don't create a fresh container inside frequently recreated views
- Don't mix production and in-memory configuration in the same runtime path
- Don't hide container construction in unrelated helper APIs
- Don't spread schema type lists across many files

## Wrong vs Correct

```swift
import SwiftData
import SwiftUI

@Model
final class Project {
    var name: String

    init(name: String) {
        self.name = name
    }
}

// WRONG: New container is created whenever this view is re-initialized.
struct ProjectListView: View {
    var body: some View {
        List { Text("Projects") }
            .modelContainer(for: [Project.self])
    }
}

// CORRECT: Own one shared container at app composition boundary.
@main
struct ExampleApp: App {
    private let container: ModelContainer

    init() {
        let schema = Schema([Project.self])
        let configuration = ModelConfiguration("Primary")
        do {
            container = try ModelContainer(for: schema, configurations: [configuration])
        } catch {
            fatalError("Failed to create ModelContainer: \(error)")
        }
    }

    var body: some Scene {
        WindowGroup {
            ProjectListView()
        }
        .modelContainer(container)
    }
}
```

## Single vs Multiple Configuration Decision
- Use **single configuration** when all models share the same durability and storage policy
- Use **multiple configurations** when storage concerns differ, for example:
  - user data vs cache-like data
  - sensitive models with stricter retention rules
  - separated stores for bounded synchronization domains

## Pitfalls
- Accidentally creating duplicate containers causes confusing stale reads and inconsistent saves
- Migrating one configuration but forgetting another leads to partial upgrade failures
- In-memory configuration leaking into production paths masks real persistence issues
