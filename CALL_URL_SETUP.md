# G&S 2.0 — Call URL Action Setup (DEFINITIVE)

> **This is the one file you need.** Copy-paste everything below into TeamDesk.
> **Table:** Fees
> **Database:** CSP (AppID 76449)
> **Last Updated:** July 13, 2026
> **Scenario 1 tested:** ✅ Working

---

## Quick Context

You have ONE Call URL action on the Fees table. The conditional blocks (`<%? %>`) inside the body handle ALL 8 scenarios automatically — the same Call URL fires for every fee record regardless of which scenario it is. The flags on the fee record (Equipment, All Assets?, Follow Rent?, Recurring Billable?) control which JSON sections get included.

---

## Step 1: Call URL Action Setup

Go to: **Setup → Tables → Fees → Triggers / Actions → [your existing Call URL action]**

If you don't have one yet:
- Setup → Tables → Fees → Actions → Add → Call URL

---

## Step 2: Method

```
POST
```

---

## Step 3: URL

```
<%* Var[Aspire Scheduled Billable Endpoint] %>
```

Or hardcode it if you don't have a DB variable:
```
https://YOUR-ASPIRE-INSTANCE/api/v1/ScheduledBillables
```

> Replace with the actual endpoint Cody gave you.

---

## Step 4: Headers

```
Content-Type: application/json
Authorization: Bearer <%* Var[Aspire API Token] %>
```

Or if using a TeamDesk 3rd-Party Account, select it in the auth dropdown and skip the Authorization header.

---

## Step 5: Body (TEXT format — paste this EXACTLY)

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

---

## Step 6: Response Format

```
JSON
```

---

## Step 7: On Error

```
Continue Execution
```

(So we can still write the error status back to the record)

---

## Step 8: Assignments (what happens after the call)

### On Success (HTTP 200 or 201):

| Column | Formula |
|--------|---------|
| `Status` | `"Sent"` |
| `Date Sent` | `Now()` |
| `Scheduled Billable API Response Status` | `ToText(ResponseStatus())` |
| `Scheduled Billable API Response Body` | `Response()` |
| `Scheduled Billable API Error Message` | (leave empty / null) |

### On Error (HTTP 4xx/5xx):

| Column | Formula |
|--------|---------|
| `Status` | `"Error"` |
| `Scheduled Billable API Response Status` | `ToText(ResponseStatus())` |
| `Scheduled Billable API Response Body` | `Response()` |
| `Scheduled Billable API Error Message` | See Error Message formula below |

### Combined Assignment Formulas (use these if TeamDesk doesn't separate success/error):

**Status:**
```
If(ResponseStatus() >= 200 and ResponseStatus() < 300, "Sent", "Error")
```

**Date Sent:**
```
If(ResponseStatus() >= 200 and ResponseStatus() < 300, Now(), null)
```

**Scheduled Billable API Response Status:**
```
ToText(ResponseStatus())
```

**Scheduled Billable API Response Body:**
```
Response()
```

**Scheduled Billable API Error Message:**
```
If(ResponseStatus() >= 400, Concat("HTTP ", ToText(ResponseStatus()), ": ", Response()), null)
```

---

## Step 9: Trigger / Button Configuration

### Option A: Button on each Fee record (for testing)

- Add a **Custom Button** to the Fees form
- Name: "Send to Aspire"
- Action: The Call URL action above
- Condition (optional): `[Status] <> "Sent"` (prevent re-sending)

### Option B: Button on parent G&S record (for production)

- Add a **Custom Button** on the Dealer Goods & Services form
- Name: "Send All Fees to Aspire"
- Action type: **For Each Record** using a RecordSet column
- RecordSet: Create one on G&S table filtered to child Fees where `[Status] is empty`
- For each record in the set → fire the Call URL action above

> **Start with Option A for testing.** Switch to Option B once all scenarios pass.

---

## What Each Scenario Produces

Here's the exact JSON output for each scenario (so you can compare against what Aspire receives):

### Scenario 1 — No assets, no follow rent ✅ TESTED
```json
{
  "RecordId": "GS-00001",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "StartDate": "8/1/2026",
      "Occurrences": "1",
      "Frequency": "Monthly",
      "Amount": "100"
    }
  ],
  "RelatedAssets": [],
  "DoNotRelateAssets": true,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": false
}
```

Fields active: `REF#`, `Description`, `TransCode`, `API ContractId`, `First Payment Date`, `Fee Amount`
Fields that matter: Equipment=empty, All Assets=unchecked, Follow Rent=unchecked

---

### Scenario 7 — No assets, follow rent
```json
{
  "RecordId": "GS-00007",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "Amount": "100"
    }
  ],
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

**Difference from Scenario 1:** No StartDate/Occurrences/Frequency in Payments. FollowRent=true.
**Fee record setup:** Same as 1 but check `Follow Rent?` and leave `First Payment Date` empty.

---

### Scenario 2 — One asset, no follow rent
```json
{
  "RecordId": "GS-00002",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "StartDate": "8/1/2026",
      "Occurrences": "1",
      "Frequency": "Monthly",
      "Amount": "100"
    }
  ],
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": {
        "Value": "80975",
        "Type": "transaction"
      },
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

**Fee record setup:** Pick an Equipment from the dropdown. Confirm `Equipment ContractAssetId` shows a value.

---

### Scenario 5 — All assets, no follow rent
```json
{
  "RecordId": "GS-00005",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "StartDate": "8/1/2026",
      "Occurrences": "1",
      "Frequency": "Monthly",
      "Amount": "100"
    }
  ],
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
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

**Key:** Empty RelatedAssets + DoNotRelateAssets=false = Aspire prorates across ALL contract assets.
**Fee record setup:** Check `All Assets?`, leave `Equipment` empty.

---

### Scenario 8 — Assets + follow rent (per-fee approach: 2 separate calls)

**Fee A ($50 to Asset 1):**
```json
{
  "RecordId": "GS-00008a",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "Amount": "50"
    }
  ],
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": {
        "Value": "80975",
        "Type": "transaction"
      }
    }
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

**Note:** No PaymentAmount/FirstPaymentDate on the asset because FollowRent=true.
**Fee record setup:** Check `Follow Rent?`, pick Equipment, enter $50.

---

### Scenario 3 — Multi-asset split (per-fee approach: 2 separate calls)

**Fee A ($25 to Asset 1):**
```json
{
  "RecordId": "GS-00003a",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "StartDate": "8/1/2026",
      "Occurrences": "1",
      "Frequency": "Monthly",
      "Amount": "25"
    }
  ],
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": {
        "Value": "APIE-108297",
        "Type": "record"
      },
      "PaymentAmount": "25",
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

**Fee B ($75 to Asset 2):** Same structure, different asset/amount.

---

### Scenario 6 — Recurring 3 months (per-fee approach)

```json
{
  "RecordId": "GS-00006",
  "Description": "Cleaning Fee",
  "TransactionCode": {"Code": "CLEANING"},
  "ContractId": {"Value": "55854", "Type": "transaction"},
  "Payments": [
    {
      "StartDate": "8/1/2026",
      "Occurrences": "3",
      "Frequency": "Monthly",
      "Amount": "50"
    }
  ],
  "ProrationMethod": {
    "Code": 1,
    "Description": "EquipmentCost"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": {
        "Value": "80975",
        "Type": "transaction"
      }
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

**Fee record setup:** Check `Recurring Billable?`, set `Number of Occurrences` = 3, `Recurring Frequency` = "Monthly". Pick Equipment.

**Note:** For proportional distribution (spec says omit PaymentAmount), the per-fee approach sends a specific amount per asset. If Aspire needs proportional on ONE call, we'll need the grouped approach.

---

## AssetRecordId Type — How to Determine

Your `Equipment ContractAssetId` lookup will have values like:
- `"80975"` → these are **transaction** type
- `"APIE-108297"` → these are **record** type

**In the Call URL body above, I hardcoded `"Type": "transaction"`.** If you have assets with APIE- prefix IDs, you'll need to add a formula column to Fees:

### New Formula Column: `API Asset Id Type`
**Type:** Formula - Text

```
If(
  Begins(Nz([Equipment ContractAssetId]), "APIE-"), "record",
  "transaction"
)
```

Then change the Call URL body's AssetRecordId section to:
```
      "AssetRecordId": {
        "Value": <%=[Equipment ContractAssetId]%>,
        "Type": <%=[API Asset Id Type]%>
      }
```

> Only do this if you actually have mixed ID formats. If all your ContractAssetIds are numeric, "transaction" is fine for all.

---

## Differences From What You Used for Scenario 1

If Scenario 1 worked with the format you already have, here's what to check:

| Field | Spec says | You may have sent | Impact |
|-------|-----------|-------------------|--------|
| TransactionCode | `{"Code": "CLEANING"}` | `{"Value": "CLEANING", "Type": "transaction"}` | Aspire might accept both — if 1 worked, keep your format |
| Amount | `"100"` (string) | `100` (number) | Likely both work |
| Dates | `"6/9/2026"` | `"2026-06-09"` | If 1 worked with ISO, keep ISO |

**Bottom line:** If Scenario 1 passed, DON'T change the format for the fields that worked. Only adjust the new sections (RelatedAssets, ProrationMethod, FollowRent logic) which haven't been tested yet.

---

## Testing Sequence

1. **Scenario 7** — just check Follow Rent? on a fee record and fire. Confirms the conditional block for FollowRent works.
2. **Scenario 2** — pick an equipment, fire. Confirms RelatedAssets + ProrationMethod section.
3. **Scenario 5** — check All Assets?, fire. Confirms the "All Assets" conditional path.
4. **Scenario 8** — equipment + Follow Rent, fire. Confirms both conditionals together.
5. **Scenario 3** — two fee records, fire each. Confirms per-fee multi-asset works.
6. **Scenario 6** — recurring flags, fire. Confirms occurrences/frequency logic.

Post each to the group chat as you complete it (per Cody's instruction from 07-08 meeting).
