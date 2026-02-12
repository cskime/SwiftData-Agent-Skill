# SwiftData Performance Patterns

## Scope
Optimize fetch and write workloads for large datasets. Keep startup paths bounded, reduce unnecessary relationship fan-out, and use batch processing for maintenance tasks.

## Do
- Bound high-cardinality fetches with limits and focused predicates
- Use batch-oriented loops for import, cleanup, and archival tasks
- Reduce over-fetching by querying only data needed for the current workflow
- Profile memory usage under realistic data volumes

## Don't
- Don't fetch entire tables for simple counters or status badges
- Don't save after every single item in bulk import jobs
- Don't keep large object graphs alive longer than needed
- Don't run full-store maintenance work on the main actor

## Wrong vs Correct

```swift
import SwiftData

@Model
final class LogEntry {
    var level: Int
    var createdAt: Date

    init(level: Int, createdAt: Date = .now) {
        self.level = level
        self.createdAt = createdAt
    }
}

// WRONG: Unbounded fetch and per-item save.
func wrongBulkInsert(context: ModelContext, count: Int) throws {
    _ = try context.fetch(FetchDescriptor<LogEntry>())

    for idx in 0..<count {
        context.insert(LogEntry(level: idx % 5))
        try context.save()
    }
}

// CORRECT: Batch inserts and bounded fetches.
func correctBulkInsert(context: ModelContext, count: Int) throws {
    let batchSize = 500

    for idx in 0..<count {
        context.insert(LogEntry(level: idx % 5))

        if (idx + 1).isMultiple(of: batchSize) {
            try context.save()
        }
    }

    try context.save()
}

func fetchRecentErrors(context: ModelContext) throws -> [LogEntry] {
    var descriptor = FetchDescriptor<LogEntry>(
        predicate: #Predicate { $0.level >= 3 },
        sortBy: [SortDescriptor(\LogEntry.createdAt, order: .reverse)]
    )
    descriptor.fetchLimit = 200
    return try context.fetch(descriptor)
}
```

## Memory Pressure Guidance
- Process large datasets in windows, not all at once
- Favor predictable sorted windows for pagination/retry safety
- Clear temporary buffers between batch phases

## Pitfalls
- Per-item save loops create severe write amplification
- Unbounded startup fetches can block first render
- Relationship-heavy projections can explode memory usage unexpectedly
