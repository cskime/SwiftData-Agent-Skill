# SwiftData Concurrency and Context Isolation

## Scope
Treat each context as actor-bound mutable state. Isolate write workflows and communicate across actors using IDs or value snapshots.

## Do
- Keep UI mutations on main actor-owned context boundaries
- Perform import and maintenance writes in dedicated model actors
- Pass identifiers between actors, then re-fetch in destination context
- Save at explicit workflow milestones

## Don't
- Don't mutate the same model object from concurrent tasks
- Don't pass mutable model instances into detached/background tasks
- Don't hide cross-actor writes behind utility methods
- Don't mix UI rendering and background mutation logic

## Wrong vs Correct

```swift
import SwiftData

@Model
final class Job {
    var title: String
    var processed: Bool

    init(title: String, processed: Bool = false) {
        self.title = title
        self.processed = processed
    }
}

// WRONG: Reuses UI-owned model instance in detached task.
func wrongProcess(job: Job, context: ModelContext) {
    Task.detached {
        job.processed = true
        try? context.save()
    }
}

// CORRECT: Isolate writes in a model actor and re-fetch by identifier.
@ModelActor
actor JobProcessor {
    func markProcessed(jobID: PersistentIdentifier) throws {
        let descriptor = FetchDescriptor<Job>(predicate: #Predicate { $0.persistentModelID == jobID })
        guard let job = try modelContext.fetch(descriptor).first else { return }

        job.processed = true
        try modelContext.save()
    }
}
```

## Isolation Rules
- One workflow owns one mutable context boundary at a time
- A model actor should hide write details and expose intention-revealing methods
- UI layer should request writes, not orchestrate low-level mutation sequencing

## Pitfalls
- Detached writes to UI-owned state cause rare but severe consistency bugs
- Sharing mutable model references across actors can produce nondeterministic behavior
- Save storms from many tiny concurrent writes increase lock contention
