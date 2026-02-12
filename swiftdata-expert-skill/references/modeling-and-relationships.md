# SwiftData Modeling and Relationships Reference

## Scope
Model data invariants directly in type definitions using `@Model`, `@Attribute`, and `@Relationship`. Relationship rules should mirror business rules, not UI convenience.

## Do
- Use `@Attribute(.unique)` only when duplicates are truly invalid
- Define delete rules explicitly to reflect ownership semantics
- Add inverse relationships for bidirectional navigation when needed
- Keep enums and value types small, explicit, and stable

## Don't
- Don't rely on implicit relationship behavior for critical cleanup logic
- Don't apply `.unique` to fields with unstable external values
- Don't encode ownership policy only in comments
- Don't create deep relationship graphs without clear traversal boundaries

## Wrong vs Correct

```swift
import SwiftData

@Model
final class Team {
    @Attribute(.unique) var code: String
    @Relationship(deleteRule: .cascade, inverse: \Member.team) var members: [Member]

    init(code: String, members: [Member] = []) {
        self.code = code
        self.members = members
    }
}

@Model
final class Member {
    var name: String
    @Relationship(deleteRule: .nullify) var team: Team?

    init(name: String, team: Team? = nil) {
        self.name = name
        self.team = team
    }
}

// WRONG: Delete behavior is not explicit, ownership is unclear.
@Model
final class Board {
    var title: String
    var cards: [Card]

    init(title: String, cards: [Card] = []) {
        self.title = title
        self.cards = cards
    }
}

@Model
final class Card {
    var text: String

    init(text: String) {
        self.text = text
    }
}

// CORRECT: Relationship and delete behavior are explicit.
@Model
final class Project {
    @Attribute(.unique) var id: UUID
    var name: String
    @Relationship(deleteRule: .cascade, inverse: \Task.project) var tasks: [Task]

    init(id: UUID = UUID(), name: String, tasks: [Task] = []) {
        self.id = id
        self.name = name
        self.tasks = tasks
    }
}

@Model
final class Task {
    var title: String
    @Relationship(deleteRule: .nullify) var project: Project?

    init(title: String, project: Project? = nil) {
        self.title = title
        self.project = project
    }
}
```

## Enum and Value-Type Guidance
- Prefer small enums with stable raw values
- Keep frequently changing or large payloads outside hot model fields
- Avoid storing derived display strings; compute them at render time

## Pitfalls
- Over-aggressive uniqueness can turn valid user edits into save failures
- Missing inverse definitions can make updates look inconsistent across screens
- Incorrect delete rules often surface as orphan records or unintended cascades
