# G&S 2.0 — Scheduled Billable API: Meeting Status Summary

> **Created:** July 13, 2026 | **Last Modified:** July 13, 2026
>
> **Author:** Malik Falana | **Modified By:** Malik Falana

**Meeting Date:** July 15, 2026
**Presenter:** Malik Falana
**Audience:** Cody Seher (Project Lead)
**Document Level:** Process (SOP) — Meeting Status Summary

---

## Summary

All Scheduled Billable API testing was completed July 13, 2026 against the Aspire test environment. The API deadline of July 15 has been met. End-user testing begins July 21.

---

## What Was Accomplished

### API Calls Validated — All 200 OK, Result: "Commit"

| Scenario # | Description | G&S Record | Contract | Aspire Token |
|---|---|---|---|---|
| 1 | $100 without assets | REF#49793 | 42712 | 380afdc6... |
| 2 | $100 assigned to one asset (APIE-93067) | REF#49794 | 42712 | 52f8b278... |
| 3 | $100 split $25/$75 across two assets (APIMU-161186, APIE-153343) | REF#49797 | 104762 | ae3f734c... |
| 5 | $100 selecting all assets (proportional) | REF#49798 | 104762 | 7b24eafe... |
| 6 | $100 recurring 3 months, 2 assets, Occurrences=3 | REF#49799 | 104762 | 3aaa7e5b... |

### Architecture Summary

- Call URL fires from the **Dealer Goods & Services** parent record
- All child Fee records are automatically combined into a single API payload
- Multi-asset support works through a summary column that concatenates each fee's asset JSON
- DoNotRelateAssets correctly differentiates "no assets" (`true`) vs "all assets" (`false`)
- Button: **"Send Scheduled Billable Test"** on G&S Preview Page

### Key Technical Decisions

| Decision | Resolution |
|----------|------------|
| **FollowRent** | Dropped from scope (Scenarios 7 & 8 removed). Always sends `false`. |
| **SuspendInvoicing** | Always `true` during testing. Per Cody (06-30 meeting): flip to `false` after first week of production. |
| **ProrationMethod** | Always included per Cody's spec (present in all 8 Appendix A examples). |
| **AssetRecordId Type** | All assets use `"record"` type with APIE-/APIMU- prefixed IDs. |
| **ContractId Type** | Always `"transaction"` with numeric value. |
| **Dates** | yyyy-MM-dd format (ISO). |
| **Scenario 4 (Custom billing codes)** | Deferred — pending business decision on where end user specifies DelinquencyCode/InvoiceCode/BillingType. |

---

## What's Out of Scope

| Item | Reason |
|---|---|
| Scenario 4 — Custom billing codes | TBD: who decides DelinquencyCode/InvoiceCode/BillingType values? |
| Error retry / failure path | Acknowledged as future work (per 06-16 meeting) |
| Proportional distribution (Scenario 6 spec-style: omit PaymentAmount) | Confirm if needed vs. explicit dollar split |

---

## What Remains Before End-User Testing (07/21)

| Item | Estimated Effort | Status |
|---|---|---|
| *(Resolved)* Reorder button actions (Call URL should fire before marking fees Sent) | 5 min | Done (07/13) |
| Prevent duplicate sends (button condition) | 15 min | Not done |
| Button relabeling for production (remove "Test" from name) | 5 min | Not done |
| End-user guide for Taylor | Done | Created 07/13 |
| Migration from CSP test to BN WD production | TBD | Not started |

---

## Questions for This Meeting

1. **Scenario 4**: Are custom billing codes (DelinquencyCode, InvoiceCode, BillingType) needed for go-live, or can they be deferred?
2. **Proportional distribution**: Does Taylor ever need a "let Aspire decide amounts" option (omit PaymentAmount on assets), or is explicit dollar split always the workflow?
3. **SuspendInvoicing**: Confirm plan — `true` for first week of production, then flip to `false`?
4. **Production migration**: When do we move from CSP (test) to BN WD (production)? Same day as end-user testing, or separate phase?
5. **End-user testing format**: Does Taylor need a written test script, or just access to the system?
6. **Aspire validation**: Can Patrick/Cody confirm the 5 test billables landed correctly in Aspire Test 2?

---

## Go-Live Readiness Assessment

| Component | Status |
|---|---|
| API payload construction | Complete |
| Multi-asset support | Complete |
| Recurring billing support | Complete |
| All-assets proportional | Complete |
| No-assets one-time | Complete |
| DoNotRelateAssets logic | Complete (spec-correct) |
| Response handling | Complete (Call URL fires first, then marks fees Sent) |
| Error handling | Not built |
| Duplicate prevention | Not built |
| Production deployment | Not started |

---

## Bottom Line

The API layer is **functionally complete and validated**. Remaining work is operational safeguards (2–4 hours). The SuspendInvoicing safety net means even if something goes wrong in production, no actual invoices will be generated until manually enabled.


