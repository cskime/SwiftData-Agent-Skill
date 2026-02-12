# SwiftData History and Change Tracking

## Scope
Use SwiftData history to process incremental transactions with durable checkpoints. This is essential when app targets multiple processes (widgets/extensions) or synchronization pipelines.

## Do
- Read transactions in bounded windows
- Persist history token after successful processing
- Build idempotent processors so replay is safe
- Keep history ingestion separate from view rendering

## Don't
- Don't scan full history from the beginning on every launch
- Don't update checkpoint token before work is committed
- Don't couple token storage to volatile memory only
- Don't process unlimited transaction windows in one pass

## Wrong vs Correct

```swift
import SwiftData

// WRONG: No checkpointing, replays entire history every time.
func wrongHistoryReplay(container: ModelContainer) throws {
    let descriptor = HistoryDescriptor<ModelTransaction>(fetchLimit: 500)
    let transactions = try container.mainContext.fetchHistory(descriptor)

    for transaction in transactions {
        // expensive work repeated on every launch
        _ = transaction.changes
    }
}

// CORRECT: Use incremental processing and durable token storage.
func processHistory(
    container: ModelContainer,
    lastToken: HistoryToken?
) throws -> HistoryToken? {
    let descriptor = HistoryDescriptor<ModelTransaction>(fetchLimit: 100)
    let transactions = try container.mainContext.fetchHistory(descriptor)
    let newTransactions = transactions.filter { transaction in
        isAfterCheckpoint(transaction.token, checkpoint: lastToken)
    }
    guard !newTransactions.isEmpty else { return lastToken }

    for transaction in newTransactions {
        // idempotent processing based on transaction changes
        _ = transaction.changeCount
    }

    return newTransactions.last?.token
}

func isAfterCheckpoint(_ token: HistoryToken, checkpoint: HistoryToken?) -> Bool {
    guard let checkpoint else { return true }
    // Implement app-specific token comparison logic here.
    return token != checkpoint
}
```

## Token Handling Guidance
- Write token only after downstream side effects are durable
- Keep token scope explicit (per store/per consumer)
- Guard against duplicate processing by design, not by luck

## Pitfalls
- Token loss forces expensive full reprocessing and can duplicate side effects
- Large history windows can block launch workflows
- Mixing history handling with UI state often causes lifecycle race conditions
