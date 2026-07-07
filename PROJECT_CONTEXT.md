# G&S 2.0 — Full Project Context

> **Last Updated:** July 7, 2026
> **Status:** Pre-testing / End-user presentation phase
> **Next Milestone:** End-user demo — Thursday, July 3, 2026 @ 12:30 PM Pacific

---

## 1. What This Project Is

**G&S 2.0** (Goods & Services 2.0) is a redesign of how **Bottleless Nation** enters and bills dealer goods-and-services charges. The goal is to replace a hard-coded, inflexible billing workflow with a line-item-based system where each charge is tied to specific equipment/assets and sent to **Aspire** (the billing/contract management platform) via a Scheduled Billable API.

The work lives inside the **Water Desk**, a dealer-facing application built on **TeamDesk** (a low-code database platform).

---

## 2. Systems & Identifiers

| System | Full Name | AppID | Role |
|--------|-----------|-------|------|
| **BN WD** | Bottleless Nation Water Desk | 101719 | Dealer-facing database where end users enter work |
| **CSP** | ~PWP Customer Service Portal | 76449 | Customer service portal; houses test records |
| **Aspire** | — | — | Core billing/contract management; receives API calls |
| **TeamDesk** | — | — | Platform both BN WD and CSP are built on |

---

## 3. Team

| Person | Role |
|--------|------|
| **Cody Seher** ("Patrick") | Project lead from Aspire/billing side; owns API specs |
| **Kyle Lawson** | Senior developer on Water Desk; leads the technical build |
| **Malik Falana** | Developer responsible for building API calls and testing |
| **Taylor Jenkins** | End user currently entering G&S charges manually |
| **Megan** | Admin support & end-user feedback |
| **Marie** | Reports on equipment-linking progress |
| **Gary** (deceased) | Previous developer who started an earlier attempt at this build |

---

## 4. Problem Statement

Today, one person (Taylor) manually enters goods-and-services charges in a "gross" hard-coded way — no line-item granularity, no equipment association. When field techs complete work orders (preventive maintenance, filter changes, lift charges, etc.), the resulting billables can't be tracked back to specific assets or controlled at a per-unit level.

**G&S 2.0 fixes this by:**
1. Letting end users enter charges on a per-unit, line-item basis inside Water Desk.
2. Associating each charge with specific equipment/serial numbers.
3. Constructing and sending a Scheduled Billable API call to Aspire for each entry.
4. Supporting both one-time and recurring charges, follow-rent logic, proration, custom billing settings, and multi-asset splits.

---

## 5. Architecture & Workflow

```
┌──────────────────────┐       ┌───────────────────────┐       ┌─────────────┐
│  End User (Taylor)   │──▶    │  Water Desk (BN WD)   │──▶    │   Aspire    │
│  enters G&S charges  │       │  builds API payload   │       │  creates    │
│  via GUI sub-tabs    │       │  via TeamDesk scripts  │       │  scheduled  │
└──────────────────────┘       └───────────────────────┘       │  billable   │
                                                                └─────────────┘
```

The GUI uses sub-tabs inside the **Dealer Goods and Services table** (similar to how credit applications have attached units). The system collects fees, equipment serials, contract info, and constructs the JSON payload.

---

## 6. Scheduled Billable API — Key Fields

| Field | Type | Notes |
|-------|------|-------|
| `RecordId` | string | Unique alpha-numeric identifier |
| `Description` | string | Description of the charge |
| `TransactionCode` | object | Type of charge (e.g., "CLEANING"); configured in Aspire Admin |
| `ContractId` | object | Contract identifier — Value + Type ("transaction", "record", or "ID") |
| `Payments` | array | Payment stream(s): StartDate, Occurrences, Frequency, Amount |
| `ProrationMethod` | object | 1=EquipmentCost, 2=EquipmentRental, 3=PurchaseOptionAmount |
| `RelatedAssets` | array | Assets the billable applies to (AssetRecordId, PaymentAmount, FirstPaymentDate) |
| `DoNotRelateAssets` | boolean | True = no assets associated |
| `DelinquencyCode` | object | Null = match contract's main stream |
| `InvoiceCode` | object | Null = match contract's main stream |
| `InvoiceLeadDays` | object | Days before due date to generate invoice |
| `SuspendInvoicing` | boolean | True = won't invoice until manually changed |
| `BillingType` | string | "Arrears", "Advanced", or null (match contract) |
| `FollowRent` | boolean | True = matches main payment stream's dates/frequency/occurrences |

### Critical API Rules

- **FollowRent = true**: Omit `StartDate`, `Occurrences`, `Frequency` from Payments; omit `PaymentAmount` and `FirstPaymentDate` from assets.
- **Multiple payment streams**: Amounts are COMBINED and used for ALL streams.
- **Asset payment amounts** must equal the total payment amount.
- **Empty RelatedAssets + DoNotRelateAssets=false** → ALL assets are selected and prorated proportionally.
- **Payment distribution** across assets is proportional based on rent percentage.
- **Cannot use FollowRent** on contracts in renewal status.
- **InvoiceLeadDays**: If specifying lead days, add them to the StartDate.

---

## 7. The 8 Test Scenarios

These scenarios cover the full range of billing configurations the system must support. Each has a corresponding test record in CSP (AppID 76449, table 1110357).

| # | Description | Key Flags | Record |
|---|-------------|-----------|--------|
| 1 | $100 without assets | DoNotRelateAssets=true, no proration | 49703 |
| 2 | $100 assigned to one asset | Single asset, full amount | 49704 |
| 3 | $100 split $25/$75 to two assets | Specific dollar amounts per asset | 49707 |
| 4 | $100 split $33.33/$66.67, custom billing | Advanced billing, 30-day lead, 30-day delinquency @ 15%, consolidated invoice | — |
| 5 | $100 selecting ALL assets | DoNotRelateAssets=false, empty RelatedAssets → proportional | 49708 |
| 6 | $100 to 2 assets proportionally, 3 months | Proportional (omit PaymentAmount), 30-day lead, 15-day delinquency, 3 occurrences | 49709 |
| 7 | $100 without assets, follow rent | FollowRent=true, remainder of contract term | 49710 |
| 8 | $100 to 2 assets, follow rent | FollowRent=true with specific asset amounts | 49711 |

### UI Mapping Notes

- **"Follow Rent" flag** → matches billing to contract main billing stream.
- **"All Assets" checkbox** → proportional proration across all equipment.
- **Blank "Number of Occurrences"** → charge runs until contract ends.
- **No Follow Rent** → user enters a first payment date instead.
- **Advanced billing / delinquency codes** → likely pulled from the Contract record (still TBD for end-user decision).
- **Scenario 8 implementation** → 2 fee records each with $50 and a recurring flag.

---

## 8. Key Technical Challenges (Resolved & Open)

### Resolved ✅

| Issue | Resolution | Date |
|-------|-----------|------|
| Multi-unit/single-asset serial problem (same serial repeated for all units) | Built a parallel process to grab individual serials correctly | 06-23-2026 |
| Per-unit G&S GUI | Demonstrated working with sub-tab approach | 06-16-2026 |
| Recurring charges proof-of-concept | Demonstrated working | 06-16-2026 |
| All 8 scenario UIs built | Ready for end-user review | 06-30-2026 |

### Open / TBD 🔲

| Issue | Notes |
|-------|-------|
| Record ID vs Transaction Number | Which identifier type to use for equipment — needs decision |
| Equipment linking to contracts | ~1,300 records still needed linking (being completed in parallel) |
| Failure path | What happens when an API call fails? Not yet addressed |
| Advanced billing / delinquency UI | Where does the end user specify these? Likely pulled from Contracts |
| Actual API testing through Aspire | Not yet started — pending end-user feedback |

---

## 9. Go-Live Strategy

1. **Build all UI scenarios** ✅ Complete
2. **Present to end users** (Taylor + Megan) for feedback — July 3, 2026
3. **Clean feedback → begin actual API testing** through Aspire
4. **Go-live with `SuspendInvoicing=true`** for the first week (catches errors before invoices go out)
5. **Validate first few batches**, then turn off SuspendInvoicing
6. **Post-implementation monitoring**

---

## 10. Meeting Timeline

| Date | Meeting | Key Outcomes |
|------|---------|--------------|
| 06-09-2026 | G&S 2.0 kickoff | Scope alignment, discovery items identified |
| 06-15-2026 | Initial Building Blocks (Kyle → Malik) | Walkthrough of existing TeamDesk structures, Gary's previous work |
| 06-16-2026 | Weekly sync | Per-unit GUI demo, recurring proof-of-concept, action items assigned |
| 06-23-2026 | Weekly sync | Multi-unit serial problem solved, parallel process built |
| 06-30-2026 | Weekly sync | All 8 scenarios UI-ready, Thursday demo prep |
| **07-03-2026** | **End-user presentation** | **Present to Taylor & Megan for feedback** |

---

## 11. File Inventory

| File | Contents |
|------|----------|
| `README.md` | App IDs and token placeholders for BN WD and CSP |
| `Reference Docs/G&S 2.0 - Transcript (06-09-26).docx` | Kickoff meeting transcript |
| `Reference Docs/G&S 2.0 - Transcript (06-16-26).docx` | Week 2 sync transcript |
| `Reference Docs/G&S 2.0 - Transcript (06-23-26).docx` | Week 3 sync transcript |
| `Reference Docs/G&S 2.0 - Transcript (06-30-26).docx` | Week 4 sync transcript |
| `Reference Docs/G&S 2.0 Initial Building Blocks - Transcript (06-15-26).docx` | Kyle's walkthrough for Malik |
| `Reference Docs/New Scheduled Billable API (1).docx` | Full API specification from Aspire |
| `Reference Docs/Scenario Example Runthroughs - GS 2.docx` | UI implementation notes for all 8 test scenarios |

---

## 12. Terminology Quick Reference

| Term | Meaning |
|------|---------|
| **G&S** | Goods & Services — billable charges for work performed |
| **Scheduled Billable** | A recurring or one-time charge record in Aspire |
| **Follow Rent** | Billable mirrors the contract's main payment stream dates/frequency |
| **Proration** | Distributing a charge proportionally across multiple assets |
| **Transaction Code** | The type/category of a charge (e.g., CLEANING, PM) in Aspire |
| **Asset / Related Asset** | A piece of equipment (water cooler, filter system) tied to a contract |
| **Suspend Invoicing** | Safety flag — charge exists but won't generate an invoice until toggled off |
| **Contract Main Stream** | The primary billing schedule on a customer's contract |
| **Lift Charge** | A one-time fee during equipment installation |
| **PM** | Preventive Maintenance |
