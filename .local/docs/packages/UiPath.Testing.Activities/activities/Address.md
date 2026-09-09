# Generate Address

Generates a random postal address and returns it as a `Dictionary<String, String>`. The dictionary contains keys: `Country`, `City`, `State`, `StreetNumber`, `StreetName`, `PostalCode`.

**Class:** `UiPath.Testing.Activities.TestData.Address`
**Assembly:** `UiPath.Testing.Activities`
**Category:** Testing > Data

```xml
xmlns:utad="clr-namespace:UiPath.Testing.Activities.TestData;assembly=UiPath.Testing.Activities"
```

---

## Output

| Property | Type | Description |
|----------|------|-------------|
| `AddressResult` | `OutArgument<Dictionary<String, String>>` | A dictionary containing address fields. Supported keys: `Country`, `City`, `State`, `StreetNumber`, `StreetName`, `PostalCode`. |

---

## Filters (designer-only)

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `Country` | `InArgument<String>` | No | *(any)* | Restricts generation to a country. `[Browsable(false)]` — set through the designer's dropdown, not the properties grid. |
| `City` | `InArgument<String>` | No | *(any)* | Restricts generation to a city. `[Browsable(false)]` — set through the designer's dropdown. |

When no restriction is chosen, Studio serializes the sentinel literals `<Random Country>` / `<Random City>` rather than omitting the attributes:

```xml
City="&lt;Random City&gt;" Country="&lt;Random Country&gt;"
```

Both forms are valid — omitting the attributes entirely also generates a fully random address. Emit the sentinels if you want to match what Studio writes.

---

## Notes

- The activity generates random addresses from a built-in dataset. Data is not fetched from any external API.
- Dictionary keys are case-sensitive: use `addressDict("City")`, not `addressDict("city")`.

---

## XAML Example

```xml
<!-- Fully random address (lean form) -->
<utad:Address
  DisplayName="Generate Address"
  AddressResult="[addressDict]" />

<!-- Same thing, matching Studio's serialization -->
<utad:Address
  DisplayName="Generate Address"
  City="&lt;Random City&gt;"
  Country="&lt;Random Country&gt;"
  AddressResult="[addressDict]" />
```

Extract fields into typed variables:
```xml
<Assign x:TypeArguments="x:String" DisplayName="Extract City"
        To="[city]" Value="[addressDict(&quot;City&quot;)]" />
<Assign x:TypeArguments="x:String" DisplayName="Extract Country"
        To="[country]" Value="[addressDict(&quot;Country&quot;)]" />
```

After execution, access individual fields:
```vb
addressDict("City")        ' e.g. "Chicago"
addressDict("PostalCode")  ' e.g. "60601"
```
