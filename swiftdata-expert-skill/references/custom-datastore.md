# SwiftData Custom Data Store Reference

## Scope
Use custom `DataStore` only when hard product constraints cannot be met by the default store. Treat custom persistence as infrastructure with explicit contracts, not as app-layer convenience.

## Adoption Gate (must satisfy at least one)
- Existing backend must remain source of truth
- Regulatory/compliance rules require non-default persistence behavior
- Multi-platform engine compatibility requires shared backend semantics
- Data locality/transport controls exceed default store capabilities

## Do
- Document the exact requirement gap before adoption
- Keep a strict adapter boundary between domain and backend
- Preserve stable identity mapping across snapshots
- Define explicit conflict and retry policy
- Add contract tests for create/fetch/update/delete/ordering/transaction semantics

## Don't
- Don't adopt custom store for stylistic architecture preferences
- Don't leak backend details into model, view, or feature layers
- Don't assume default-store edge behavior will match custom behavior
- Don't ship without failure-mode tests (timeouts, partial writes, retries)

## Boundary Pattern

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

// Domain-facing contract (stable for app layers).
protocol NotePersistence {
    func fetchAll() async throws -> [Note]
    func upsert(_ note: Note) async throws
    func delete(id: UUID) async throws
}

// Backend-facing record contract (isolated from SwiftData models).
struct NoteRecord {
    let id: UUID
    let body: String
}

protocol BackendNoteStore {
    func fetchAll() async throws -> [NoteRecord]
    func write(_ records: [NoteRecord]) async throws
    func delete(id: UUID) async throws
}

actor NoteStoreAdapter: NotePersistence {
    private let backend: BackendNoteStore

    init(backend: BackendNoteStore) {
        self.backend = backend
    }

    func fetchAll() async throws -> [Note] {
        let records = try await backend.fetchAll()
        return records.map { Note(id: $0.id, body: $0.body) }
    }

    func upsert(_ note: Note) async throws {
        try await backend.write([NoteRecord(id: note.id, body: note.body)])
    }

    func delete(id: UUID) async throws {
        try await backend.delete(id: id)
    }
}
```

## Contract Test Matrix
- Identity stability: same backend identity maps to one model identity
- Ordering guarantees: repeated reads preserve declared sort contract
- Transaction behavior: partial failures do not silently reorder or drop writes
- Retry behavior: duplicate delivery does not create duplicate domain records
- Error mapping: backend errors map to deterministic app-level failures
- Migration compatibility: old snapshots decode and map without identity drift

## Review Checklist
- Requirement gap for custom store is written and approved
- Adapter boundary is isolated from UI/model layers
- Conflict policy (last-write-wins, merge, reject) is explicit
- Retry/idempotency policy is test-backed
- Failure telemetry and alerting paths are defined

## Pitfalls
- Identity drift between backend and SwiftData snapshots causes duplicate rows
- Hidden backend retries can reorder writes without conflict policy
- Thin test coverage misses migration and retry edge failures
