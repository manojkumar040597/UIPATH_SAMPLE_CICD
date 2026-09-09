# Verify Expression with Operator

Compares two expressions using a specified operator and asserts the comparison is true. Supports equality, inequality, relational comparisons, string containment, and regex matching. If the assertion fails, the test case is marked as failed.

**Class:** `UiPath.Testing.Activities.VerifyExpressionWithOperator`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Verification

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
```

---

## Input

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `FirstExpression` | `InArgument` (non-generic) | Yes | — | The left-hand side of the comparison (the actual value). **Non-generic** — element syntax with `x:TypeArguments` required. |
| `SecondExpression` | `InArgument` (non-generic) | Yes | — | The right-hand side of the comparison (the expected value). **Non-generic.** |
| `Operator` | `Comparison` | Yes | `Equality` | The operator used to compare the two expressions. See enum values below. |
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

`FirstExpression` and `SecondExpression` are declared as **non-generic** `InArgument`, so they carry no implicit type. Setting them as attributes fails to load the file:

```xml
<!-- WRONG — fails with: Set property 'UiPath.Testing.Activities.VerifyExpressionWithOperator.FirstExpression' threw an exception. -->
<uta:VerifyExpressionWithOperator FirstExpression="[actualTotal]" Operator="Equality" SecondExpression="[expectedTotal]" />
```

Use element syntax with an explicit `x:TypeArguments` on each. The two argument types are checked against `Operator` at design time, so they must be compatible — e.g. both `x:String` for `Contains` / `RegexMatch`, both numeric for the relational operators. Use `x:Object` on one side only when the value really is loosely typed.

---

## Validation Constraints

`Verify Expression with Operator` cannot be placed inside a **Verify Control Attribute**'s `ActivityToTest` body — it declares a `HasNoParent<VerifyControlAttribute>` constraint and validation fails if nested there.

---

## Enum: `Comparison`

| Value | Symbol | Description |
|-------|--------|-------------|
| `Equality` | `=` | Asserts `FirstExpression = SecondExpression`. |
| `Inequality` | `<>` | Asserts `FirstExpression <> SecondExpression`. |
| `GreaterThan` | `>` | Asserts `FirstExpression > SecondExpression`. |
| `GreaterThanOrEqual` | `>=` | Asserts `FirstExpression >= SecondExpression`. |
| `LessThan` | `<` | Asserts `FirstExpression < SecondExpression`. |
| `LessThanOrEqual` | `<=` | Asserts `FirstExpression <= SecondExpression`. |
| `Contains` | `Contains` | Asserts that `FirstExpression` contains `SecondExpression` (string containment). |
| `RegexMatch` | `Regex-Match` | Asserts that `FirstExpression` matches the regex pattern in `SecondExpression`. |

In XAML, `Operator` takes the **enum member name** (`Operator="RegexMatch"`), not the symbol.

---

## Project Settings

| Property | Setting Key | Description |
|----------|-------------|-------------|
| `OutputMessageFormat` | `VerifyActivitiesOutputFormat` / `VerifyExpressionWithOperatorOutputFormat` | Default message format for all instances in the project. |

---

## XAML Example

```xml
<!-- Assert actual equals expected -->
<uta:VerifyExpressionWithOperator
    DisplayName="Verify Order Total"
    ContinueOnFailure="True"
    Operator="Equality"
    TakeScreenshotInCaseOfFailingAssertion="True"
    TakeScreenshotInCaseOfSucceedingAssertion="False">
  <uta:VerifyExpressionWithOperator.FirstExpression>
    <InArgument x:TypeArguments="x:Decimal">[actualTotal]</InArgument>
  </uta:VerifyExpressionWithOperator.FirstExpression>
  <uta:VerifyExpressionWithOperator.SecondExpression>
    <InArgument x:TypeArguments="x:Decimal">[expectedTotal]</InArgument>
  </uta:VerifyExpressionWithOperator.SecondExpression>
</uta:VerifyExpressionWithOperator>

<!-- Assert string contains substring -->
<uta:VerifyExpressionWithOperator
    DisplayName="Verify statusCode contains OK"
    ContinueOnFailure="True"
    Operator="Contains"
    TakeScreenshotInCaseOfFailingAssertion="True">
  <uta:VerifyExpressionWithOperator.FirstExpression>
    <InArgument x:TypeArguments="x:String">[statusCode]</InArgument>
  </uta:VerifyExpressionWithOperator.FirstExpression>
  <uta:VerifyExpressionWithOperator.SecondExpression>
    <InArgument x:TypeArguments="x:String">["OK"]</InArgument>
  </uta:VerifyExpressionWithOperator.SecondExpression>
</uta:VerifyExpressionWithOperator>

<!-- Assert value matches regex -->
<uta:VerifyExpressionWithOperator
    DisplayName="Verify Email Format"
    ContinueOnFailure="True"
    Operator="RegexMatch"
    TakeScreenshotInCaseOfFailingAssertion="True">
  <uta:VerifyExpressionWithOperator.FirstExpression>
    <InArgument x:TypeArguments="x:String">[customerEmail]</InArgument>
  </uta:VerifyExpressionWithOperator.FirstExpression>
  <uta:VerifyExpressionWithOperator.SecondExpression>
    <InArgument x:TypeArguments="x:String">^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$</InArgument>
  </uta:VerifyExpressionWithOperator.SecondExpression>
</uta:VerifyExpressionWithOperator>
```

> The regex above is written as a **plain literal** — no surrounding `[...]`, so the pattern's own `[` / `]` are never read as a VB expression. The quoted form `["^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"]` is equivalent; both validate and match identically. Prefer the plain literal for regexes, since it avoids having to reason about bracket nesting.
