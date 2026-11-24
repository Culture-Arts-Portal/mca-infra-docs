# form_schema format (v1)

Each service stores its form as JSON in `services.form_schema`. The frontend's
`DynamicForm` reads it to draw the form, and the backend's `validate` middleware
reads the same JSON to check the answers. One JSON, two uses, no hand-built pages.

## What is NOT in the schema

The platform adds these to every service automatically, so they never appear in a schema:

- The "From your account" card (name, NRC, phone, email come from the `users` table)
- The final **Review** step (built from the answers) and **payment** (from `services.fee`)
- The "Save draft" button (saves with status `DRAFT`)

## Top level

| Key           | Type   | Meaning                                              |
|---------------|--------|------------------------------------------------------|
| `version`     | number | Schema version. Bump it when a published form changes |
| `steps`       | array  | The steps, in order                                  |
| `declaration` | string | Checkbox text the applicant must tick before submitting |
| `feeRules`    | object | Optional. Fee depends on an answer instead of the fixed `services.fee`: `{ "field": "expectedAttendance", "amounts": { "under-500": 200, ... } }`. Amounts in kwacha; `null` means not confirmed by the ministry yet |
| `requires`    | object | Optional. The applicant must already hold another approved service: `{ "service": "artist-registration", "status": "APPROVED", "message": "...", "reuseDocuments": ["nrcCopy"] }`. `reuseDocuments` attaches those files from that application, so they aren't uploaded again |

## A step

| Key      | Type   | Meaning                         |
|----------|--------|---------------------------------|
| `id`     | string | Short id, e.g. `about`          |
| `title`  | string | Shown as the step heading       |
| `fields` | array  | The fields on this step         |
| `intro`  | string | Optional note shown at the top of the step |

## A field

| Key           | Required | Meaning |
|---------------|----------|---------|
| `name`        | yes | Key the answer is saved under in `applications.form_data`. camelCase, unique in the form |
| `type`        | yes | One of the types below |
| `label`       | yes | Text above the field |
| `required`    | yes | `true` or `false` |
| `hint`        | no  | Small help text under the field |
| `placeholder` | no  | Example text inside an empty field |
| `default`     | no  | Starting value |
| `options`     | for `choice` and `select` | List of `{ "value", "label" }`. `value` is saved, `label` is shown |
| `style`       | no  | For `choice`: `"buttons"` (default) or `"chips"` |
| `showIf`      | no  | Only show (and only require) the field when another answer matches: `{ "field": "registeringAs", "equals": "group" }` |
| `validation`  | no  | Rules, see below |

### Types

| Type       | Draws as | Saved as |
|------------|----------|----------|
| `text`     | One-line input | string |
| `textarea` | Multi-line input | string |
| `number`   | Number input | number |
| `date`     | Date picker | `"YYYY-MM-DD"` |
| `time`     | Time picker | `"HH:MM"` |
| `email`, `tel`, `url` | Input with that keyboard | string |
| `select`   | Dropdown | the option's `value` |
| `choice`   | Buttons or chips, pick one | the option's `value` |
| `checkbox` | One tick box | `true` / `false` |
| `file`     | Upload box. Also uses `accept` (MIME types), `maxSizeMB`, `maxFiles` | list of document ids from the `documents` table |
| `repeater` | A list the user adds rows to. Its own `fields` describe one row. Also uses `addLabel` | list of objects |

### Validation rules

| Rule | For | Example |
|------|-----|---------|
| `minLength`, `maxLength` | text types | `{ "maxLength": 500 }` |
| `min`, `max` | `number`, `date` (`"today"` and `"currentYear"` allowed) | `{ "max": "today" }` |
| `notBefore` | `date` | Can't be earlier than another date field: `{ "notBefore": "startDate" }` |
| `maxWords` | `textarea` | `{ "maxWords": 300 }` |
| `mustBe`, `mustBeMessage` | `choice` | Only one answer lets the applicant continue; the message says why |
| `notMoreThanTotal` | `number` | Can't exceed the sum of a repeater's number field: `{ "notMoreThanTotal": "budgetItems.amount" }` |
| `pattern`, `patternMessage` | text types | NRC: `"^\\d{6}/\\d{2}/\\d$"` |
| `minItems`, `maxItems` | `repeater` | `{ "minItems": 2 }` |

## Rules for the backend

1. A field hidden by `showIf` is skipped: not required, and any answer for it is dropped.
2. `required` and `validation` are checked again on the server. Never trust the browser alone.
3. Unknown field names in a submission are rejected.

## Example answers (`applications.form_data`)

```json
{
  "registeringAs": "individual",
  "stageName": "Kalindula King",
  "dateOfBirth": "1998-04-12",
  "province": "lusaka",
  "district": "Chongwe",
  "address": "Plot 12, Kabulonga Road",
  "artForm": "music",
  "genre": "Kalindula",
  "yearsActive": "4-10",
  "bio": "Kalindula musician from Chongwe, performing at weddings and festivals across Lusaka Province since 2019.",
  "nrcCopy": [41],
  "photo": [42],
  "workSamples": [43, 44]
}
```
