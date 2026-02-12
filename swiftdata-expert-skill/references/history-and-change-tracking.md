# SwiftData History and Change Tracking

## Scope
Use SwiftData history for incremental processing with durable checkpoints. Prioritize idempotent consumers, bounded batches, and strict checkpoint write ordering.

## Do
- Process history in bounded windows
- Keep checkpoint token storage durable (disk/db/keychain, not memory only)
- Apply side effects idempotently per transaction
- Write the new checkpoint only after downstream side effects are committed
- Separate ingestion workers from UI presentation code

## Don't
- Don't re-scan full history on every launch
- Don't persist a checkpoint before work success is durable
- Don't compare tokens using ad-hoc ordering logic
- Don't process unlimited windows in one pass

## Wrong vs Correct

```swift
import SwiftData

enum HistoryProcessingError: Error {
    case checkpointNotInWindow
}

// WRONG: Updates checkpoint too early and uses weak token filtering.
func wrongHistoryReplay(container: ModelContainer, lastToken: HistoryToken?) throws -> HistoryToken? {
    let descriptor = HistoryDescriptor<ModelTransaction>(fetchLimit: 500)
    let transactions = try container.mainContext.fetchHistory(descriptor)

    // Incorrect: token inequality is not a safe progression strategy.
    let newTransactions = transactions.filter { $0.token != lastToken }
    let optimisticToken = newTransactions.last?.token

    // Incorrect: checkpoint moves before side effects are durable.
    return optimisticToken
}

// CORRECT: Process incrementally and checkpoint after durable work.
func processHistoryWindow(
    container: ModelContainer,
    lastToken: HistoryToken?,
    batchSize: Int = 100
) throws -> HistoryToken? {
    let descriptor = HistoryDescriptor<ModelTransaction>(fetchLimit: batchSize)
    let transactions = try container.mainContext.fetchHistory(descriptor)
    guard !transactions.isEmpty else { return lastToken }

    let startIndex: Int
    if let lastToken {
        // In production, prefer an SDK query anchored at `lastToken` when available.
        guard let checkpointIndex = transactions.lastIndex(where: { $0.token == lastToken }) else {
            throw HistoryProcessingError.checkpointNotInWindow
        }
        startIndex = checkpointIndex + 1
    } else {
        startIndex = transactions.startIndex
    }

    var nextToken = lastToken
    for transaction in transactions[startIndex...] {
        try applyIdempotentSideEffects(transaction)
        nextToken = transaction.token
    }

    // Persist nextToken only after side effects are committed.
    return nextToken
}

func applyIdempotentSideEffects(_ transaction: ModelTransaction) throws {
    // Use transaction token (or equivalent unique transaction key)
    // to deduplicate side effects in downstream storage.
    _ = transaction.changeCount
}
```

## Checkpoint Durability Contract
- Scope checkpoint by consumer and store (for example: `widget-sync:primary-store`)
- Store checkpoint atomically with downstream commit state
- On failure, keep previous checkpoint and retry safely
- If checkpoint cannot be found in the fetched window, trigger controlled backfill or widen window

## Operational Pattern
1. Load last durable checkpoint
2. Fetch bounded history window
3. Process transactions in commit order
4. Commit side effects
5. Persist new checkpoint atomically
6. Repeat until window is empty or execution budget is reached

## Pitfalls
- Early checkpoint writes can permanently skip unprocessed changes
- Non-idempotent consumers create duplicate side effects on retry
- Missing checkpoint scoping causes cross-consumer corruption
- Oversized windows can block launch and increase memory pressure
