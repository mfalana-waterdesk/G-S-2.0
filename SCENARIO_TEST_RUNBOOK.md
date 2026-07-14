# G&S 2.0 — Scenario Test Runbook

> **Goal:** Test all 8 scenarios via the Call URL on the Fees table in CSP (AppID 76449).
> **Deadline:** Post each completed scenario to the group chat. End-user testing starts July 21.
> **Contract for all tests:** Transaction# `55854` (as used in Cody's spec examples)

---

## CRITICAL: API Payload Corrections

Based on the official API spec (`New Scheduled Billable API (1).docx`), the correct field structures are:

| Field | CORRECT Format | WRONG (what our formulas had) |
|-------|---------------|------|
| TransactionCode | `{"Code": "CLEANING"}` | ~~{"Value": "CLEANING", "Type": "transaction"}~~ |
| AssetRecordId | `{"Value": "80975", "Type": "transaction"}` | ~~plain string "80975"~~ |
| ProrationMethod | `{"Code": 1, "Description": "EquipmentCost"}` | ~~{"Value": "1"}~~ |
| DelinquencyCode | `{"Code": "15-15"}` or `null` | (was correct as null) |
| InvoiceCode | `{"Code": "BTS"}` or `null` | (was correct as null) |
| InvoiceLeadDays | `{"Code": "30"}` or `null` | (was correct as null) |

### AssetRecordId Type Values
- `"transaction"` — use when the value is a transaction number (e.g., "80975")
- `"record"` — use when the value is an Aspire Record ID (e.g., "APIE-108297")

Your `Equipment ContractAssetId` column on Fees — check what format those IDs are in. If they look like "APIE-xxxxx", use `"record"`. If they're numeric like "80975", use `"transaction"`.

---

## Corrected Call URL Body (per Fee record)

This replaces the body in `CALL_URL_ACTIONS.md`:

```
{
  "RecordId": <%=[REF#]%>,
  "Description": <%=[Description]%>,
  "TransactionCode": {"Code": <%=[TransCode]%>},
  "ContractId": {
    "Value": <%=[API ContractId]%>,
    "Type": "transaction"
  },
  "Payments": [
    {
<%? [Follow Rent?] = false %>
      "StartDate": <%=Format([First Payment Date], "M/d/yyyy")%>,
      "Occurrences": <%=If([Recurring Billable?] = true and [Number of Occurrences] > 0, Format([Number of Occurrences], "0"), "1")%>,
      "Frequency": <%=If([Recurring Billable?] = true and not IsNull([Recurring Frequency]), [Recurring Frequency], "Monthly")%>,
<%?%>
      "Amount": <%=Format([Fee Amount], "0.##")%>
    }
  ],
<%? not IsNull([Equipment]) or [All Assets?] = true %>
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
<%?%>
<%? not IsNull([Equipment]) and [All Assets?] = false %>
  "RelatedAssets": [
    {
      "AssetRecordId": {
        "Value": <%=[Equipment ContractAssetId]%>,
        "Type": "transaction"
      }
<%? [Follow Rent?] = false %>
      ,"PaymentAmount": <%=Format([Fee Amount], "0.##")%>,
      "FirstPaymentDate": <%=Format([First Payment Date], "M/d/yyyy")%>
<%?%>
    }
  ],
  "DoNotRelateAssets": false,
<%?%>
<%? [All Assets?] = true %>
  "RelatedAssets": [],
  "DoNotRelateAssets": false,
<%?%>
<%? IsNull([Equipment]) and [All Assets?] = false %>
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

### Key Notes:
- **Date format:** The spec uses `"6/9/2026"` (M/d/yyyy), not ISO format
- **Amount format:** The spec uses `"100"` not `"100.00"` — use `Format([Fee Amount], "0.##")` to drop trailing zeros
- **Occurrences:** Sent as a string `"1"` in the spec examples
- **SuspendInvoicing:** Set to `true` during testing (we don't want actual invoices generated)

---

## Test Records (Already Created in CSP)

From the Scenario Runthroughs doc, these G&S parent records already exist:

| Scenario | Record ID | URL |
|----------|-----------|-----|
| 1 | 49703 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49703 |
| 2 | 49704 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49704 |
| 3 | 49707 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49707 |
| 4 | (same structure as 3) | — |
| 5 | 49708 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49708 |
| 6 | 49709 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49709 |
| 7 | 49710 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49710 |
| 8 | 49711 | https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49711 |

---

## Scenario-by-Scenario Test Steps

### Scenario 1 — ✅ DONE
$100, no assets, no follow rent.

---

### Scenario 7 — No assets, Follow Rent (Do Next)

**Why next:** Same as Scenario 1 but adds FollowRent=true. Minimal change.

**Fee Record Setup:**

| Field | Value |
|-------|-------|
| Fee Amount | 100 |
| TransCode | CLEANING |
| Description | New SB Test 7 |
| Equipment | *(empty)* |
| All Assets? | ☐ unchecked |
| Follow Rent? | ☑ **checked** |
| First Payment Date | *(leave empty — not used)* |
| Recurring Billable? | ☐ unchecked |
| Number of Occurrences | *(empty)* |

**Expected JSON sent:**
```json
{
  "RecordId": "...",
  "Description": "New SB Test 7",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [{"Amount": "100"}],
  "RelatedAssets": [],
  "DoNotRelateAssets": true,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": true
}
```

**What to verify in Aspire:**
- Scheduled billable created with no assets
- Payment dates/frequency/occurrences match the contract's main stream
- Charge runs for remainder of contract term

**Use existing record:** https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49710

---

### Scenario 2 — Single Asset, No Follow Rent

**Fee Record Setup:**

| Field | Value |
|-------|-------|
| Fee Amount | 100 |
| TransCode | CLEANING |
| Description | New SB Test 2 |
| Equipment | **Pick one with a ContractAssetId** |
| All Assets? | ☐ unchecked |
| Follow Rent? | ☐ unchecked |
| First Payment Date | 8/1/2026 |
| Recurring Billable? | ☐ unchecked |

**Expected JSON sent:**
```json
{
  "RecordId": "...",
  "Description": "New SB Test 2",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [{"StartDate": "8/1/2026", "Occurrences": "1", "Frequency": "Monthly", "Amount": "100"}],
  "ProrationMethod": {"Code": 1, "Description": "EquipmentCost"},
  "RelatedAssets": [
    {
      "AssetRecordId": {"Value": "<ContractAssetId>", "Type": "transaction"},
      "PaymentAmount": "100",
      "FirstPaymentDate": "8/1/2026"
    }
  ],
  "DoNotRelateAssets": false,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": false
}
```

**Pre-check:** Open the fee record and confirm `Equipment ContractAssetId` (lookup column) shows a value. If it's empty, the Miscellaneous Unit doesn't have a ContractAssetId populated.

**Use existing record:** https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49704

---

### Scenario 5 — All Assets, No Follow Rent

**Fee Record Setup:**

| Field | Value |
|-------|-------|
| Fee Amount | 100 |
| TransCode | CLEANING |
| Description | New SB Test 5 |
| Equipment | *(empty)* |
| All Assets? | ☑ **checked** |
| Follow Rent? | ☐ unchecked |
| First Payment Date | 8/1/2026 |
| Recurring Billable? | ☐ unchecked |

**Expected JSON sent:**
```json
{
  "RecordId": "...",
  "Description": "New SB Test 5",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [{"StartDate": "8/1/2026", "Occurrences": "1", "Frequency": "Monthly", "Amount": "100"}],
  "ProrationMethod": {"Code": 1, "Description": "EquipmentCost"},
  "RelatedAssets": [],
  "DoNotRelateAssets": false,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": false
}
```

**What makes this different:** Empty RelatedAssets + DoNotRelateAssets=false tells Aspire to prorate across ALL assets on the contract proportionally.

**Use existing record:** https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49708

---

### Scenario 8 — Assets + Follow Rent

**Fee Record Setup (2 fee records under one G&S parent):**

**Fee A:**

| Field | Value |
|-------|-------|
| Fee Amount | 50 |
| TransCode | CLEANING |
| Description | New SB Test 8 |
| Equipment | **Asset #1** |
| All Assets? | ☐ unchecked |
| Follow Rent? | ☑ **checked** |
| Recurring Billable? | ☐ unchecked |

**Fee B:**

| Field | Value |
|-------|-------|
| Fee Amount | 50 |
| TransCode | CLEANING |
| Description | New SB Test 8 |
| Equipment | **Asset #2** |
| All Assets? | ☐ unchecked |
| Follow Rent? | ☑ **checked** |
| Recurring Billable? | ☐ unchecked |

**Per-fee approach:** Each fee fires separately. Each produces:
```json
{
  "RecordId": "...",
  "Description": "New SB Test 8",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [{"Amount": "50"}],
  "ProrationMethod": {"Code": 1, "Description": "EquipmentCost"},
  "RelatedAssets": [
    {"AssetRecordId": {"Value": "<ContractAssetId>", "Type": "transaction"}}
  ],
  "DoNotRelateAssets": false,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": true
}
```

**Note:** With FollowRent=true, we omit PaymentAmount and FirstPaymentDate from the asset object.

**⚠️ QUESTION FOR CODY:** The spec's Scenario 8 example INCLUDES PaymentAmount/FirstPaymentDate on assets even with FollowRent=true, but has a comment "Asset pricing is not available when FollowRent = true." This is contradictory. Ask Cody: "For Scenario 8, do I include PaymentAmount on the assets or omit it?" If the answer is include it, add those fields back.

**Use existing record:** https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49711

---

### Scenario 6 — Recurring, 3 months, 2 assets proportionally

**Fee Record Setup (2 fee records):**

**Fee A:**

| Field | Value |
|-------|-------|
| Fee Amount | 50 |
| TransCode | CLEANING |
| Description | New SB Test 6 |
| Equipment | **Asset #1** |
| All Assets? | ☐ unchecked |
| Follow Rent? | ☐ unchecked |
| First Payment Date | 8/1/2026 |
| Recurring Billable? | ☑ **checked** |
| Number of Occurrences | 3 |
| Recurring Frequency | Monthly |

**Fee B:** Same but with Asset #2.

**Per-fee approach:** Each fee sends independently:
```json
{
  "RecordId": "...",
  "Description": "New SB Test 6",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [{"StartDate": "8/1/2026", "Occurrences": "3", "Frequency": "Monthly", "Amount": "50"}],
  "ProrationMethod": {"Code": 1, "Description": "EquipmentCost"},
  "RelatedAssets": [
    {"AssetRecordId": {"Value": "<ContractAssetId>", "Type": "transaction"}}
  ],
  "DoNotRelateAssets": false,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": false
}
```

**Note on proportional:** The spec's Scenario 6 omits PaymentAmount/FirstPaymentDate from assets to get proportional distribution. But with per-fee calls, each asset gets its own billable with a specific amount. Ask Cody if this is acceptable or if proportional MUST be one call.

**Use existing record:** https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49709

---

### Scenario 3 — Multi-asset split ($25/$75)

**Fee Record Setup (2 fee records):**

**Fee A:**

| Field | Value |
|-------|-------|
| Fee Amount | 25 |
| Equipment | **Asset #1** (the one with ID "APIE-108297" or equivalent) |
| Follow Rent? | ☐ unchecked |
| First Payment Date | 8/1/2026 |

**Fee B:**

| Field | Value |
|-------|-------|
| Fee Amount | 75 |
| Equipment | **Asset #2** (the one with ID "80975" or equivalent) |
| Follow Rent? | ☐ unchecked |
| First Payment Date | 8/1/2026 |

**Per-fee approach:** Two independent calls. Fee A sends $25 to Asset 1, Fee B sends $75 to Asset 2.

**Use existing record:** https://waterdesk.teamdesk.net/secure/db/76449/preview.aspx?t=1110357&id=49707

---

### Scenario 4 — Multi-asset + Custom Billing (SKIP FOR NOW)

Same as Scenario 3 but with:
- `"DelinquencyCode": {"Code": "30-15"}`
- `"InvoiceCode": {"Code": "BTF"}`
- `"InvoiceLeadDays": {"Code": "30"}`
- `"BillingType": "Advanced"`
- StartDate adjusted +30 days for lead time

**Skip this for initial testing.** It's the same mechanical process as Scenario 3 with different null values filled in. Come back to it after 3 works.

---

## Multi-Asset Decision

For Scenarios 3, 6, and 8 — sending per-fee (one call per asset) vs. grouped (one call with multiple RelatedAssets):

**Per-fee works if:** Aspire is fine with 2 separate Scheduled Billables of $25 and $75 (instead of 1 Scheduled Billable of $100 split across 2 assets).

**You need grouped if:** The spec's behavior of proportional distribution, combined amounts, or single-billable tracking in Aspire is required.

**Action item:** Test Scenario 3 per-fee first. Then check in Aspire whether the result looks correct to Cody. If it does → you're done. If not → build the grouped approach (Option C).

---

## Test Execution Order (recommended)

| Order | Scenario | Complexity | Blockers |
|-------|----------|-----------|----------|
| ✅ | 1 | Simplest | Done |
| 1st | 7 | Same as 1 + FollowRent flag | None |
| 2nd | 2 | Adds one asset | Need Equipment with ContractAssetId |
| 3rd | 5 | All assets (empty array trick) | None |
| 4th | 8 | Asset + FollowRent | Clarify PaymentAmount question |
| 5th | 3 | Multi-asset (per-fee approach) | None |
| 6th | 6 | Recurring + multi-asset | None |
| Next | 4 | Custom billing codes | Needs billing code fields |

---

## Hooking Up to the UI

The end-user flow:

1. **Taylor opens a Dealer Goods & Services record** (already has contract selected)
2. **Clicks into the Fees sub-tab** → sees line items
3. **Adds fee records** with the appropriate flags
4. **Button on the G&S parent** → "Send to Aspire"
   - This button triggers a workflow that processes the Fees RecordSet
   - Each fee gets the Call URL fired
   - Status updates to "Sent" on success

**To wire this up:**
1. Create a **RecordSet column** on G&S filtered to Fees where `[Status] is empty`
2. Create a **Custom Button** on the G&S form: "Send to Aspire"
3. Button triggers an **Action Sequence**:
   - Action 1: **For Each Record** in the RecordSet → Call URL (the body above)
   - Action 2: On success → Set `[Status] = "Sent"`, `[Date Sent] = Now()`
   - Action 3: On error → Set `[Status] = "Error"`, `[Scheduled Billable API Error Message] = Response()`

The button lives on the **G&S form**, but the Call URL fires in the context of each **Fee record**. This is standard TeamDesk RecordSet + Call URL pattern.

---

## Pre-Flight Checklist

Before testing each scenario:

- [ ] Fee record has `API ContractId` populated (summary from parent G&S)
- [ ] Fee record has `REF#` populated
- [ ] Fee record has `TransCode` filled in
- [ ] If using Equipment: `Equipment ContractAssetId` lookup shows a value
- [ ] Call URL endpoint is correct and accessible
- [ ] Auth token is valid
- [ ] `SuspendInvoicing` is set to `true` in the body (safety net)
