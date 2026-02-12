# SwiftData Custom Data Store Reference

## Scope
Use `DataStore` only when product requirements cannot be met by the default store. Keep custom store concerns isolated from domain models and UI code.

## Do
- Start with default store and document the exact gap before custom store adoption
- Keep custom store adapter boundaries narrow and testable
- Preserve stable model identity mapping across snapshots
- Add integration tests for fetch/write/delete/transaction semantics

## Don't
- Don't introduce custom store for stylistic reasons
- Don't leak backend-specific details into view or model layers
- Don't skip transaction and error propagation tests
- Don't assume custom store behavior matches default store edge cases

## Wrong vs Correct

```swift
import SwiftData

@Model
final class Note {
    @Attribute(.unique) var id: UUID
    var body: String

    init(id: UUID = UUID(), body: String) {
        self.id = id
        self.body = body
    }
}

// WRONG: Backend concerns leak through app layers.
struct NoteService {
    func writeRawSQL(_ statement: String) {
        // hard-coded backend detail in domain service
    }
}

// CORRECT: Keep custom persistence behind a store adapter boundary.
protocol NotePersistence {
    func fetchAll() throws -> [Note]
    func upsert(_ note: Note) throws
    func delete(id: UUID) throws
}

struct SwiftDataNoteRepository {
    let context: ModelContext

    func fetchAll() throws -> [Note] {
        try context.fetch(FetchDescriptor<Note>())
    }

    func upsert(_ note: Note) throws {
        context.insert(note)
        try context.save()
    }

    func delete(id: UUID) throws {
        let descriptor = FetchDescriptor<Note>(predicate: #Predicate { $0.id == id })
        guard let note = try context.fetch(descriptor).first else { return }
        context.delete(note)
        try context.save()
    }
}
```

## Adoption Gate
Adopt custom store only when one or more are true:
- Existing backend must remain source of truth
- Regulatory rules require non-default persistence behavior
- Cross-platform engine compatibility must be preserved
- Data locality/transport controls exceed default store capabilities

## Pitfalls
- Identity mismatches between store snapshots and in-memory models cause duplication bugs
- Implicit backend retries can reorder writes without clear conflict policy
- Under-tested custom store adapters fail on rare migration edges
