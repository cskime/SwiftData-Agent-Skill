# SwiftData Inheritance and Polymorphism

## Scope
Use inheritance for natural domain hierarchies where shared fields and polymorphic queries improve clarity. Keep hierarchies shallow and query intent explicit.

## Do
- Use a base model only when subclasses truly share stable behavior and fields
- Keep hierarchy depth small and domain-oriented
- Validate base-type and subtype query behavior in tests
- Use polymorphism where it reduces duplicate schema and query logic

## Don't
- Don't use inheritance to avoid writing explicit composition models
- Don't create deep hierarchies that obscure persistence behavior
- Don't assume subtype-only constraints are enforced at base query call sites
- Don't mix unrelated concepts into one base class

## Wrong vs Correct

```swift
import SwiftData

// WRONG: Unrelated types forced into one inheritance tree.
@Model
class Record {
    var title: String

    init(title: String) {
        self.title = title
    }
}

@Model
final class PaymentRecord: Record {
    var amount: Decimal

    init(title: String, amount: Decimal) {
        self.amount = amount
        super.init(title: title)
    }
}

@Model
final class WorkoutRecord: Record {
    var durationMinutes: Int

    init(title: String, durationMinutes: Int) {
        self.durationMinutes = durationMinutes
        super.init(title: title)
    }
}

// CORRECT: Base type expresses meaningful shared work-item semantics.
@Model
class WorkItem {
    @Attribute(.unique) var id: UUID
    var createdAt: Date
    var title: String

    init(id: UUID = UUID(), createdAt: Date = .now, title: String) {
        self.id = id
        self.createdAt = createdAt
        self.title = title
    }
}

@Model
final class Bug: WorkItem {
    var severity: Int

    init(title: String, severity: Int) {
        self.severity = severity
        super.init(title: title)
    }
}

@Model
final class Feature: WorkItem {
    var specURL: URL?

    init(title: String, specURL: URL?) {
        self.specURL = specURL
        super.init(title: title)
    }
}
```

## Query Strategy
- Use base-type queries for shared workflows (timeline, activity feed)
- Use subtype queries for specialized workflows (severity triage, feature planning)
- Keep predicate logic explicit when mixing base and subtype data

## Pitfalls
- Deep hierarchy chains make migration and review harder
- Misusing inheritance for convenience causes confusing model boundaries
- Ambiguous subtype semantics produce brittle query assumptions
