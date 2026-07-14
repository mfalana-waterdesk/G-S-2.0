# G&S 2.0 — End User Guide: Submitting Scheduled Billable Charges

> **Created:** July 13, 2026 | **Author:** Malik Falana 

> **Last Modified:** July 13, 2026  | **Modified By:** Malik Falana

---

## What This Guide Covers

This guide walks you through submitting Goods & Services charges (like trip fees, cleaning fees, filter replacements, etc.) from the Water Desk to Aspire. Once submitted, the charges appear in Aspire automatically — no manual entry needed.

**Who this is for:** Taylor Jenkins (primary), Megan (admin support)

---

## Before You Start

- You need access to the **Dealer Goods & Services** table in the Water Desk
- You need a contract already selected on your G&S record
- You should know:
  - What type of charge you're entering (e.g., Cleaning Fee, Trip Charge)
  - The dollar amount
  - Whether it applies to a specific piece of equipment or not
  - When billing should start (First Payment Date)

---

## Step 1: Open Your G&S Record

1. Navigate to **Dealer Goods & Services**
2. Find the record for the contract you want to bill, or create a new one by selecting the contract using the Transaction# picker
3. The system will auto-fill contract details for you

---

## Step 2: Add Fee Line Items

1. Scroll to the bottom of the G&S record and click the **$ FEES** sub-tab
2. Click **NEW** to add a fee
3. Fill in the following fields:

| Field | What to Enter |
|-------|---------------|
| **Equipment** | Select the unit from the dropdown. Leave as "--None--" if the charge isn't tied to a specific piece of equipment. |
| **Serial Number** | Auto-fills when you select Equipment. Use this to verify you picked the right unit. |
| **Description** | Select the charge type: Ice Mats, Cleaning Fee, Trip Charge, CO2 Exchange, Filter, PM, Supplies, Damage, etc. |
| **Fee Amount** | Enter the dollar amount (e.g., 100.00) |
| **First Payment Date** | The date billing should start. **Required.** |
| **All Assets?** | Check this ONLY if you want Aspire to split the charge proportionally across ALL equipment on the contract. |
| **Recurring Billable?** | Check this if the charge repeats for multiple months. |
| **Recurring Frequency** | Only fill if Recurring Billable is checked. Options: Monthly, Quarterly, Annual. |
| **Number of Occurrences** | Only fill if Recurring Billable is checked. How many billing periods (e.g., "3" for 3 months). Leave blank if the charge should continue for the remainder of the contract term. |

4. Click **SAVE**
5. Repeat for additional fee line items if needed (for example, when splitting a charge across multiple pieces of equipment)

---

## Step 3: Send to Aspire

1. Go back to the main G&S record (click **BACK** or navigate up from the fee record)
2. Click the **SEND SCHEDULED BILLABLE TEST** button
3. The system will:
   - Gather all your saved fee line items
   - Send them to Aspire as one submission
   - Mark each fee as "Sent" when successful

That's it. Your charges are now in Aspire.

---

## Common Scenarios

### Scenario 1: One Charge, No Specific Equipment

Use this when the charge isn't tied to any particular unit (e.g., a general trip charge for the whole contract).

1. Add 1 fee in the **$ FEES** sub-tab
2. Leave **Equipment** as "--None--"
3. Leave **All Assets?** unchecked
4. Enter your **Description**, **Fee Amount**, and **First Payment Date**
5. Save
6. Go back to the G&S record and click **SEND SCHEDULED BILLABLE TEST**

---

### Scenario 2: One Charge to One Piece of Equipment

Use this when the charge is for a specific unit (e.g., a PM fee for a particular water cooler).

1. Add 1 fee in the **$ FEES** sub-tab
2. Select the unit from the **Equipment** dropdown
3. Verify the **Serial Number** auto-fills correctly
4. Enter your **Description**, **Fee Amount**, and **First Payment Date**
5. Save
6. Go back to the G&S record and click **SEND SCHEDULED BILLABLE TEST**

---

### Scenario 3: Split a Charge Across Multiple Pieces of Equipment

Use this when you need to divide a total amount between specific units (e.g., $25 to Unit A, $75 to Unit B).

1. Add a fee for the first unit:
   - Select the equipment
   - Enter that unit's portion of the total (e.g., $25.00)
   - Enter **Description** and **First Payment Date**
   - Save
2. Add another fee for the second unit:
   - Select the equipment
   - Enter that unit's portion (e.g., $75.00)
   - Enter **Description** and **First Payment Date**
   - Save
3. Repeat for additional units if needed
4. Go back to the G&S record and click **SEND SCHEDULED BILLABLE TEST**

All fees are sent together in one submission.

---

### Scenario 5: Charge Prorated Across ALL Equipment on the Contract

Use this when you want Aspire to automatically divide the amount proportionally across every unit on the contract.

1. Add 1 fee in the **$ FEES** sub-tab
2. Leave **Equipment** as "--None--"
3. Check the **All Assets?** checkbox
4. Enter your **Description**, **Fee Amount**, and **First Payment Date**
5. Save
6. Go back to the G&S record and click **SEND SCHEDULED BILLABLE TEST**

Aspire handles the proportional split — you only enter the total amount.

---

### Scenario 6: Recurring Monthly Charge

Use this for charges that repeat over multiple months (e.g., a filter replacement every month for 3 months).

1. Add your fee(s) following any of the scenarios above
2. Check the **Recurring Billable?** checkbox
3. Set **Recurring Frequency** to "Monthly" (or Quarterly/Annual as needed)
4. Set **Number of Occurrences** to the number of billing periods (e.g., "3" for 3 months)
   - Leave blank if the charge should continue for the remainder of the contract term
5. Save
6. Go back to the G&S record and click **SEND SCHEDULED BILLABLE TEST**

---

## Important Things to Know

- **Testing mode is active.** Charges appear in Aspire but will NOT generate invoices yet. This gives you a chance to verify everything looks correct without affecting customers.
- **Sent means sent.** Once a fee is marked "Sent," it cannot be re-sent or edited. If you need to correct a sent charge, contact **Kyle** or **Malik**.
- **One submission per click.** When you click the Send button, ALL saved fees on that G&S record go together. Make sure all your fee line items are entered and saved before clicking.
- **Serial Number is your safety check.** After selecting Equipment, always glance at the Serial Number column to confirm you picked the right unit.

---

## Troubleshooting

| Problem | What to Do |
|---------|------------|
| The **SEND SCHEDULED BILLABLE TEST** button doesn't appear | Make sure you have at least one fee record saved in the **$ FEES** sub-tab. |
| Error after clicking Send | Check that every fee has a **First Payment Date** and a **Fee Amount** filled in. Both are required. |
| Selected the wrong equipment | Edit the fee record, change the **Equipment** dropdown, and save. Then try sending again. |
| Charges not showing up in Aspire | Check the **Status** field on your fee records. If it says "Sent" — it went through; give Aspire a moment. If it says "Error" — contact Malik or Kyle. |
| Need to change a charge that's already sent | Contact Kyle or Malik. Sent charges cannot be modified from the Water Desk. |

---

## Quick Reference

| Action | Where |
|--------|-------|
| Add fee line items | **$ FEES** sub-tab (bottom of G&S record) |
| Send charges to Aspire | **SEND SCHEDULED BILLABLE TEST** button (main G&S record) |
| Verify equipment | Check **Serial Number** column after selecting Equipment |
| Make it recurring | Check **Recurring Billable?**, set Frequency and Occurrences |
| Prorate across all units | Check **All Assets?**, leave Equipment as "--None--" |

---

## Need Help?

- For system errors or corrections to sent charges: Contact **Kyle** or **Malik**
- For questions about which charge type to use: Check with your manager

---

## See Also

- G&S 2.0 Scenario Test Runbook (for detailed test validation steps)
