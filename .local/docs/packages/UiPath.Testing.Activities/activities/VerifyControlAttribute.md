# Verify Control Attribute

Wraps a single UI Automation activity and asserts one of that activity's **output properties** against an expected value, using a `Comparison` operator. The read and the check are fused into one activity — you do **not** capture the UI value into a variable and assert on it separately.

**Class:** `UiPath.Testing.Activities.VerifyControlAttribute`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Verification
**Platform:** Windows only. The activity is hidden (`[Browsable(false)]`) in cross-platform projects, so `project.json` must have `"targetFramework": "Windows"`.

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
```

---

## Input

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `ActivityToTest` | `ActivityAction` | Yes | — | Container holding **exactly one** UI Automation activity whose output is asserted (e.g. `uix:NGetText`, `uix:NGetAttribute`). Must not be empty; must not contain a nested `VerifyControlAttribute`. |
| `OutputArgument` | `String` | Yes | — | The **name of the output property** on the wrapped activity to assert on — e.g. `"TextString"` for `NGetText`. Not an expression: a plain property name. |
| `Operator` | `Comparison` | Yes | `Equality` | Operator used to compare the wrapped activity's output against `Expression`. |
| `Expression` | `InArgument` (non-generic) | Yes | — | The expected value. **Non-generic** — must be written with element syntax and an explicit `x:TypeArguments`. See [XAML syntax](#xaml-syntax-non-generic-inargument). |
| `ContinueOnFailure` | `InArgument<Boolean>` | No | `true` | When `false`, a failing assertion throws `TestingActivitiesException` and aborts the test case. When `true`, the test case is marked failed but execution continues. |

## Output

| Property | Type | Description |
|----------|------|-------------|
| `Result` | `OutArgument<Boolean>` | `true` if the assertion passed. |

## Messages

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `AlternativeVerificationTitle` | `InArgument<String>` | No | *(DisplayName)* | Overrides the verification title reported to Orchestrator / the test report. |
| `OutputMessageFormat` | `InArgument<String>` | No | *(project setting)* | Custom result-message format. Placeholders: `{LeftExpression}`, `{LeftExpressionText}`, `{RightExpression}`, `{RightExpressionText}`, `{Result}`, `{Operator}`. |

## Common

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `TakeScreenshotInCaseOfFailingAssertion` | `InArgument<Boolean>` | No | **`true`** | Takes a screenshot when the assertion fails. Note this default differs from the other verify activities, which default to `false`. |
| `TakeScreenshotInCaseOfSucceedingAssertion` | `InArgument<Boolean>` | No | `false` | Takes a screenshot when the assertion passes. |

---

## Validation Constraints

- `ActivityToTest.Handler` must be set, or validation fails with *"activity is missing"*.
- The wrapped activity must expose at least one browsable output property, or validation fails with *"no output argument"*.
- `OutputArgument` must name one of those output properties, or validation fails with *"no output argument"*.
- `Verify Expression`, `Verify Expression with Operator`, `Verify Range`, and a nested `Verify Control Attribute` **cannot** appear inside `ActivityToTest` — each declares a `HasNoParent<VerifyControlAttribute>` constraint.
- Types must be compatible: the wrapped output property's type and `Expression`'s type are checked against `Operator` at design time. For `NGetText.TextString` (a `String`), `Expression` must be a `String`.

---

## XAML syntax: non-generic `InArgument`

`Expression` is declared as a **non-generic** `InArgument`, so it has no implicit type. Setting it as an attribute fails to load:

```xml
<!-- WRONG — fails with: Set property 'UiPath.Testing.Activities.VerifyControlAttribute.Expression' threw an exception. -->
<uta:VerifyControlAttribute Expression="&quot;15&quot;" ... />
```

Always use element syntax with an explicit `x:TypeArguments`:

```xml
<uta:VerifyControlAttribute.Expression>
  <InArgument x:TypeArguments="x:String">["15"]</InArgument>
</uta:VerifyControlAttribute.Expression>
```

---

## Output property names for common UI Automation activities

`OutputArgument` takes the **property name**, not the display name:

| Wrapped activity | Useful `OutputArgument` values |
|------------------|-------------------------------|
| `uix:NGetText` (Get Text) | `TextString` (the extracted text, `String`); also `Text`, `WordsInfo` |
| `uix:NGetAttribute` (Get Attribute) | `Result` (the attribute value) |
| `uix:NCheckState` (Check App State) | `Exists` (`Boolean`) |

When in doubt, read the wrapped activity's own doc and pick a browsable output property.

---

## XAML Example

A complete, validated test-case fragment: click through the Windows Calculator and assert the result display reads `15`. Only `ActivityToTest`, `OutputArgument`, `Operator` and `Expression` need to be authored — Studio generates the internal `ArgumentsBridge` plumbing and the `_autogenerated_*` variables that bind the wrapped activity's outputs when the file is loaded, so they must **not** be hand-written.

```xml
<uix:NApplicationCard AttachMode="ByInstance" CloseMode="Never" DisplayName="Use Calculator"
                      OpenMode="IfNotOpen" ScopeGuid="b1b1b1b1-0001-4001-8001-000000000001" Version="V2">
  <uix:NApplicationCard.Body>
    <ActivityAction x:TypeArguments="x:Object">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="x:Object" Name="WSSessionData" />
      </ActivityAction.Argument>
      <Sequence DisplayName="Then - verify the result display">
        <uta:VerifyControlAttribute
            DisplayName="Verify result display equals 15"
            ContinueOnFailure="True"
            Operator="Equality"
            OutputArgument="TextString"
            TakeScreenshotInCaseOfFailingAssertion="True"
            TakeScreenshotInCaseOfSucceedingAssertion="False">
          <uta:VerifyControlAttribute.ActivityToTest>
            <ActivityAction>
              <uix:NGetText DisplayName="Get Text - Result display"
                            ScopeIdentifier="b1b1b1b1-0001-4001-8001-000000000001" Version="V5">
                <uix:NGetText.Target>
                  <uix:TargetAnchorable ElementType="Text"
                                        FullSelectorArgument="&lt;uia automationid='CalculatorResults' /&gt;"
                                        SearchSteps="Selector" Version="V6" />
                </uix:NGetText.Target>
              </uix:NGetText>
            </ActivityAction>
          </uta:VerifyControlAttribute.ActivityToTest>
          <uta:VerifyControlAttribute.Expression>
            <InArgument x:TypeArguments="x:String">["15"]</InArgument>
          </uta:VerifyControlAttribute.Expression>
        </uta:VerifyControlAttribute>
      </Sequence>
    </ActivityAction>
  </uix:NApplicationCard.Body>
  <uix:NApplicationCard.TargetApp>
    <uix:TargetApp FilePath="C:\Windows\System32\calc.exe"
                   Selector="&lt;wnd app='applicationframehost.exe' title='Calculator' /&gt;" Version="V3">
      <uix:TargetApp.Arguments>
        <InArgument x:TypeArguments="x:String" />
      </uix:TargetApp.Arguments>
    </uix:TargetApp>
  </uix:NApplicationCard.TargetApp>
</uix:NApplicationCard>
```

Requires the UI Automation namespace alongside `uta`:

```xml
xmlns:uix="http://schemas.uipath.com/workflow/activities/uix"
```

---

## Enum: `Comparison`

| Value | Symbol | Description |
|-------|--------|-------------|
| `Equality` | `=` | Asserts the output equals `Expression`. |
| `Inequality` | `<>` | Asserts the output differs from `Expression`. |
| `GreaterThan` | `>` | Asserts the output is greater than `Expression`. |
| `GreaterThanOrEqual` | `>=` | Asserts the output is greater than or equal to `Expression`. |
| `LessThan` | `<` | Asserts the output is less than `Expression`. |
| `LessThanOrEqual` | `<=` | Asserts the output is less than or equal to `Expression`. |
| `Contains` | `Contains` | Asserts the output contains `Expression`. |
| `RegexMatch` | `Regex-Match` | Asserts the output matches the regex in `Expression`. |

In XAML, `Operator` is written with the **enum member name** (`Operator="Equality"`), not the symbol.

---

## Project Settings

| Property | Setting Key | Description |
|----------|-------------|-------------|
| `OutputMessageFormat` | `VerifyActivitiesOutputFormat` / `VerifyControlAttributeOutputFormat` | Default message format for all Verify Control Attribute instances in the project. |
