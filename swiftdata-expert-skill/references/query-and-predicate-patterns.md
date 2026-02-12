# SwiftData Query and Predicate Patterns

## Scope
Choose query APIs by workload. Use `@Query` for live UI projections and `FetchDescriptor` for workflow logic. Keep filtering and sorting at data fetch boundaries.

## Do
- Use `@Query` for views that should stay synchronized with model changes
- Use `FetchDescriptor` for imports, exports, audits, and maintenance tasks
- Push filter/sort into `#Predicate` and `SortDescriptor`
- Bound result sets for large collections

## Don't
- Don't fetch broad datasets and filter in `ForEach`
- Don't couple migration or import logic to `@Query`
- Don't leave ordering implicit for user-visible lists
- Don't run expensive derived sorting on every view update

## Wrong vs Correct

```swift
import SwiftData
import SwiftUI

@Model
final class Task {
    var title: String
    var isDone: Bool
    var updatedAt: Date

    init(title: String, isDone: Bool = false, updatedAt: Date = .now) {
        self.title = title
        self.isDone = isDone
        self.updatedAt = updatedAt
    }
}

// WRONG: Over-fetch then filter/sort in view rendering.
struct WrongTaskList: View {
    @Query private var allTasks: [Task]

    var body: some View {
        List {
            ForEach(allTasks.filter { !$0.isDone }.sorted { $0.updatedAt > $1.updatedAt }) { task in
                Text(task.title)
            }
        }
    }
}

// CORRECT: Keep filtering/sorting in query definition.
struct CorrectTaskList: View {
    @Query(
        filter: #Predicate<Task> { task in !task.isDone },
        sort: [SortDescriptor(\Task.updatedAt, order: .reverse)]
    ) private var openTasks: [Task]

    var body: some View {
        List(openTasks) { task in
            Text(task.title)
        }
    }
}

func fetchRecentDoneTasks(context: ModelContext) throws -> [Task] {
    var descriptor = FetchDescriptor<Task>(
        predicate: #Predicate<Task> { $0.isDone },
        sortBy: [SortDescriptor(\Task.updatedAt, order: .reverse)]
    )
    descriptor.fetchLimit = 100
    return try context.fetch(descriptor)
}
```

## Projection Guidance
- Prefer lightweight domain projections for reporting workflows
- Avoid hydrating deep relationship trees when only scalar fields are needed
- Keep projection logic reusable outside view bodies

## Pitfalls
- Inline filtering in UI code causes avoidable CPU spikes on large lists
- Missing deterministic sort order produces unstable result presentation
- Unbounded fetches in startup code frequently trigger memory pressure
