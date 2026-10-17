# Custom Jotform Widget
We have a Jotform (id 262811636939366) that's going to register data about distribution deviations in different regions. In addition to general data about the region, we need to collect detailed information for each pickup point within the region. I cannot find a Jotform widget that meets this requirement in a good way, especially when it comes to fitting many fields per pickup point on a mobile screen.

So I need a new custom Jotform widget. The widget will only be used for a couple internal specific forms, and will mostly be filled in on mobile.

## Input (widget settings)
- `columns`: a JSON list of field definitions (despite the name, each one is rendered as a table **row**):
  ```
  [
    { name: string, type: string, label: string, validate?: string, required?: boolean, options?: string[] }
  ]
  ```
  - `type`: one of `["dropdown", "number", "text", "checkbox", "timestamp"]`
  - `label`: shown in the first column. Defaults to `name`.
  - `validate`: optional regex. It must match the whole value (it is anchored automatically). Ignored for `checkbox` and `dropdown`.
  - `required`: optional, default `false`. Ignored for `checkbox` (unchecked is a valid `false`).
  - `options`: required for `dropdown`, ignored otherwise.
- `pickupPoints`: the list of pickup points, one per line (a JSON array of strings is also accepted). At most 4.

## Layout (transposed table)
- One **column** per pickup point, with the pickup point name as the column header.
- One **row** per field, with the field label in the first column.
- Timestamp and number inputs are narrow (about 5 characters). A timestamp is a time of day only, no date. The accepted formats are defined with `validate` and parsed on the server side.
- Checkbox cells contain just one checkbox (true/false) with no additional text.
- Must work well on mobile. If the table doesn't fit, it scrolls horizontally inside the widget while the label column stays visible.

## Dynamic pickup point lists
Jotform custom widgets run in a cross-origin iframe and cannot read other form fields, so the list cannot be changed by form rules. Instead, we add **one widget instance per region**, each with its own `pickupPoints` setting, and show/hide them with standard Jotform conditions on the region field. The server reads whichever instance has a value.

## Output
A JSON array (as a string), with one entry per pickup point. Untouched pickup points are included too:
```json
[{ "pickupPoint": "Kiwi Sentrum", "values": { "deviation": "Late", "count": 3, "ok": true, "time": "08:45" } }]
```
- `values` is keyed by column `name`.
- Empty values: `""` for text/timestamp/dropdown, `null` for number, `false` for checkbox.
- Numbers are output as JSON numbers (a decimal comma is converted to a point). If a custom `validate` lets a non-numeric value through, it is output as the raw string.
