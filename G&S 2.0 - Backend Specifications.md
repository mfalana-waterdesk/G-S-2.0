# G&S 2.0 — Scheduled Billable API: Backend Technical Reference

> **Created:** July 13, 2026 | **Last Modified:** July 13, 2026
>
> **Author:** Malik Falana | **Modified By:** Malik Falana

---

## 1. Overview

**Purpose:** Send Goods & Services (G&S) charges from the Water Desk to Aspire as Scheduled Billables via a single POST call per submission.

**Architecture:** One Call URL action on the Dealer Goods & Services table fires per-submission, combining all child Fee records into a single API payload using formula columns and summary roll-ups.

| Property | Value |
|----------|-------|
| Database | CSP (AppID 76449) — test environment |
| Endpoint | `POST /1/contract/scheduledbillables` |
| Base URL | `https://purewaterpartnerstest2.leaseteam.net/LeaseTeam.Aspire.Api` |
| Auth | Bearer token (Aspire API key) |

---

## 2. Data Flow

```
Taylor enters fees in Fees sub-tab
    ↓
Each fee's [API Asset Fragment] formula auto-computes JSON chunk
    ↓
G&S parent [API Assembled RelatedAssets] summary concatenates all chunks
G&S parent [Total Fee Amount] summary sums amounts
G&S parent [API First Fee *] summaries pull shared fields
G&S parent [API DoNotRelateAssets Value] formula computes true/false
    ↓
Taylor clicks "Send Scheduled Billable Test" button on G&S Preview Page
    ↓
Action 1: Send Scheduled Billable to Aspire (Call URL) — POST to Aspire
Action 2: Mark Related Fees Sent (Update Record) — stamps Status="Sent" (fires after API)
    ↓
Aspire returns 200 OK with Result: "Commit" (or 422 with error details)
```

---

## 3. Button Configuration

### Button: "Send Scheduled Billable Test"

| Property | Value |
|----------|-------|
| Location | Preview Page |
| Table | Dealer Goods & Services |
| Filter | All records |
| Roles | Default Role |
| Assignments | None |

**Actions (ordered):**

| Order | Action Name | Type | What It Does |
|-------|-------------|------|--------------|
| 1 | Send Scheduled Billable to Aspire | Call URL | POST payload to Aspire endpoint |
| 2 | Mark Related Fees Sent (Parent Test) | Update Record | Sets related Fees records Status = "Sent" |

> **Note:** Action order was corrected on 07/13/2026. Call URL fires first; fees are only marked "Sent" after the API call completes.

---

## 4. API Endpoint

| Method | URL | Headers | Body |
|--------|-----|---------|------|
| POST | `https://purewaterpartnerstest2.leaseteam.net/LeaseTeam.Aspire.Api/1/contract/scheduledbillables` | Authorization: Bearer {token}, Content-Type: application/json | See §5 below |

---

## 5. Call URL Body (Exact — As Saved in TeamDesk)

### Description
Constructs the Scheduled Billable JSON payload by pulling values from the G&S parent record's formula and summary columns.

### Why This Step Exists
Aspire requires a single JSON object per Scheduled Billable submission. The payload combines header-level fields (ContractId, Description, TransactionCode) with per-asset detail (RelatedAssets array) rolled up from child Fee records.

### What TeamDesk Does
On button click, TeamDesk evaluates the `<%= %>` and `<%* %>` tokens against the current G&S record's column values, assembles the JSON, and sends the HTTP POST.

### Call URL Body

```json
{
  "RecordId": <%=[Ref#]%>,
  "Description": <%=[API First Fee Description]%>,
  "TransactionCode": {"Code": <%=[API First Fee TransCode]%>},
  "ContractId": {
    "Value": <%=[Contract RecordId]%>,
    "Type": "transaction"
  },
  "Payments": [
    {
      "StartDate": <%=Format([API First Fee Payment Date], "yyyy-MM-dd")%>,
      "Occurrences": <%=[API First Fee Occurrences]%>,
      "Frequency": <%=[API First Fee Frequency]%>,
      "Amount": <%=[Total Fee Amount]%>
    }
  ],
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
  "RelatedAssets": [<%*[API Assembled RelatedAssets]%>],
  "DoNotRelateAssets": <%*[API DoNotRelateAssets Value]%>,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": false
}
```

### Result
Aspire creates a Scheduled Billable record and returns HTTP 200 with `"Result": "Commit"` on success, or HTTP 422 with error details on validation failure.

### Token Type Reference

| Token | Behavior |
|-------|----------|
| `<%= %>` | Auto-encodes value (wraps strings in quotes, leaves numbers raw) |
| `<%* %>` | Passthrough — inserts raw value with NO encoding (required for pre-computed JSON fragments and boolean literals) |

---

## 6. Table: Fees (Child)

### Purpose
Stores individual G&S charges. Each Fee record represents one line item that contributes to the parent G&S submission.

### Input Columns

| Column | Type | Notes |
|--------|------|-------|
| Equipment | Reference → Miscellaneous Units (DropDown) | Links to an asset; null = no asset |
| Description | Text (dropdown) | Ice Mats, Cleaning Fee, Trip Charge, etc. |
| Fee Amount | Numeric | Dollar amount for this line item |
| First Payment Date | Date | When billing begins |
| TransCode | Text | CLEANING, PM, LIFT, TRIP, FILTER, CLOG, CO2EXCH, ICEMAT |
| All Assets? | Checkbox | If true, Aspire prorates across all contract assets |
| Recurring Billable? | Checkbox | Enables multi-occurrence billing |
| Number of Occurrences | Numeric | How many billing cycles (used when Recurring = true) |
| Recurring Frequency | Text | Monthly, Quarterly, Annual |

### Lookup Columns (from Equipment → Miscellaneous Units)

| Column | Source |
|--------|--------|
| Serial Number | Miscellaneous Units → Serial Number |
| Equipment ContractAssetId | Miscellaneous Units → ContractAssetId |
| Equipment Aspire Record Id | Miscellaneous Units → Aspire Record Id |
| Equipment Contract Asset Id Raw | Miscellaneous Units → Contract Asset Id Raw |

### Summary Columns (from Parent G&S)

| Column | Summary Function | Source |
|--------|-----------------|--------|
| API ContractId | Concatenate | Parent G&S → Selected ContractId |
| API Contract RecordId | Concatenate | Parent G&S → Contract RecordId |
| Parent Contract Status | Concatenate | Parent G&S → Contract Status |
| Parent Delinquency Code | Concatenate | Parent G&S → Delinquency Code |
| Parent Invoice Code | Concatenate | Parent G&S → Invoice Code |

### Key Formula: API Asset Fragment

**Column Type:** Formula - Text

**Purpose:** Builds the per-fee JSON object for the RelatedAssets array.

**Why this exists:** Each Fee with an equipment reference contributes one entry to the RelatedAssets array. This formula pre-computes the JSON fragment so the parent's summary column can concatenate them all.

```json
If(
    IsNull([Equipment Aspire Record Id]),
    "",
    '{"AssetRecordId":{"Value":' & JSONFormat([Equipment Aspire Record Id]) & ',"Type":"record"},"PaymentAmount":' & ToText([Fee Amount]) & ',"FirstPaymentDate":' & JSONFormat(XMLFormat([First Payment Date])) & '}'
)
```

**Result:** Produces a JSON object like:
```json
{"AssetRecordId":{"Value":"APIE-93067","Type":"record"},"PaymentAmount":100,"FirstPaymentDate":"2026-06-29"}
```

Or empty string `""` if no equipment is linked.

### Other Formula Columns

| Column | Type | Formula | Purpose |
|--------|------|---------|---------|
| API Has Equipment | Checkbox | `not IsNull([Equipment])` | Guard check for asset-related logic |
| API DoNotRelateAssets | Checkbox | `IsNull([Equipment]) and not [All Assets?]` | True when charge has no asset association |
| API Frequency | Text | `If([Recurring Billable?] and not IsNull([Recurring Frequency]), [Recurring Frequency], "Monthly")` | Defaults to "Monthly" for one-time |
| API Occurrences | Numeric | `If([Recurring Billable?] and [Number of Occurrences] > 0, [Number of Occurrences], 1)` | Defaults to 1 for one-time |
| API Follow Rent Allowed | Checkbox | `If([Follow Rent?] and [Parent Contract Status] = "Renewal", false, [Follow Rent?])` | Guard: prevents Follow Rent on Renewal contracts |

### Response Columns

| Column | Type | Purpose |
|--------|------|---------|
| Scheduled Billable API Response Status | Text (Updatable) | HTTP status code from Call URL response |
| Scheduled Billable API Response Body | Text (Updatable) | Full response body |
| Scheduled Billable API Error Message | Text (Updatable) | Parsed error detail |
| API Proration Method Description | Text (Updatable) | Confirmation of proration method used |

---

## 7. Table: Dealer Goods & Services (Parent)

### Purpose
Groups Fee records into a single billable submission. Holds the rolled-up summary data and fires the Call URL action.

### Key Columns for API

| Column | Type | Source/Formula | Purpose |
|--------|------|----------------|---------|
| Ref# | Autonumber | System-generated | Used as `RecordId` in payload (prefixed "REF#") |
| Contract RecordId | Lookup | Contract table → numeric Aspire contract ID | Used as `ContractId.Value` |
| Selected ContractId | Lookup | Contract table → contract number | Display only, NOT used in API |

### Summary Columns (Pulling from Child Fees)

| Column | Summary Function | Sort | Source (Fees Table) | API Field |
|--------|-----------------|------|---------------------|-----------|
| API Assembled RelatedAssets | Concatenate (separator: `,`) | — | API Asset Fragment | `RelatedAssets` array contents |
| API First Fee Description | First | Id Asc | Description | `Description` |
| API First Fee TransCode | First | Id Asc | TransCode | `TransactionCode.Code` |
| API First Fee Occurrences | First | Id Asc | API Occurrences | `Payments[].Occurrences` |
| API First Fee Frequency | First | Id Asc | API Frequency | `Payments[].Frequency` |
| API First Fee Payment Date | First | Id Asc | First Payment Date | `Payments[].StartDate` |
| API First Fee AllAssets | First | Id Asc | All Assets? | Controls `DoNotRelateAssets` logic |
| API Units Count | # of records | — | Equipment (non-null only) | Controls `DoNotRelateAssets` logic |
| Total Fee Amount | Total | — | Fee Amount | `Payments[].Amount` |
| API First Matching Fee Status | First | — | Status | Readiness check |
| API Related Fee Count | Count | — | Id | Readiness check |

### Key Formula: API DoNotRelateAssets Value

**Column Type:** Formula - Text

**Purpose:** Produces the literal string `"true"` or `"false"` for the `DoNotRelateAssets` field in the JSON payload.

**Why this exists:** TeamDesk's `<%* %>` passthrough token inserts this raw value directly into the JSON. A checkbox would output `True`/`False` (capitalized), which is invalid JSON.

```
If([API Units Count] = 0 and not [API First Fee AllAssets], "true", "false")
```

**Logic:**
- `true` when: No fees have equipment selected AND "All Assets?" is not checked (Scenario 1)
- `false` when: At least one fee has equipment, OR "All Assets?" is checked (Scenarios 2, 3, 5, 6)

### Other Formula Columns

| Column | Formula | Purpose |
|--------|---------|---------|
| API ContractId Value | `Nz([Selected ContractId])` | Pulls ContractId for payload |
| API ContractId Type | `"transaction"` | Always "transaction" |
| API Ready for Single-Asset Test | `not IsNull([Selected ContractId]) and [API Related Fee Count] > 0 and IsNull([API First Matching Fee Status])` | Readiness guard |
| API Payload Preview | Status indicator for multi-asset grouping | Preview/debugging only |
| Aspire Http Link | Formula - Text | Constructs URL to view billable in Aspire |

---

## 8. Aspire API Field Formats (Confirmed Working)

| Field | Format | Example | Notes |
|-------|--------|---------|-------|
| TransactionCode | `{"Code": "<string>"}` | `{"Code": "CLEANING"}` | Valid codes: CLEANING, PM, LIFT, TRIP, FILTER, CLOG, CO2EXCH, ICEMAT |
| ContractId | `{"Value": "<numeric>", "Type": "transaction"}` | `{"Value": "104762", "Type": "transaction"}` | Value MUST be numeric string |
| AssetRecordId | `{"Value": "<string>", "Type": "record"}` | `{"Value": "APIE-93067", "Type": "record"}` | Uses Aspire Record Id format |
| ProrationMethod | `{"Code": <int>, "Description": "<string>"}` | `{"Code": 1, "Description": "EquipmentCost"}` | Always present in current implementation |
| Dates | `yyyy-MM-dd` | `"2026-06-29"` | ISO format |
| Amounts | Raw numeric | `100` | No quotes, no currency symbols |
| FollowRent | Boolean literal | `false` | Always false (dropped from scope) |
| SuspendInvoicing | Boolean literal | `true` | Always true during rollout; flip to false after first week (per Cody, 06-30 meeting) |
| DoNotRelateAssets | Boolean literal | `true` or `false` | Computed by formula |
| RecordId | String | `"REF#49793"` | G&S Autonumber |

---

## 9. Validation Rules (Learned from Testing)

These constraints were identified during test validation and will cause Aspire to return HTTP 422 if violated:

| Rule | Error Behavior | Prevention |
|------|---------------|------------|
| ContractId.Value MUST be numeric when Type="transaction" | 422 Unprocessable Entity | Ensure `[Contract RecordId]` lookup resolves to a numeric value |
| Asset FirstPaymentDate MUST be >= Payments.StartDate | 422 Unprocessable Entity | Validate dates in Fee entry form or via TeamDesk validation rule |
| TransactionCode.Code must exist in Aspire Admin billing configuration | 422 Unprocessable Entity | Restrict dropdown values to known codes |
| Sum of RelatedAssets[].PaymentAmount must equal Payments[].Amount | 422 Unprocessable Entity | `[Total Fee Amount]` summary ensures amounts sum correctly |
| TeamDesk `<%? %>` conditional blocks do NOT reliably evaluate summary/formula columns | Malformed JSON sent to API | Use `<%* %>` passthrough with pre-computed formula columns instead |

---

## 10. Validated Test Results (July 13, 2026)

| Scenario | Description | RecordId | ContractId | DoNotRelateAssets | RelatedAssets | HTTP Status | Aspire Token |
|----------|-------------|----------|------------|-------------------|---------------|-------------|--------------|
| 1 | No assets, one-time $100 | REF#49793 | 42712 | true | [] | 200 OK | 380afdc6... |
| 2 | Single asset, $100 | REF#49794 | 42712 | false | 1 asset (APIE-93067) | 200 OK | 52f8b278... |
| 3 | Two assets, $25/$75 split | REF#49797 | 104762 | false | 2 assets (APIMU-161186, APIE-153343) | 200 OK | ae3f734c... |
| 5 | All assets proportional, $100 | REF#49798 | 104762 | false | [] (Aspire prorates) | 200 OK | 7b24eafe... |
| 6 | Recurring 3 months, 2 assets | REF#49799 | 104762 | false | 2 assets | 200 OK | 3aaa7e5b... |
| 4 | Custom billing codes | — | — | — | — | Pending | — |
| 7 | Follow Rent, no assets | — | — | — | — | DROPPED | — |
| 8 | Follow Rent, 2 assets | — | — | — | — | DROPPED | — |

### Scenario Logic Matrix

| Scenario | Equipment? | All Assets? | Recurring? | DoNotRelateAssets | RelatedAssets Content |
|----------|-----------|-------------|------------|-------------------|----------------------|
| 1 | No | No | No | true | Empty `[]` |
| 2 | Yes (1) | No | No | false | 1 asset with PaymentAmount + FirstPaymentDate |
| 3 | Yes (2+) | No | No | false | N assets, amounts sum to total |
| 5 | No | Yes | No | false | Empty `[]` — Aspire prorates across all |
| 6 | Yes (2+) | No | Yes | false | N assets, Occurrences > 1 |

---

## 11. Potential Issues

| # | Issue | Impact | Mitigation |
|---|-------|--------|------------|
| 1 | Multi-asset assumes same TransCode/Description | The "First" summaries pull TransCode and Description from the first fee only. If fees under one G&S parent have different TransCodes, only the first is sent to Aspire | Either enforce same TransCode per G&S submission (UI/validation), or split into separate API calls per TransCode group |
| 2 | Multi-asset assumes same First Payment Date | Payments.StartDate uses `API First Fee Payment Date` (first fee's date). If fees have different dates, asset FirstPaymentDates could fall outside the Payment date range, causing a 422 error | Either enforce same date per submission, or use the earliest date across all fees |
| 3 | No validation on Equipment Aspire Record Id | If a user selects equipment that hasn't been linked in Aspire (Aspire Record Id is empty), the asset fragment will be empty and the asset won't be included in the payload — silently dropped | Add a validation rule or form message: if Equipment is selected but `Equipment Aspire Record Id` is null, warn the user |
| 4 | FollowRent checkbox exists in schema but not on form | Column `Follow Rent?` exists on Fees table but is not displayed. If data gets set to true by accident (import, API, etc.), the per-fee payload formula could behave unexpectedly | Remove the FollowRent branch from formulas entirely, or delete/hide the column |

---

## 12. Remaining Scenarios to Test

| Scenario | Description | Status |
|----------|-------------|--------|
| 4 | Custom billing codes (DelinquencyCode, InvoiceCode, InvoiceLeadDays, BillingType) | Pending — requires fields for user to specify billing codes |
| 7 | Follow Rent, no assets (FollowRent=true, Payments has only Amount) | Pending — FollowRent dropped from UI but API supports it |
| 8 | Follow Rent, 2 assets (FollowRent=true, assets without PaymentAmount) | Pending — same as above |

---

## 13. Technical Notes & Gotchas

### TeamDesk Token Behavior
- `<%= %>` auto-encodes: wraps strings in quotes, leaves numbers raw. Use for most column references.
- `<%* %>` passthrough: inserts raw value. Required for:
  - Pre-computed JSON arrays (API Assembled RelatedAssets)
  - Boolean literals (API DoNotRelateAssets Value)
- `<%? %> ... <%?%>` conditional blocks: **DO NOT** reliably evaluate summary or formula columns. Avoid for API payloads.

### Null Handling
- Use `Nz()` in formulas. If a field is null and concatenated with `&`, the entire expression returns null.
- `Concat()` treats nulls as empty strings — safer for JSON construction.
- Always validate that lookup columns resolve before submission.

### Date Formatting
- Aspire expects `yyyy-MM-dd` (ISO format).
- TeamDesk `Format([Date], "yyyy-MM-dd")` produces the correct output.
- `XMLFormat([Date])` can also be used inside `JSONFormat()` for proper quoting.

### Amount Formatting
- Send raw numeric values (no quotes, no currency symbols).
- `ToText([Fee Amount])` produces raw numeric in TeamDesk formula context.
- In Call URL body, `<%= %>` on a Numeric column outputs unquoted number.

### Multi-Asset Grouping
- TeamDesk formulas cannot iterate child records.
- The solution uses a Summary column (`API Assembled RelatedAssets`) with Concatenate function to join all child Fee `[API Asset Fragment]` values with comma separator.
- The Call URL body wraps this in `[ ]` to form a valid JSON array.

---

## 14. Setup Links

| Item | TeamDesk Path |
|------|---------------|
| Fees table columns | Setup > Tables > Fees > Columns |
| G&S table columns | Setup > Tables > Dealer Goods & Services > Columns |
| Call URL action | Setup > Tables > Dealer Goods & Services > Actions > Send Scheduled Billable to Aspire |
| Button configuration | Setup > Tables > Dealer Goods & Services > Buttons > Send Scheduled Billable Test |
| Equipment table | Setup > Tables > Miscellaneous Units > Columns |

