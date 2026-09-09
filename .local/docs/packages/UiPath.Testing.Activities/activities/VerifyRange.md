# Verify Range

Verifies that a value falls within (or outside) a specified range defined by a lower and upper limit. If the assertion fails, the test case is marked as failed.

**Class:** `UiPath.Testing.Activities.VerifyRange`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Verification

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
```

---

## Input

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `Expression` | `InArgument` (non-generic) | Yes | — | The value to verify against the range. **Non-generic** — element syntax with `x:TypeArguments` required. |
| `LowerLimit` | `InArgument` (non-generic) | Yes | — | The lower bound of the range (inclusive). **Non-generic.** |
| `UpperLimit` | `InArgument` (non-generic) | Yes | — | The upper bound of the range (inclusive). **Non-generic.** |
| `VerificationType` | `VerificationType` | Yes | `IsWithin` | Whether the value must be inside or outside the range. In XAML this is written as `"is within"` / `"is not within"` — see [Enum](#enum-verificationtype). |
| `ContinueOnFailure` | `InArgument<Boolean>` | No | `true` | When `false`, a failing assertion throws `TestingActivitiesException` and aborts the test case. When `true`, the test case is marked failed but execution continues. |

## Output

| Property | Type | Description |
|----------|------|-------------|
| `Result` | `OutArgument<Boolean>` | `true` if the assertion passed. |

## Messages

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `AlternativeVerificationTitle` | `InArgument<String>` | No | *(DisplayName)* | Overrides the verification title reported to Orchestrator / the test report. |
| `OutputMessageFormat` | `InArgument<String>` | No | *(project setting)* | Custom format string for the result message. Supported placeholders: `{LeftExpression}`, `{LeftExpressionText}`, `{RightExpression}`, `{RightExpressionText}`, `{Result}`, `{Operator}`. |

## Common

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `TakeScreenshotInCaseOfFailingAssertion` | `InArgument<Boolean>` | No | `false` | If `true`, takes a screenshot when the assertion fails. |
| `TakeScreenshotInCaseOfSucceedingAssertion` | `InArgument<Boolean>` | No | `false` | If `true`, takes a screenshot when the assertion passes. |

---

## XAML syntax: non-generic `InArgument`

`Expression`, `LowerLimit` and `UpperLimit` are declared as **non-generic** `InArgument`, so they carry no implicit type. Setting them as attributes fails to load the file:

```xml
<!-- WRONG — fails with: Set property 'UiPath.Testing.Activities.VerifyRange.Expression' threw an exception. -->
<uta:VerifyRange Expression="[score]" LowerLimit="[1]" UpperLimit="[100]" ... />
```

Use element syntax with an explicit `x:TypeArguments` on each. Give all three the **same** type — one the comparison supports (`x:Double`, `x:Int32`, `x:Decimal`, `s:DateTime`, …); mixed types are not checked at design time and can fail when the comparison runs. Convert first if the source variable is a different numeric type — e.g. `[CDbl(orderTotal)]` for a `Decimal` variable compared against `x:Double` bounds.

---

## Validation Constraints

`Verify Range` cannot be placed inside a **Verify Control Attribute**'s `ActivityToTest` body — it declares a `HasNoParent<VerifyControlAttribute>` constraint and validation fails if nested there.

---

## Enum: `VerificationType`

| Value | XAML literal | Description |
|-------|--------------|-------------|
| `IsWithin` | `"is within"` | Asserts the expression is within `[LowerLimit, UpperLimit]` (inclusive on both ends). |
| `IsNotWithin` | `"is not within"` | Asserts the expression is outside the range. |

> **Important:** `VerificationType` is serialized through a description-based `TypeConverter`, so the XAML attribute takes the **literal** in the middle column — *not* the C# enum member name. Writing `VerificationType="IsNotWithin"` does **not** fail validation; it silently falls back to `IsWithin`, producing a test that passes for the wrong reason. Always write `"is within"` or `"is not within"`.

---

## Project Settings

| Property | Setting Key | Description |
|----------|-------------|-------------|
| `OutputMessageFormat` | `VerifyActivitiesOutputFormat` / `VerifyRangeOutputFormat` | Default message format for all Verify Range instances in the project. |

---

## XAML Example

```xml
<!-- Assert orderTotal is within [0, 10000] -->
<uta:VerifyRange
    DisplayName="Verify orderTotal is within [0, 10000]"
    ContinueOnFailure="True"
    VerificationType="is within"
    TakeScreenshotInCaseOfFailingAssertion="True"
    TakeScreenshotInCaseOfSucceedingAssertion="False">
  <uta:VerifyRange.Expression>
    <InArgument x:TypeArguments="x:Double">[CDbl(orderTotal)]</InArgument>
  </uta:VerifyRange.Expression>
  <uta:VerifyRange.LowerLimit>
    <InArgument x:TypeArguments="x:Double">[0.0]</InArgument>
  </uta:VerifyRange.LowerLimit>
  <uta:VerifyRange.UpperLimit>
    <InArgument x:TypeArguments="x:Double">[10000.0]</InArgument>
  </uta:VerifyRange.UpperLimit>
</uta:VerifyRange>

<!-- Assert balance is OUTSIDE [-10, 0] -->
<uta:VerifyRange
    DisplayName="Verify Positive Balance"
    ContinueOnFailure="True"
    VerificationType="is not within"
    Result="[balanceOk]">
  <uta:VerifyRange.Expression>
    <InArgument x:TypeArguments="x:Double">[balance]</InArgument>
  </uta:VerifyRange.Expression>
  <uta:VerifyRange.LowerLimit>
    <InArgument x:TypeArguments="x:Double">[-10.0]</InArgument>
  </uta:VerifyRange.LowerLimit>
  <uta:VerifyRange.UpperLimit>
    <InArgument x:TypeArguments="x:Double">[0.0]</InArgument>
  </uta:VerifyRange.UpperLimit>
</uta:VerifyRange>
```
