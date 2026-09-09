# UiPath.Testing.Activities — Activity Reference

Activities for test case authoring, test data generation, test data queue management, and assertion-based verification. Requires UiPath Studio v2020.4+ and Orchestrator v2020.4+ for queue-based activities.

**Package ID:** `UiPath.Testing.Activities`
**Platform:** Cross-platform (Windows & Linux)

---

## XML Namespace Declarations

```xml
<!-- Main activities (verification, comparison, test data queues) -->
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"

<!-- Test data generation activities -->
xmlns:utad="clr-namespace:UiPath.Testing.Activities.TestData;assembly=UiPath.Testing.Activities"

<!-- Types used in variable declarations: ComparisonRule, Difference, SemanticDifferences -->
xmlns:utam="clr-namespace:UiPath.Testing.Activities.Models;assembly=UiPath.Testing.Activities"
```

The alias names are arbitrary — only the `clr-namespace` matters. These docs use `utad` in their examples, while Studio-generated XAML commonly emits `utat` for the same test-data namespace (and `utam` for the models namespace) — both resolve identically.

To write these types unqualified inside `[...]` expressions, also add them to `TextExpression.NamespacesForImplementation`:

```xml
<x:String>UiPath.Testing.Activities</x:String>
<x:String>UiPath.Testing.Activities.Models</x:String>
<x:String>UiPath.Platform.ResourceHandling</x:String>   <!-- for LocalResource.FromPath -->
```

and `UiPath.Testing.Activities` to `TextExpression.ReferencesForImplementation`.

---

## Test Data Queues

Interact with Orchestrator Test Data Queues to supply parameterized test data to test cases.

| Activity | Class | Description |
|----------|-------|-------------|
| [Add Test Data Queue Item](activities/NewAddTestDataQueueItem.md) | `NewAddTestDataQueueItem` | Adds a single item (key-value dictionary) to a Test Data Queue. |
| [Bulk Add Test Data Queue Items](activities/BulkAddTestDataQueue.md) | `BulkAddTestDataQueue` | Adds multiple items from a DataTable to a Test Data Queue. |
| [Delete Test Data Queue Items](activities/DeleteTestDataQueueItems.md) | `DeleteTestDataQueueItems` | Deletes a list of test data queue items from Orchestrator. |
| [Get Test Data Queue Item](activities/GetTestDataQueueItem.md) | `GetTestDataQueueItem` | Retrieves and optionally consumes the next item from a queue. Returns `Dictionary<String, Object>`. |
| [Get Test Data Queue Items](activities/GetTestDataQueueItems.md) | `GetTestDataQueueItems` | Retrieves all items (or a filtered/paginated subset) from a queue. Returns `List<TestDataQueueItem>`. |

---

## Test Data

Generate synthetic test data and attach evidence to test cases.

| Activity | Class | Description |
|----------|-------|-------------|
| [Generate Address](activities/Address.md) | `Address` | Generates a random postal address as `Dictionary<String, String>`. Keys: `Country`, `City`, `State`, `StreetNumber`, `StreetName`, `PostalCode`. |
| [Generate Given Name](activities/GivenName.md) | `GivenName` | Generates a random first name. |
| [Generate Last Name](activities/LastName.md) | `LastName` | Generates a random last name. |
| [Generate Random Date](activities/RandomDate.md) | `RandomDate` | Generates a random `DateTime` within a specified range. |
| [Generate Random Number](activities/RandomNumber.md) | `RandomNumber` | Generates a random `Decimal` number with optional min, max, and decimal places. |
| [Generate Random String](activities/RandomString.md) | `RandomString` | Generates a random string of a specified length and casing (`LowerCase`, `UpperCase`, `CamelCase`, `Mixed`). |
| [Generate Random Value](activities/RandomValue.md) | `RandomValue` | Picks a random line from a `.txt` or `.csv` file and returns it as a string. |
| [Attach Document](activities/AttachDocument.md) | `AttachDocument` | Attaches a file to the current test case in Orchestrator. |

---

## Verification

Assert values, compare expressions, and compare text or PDF documents in test case workflows.

| Activity | Class | Description |
|----------|-------|-------------|
| [Verify Expression](activities/VerifyExpression.md) | `VerifyExpression` | Asserts a Boolean expression is `true`. Marks the test case as failed if not. |
| [Verify Expression with Operator](activities/VerifyExpressionWithOperator.md) | `VerifyExpressionWithOperator` | Compares two values using an operator (`=`, `<>`, `>`, `>=`, `<`, `<=`, `Contains`, `Regex-Match`). |
| [Verify Range](activities/VerifyRange.md) | `VerifyRange` | Asserts a value is within (or outside) a lower/upper bound range. |
| [Verify Control Attribute](activities/VerifyControlAttribute.md) | `VerifyControlAttribute` | **Windows only.** Wraps a single UI Automation activity and asserts one of its output properties — the read and the assertion are one activity. |
| [Create Comparison Rule](activities/CreateComparisonRule.md) | `CreateComparisonRule` | Creates a `ComparisonRule` (regex or wildcard) for excluding dynamic sections in text/PDF comparisons. |
| [Compare Text](activities/CompareText.md) | `CompareText` | Compares two text strings at line/word/character granularity. Supports rules to exclude dynamic content and optional Autopilot semantic interpretation. |
| [Compare PDF Documents](activities/ComparePdfDocuments.md) | `ComparePdfDocuments` | Compares two PDF files for text and image equivalence. Supports comparison rules and Autopilot semantic interpretation. |

---

## XAML Authoring Rules

Four traps account for most invalid hand-written XAML in this package. Each is verified against Studio validation.

### 1. Non-generic `InArgument` properties need element syntax

Six properties are declared as **non-generic** `InArgument`, so they carry no implicit type and **cannot** be set as XAML attributes:

| Activity | Non-generic properties |
|----------|------------------------|
| `VerifyExpressionWithOperator` | `FirstExpression`, `SecondExpression` |
| `VerifyRange` | `Expression`, `LowerLimit`, `UpperLimit` |
| `VerifyControlAttribute` | `Expression` |

```xml
<!-- WRONG — Set property '…FirstExpression' threw an exception. -->
<uta:VerifyExpressionWithOperator FirstExpression="[statusCode]" Operator="Contains" SecondExpression="&quot;OK&quot;" />

<!-- CORRECT -->
<uta:VerifyExpressionWithOperator Operator="Contains">
  <uta:VerifyExpressionWithOperator.FirstExpression>
    <InArgument x:TypeArguments="x:String">[statusCode]</InArgument>
  </uta:VerifyExpressionWithOperator.FirstExpression>
  <uta:VerifyExpressionWithOperator.SecondExpression>
    <InArgument x:TypeArguments="x:String">["OK"]</InArgument>
  </uta:VerifyExpressionWithOperator.SecondExpression>
</uta:VerifyExpressionWithOperator>
```

Every other `InArgument<T>` / `OutArgument<T>` property in the package is generic and may be written as an attribute.

### 2. Two enums serialize by description, not by member name

`VerificationType` and `TestDataQueueItemStatus` carry a description-based `TypeConverter`. Writing the C# member name **does not fail validation** — it silently falls back to the enum's default value, producing a test that passes for the wrong reason.

| Enum | Member | XAML literal |
|------|--------|--------------|
| `VerificationType` | `IsWithin` | `"is within"` |
| `VerificationType` | `IsNotWithin` | `"is not within"` |
| `TestDataQueueItemStatus` | `All` | `"All"` |
| `TestDataQueueItemStatus` | `OnlyConsumed` | `"Items Consumed"` |
| `TestDataQueueItemStatus` | `OnlyNotConsumed` | `"Items Not Consumed"` |

All other enums in the package (`Comparison`, `ComparisonType`, `ComparisonRuleType`, `Case`) serialize by member name: `Operator="RegexMatch"`, `ComparisonType="Line"`, `Case="UpperCase"`.

### 3. String literals need brackets or nothing — never bare `&quot;`

For an `InArgument<String>` attribute, `&quot;value&quot;` stores the **quote characters** as part of the value. Use a bracketed VB expression (what Studio emits) or plain text:

```xml
QueueName="[&quot;MyTestQueue&quot;]"     <!-- OK -->
QueueName="MyTestQueue"                   <!-- OK -->
QueueName="&quot;MyTestQueue&quot;"       <!-- WRONG: value is "MyTestQueue" *with* quotes -->
```

This silently breaks paths, queue names, and regex patterns at run time.

### 4. PDF paths are `IResource`, not strings

`ComparePdfDocuments.BaselinePath` / `TargetPath` are `InArgument<IResource>`. A string literal is a design-time error (*"The type UiPath.Platform.ResourceHandling.IResource cannot be used as a literal"*). Build the resource explicitly:

```xml
BaselinePath="[UiPath.Platform.ResourceHandling.LocalResource.FromPath(System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), &quot;Baseline\invoice_baseline.pdf&quot;))]"
```

---

## Shared Assertion Properties

`Verify Expression`, `Verify Expression with Operator`, `Verify Range` and `Verify Control Attribute` all derive from `AssertionActivity` and share `ContinueOnFailure` (default `true`; when `false` a failing assertion throws `TestingActivitiesException`), `Result` (`OutArgument<Boolean>`), `AlternativeVerificationTitle`, and the two `TakeScreenshot*` inputs — each activity's doc has the full table. Two cross-cutting gotchas:

- `TakeScreenshotInCaseOfFailingAssertion` defaults to **`true`** on `Verify Control Attribute` but **`false`** on the other three — set it explicitly whenever a failure screenshot is required.
- None of the four may be placed inside a `Verify Control Attribute`'s `ActivityToTest` body — each declares a `HasNoParent<VerifyControlAttribute>` constraint.

---

## Test Project Requirements

Key `project.json` fields when authoring test projects:

| `project.json` field | Value |
|----------------------|-------|
| `designOptions.outputType` | `"Tests"` — marks the project as a Test Automation project |
| `dependencies` | must include `UiPath.Testing.Activities` |
| `expressionLanguage` | decides the expression dialect — match it. In a `"VisualBasic"` project expressions use the `[expr]` bracket form and `CSharpValue` / `CSharpReference` are invalid; in a `"CSharp"` project it is the reverse. |
| `targetFramework` | `"Windows"` is required only for **Verify Control Attribute**, which is hidden in cross-platform projects |

---

## Key Types

| Type | Description |
|------|-------------|
| `TestDataQueueItem` | Represents a single item in a Test Data Queue. Contains item ID, status, and field data. |
| `ComparisonRule` | A rule (regex or wildcard) that excludes matched text from comparisons. Created by **Create Comparison Rule**. |
| `Difference` | A single diff entry with `Operation` (`Equal`, `Inserted`, `Deleted`) and `Text`. |
| `SemanticDifferences` | AI-interpreted semantic diff result from Autopilot. |
| `Comparison` | Enum of comparison operators: `Equality`, `Inequality`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`, `Contains`, `RegexMatch`. |
| `ComparisonType` | Enum of text comparison granularities: `Line`, `Word`, `Character`. |
| `VerificationType` | Enum: `IsWithin`, `IsNotWithin` (for Verify Range). |
| `TestDataQueueItemStatus` | Enum: `All`, `OnlyConsumed`, `OnlyNotConsumed` (for Get Test Data Queue Items filter). |
| `Case` | Enum of string casing: `LowerCase`, `UpperCase`, `CamelCase`, `Mixed` (for Generate Random String). |
| `ComparisonRuleType` | Enum: `RegexRule`, `WildcardRule` (for Create Comparison Rule). |
