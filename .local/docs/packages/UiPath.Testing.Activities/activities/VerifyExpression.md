# Verify Expression

Evaluates a Boolean expression and asserts that it is `true`. If the expression evaluates to `false`, the test case is marked as failed. Optionally captures screenshots on pass or fail and writes a formatted result message to the test report.

**Class:** `UiPath.Testing.Activities.VerifyExpression`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Verification

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
```

---

## Input

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `Expression` | `InArgument<Boolean>` | Yes | — | The Boolean expression to verify. The assertion passes when this evaluates to `true`. Generic, so the attribute form `Expression="[actual = expected]"` is valid. |
| `ContinueOnFailure` | `InArgument<Boolean>` | No | `true` | When `false`, a failing assertion throws `TestingActivitiesException` and aborts the test case. When `true`, the test case is marked failed but execution continues. |

## Output

| Property | Type | Description |
|----------|------|-------------|
| `Result` | `OutArgument<Boolean>` | `true` if the assertion passed. |

## Messages

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `AlternativeVerificationTitle` | `InArgument<String>` | No | *(DisplayName)* | Overrides the verification title reported to Orchestrator / the test report. |
| `OutputMessageFormat` | `InArgument<String>` | No | *(project setting)* | Custom format string for the result message written to the test report. Supported placeholders: `{Expression}`, `{Result}`. Example: `"{Expression} has result {Result}"`. |

## Common

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `TakeScreenshotInCaseOfFailingAssertion` | `InArgument<Boolean>` | No | `false` | If `true`, takes a screenshot when the assertion fails. |
| `TakeScreenshotInCaseOfSucceedingAssertion` | `InArgument<Boolean>` | No | `false` | If `true`, takes a screenshot when the assertion passes. |

---

## Validation Constraints

`Verify Expression` cannot be placed inside a **Verify Control Attribute**'s `ActivityToTest` body — it declares a `HasNoParent<VerifyControlAttribute>` constraint and validation fails if nested there.

---

## Project Settings

The `OutputMessageFormat` default is configurable in UiPath Studio under **Project Settings**:

| Property | Setting Key | Description |
|----------|-------------|-------------|
| `OutputMessageFormat` | `VerifyActivitiesOutputFormat` / `VerifyExpressionOutputFormat` | Default message format for all Verify Expression instances in the project. |

---

## XAML Example

```xml
<!-- Basic assertion -->
<uta:VerifyExpression
  DisplayName="Then discountedTotal = 900"
  ContinueOnFailure="True"
  Expression="[actual = expected]"
  Result="[assertionPassed]"
  TakeScreenshotInCaseOfFailingAssertion="True"
  TakeScreenshotInCaseOfSucceedingAssertion="False" />

<!-- With custom message format -->
<uta:VerifyExpression
  DisplayName="Verify Login Succeeded"
  ContinueOnFailure="True"
  Expression="[isLoggedIn]"
  OutputMessageFormat="[&quot;Login check: {Expression} → {Result}&quot;]"
  TakeScreenshotInCaseOfFailingAssertion="True" />
```
