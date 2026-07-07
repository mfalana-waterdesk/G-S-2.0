# G&S 2.0 — API Build Blueprint (with Real Field Mapping)

> Your task: For each Fee record in CSP, build the correct JSON payload and POST it to Aspire's Scheduled Billable endpoint.
> Target: ~5 hours of focused work.

---

## The Data You're Working With

### Source Tables (CSP — AppID 76449)

**Dealer Goods & Services** (table 1110357) — The parent record per submission:
- `Trans#` — the contract transaction number
- `Select Your Transaction#` — the contract reference (links to Contract table)
- `First Payment Date` — default first payment date
- `Ref#` — auto-generated reference
- `Selected ContractId` — lookup to Contract table's ContractId
- `Delinquency Code` — lookup from Contract
- `Del Code Formula` — formula-derived delinquency code
- `ContractAssetId(Holding)` — holds asset IDs grabbed by the parallel process
- `Asset Grab Flag 2.0` / `Asset Grab Status 2.0` — controls the serial-grabbing process

**Fees** (table 1110358) — One record per line-item charge:
- `Fee Amount` — dollar amount
- `TransCode` — Aspire transaction code (CLEANING, PM, LIFT, FILTER, etc.)
- `Description` — description text
- `First Payment Date` — per-fee start date
- `Equipment` — reference to Miscellaneous Units (asset)
- `Serial Number` — lookup from Equipment
- `Follow Rent?` — checkbox
- `All Assets?` — checkbox
- `Recurring Billable?` — checkbox
- `Number of Occurrences` — numeric (blank = follow contract term)
- `Recurring Frequency` — Monthly / Quarterly / Annual
- `Trans#` — links back to parent G&S record
- `Status` — tracks sent/pending
- `REF#` — reference number
- `Date Sent` — timestamp when API call was made

**Miscellaneous Units** (table 1110356) — Equipment/asset records:
- `ContractAssetId` — the asset ID for Aspire's `RelatedAssets[].AssetRecordId`
- `Aspire Record Id` — alternative identifier
- `Serial Number` — physical serial
- `Id` — internal TeamDesk key (what `Equipment` field in Fees references)

**Contract** (table 763145) — Contract details:
- `ContractId` — Aspire contract identifier → `ContractId.Value`
- `Status` — contract status (check for "Renewal" to block FollowRent)
- `Delinquency Code` — for the API's `DelinquencyCode`
- `InvoiceCode` — for the API's `InvoiceCode`
- `InvoiceLeadDays` — for the API's `InvoiceLeadDays`
- `Billing Frequency` — contract's main billing frequency
- `--RecordId Value` / `--RecordId Type` — ContractId type info

---

## The Decision Logic (Per Fee Record)

For EACH Fee record that hasn't been sent (`Status` is null/empty):

```
┌─────────────────────────────────────────────────────────────┐
│ READ: Fee Amount, TransCode, Description, First Payment Date │
│       Equipment, Follow Rent?, All Assets?,                  │
│       Recurring Billable?, Number of Occurrences,            │
│       Recurring Frequency                                    │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ READ from parent G&S: Trans#, Selected ContractId,           │
│       Delinquency Code                                       │
│ READ from Misc Units (if Equipment set): ContractAssetId     │
│ READ from Contract: ContractId, Status, InvoiceCode,         │
│       InvoiceLeadDays, Billing Frequency                     │
└──────────────────────────┬──────────────────────────────────┘
                           ▼
              ┌─── BUILD PAYLOAD ───┐
              │                     │
              ▼                     ▼
   ┌── ASSET LOGIC ──┐    ┌── PAYMENT LOGIC ──┐
   │                  │    │                   │
   │ Equipment null   │    │ Follow Rent? ──┐  │
   │ + All Assets?    │    │   YES: Amount   │  │
   │   = false        │    │   only          │  │
   │ → DoNotRelate    │    │                 │  │
   │   Assets = true  │    │   NO: Full      │  │
   │                  │    │   Payments obj  │  │
   │ Equipment null   │    │   (StartDate,   │  │
   │ + All Assets?    │    │    Occurrences, │  │
   │   = true         │    │    Frequency,   │  │
   │ → Empty Related  │    │    Amount)      │  │
   │   Assets array   │    └─────────────────┘  │
   │   DoNotRelate    │                         │
   │   = false        │                         │
   │                  │                         │
   │ Equipment set    │                         │
   │ → Populate       │                         │
   │   RelatedAssets  │                         │
   │   with asset ID  │                         │
   └──────────────────┘                         │
              │                                 │
              └─────────┬───────────────────────┘
                        ▼
              ┌── SEND HTTP POST ──┐
              │ To Aspire endpoint │
              └────────┬───────────┘
                       ▼
              ┌── UPDATE FEE RECORD ──┐
              │ Status = "Sent"       │
              │ Date Sent = Now()     │
              └───────────────────────┘
```

---

## Payload Templates (Copy-Paste Ready)

### Template A: No Assets, No Follow Rent (Scenario 1)

When: `Equipment` = null, `All Assets?` = false, `Follow Rent?` = false

```json
{
  "RecordId": "{REF#}",
  "Description": "{Description}",
  "TransactionCode": {
    "Value": "{TransCode}",
    "Type": "transaction"
  },
  "ContractId": {
    "Value": "{ContractId from Contract table}",
    "Type": "transaction"
  },
  "Payments": [
    {
      "StartDate": "{First Payment Date: YYYY-MM-DD}",
      "Occurrences": 1,
      "Frequency": "Monthly",
      "Amount": {Fee Amount}
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

---

### Template B: Specific Asset(s), No Follow Rent (Scenarios 2, 3, 4)

When: `Equipment` is set, `Follow Rent?` = false

```json
{
  "RecordId": "{REF#}",
  "Description": "{Description}",
  "TransactionCode": {
    "Value": "{TransCode}",
    "Type": "transaction"
  },
  "ContractId": {
    "Value": "{ContractId}",
    "Type": "transaction"
  },
  "Payments": [
    {
      "StartDate": "{First Payment Date}",
      "Occurrences": {Number of Occurrences OR 1 if blank and not recurring},
      "Frequency": "{Recurring Frequency OR 'Monthly'}",
      "Amount": {Fee Amount}
    }
  ],
  "ProrationMethod": {
    "Value": "1"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": "{ContractAssetId from Misc Units}",
      "PaymentAmount": {Fee Amount},
      "FirstPaymentDate": "{First Payment Date}"
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

**For multiple assets (Scenario 3):** You'll have multiple Fee records with the same `REF#` — group them. Each fee's `Fee Amount` becomes that asset's `PaymentAmount`. The total `Payments[].Amount` = sum of all fee amounts in that group.

---

### Template C: All Assets, No Follow Rent (Scenario 5)

When: `All Assets?` = true, `Follow Rent?` = false

```json
{
  "RecordId": "{REF#}",
  "Description": "{Description}",
  "TransactionCode": {
    "Value": "{TransCode}",
    "Type": "transaction"
  },
  "ContractId": {
    "Value": "{ContractId}",
    "Type": "transaction"
  },
  "Payments": [
    {
      "StartDate": "{First Payment Date}",
      "Occurrences": {Number of Occurrences OR 1},
      "Frequency": "{Recurring Frequency OR 'Monthly'}",
      "Amount": {Fee Amount}
    }
  ],
  "ProrationMethod": {
    "Value": "2"
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

---

### Template D: No Assets, Follow Rent (Scenario 7)

When: `Equipment` = null, `Follow Rent?` = true

```json
{
  "RecordId": "{REF#}",
  "Description": "{Description}",
  "TransactionCode": {
    "Value": "{TransCode}",
    "Type": "transaction"
  },
  "ContractId": {
    "Value": "{ContractId}",
    "Type": "transaction"
  },
  "Payments": [
    {
      "Amount": {Fee Amount}
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

---

### Template E: Specific Asset(s), Follow Rent (Scenario 8)

When: `Equipment` is set, `Follow Rent?` = true

```json
{
  "RecordId": "{REF#}",
  "Description": "{Description}",
  "TransactionCode": {
    "Value": "{TransCode}",
    "Type": "transaction"
  },
  "ContractId": {
    "Value": "{ContractId}",
    "Type": "transaction"
  },
  "Payments": [
    {
      "Amount": {Fee Amount}
    }
  ],
  "ProrationMethod": {
    "Value": "1"
  },
  "RelatedAssets": [
    {
      "AssetRecordId": "{ContractAssetId from Misc Units}"
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

---

### Template F: All Assets, Follow Rent

When: `All Assets?` = true, `Follow Rent?` = true

```json
{
  "RecordId": "{REF#}",
  "Description": "{Description}",
  "TransactionCode": {
    "Value": "{TransCode}",
    "Type": "transaction"
  },
  "ContractId": {
    "Value": "{ContractId}",
    "Type": "transaction"
  },
  "Payments": [
    {
      "Amount": {Fee Amount}
    }
  ],
  "ProrationMethod": {
    "Value": "2"
  },
  "RelatedAssets": [],
  "DoNotRelateAssets": false,
  "DelinquencyCode": null,
  "InvoiceCode": null,
  "InvoiceLeadDays": null,
  "SuspendInvoicing": true,
  "BillingType": null,
  "FollowRent": true
}
```

---

## Pseudocode: The Actual Logic

```
FUNCTION buildScheduledBillablePayload(feeRecord, parentGS, contract, miscUnit):

  // --- Base fields (always present) ---
  payload = {
    RecordId: feeRecord.REF#,
    Description: feeRecord.Description,
    TransactionCode: { Value: feeRecord.TransCode, Type: "transaction" },
    ContractId: { Value: contract.ContractId, Type: "transaction" },
    SuspendInvoicing: true,
    FollowRent: feeRecord.FollowRent
  }

  // --- Payments ---
  IF feeRecord.FollowRent == true:
    payload.Payments = [{ Amount: feeRecord.FeeAmount }]
  ELSE:
    payment = {
      Amount: feeRecord.FeeAmount,
      StartDate: formatDate(feeRecord.FirstPaymentDate),
      Frequency: feeRecord.RecurringFrequency OR "Monthly"
    }
    IF feeRecord.RecurringBillable == true AND feeRecord.NumberOfOccurrences > 0:
      payment.Occurrences = feeRecord.NumberOfOccurrences
    ELSE:
      payment.Occurrences = 1
    END IF
    payload.Payments = [payment]
  END IF

  // --- Assets ---
  IF feeRecord.AllAssets == true:
    payload.RelatedAssets = []
    payload.DoNotRelateAssets = false
    payload.ProrationMethod = { Value: "2" }
  ELSE IF feeRecord.Equipment is not empty:
    asset = { AssetRecordId: miscUnit.ContractAssetId }
    IF feeRecord.FollowRent == false:
      asset.PaymentAmount = feeRecord.FeeAmount
      asset.FirstPaymentDate = formatDate(feeRecord.FirstPaymentDate)
    END IF
    payload.RelatedAssets = [asset]
    payload.DoNotRelateAssets = false
    payload.ProrationMethod = { Value: "1" }
  ELSE:
    // No equipment, not all assets → no assets
    payload.RelatedAssets = []
    payload.DoNotRelateAssets = true
  END IF

  // --- Custom billing (null = inherit from contract) ---
  payload.DelinquencyCode = null
  payload.InvoiceCode = null
  payload.InvoiceLeadDays = null
  payload.BillingType = null

  RETURN payload
END FUNCTION
```

---

## Multi-Asset Grouping (Scenarios 3, 4, 6, 8)

When multiple Fee records share the same `REF#` and have different `Equipment` values, they represent a **split across assets**. You need to:

1. Group Fee records by `REF#`
2. If group has >1 record with Equipment set:
   - Sum all `Fee Amount` → that's your `Payments[].Amount`
   - Each fee becomes one entry in `RelatedAssets[]`
   - Each asset's `PaymentAmount` = that fee's `Fee Amount`
3. Send ONE API call for the group (not one per fee)

```
FUNCTION buildMultiAssetPayload(feeGroup, parentGS, contract):
  // feeGroup = array of Fee records with same REF#

  totalAmount = SUM(feeGroup.map(f => f.FeeAmount))
  firstFee = feeGroup[0]  // Use first record for shared fields

  payload = buildScheduledBillablePayload(firstFee, parentGS, contract, null)
  payload.Payments[0].Amount = totalAmount

  payload.RelatedAssets = []
  FOR EACH fee IN feeGroup:
    miscUnit = lookupMiscUnit(fee.Equipment)
    asset = { AssetRecordId: miscUnit.ContractAssetId }
    IF firstFee.FollowRent == false:
      asset.PaymentAmount = fee.FeeAmount
      asset.FirstPaymentDate = formatDate(fee.FirstPaymentDate)
    END IF
    payload.RelatedAssets.PUSH(asset)
  END FOR

  RETURN payload
END FUNCTION
```

---

## Work Plan (5-Hour Target)

| Block | Time | What to do |
|-------|------|-----------|
| **1** | 1 hr | Set up the Call URL action shell: HTTP POST to Aspire endpoint, auth headers, capture response. Test with a hardcoded Scenario 1 payload to confirm connectivity. |
| **2** | 2 hr | Build `buildScheduledBillablePayload` logic in TeamDesk formulas. Handle all 5 template paths (A–E). Use the pseudocode above as your guide. |
| **3** | 1 hr | Build multi-asset grouping (REF#-based grouping, sum amounts, populate RelatedAssets array). |
| **4** | 0.5 hr | Wire up the Status/Date Sent update on success. |
| **5** | 0.5 hr | Spot-check: manually verify payload output for 2-3 scenarios before the meeting. |

### What to skip (do after meeting):
- Custom billing fields (Scenarios 4, 6 extras) — leave as null for now
- Error handling / retry mechanism
- Renewal status guard for FollowRent
- Proportional distribution without PaymentAmount (Scenario 6 nuance)

This gets you a working API call that handles the core 80% of scenarios. The custom billing and edge cases can be layered on after your test meeting.

---

## Quick Reference: TeamDesk → API Field

| You read from... | You put into... |
|-----------------|----------------|
| `Fees.REF#` | `RecordId` |
| `Fees.Description` | `Description` |
| `Fees.TransCode` | `TransactionCode.Value` |
| `Contract.ContractId` | `ContractId.Value` |
| `Fees.Fee Amount` | `Payments[].Amount` |
| `Fees.First Payment Date` | `Payments[].StartDate` (if not FollowRent) |
| `Fees.Recurring Frequency` | `Payments[].Frequency` (if not FollowRent) |
| `Fees.Number of Occurrences` | `Payments[].Occurrences` (if not FollowRent) |
| `Fees.Follow Rent?` | `FollowRent` |
| `Fees.All Assets?` | controls `RelatedAssets` + `DoNotRelateAssets` |
| `Fees.Equipment` → `Misc Units.ContractAssetId` | `RelatedAssets[].AssetRecordId` |
| `Fees.Fee Amount` (per asset) | `RelatedAssets[].PaymentAmount` (if not FollowRent) |
| `Fees.First Payment Date` | `RelatedAssets[].FirstPaymentDate` (if not FollowRent) |
| Always `true` during rollout | `SuspendInvoicing` |
| Always `"transaction"` | `TransactionCode.Type`, `ContractId.Type` |
