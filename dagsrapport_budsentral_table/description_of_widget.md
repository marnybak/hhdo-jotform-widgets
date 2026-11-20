# Custom Jotform Widget
We have a Jotform (id 262811636939366) that's going to register data about distribution deviations in different regions. In addition to general data about the region, we need to collect detailed information for each pickup point within the region. I cannot find a Jotform widget that meets this requirement in a good way, especially when it comes to fitting many fields per pickup point on a mobile screen.

So I need a new custom Jotform widget. The widget will only be used for a couple internal specific forms, and will mostly be filled in on mobile.

## Input (widget settings)
- `columns`: a JSON list of field definitions (despite the name, each one is rendered as a table **row**):
  ```
  [
    { name: string, type: string, label: string, validate?: string, required?: boolean, requiredIf?: string, options?: string[], rows?: number }
  ]
  ```
  - `type`: one of `["dropdown", "number", "text", "textarea", "checkbox", "timestamp"]`
  - `label`: shown in the first column. Defaults to `name`.
  - `validate`: optional regex. It must match the whole value (it is anchored automatically). Only used for `text`, `textarea` and `number`.
  - `required`: optional, default `false`. Ignored for `checkbox` (unchecked is a valid `false`).
  - `requiredIf`: optional `name` of another field. This field becomes required (for the same pickup point) when that field is filled in: a `number` greater than 0, a checked `checkbox`, or any other non-empty value. Ignored for `checkbox`. Example: `{ "name": "ekstra_tid_sortering_aarsak", "type": "textarea", "requiredIf": "ekstra_tid_sortering_min" }`.
  - `options`: required for `dropdown`, ignored otherwise.
  - `rows`: number of visible lines for `textarea`, default `3`. Ignored for other types.
- `pickupPoints`: the list of pickup points, one per line (a JSON array of strings is also accepted). At most 4.

## Layout (transposed table)
- One **column** per pickup point, with the pickup point name as the column header.
- One **row** per field, with the field label in the first column.
- Timestamp fields use [flatpickr](https://flatpickr.js.org/) in 24-hour mode on desktop (native `<input type="time">` follows the browser language, not the OS clock setting, and can show AM/PM), and the device's native time picker on mobile. Typed values like `8:45`, `08.45` and `0845` are accepted. A timestamp is a time of day only, no date, and is always output as 24-hour `HH:MM`.
- Validation errors show on a field when the user leaves it (focus out), and on every field when the form is submitted.
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
- Empty values: `""` for text/textarea/timestamp/dropdown, `null` for number, `false` for checkbox.
- Numbers are output as JSON numbers (a decimal comma is converted to a point). If a custom `validate` lets a non-numeric value through, it is output as the raw string.


Transposed configurable list
A table that creates a column for each value in a static list, and one row for each field. Fields can be dropdowns, numbers, text, checkbox or time values.
https://marnybak.github.io/hhdo-jotform-widgets/dagsrapport_budsentral_table/widget.html
