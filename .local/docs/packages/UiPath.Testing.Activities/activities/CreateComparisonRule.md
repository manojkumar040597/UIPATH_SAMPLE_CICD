# Create Comparison Rule

Creates a `ComparisonRule` object that can be passed to **Compare Text** or **Compare PDF Documents** to exclude dynamic sections of text from comparison. Rules match patterns using either wildcard or regex syntax, replacing matched portions with a placeholder in the diff output.

**Class:** `UiPath.Testing.Activities.CreateComparisonRule`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Verification

```xml
xmlns:uta="clr-namespace:UiPath.Testing.Activities;assembly=UiPath.Testing.Activities"
xmlns:utam="clr-namespace:UiPath.Testing.Activities.Models;assembly=UiPath.Testing.Activities"
```

The `utam` alias is needed to declare the `ComparisonRule` variable that receives the output.

---

## Input

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `RuleName` | `InArgument<String>` | No | — | A name for the rule. Used as the placeholder text in diff output when `UsePlaceholder` is `true`. |
| `ComparisonRuleType` | `ComparisonRuleType` | No | `RegexRule` | The matching technique: `RegexRule` or `WildcardRule`. See enum below. |
| `Pattern` | `InArgument<String>` | No | — | The pattern to match in the text. Use regex syntax for `RegexRule`, glob-style wildcards (`*`, `?`) for `WildcardRule`. |
| `UsePlaceholder` | `InArgument<Boolean>` | No | `true` | If `true`, matched text in the diff is replaced by `[RuleName]`. If `false`, matched text is simply omitted. |

## Output

| Property | Type | Description |
|----------|------|-------------|
| `ComparisonRule` | `OutArgument<ComparisonRule>` | The created comparison rule object. Pass this to the `RulesList` property of **Compare Text** or **Compare PDF Documents**. |

## Common

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `ContinueOnError` | `InArgument<Boolean>` | No | `false` | If `true`, execution continues when the activity throws an error. |

---

## Whitespace in PDF text

When **Compare PDF Documents** extracts text in `Line` or `Word` mode, it joins the items on each line with **U+2008 PUNCTUATION SPACE**, not a normal space (U+0020). A `RegexRule` whose pattern contains a literal space will therefore **not match**.

Use `\s+` (or `\s`) wherever the pattern spans a space — `\s` matches U+2008:

```xml
<!-- WRONG for PDF comparison: the literal spaces never match -->
Pattern="[&quot;Generated on \d{4}-\d{2}-\d{2}&quot;]"

<!-- CORRECT -->
Pattern="[&quot;Generated\s+on\s+\d{4}-\d{2}-\d{2}&quot;]"
```

`WildcardRule` does not need this: it internally rewrites each literal space to match either the punctuation space (U+2008) or a normal space. **Compare Text** operates on the strings you pass in, so normal spaces are fine there.

---

## String-valued attributes

`RuleName` and `Pattern` are `InArgument<String>`. Write them as bare text or as a bracketed expression — **never** as `&quot;…&quot;` without brackets, which stores the quote characters as part of the pattern and stops it matching:

```xml
Pattern="[&quot;INV-\d{8}&quot;]"      <!-- OK: VB expression (what Studio emits) -->
Pattern="INV-\d{8}"                    <!-- OK: bare literal -->
Pattern="&quot;INV-\d{8}&quot;"        <!-- WRONG: pattern becomes "INV-\d{8}" including quotes -->
```

---

## Enum: `ComparisonRuleType`

| Value | Description |
|-------|-------------|
| `RegexRule` | Pattern is a regular expression. Matches any text that satisfies the regex. |
| `WildcardRule` | Pattern uses glob-style wildcards (`*` matches any sequence of characters, `?` matches a single character). |

Written in XAML with the enum member name: `ComparisonRuleType="RegexRule"`.

---

## Notes

- Create rules before calling **Compare Text** or **Compare PDF Documents**.
- Collect multiple rules into a list and pass them via `RulesList`. Also set `Rules="{x:Null}"` on the compare activity, because the legacy `Rules` collection takes precedence over `RulesList` when non-null.
- Typical use cases: excluding dates, timestamps, IDs, and other dynamic content from document comparisons.

---

## XAML Example

```xml
<!-- Rule to ignore date patterns like "01/15/2024" -->
<uta:CreateComparisonRule
  DisplayName="Create Date Rule"
  ComparisonRuleType="RegexRule"
  ContinueOnError="False"
  RuleName="[&quot;DatePattern&quot;]"
  Pattern="[&quot;\d{2}/\d{2}/\d{4}&quot;]"
  UsePlaceholder="True"
  ComparisonRule="[dateRule]" />

<!-- Rule to ignore any order ID starting with "ORD-" -->
<uta:CreateComparisonRule
  DisplayName="Create Order ID Rule"
  ComparisonRuleType="WildcardRule"
  ContinueOnError="False"
  RuleName="[&quot;OrderID&quot;]"
  Pattern="[&quot;ORD-*&quot;]"
  UsePlaceholder="True"
  ComparisonRule="[orderIdRule]" />
```

Then pass the rules to a compare activity:
```xml
<uta:CompareText
  Rules="{x:Null}"
  RulesList="[New List(Of ComparisonRule) From {dateRule, orderIdRule}]"
  ... />
```

Declare the receiving variables with the `utam` alias:
```xml
<Variable x:TypeArguments="utam:ComparisonRule" Name="dateRule" />
<Variable x:TypeArguments="utam:ComparisonRule" Name="orderIdRule" />
```
