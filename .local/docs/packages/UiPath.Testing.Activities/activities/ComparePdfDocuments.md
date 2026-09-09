# Compare PDF Documents

Compares two PDF documents (baseline vs. target) to determine their equivalence. Supports text comparison at line/word/character granularity, optional inclusion of images/widgets, and comparison rules to exclude dynamic sections. Optionally uses Autopilot AI to interpret differences semantically.

**Class:** `UiPath.Testing.Activities.ComparePdfDocuments`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Verification

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
xmlns:utam="clr-namespace:UiPath.Testing.Activities.Models;assembly=UiPath.Testing.Activities"
```

The `utam` alias is needed to declare `ComparisonRule` / `Difference` variables. Also add `UiPath.Testing.Activities.Models` and `UiPath.Platform.ResourceHandling` to `TextExpression.NamespacesForImplementation` if you want to write `ComparisonRule` and `LocalResource` unqualified.

---

## Input

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `BaselinePath` | `InArgument<IResource>` | Yes | — | The baseline (reference) PDF. **Not a string** — see [Paths are `IResource`](#paths-are-iresource-not-strings). |
| `TargetPath` | `InArgument<IResource>` | Yes | — | The target PDF to compare against the baseline. **Not a string.** |
| `ComparisonType` | `ComparisonType` | Yes* | `Line` | Granularity of text comparison: `Line`, `Word`, or `Character`. *Hidden when `InterpretDifferencesWithAutopilot` is `true`. |
| `InterpretDifferencesWithAutopilot` | `Boolean` | No | `false` | If `true`, uses Autopilot AI to semantically interpret differences. When enabled, `ComparisonType` is hidden and `SemanticDifferences` is populated. |
| `ContinueOnFailure` | `InArgument<Boolean>` | No | `true` | See [ContinueOnFailure](#continueonfailure-leave-it-true). |
| `IgnoreIdenticalItems` | `InArgument<Boolean>` | No | `true` | If `true`, identical content (lines/items) is excluded from the diff output. |
| `IncludeImages` | `InArgument<Boolean>` | No | `true` | If `true`, images **and URI widgets** are included in the comparison. |
| `IgnoreImagesLocation` | `InArgument<Boolean>` | No | `false` | If `true`, the page and position of images/URI widgets are ignored (only their presence is compared). Only relevant when `IncludeImages` is `true`. |
| `OutputFolderPath` | `InArgument<String>` | Yes | `"."` | Directory where the two annotated diff PDFs are written. **The directory must already exist** — the activity throws `InvalidArgumentsException` ("OutputFolderPath property should be a folder") otherwise; it does not create it. Relative paths resolve against the process working directory, which is the project directory during a Studio run. |
| `Rules` | `List<InArgument<ComparisonRule>>` | No | — | *(Legacy)* Individual comparison rules added via the designer. **Takes precedence over `RulesList`** — see [Rules vs RulesList](#rules-vs-ruleslist). |
| `RulesList` | `InArgument<List<ComparisonRule>>` | No | — | A list of `ComparisonRule` objects (from **Create Comparison Rule**) to exclude dynamic sections. Preferred. |

## Output

| Property | Type | Description |
|----------|------|-------------|
| `Result` | `OutArgument<Boolean>` | `true` if documents are equivalent, `false` if differences were found. |
| `Differences` | `OutArgument<IEnumerable<Difference>>` | List of text differences. Each `Difference` has `Operation` (`Equal`, `Inserted`, `Deleted`) and `Text`. Populated when `InterpretDifferencesWithAutopilot` is `false`. |
| `SemanticDifferences` | `OutArgument<SemanticDifferences>` | Semantic interpretation from Autopilot. Only populated when `InterpretDifferencesWithAutopilot` is `true`. |

Two annotated PDFs are also written to `OutputFolderPath`, named `<baselineFileName>_result.pdf` and `<targetFileName>_result.pdf`.

---

## Paths are `IResource`, not strings

`BaselinePath` and `TargetPath` are `InArgument<IResource>`. A bare string literal is a **design-time validation error**:

```xml
<!-- WRONG — fails validation with:
     Literal only supports value types and the immutable type System.String.
     The type UiPath.Platform.ResourceHandling.IResource cannot be used as a literal. -->
<uta:ComparePdfDocuments BaselinePath="Baseline\invoice_baseline.pdf" ... />
```

Build the resource with `LocalResource.FromPath(...)` inside an expression:

```xml
BaselinePath="[UiPath.Platform.ResourceHandling.LocalResource.FromPath(System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), &quot;Baseline\invoice_baseline.pdf&quot;))]"
```

`LocalResource.FromPath` needs an absolute path, so combine with `Directory.GetCurrentDirectory()` (the project directory at run time).

---

## `Rules` vs `RulesList`

The activity evaluates `Rules ?? RulesList`. If the legacy `Rules` collection is non-null it wins and `RulesList` is **silently ignored**, even when `Rules` is empty. When passing rules programmatically, set `RulesList` and explicitly null out the other:

```xml
<uta:ComparePdfDocuments Rules="{x:Null}" RulesList="[New List(Of ComparisonRule) From {dateRule, idRule}]" ... />
```

---

## `ContinueOnFailure`: leave it `true`

The activity reports the comparison outcome through `Result` and as a test-case assertion. Leave `ContinueOnFailure = True` and assert on `Result` with a separate **Verify Expression** — that is the pattern that gives a clean pass/fail in the test report:

```xml
<uta:ComparePdfDocuments ... ContinueOnFailure="True" Result="[pdfCompareResult]" />
<uta:VerifyExpression Expression="[pdfCompareResult]" ContinueOnFailure="True"
                      TakeScreenshotInCaseOfFailingAssertion="True" />
```

> Setting `ContinueOnFailure = False` currently throws when the documents **match** rather than when they differ, so a matching comparison aborts the workflow. Keep it `True`.

---

## String-valued attributes

`OutputFolderPath` is `InArgument<String>`. Write it as bare text or as a bracketed expression — **never** as `&quot;…&quot;` without brackets, which stores the quote characters as part of the value and makes the folder lookup fail:

```xml
OutputFolderPath="Output\PDFDiff"                          <!-- OK: bare literal -->
OutputFolderPath="[&quot;Output\PDFDiff&quot;]"             <!-- OK: VB expression -->
OutputFolderPath="&quot;Output\PDFDiff&quot;"               <!-- WRONG: value becomes "Output\PDFDiff" with quotes -->
```

---

## Valid Configurations

**Standard comparison** (`InterpretDifferencesWithAutopilot = false`):
- `ComparisonType` must be set to `Line`, `Word`, or `Character`.
- `Differences` output is populated; `SemanticDifferences` is not.

**Autopilot-interpreted comparison** (`InterpretDifferencesWithAutopilot = true`):
- `ComparisonType` is hidden and ignored.
- `SemanticDifferences` output is populated; `Differences` is not.

These two modes are mutually exclusive.

---

## Enum: `ComparisonType`

| Value | Description |
|-------|-------------|
| `Line` | Compares extracted text line by line. |
| `Word` | Compares extracted text word by word. |
| `Character` | Compares extracted text character by character. |

In `Line` and `Word` modes the extracted text is assembled with U+2008 PUNCTUATION SPACE between items rather than a normal space. This matters when writing regex comparison rules — see **[Create Comparison Rule](CreateComparisonRule.md#whitespace-in-pdf-text)**.

---

## XAML Example

```xml
<!-- Line-by-line comparison, masking three dynamic fields with regex rules -->
<Sequence DisplayName="Compare invoice against baseline">
  <Sequence.Variables>
    <Variable x:TypeArguments="utam:ComparisonRule" Name="dateRule" />
    <Variable x:TypeArguments="utam:ComparisonRule" Name="invoiceNumberRule" />
    <Variable x:TypeArguments="utam:ComparisonRule" Name="timestampRule" />
    <Variable x:TypeArguments="x:Boolean" Name="pdfCompareResult" />
    <Variable x:TypeArguments="scg:IEnumerable(utam:Difference)" Name="pdfDifferences" />
  </Sequence.Variables>

  <uta:CreateComparisonRule DisplayName="Rule - Invoice Date" ComparisonRuleType="RegexRule"
                            ContinueOnError="False" ComparisonRule="[dateRule]"
                            Pattern="[&quot;\d{2}/\d{2}/\d{4}&quot;]" RuleName="[&quot;InvoiceDate&quot;]"
                            UsePlaceholder="True" />
  <uta:CreateComparisonRule DisplayName="Rule - Invoice Number" ComparisonRuleType="RegexRule"
                            ContinueOnError="False" ComparisonRule="[invoiceNumberRule]"
                            Pattern="[&quot;INV-\d{8}&quot;]" RuleName="[&quot;InvoiceNumber&quot;]"
                            UsePlaceholder="True" />
  <uta:CreateComparisonRule DisplayName="Rule - Render Timestamp" ComparisonRuleType="RegexRule"
                            ContinueOnError="False" ComparisonRule="[timestampRule]"
                            Pattern="[&quot;Generated\s+on\s+\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z&quot;]"
                            RuleName="[&quot;RenderTimestamp&quot;]" UsePlaceholder="True" />

  <uta:ComparePdfDocuments
      Rules="{x:Null}" SemanticDifferences="{x:Null}"
      DisplayName="Compare PDF Documents - baseline vs Output\invoice.pdf"
      BaselinePath="[UiPath.Platform.ResourceHandling.LocalResource.FromPath(System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), &quot;Baseline\invoice_baseline.pdf&quot;))]"
      TargetPath="[UiPath.Platform.ResourceHandling.LocalResource.FromPath(System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), &quot;Output\invoice.pdf&quot;))]"
      ComparisonType="Line"
      ContinueOnFailure="True"
      IgnoreIdenticalItems="True"
      IncludeImages="True"
      IgnoreImagesLocation="False"
      OutputFolderPath="[System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), &quot;Output\PDFDiff&quot;)]"
      RulesList="[New List(Of ComparisonRule) From {dateRule, invoiceNumberRule, timestampRule}]"
      Result="[pdfCompareResult]"
      Differences="[pdfDifferences]" />

  <uta:VerifyExpression DisplayName="Verify invoices are equivalent after rules"
                        ContinueOnFailure="True" Expression="[pdfCompareResult]"
                        TakeScreenshotInCaseOfFailingAssertion="True"
                        TakeScreenshotInCaseOfSucceedingAssertion="False" />
</Sequence>
```

```xml
<!-- Autopilot semantic interpretation -->
<uta:ComparePdfDocuments
    Rules="{x:Null}" Differences="{x:Null}" RulesList="{x:Null}"
    DisplayName="Compare PDF (Autopilot)"
    BaselinePath="[UiPath.Platform.ResourceHandling.LocalResource.FromPath(baselineAbsolutePath)]"
    TargetPath="[UiPath.Platform.ResourceHandling.LocalResource.FromPath(targetAbsolutePath)]"
    InterpretDifferencesWithAutopilot="True"
    ContinueOnFailure="True"
    OutputFolderPath="[System.IO.Path.Combine(System.IO.Directory.GetCurrentDirectory(), &quot;Output\PDFDiff&quot;)]"
    Result="[pdfResult]"
    SemanticDifferences="[semanticResult]" />
```
