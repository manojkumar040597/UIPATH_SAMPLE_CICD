# Get Test Data Queue Items

Retrieves all items (or a filtered/paginated subset) from a Test Data Queue in Orchestrator. Returns a list of `TestDataQueueItem` objects that can be iterated or passed to **Delete Test Data Queue Items**.

**Class:** `UiPath.Testing.Activities.GetTestDataQueueItems`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Test Data Queues

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
```

---

## Input

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `QueueName` | `InArgument<String>` | Yes | The name of the Test Data Queue to retrieve items from. |

## Options

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `TestDataQueueItemStatus` | `TestDataQueueItemStatus` | No | `All` | Filter items by consumption status. See enum values below. |
| `IdFilter` | `InArgument<String>` | No | — | Filter items by a specific item ID. |

## Pagination

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `Skip` | `InArgument<Int32?>` | No | — | Number of items to skip (offset). Used for paging through large queues. |
| `Top` | `InArgument<Int32?>` | No | — | Maximum number of items to return. |

## Output

| Property | Type | Description |
|----------|------|-------------|
| `TestDataQueueItems` | `OutArgument<List<TestDataQueueItem>>` | The retrieved list of test data queue items. Each item contains its field data and metadata (ID, status, etc.). |

## Common

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `FolderPath` | `InArgument<String>` | No | `null` | Orchestrator folder path. If not set, uses the folder of the running process. |
| `ContinueOnError` | `InArgument<Boolean>` | No | `false` | If `true`, execution continues when the activity throws an error. |
| `TimeoutMs` | `InArgument<Int32>` | No | — | Timeout in milliseconds for the Orchestrator API call. |

---

## Enum: `TestDataQueueItemStatus`

| Value | XAML literal | Description |
|-------|--------------|-------------|
| `All` | `"All"` | Return all items regardless of consumption status. |
| `OnlyConsumed` | `"Items Consumed"` | Return only items that have been marked as consumed. |
| `OnlyNotConsumed` | `"Items Not Consumed"` | Return only items that have not yet been consumed. |

> **Important:** `TestDataQueueItemStatus` is serialized through a description-based `TypeConverter`, so the XAML attribute takes the **literal** in the middle column — *not* the C# enum member name. Writing `TestDataQueueItemStatus="OnlyNotConsumed"` does not fail validation; it silently falls back to `All`, so the activity returns consumed items too.

---

## String-valued attributes

`QueueName`, `IdFilter` and `FolderPath` are `InArgument<String>`. Write them as bare text or as a bracketed expression — **never** as `&quot;…&quot;` without brackets, which stores the quote characters as part of the value:

```xml
QueueName="[&quot;MyTestQueue&quot;]"     <!-- OK: VB expression -->
QueueName="MyTestQueue"                   <!-- OK: bare literal -->
QueueName="&quot;MyTestQueue&quot;"       <!-- WRONG: name becomes "MyTestQueue" including quotes -->
```

---

## XAML Example

```xml
<!-- Get all unconsumed items -->
<uta:GetTestDataQueueItems
  DisplayName="Get Test Data Queue Items"
  QueueName="[&quot;MyTestQueue&quot;]"
  TestDataQueueItemStatus="Items Not Consumed"
  TestDataQueueItems="[queueItems]"
  ContinueOnError="False" />

<!-- Paginated: get first 50 items, skipping 0 -->
<uta:GetTestDataQueueItems
  DisplayName="Get Test Data Queue Items (Page 1)"
  QueueName="[&quot;MyTestQueue&quot;]"
  TestDataQueueItemStatus="All"
  Top="[50]"
  Skip="[0]"
  TestDataQueueItems="[queueItems]" />
```
