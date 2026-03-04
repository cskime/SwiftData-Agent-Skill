# SwiftData CloudKit Sync Compatibility Reference

## Scope
Use this guidance when SwiftData models are configured for iCloud sync through CloudKit. These constraints apply to the CloudKit-backed schema, not to local-only persistence.

## Required Constraints
- Do not use `@Attribute(.unique)` for synced model properties.
- Ensure every non-relationship stored property is either optional or has a default value.
- Ensure every relationship is optional.

## Why
CloudKit-backed sync has schema constraints that differ from local-only SwiftData usage. Violating these constraints causes container/model initialization failures when CloudKit integration is enabled.

## Wrong vs Correct

```swift
import SwiftData

// WRONG: CloudKit-incompatible model shape.
@Model
final class Project {
    @Attribute(.unique) var code: String
    var name: String
    var ownerID: UUID
    var tasks: [Task]

    init(code: String, name: String, ownerID: UUID, tasks: [Task] = []) {
        self.code = code
        self.name = name
        self.ownerID = ownerID
        self.tasks = tasks
    }
}

@Model
final class Task {
    var title: String
    var project: Project

    init(title: String, project: Project) {
        self.title = title
        self.project = project
    }
}

// CORRECT: CloudKit-compatible model shape.
@Model
final class SyncedProject {
    var code: String?
    var name: String = ""
    var ownerID: UUID = UUID()
    @Relationship(deleteRule: .cascade, inverse: \SyncedTask.project) var tasks: [SyncedTask]?

    init(code: String? = nil, name: String = "", ownerID: UUID = UUID(), tasks: [SyncedTask]? = nil) {
        self.code = code
        self.name = name
        self.ownerID = ownerID
        self.tasks = tasks
    }
}

@Model
final class SyncedTask {
    var title: String = ""
    @Relationship(deleteRule: .nullify) var project: SyncedProject?

    init(title: String = "", project: SyncedProject? = nil) {
        self.title = title
        self.project = project
    }
}
```

## Apple References
- SwiftData docs: <https://developer.apple.com/documentation/swiftdata/syncing-model-data-across-a-persons-devices>
- SwiftData docs: <https://developer.apple.com/documentation/swiftdata/modelcontainer>
- WWDC22: Evolve your Core Data schema: <https://developer.apple.com/videos/play/wwdc2022/10120/>
