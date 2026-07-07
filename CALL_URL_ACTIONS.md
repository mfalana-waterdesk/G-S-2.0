# G&S 2.0 — TeamDesk Call URL Action Configuration

> Copy-paste ready configurations for the Scheduled Billable API call in TeamDesk.
> These go into a **Call URL** workflow action on the **Fees** table (or triggered from the parent **Dealer Goods & Services** table).

---

## How TeamDesk Call URL Works (Quick Recap)

- **Method:** POST
- **URL:** The Aspire Scheduled Billable endpoint
- **Headers:** Auth token + Content-Type
- **Body:** JSON format, using `<%= [Column Name] %>` for auto-encoded values and `<%* expression %>` for passthrough (pre-built JSON fragments)
- **Conditional sections:** `<%? condition %> content <%?%>` — includes content only when condition is true
- **Response:** Use `Response("path")` to extract values, `ResponseStatus()` to check HTTP status
- **Assignments:** Map response data back to columns (Status, Date Sent, etc.)

---

## Architecture Decision: Per-Fee-Record vs. Grouped

TeamDesk's Call URL fires **per record**. Since each Fee record represents one line item, you have two options:

### Option A: One API call per Fee record (SIMPLER — RECOMMENDED TO START)
- Each Fee sends its own Scheduled Billable to Aspire
- Single-asset and no-asset scenarios work perfectly
- Multi-asset splits (Scenarios 3, 4, 8) → each asset is its own Scheduled Billable
- Limitation: You can't combine multiple assets into one billable in a single call

### Option B: Grouped call from parent G&S record (COMPLEX — PHASE 2)
- Build a summary/formula column on the parent that concatenates all Fee data into one JSON
- Use passthrough `<%* [JSON Summary Column] %>` to inject it
- Handles multi-asset grouping but requires more formula plumbing

**Start with Option A.** It covers Scenarios 1, 2, 5, 7 perfectly and handles 3/4/6/8 as individual calls per asset (which Aspire supports fine — each asset just gets its own Scheduled Billable).

---

## The Call URL Action (Option A — Per Fee Record)

### Setup in TeamDesk

1. **Table:** Fees
2. **Trigger:** Custom Button (or Record Change Trigger when status changes)
3. **Action Type:** Call URL

---

### Method

```
POST
```

---

### URL

```
<%* [Aspire Scheduled Billable Endpoint URL] %>
```

Replace with your actual Aspire endpoint. It might be something like:
```
https://api.youraspireinstance.com/api/v1/ScheduledBillables
```

> **Note:** You'll need to confirm this URL with Cody. It should be in the API docs or your Aspire admin panel.

---

### Headers

```
Content-Type: application/json
Authorization: Bearer <%[Aspire API Token]%>
```

Or if Aspire uses a different auth scheme, adjust accordingly. You can also set up a **3rd-Party Account** in TeamDesk under Setup > Integration API > 3rd-Party Accounts for cleaner auth management.

---

### Body (JSON format)

```json
{
  "RecordId": <%=[REF#]%>,
  "Description": <%=[Description]%>,
  "TransactionCode": {
    "Value": <%=[TransCode]%>,
    "Type": "transaction"
  },
  "ContractId": {
    "Value": <%=[Trans#]%>,
    "Type": "transaction"
  },
  "Payments": [
    {
<%? [Follow Rent?] = false %>
      "StartDate": <%=Format([First Payment Date], "yyyy-MM-dd")%>,
      "Occurrences": <%=If([Recurring Billable?] = true and [Number of Occurrences] > 0, [Number of Occurrences], 1)%>,
      "Frequency": <%=If([Recurring Billable?] = true and [Recurring Frequency] <> null, [Recurring Frequency], "Monthly")%>,
<%?%>
      "Amount": <%=[Fee Amount]%>
    }
  ],
<%? [Equipment] <> null and [All Assets?] = false %>
  "ProrationMethod": {
    "Value": "1"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": <%=[ContractAssetId]%>
<%? [Follow Rent?] = false %>
      ,"PaymentAmount": <%=[Fee Amount]%>,
      "FirstPaymentDate": <%=Format([First Payment Date], "yyyy-MM-dd")%>
<%?%>
    }
  ],
  "DoNotRelateAssets": false,
<%?%>
<%? [All Assets?] = true %>
  "ProrationMethod": {
    "Value": "2"
  },
  "RelatedAssets": [],
  "DoNotRelateAssets": false,
<%?%>
<%? [Equipment] = null and [All Assets?] = false %>
  "RelatedAssets": [],
  "DoNotRelateAssets": true,
<%?%>
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": <%=[Follow Rent?]%>
}
```

---

### Important Notes on the Body

1. **`<%=  %>`** = auto-encoded (strings get quoted, numbers stay raw, booleans are true/false)
2. **`<%?  %> ... <%?%>`** = conditional block — content only appears if condition is true
3. **`<%*  %>`** = passthrough (no encoding — use for pre-built JSON fragments)
4. **`Format([Date], "yyyy-MM-dd")`** = formats date as ISO string for the API
5. **`[ContractAssetId]`** — this needs to be a lookup column on the Fees table that pulls `ContractAssetId` from the linked Miscellaneous Units record via the `Equipment` reference

---

## Column You May Need to Add

If `ContractAssetId` isn't already accessible from the Fees table, you'll need a **Lookup column**:

- **Name:** `ContractAssetId`
- **Type:** Lookup
- **Source:** Follow the `Equipment` reference → Miscellaneous Units → `ContractAssetId` column

This gives you the Aspire asset identifier directly on each Fee record.

---

## Response Handling (Assignments)

After the call, set up these assignments:

| From (Formula) | To (Column) |
|----------------|-------------|
| `If(ResponseStatus() = 200 or ResponseStatus() = 201, "Sent", "Failed: " & Text(ResponseStatus()))` | `Status` |
| `If(ResponseStatus() = 200 or ResponseStatus() = 201, Now(), null)` | `Date Sent` |

If Aspire returns useful data in the response (like a billable ID), you can also capture it:

| From (Formula) | To (Column) |
|----------------|-------------|
| `Response("Id")` | (a new column if you want to store the Aspire record ID) |

---

## Error Message Formula

```
If(ResponseStatus() >= 400, 
  "Aspire API Error (" & Text(ResponseStatus()) & "): " & Response(), 
  null
)
```

Set **On Error** to `Continue Execution` so you can still write the failure status back to the record rather than just rolling back.

---

## Alternative: Simpler Body Using a Formula Column

If the conditional blocks in the JSON body get unwieldy in TeamDesk's editor, you can pre-build the JSON in a **Text Formula column** on the Fees table:

### Formula Column: "API Payload" (Text, Formula)

```
"{" &
  """RecordId"": " & JSONFormat([REF#]) & "," &
  """Description"": " & JSONFormat([Description]) & "," &
  """TransactionCode"": {""Value"": " & JSONFormat([TransCode]) & ", ""Type"": ""transaction""}," &
  """ContractId"": {""Value"": " & JSONFormat([Trans#]) & ", ""Type"": ""transaction""}," &
  """Payments"": [{" &
    If([Follow Rent?] = false,
      """StartDate"": " & JSONFormat(Format([First Payment Date], "yyyy-MM-dd")) & "," &
      """Occurrences"": " & Text(If([Recurring Billable?] and [Number of Occurrences] > 0, [Number of Occurrences], 1)) & "," &
      """Frequency"": " & JSONFormat(If([Recurring Billable?] and [Recurring Frequency] <> null, [Recurring Frequency], "Monthly")) & ",",
      ""
    ) &
    """Amount"": " & Text([Fee Amount]) &
  "}]," &
  If([Equipment] <> null and [All Assets?] = false,
    """ProrationMethod"": {""Value"": ""1""}," &
    """RelatedAssets"": [{""AssetRecordId"": " & JSONFormat([ContractAssetId]) &
      If([Follow Rent?] = false,
        ", ""PaymentAmount"": " & Text([Fee Amount]) &
        ", ""FirstPaymentDate"": " & JSONFormat(Format([First Payment Date], "yyyy-MM-dd")),
        ""
      ) &
    "}]," &
    """DoNotRelateAssets"": false,",
    If([All Assets?] = true,
      """ProrationMethod"": {""Value"": ""2""}," &
      """RelatedAssets"": []," &
      """DoNotRelateAssets"": false,",
      """RelatedAssets"": []," &
      """DoNotRelateAssets"": true,"
    )
  ) &
  """DelinquencyCode"": null," &
  """InvoiceCode"": null," &
  """InvoiceLeadDays"": null," &
  """SuspendInvoicing"": true," &
  """BillingType"": null," &
  """FollowRent"": " & If([Follow Rent?], "true", "false") &
"}"
```

Then your Call URL body becomes just:

```json
<%*[API Payload]%>
```

That single passthrough block injects the pre-built JSON. Much cleaner in the Call URL editor.

---

## Step-by-Step Setup Checklist

### Before you start:
- [ ] Confirm Aspire Scheduled Billable endpoint URL with Cody
- [ ] Confirm auth method (Bearer token? Basic auth? API key header?)
- [ ] Verify `ContractAssetId` is accessible as a lookup on the Fees table (add if not)
- [ ] Verify `Trans#` on Fees maps correctly to the contract's transaction number in Aspire

### Build sequence:
1. [ ] Create the "API Payload" formula column on Fees table (the formula above)
2. [ ] Create a Custom Button on Fees table (or Dealer G&S table — your call)
3. [ ] Add a Call URL action to that button
4. [ ] Set Method = POST, URL = Aspire endpoint, Headers = auth + content-type
5. [ ] Set Body type = Text, body = `<%*[API Payload]%>`
6. [ ] Set Response format = JSON
7. [ ] Set On Error = Continue Execution
8. [ ] Add Error Message formula
9. [ ] Add assignments (Status, Date Sent)
10. [ ] Test with Scenario 1 record (simplest case)

---

## Testing Tip

Before connecting to real Aspire, you can test your payload shape by:
1. Setting the URL to a request bin (like https://webhook.site — gives you a temp URL to inspect the exact payload being sent)
2. Fire the button
3. Check what arrived at the request bin
4. Compare against the expected JSON from the blueprint

This lets you validate payload structure without risking Aspire data.

---

## What Each Scenario Produces

| Scenario | Equipment | All Assets? | Follow Rent? | Recurring? | Result |
|----------|-----------|-------------|-------------|------------|--------|
| 1 | null | false | false | no | DoNotRelateAssets=true, full Payments |
| 2 | set | false | false | no | 1 asset, full Payments |
| 3 | set (x2) | false | false | no | 2 separate calls, 1 asset each |
| 4 | set (x2) | false | false | no | Same as 3 + custom billing (Phase 2) |
| 5 | null | **true** | false | no | Empty RelatedAssets, DoNotRelate=false |
| 6 | set (x2) | false | false | **yes** | 2 calls, each with Occurrences + Frequency |
| 7 | null | false | **true** | — | DoNotRelateAssets=true, Amount-only Payments |
| 8 | set (x2) | false | **true** | — | 2 calls, AssetRecordId only, Amount-only Payments |
