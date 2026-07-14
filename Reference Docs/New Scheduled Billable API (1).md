New Scheduled Billable API

The purpose of this document is to define the key/value pairs and differing structures for each type of Scheduled Billable API call. Example scenarios can be found in Appendix A, page

Key/Value Pairs Defined:

RecordId (string) – This is the record ID for the scheduled billable record. The field is alpha-numeric and is required. It is not required to be a unique value although it is highly recommended that uniqueness be enforced.
Example: “RecordId: “New_SB_00001”
Description (string) –  The description of the scheduled billable.
Example: “Description”: “Cleaning Fee”
TransactionCode (object) – Code that describes the type of charge being applied and uses a nested key/value pair. Transaction Codes can be found in Aspire Administration  Billing  Transaction Codes TAB.
Example: “TransactionCode”: {“Code”: “CLEANING”}
ContractId (object) – ID for the contract the scheduled billable will be sent to and uses nested key/values pairs. Record types include ID, Record, and Transaction. ID and Record are of type string, and Transaction is of type string or number.
Example:
“ContractId”: {
	“Value”: “12345”,
	“Type”: “transaction”
}







Payments (array of objects) – Defines the scheduled billable payment stream(s). Payment stream(s) are defined as object within the array. Multiple payment streams can be created. Each payment stream amount will be combined, and the total amount will be the value used for all payment streams. NOTE: “Frequency” will default to Monthly unless specified.
Example #1:
“Payments”: [
			{
				“StartDate”: {{First Payment Due Date}},
				“Occurrences”: {{Number of payments}},
				“Frequency”: {{Payment Frequency}},
				“Amount”: {{Amount}}
			}
		]

Example #2:
“Payments”: [
			{
				“StartDate”: {{First Payment Due Date}},
				“Occurrences”: {{Number of payments}},
				“Frequency”: {{Payment Frequency}},
				“Amount”: {{Total Amount*}}
			},
			{
				“StartDate”: {{First Payment Due Date}},
				“Occurrences”: {{Number of payments}},
				“Frequency”: {{Payment Frequency}},
				“Amount”: {{Total Amount*}}
			}
		]

*Total Amount – The amount of each payment stream will be added together and used for all payment streams. For example, if payment stream A is set to $100 and payment stream B is set to $150, then 2 payment streams will be created and each will have an amount of $250.



ProrationMethod (object) – Defines the proration method used for the scheduled billable and uses nested key/value pairs.
Proration Method Codes & Descriptions:

Example:
“ProrationMethod”: {
			“Code”: 1,
			“Description”: “EquipmentCost”
		}
RelatedAssets (array of objects) – Defines the asset(s) that the scheduled billable applies to. If no assets are defined and DoNotRelateAssets is set to false, then all assets will be selected.
AssetRecordId (object) – ID used to identify the asset in the contract. Can be a record ID or transaction number. Cannot be an asset ID.
PaymentAmount (number or string) – Amount to be allocated. NOTE: The total payment amount of all assets combined must equal the Payment Stream Amount. Do not include this key/value pair if FollowRent is set to true.
FirstPaymentDate (string) – First date of the payment. Must be a date between Payment Stream Start Date and Payment Stream Start Date + Payment Stream Frequency. This value is required if PaymentAmount is greater than $0.00 or if FollowRent is set to false. However, there is no effect on the scheduled billable.

Example #1:
“RelatedAssets”: [
			{
				“AssetRecordId”: {
					“Value”: “12345”,
					“Type”: “transaction”
                                                            },
                                                          “PaymentAmount”: “100.00”,
                                                         “FirstPaymentDate”: “5/26/2026”
			}
		]

Example #2:
“RelatedAssets”: [
			{
				“AssetRecordId”: {
					“Value”: “12345”,
					“Type”: “transaction”
				},
				“PaymentAmount”: “100.00”,
				“FirstPaymentDate”: “5/26/2026”
			},
			{
				“AssetRecordId”: {
					“Value”: “ABC123”,
					“Type”: “record”
				},
				“PaymentAmount”: “150.00”,
				“FirstPaymentDate”: “5/26/2026”
			}
		]

Example #3 (FollowRent = true):
“RelatedAssets”: [
			{
				“AssetRecordId”: {
					“Value”: “12345”,
					“Type”: “transaction”
				}
			},
			{
				“AssetRecordId”: {
					“Value”: “ABC123”,
					“Type”: “record”
				}
			}
		]
DoNotRelateAssets (boolean) – Use to specify whether assets should be related to the scheduled billable charge. If set to true, no assets will be selected for the scheduled billable.
Example:
“DoNotRelateAssets”: false
DelinquencyCode (object) – Use to specify the delinquency code for the scheduled billable. Omitting this key or setting it to ‘null’, will set this to whatever value is being used for the contracts Main Payment Stream. Delinquency Codes can be found in Aspire Administration  Billing  Delinquency Code TAB.
Example:
“DelinquencyCode”: {“Code”: “15-15”}
InvoiceCode (object) – Use to specify the Invoice Code for the scheduled billable. Omitting this key or setting it to ‘null’, will set this to whatever value is being used for the contracts Main Payment Stream. Invoice Codes can be found in Aspire Administration  Billing  Invoice Code TAB.
Example:
“InvoiceCode”: {“Code”: “BTS”}
InvoiceLeadDays (object) – Use to specify the number of days prior to the Invoice Due Date (Payment StartDate) the invoice will generate. Omitting this key or setting it to ‘null’, will set this to whatever value is being used for the contracts Main Payment Stream. Invoice Lead Days Codes can be found in Aspire Administration  Billing  Lead Days TAB.
Example:
“InvoiceLeadDays”: {“Code”: “30”}
SuspendInvoicing (boolean) – Use to set the scheduled billable to invoice or not. If set to true, scheduled billable will not invoice until a user manually changes it in the GUI. If set to false, scheduled billable will invoice when conditions are met.
Example:
“SuspendInvoicing”: false
BillingType (string) – Use to set the billing type of the scheduled billables. Can be set to “Arrears”, “Advanced”, or null. If set to null or omitted, the billing type of the main payment stream will be used.
Example:
“BillingType”: “Arrears”






FollowRent (boolean) – Use to set the scheduled billable to follow the Main Payment Stream. If set to true, Scheduled Billable First Payment Date, Occurrences, and Frequency will match the Main Payment Stream. NOTE: If set to true, DO NOT include the StartDate, Occurrences, and Frequency in the Payment array of objects. If assets have been specified, the PaymentAmount and FirstPaymentDate of each asset must be omitted. Cannot be used on contracts in renewal status.
Example:
“FollowRent”: false

Payment Amount Per Asset Distribution

The per asset payment amount is determined based on the percentage each asset contributes to the combined rent payment. If the number of assets specified in the call are less than the total number of assets on the contract, then only the monthly payment amounts for those specified assets will be used to determine the percentage for distribution.
Example #1:
- Contract has 5 assets
- Scheduled Billable is for all 5 assets
- Combined rent amount is $200.00
- Per asset rent amount:
- Asset 1: $25.00 | Percentage of rent amount: 12.5%
- Asset 2: $48.00 | Percentage of rent amount: 24.0%
- Asset 3: $37.00 | Percentage of rent amount: 18.5%
- Asset 4: $45.00 | Percentage of rent amount: 22.5%
- Asset 5: $45.00 | Percentage of rent amount: 22.5%
- Scheduled Billable amount is $100.00
- Per asset scheduled billable amount distribution:
- Asset 1: $12.50 ($100 x 12.5%)
- Asset 2: $24.00 ($100 x 24.0%)
- Asset 3: $18.50 ($100 x 18.5%)
- Asset 4: $22.50 ($100 x 22.5%)
- Asset 5: $22.50 ($100 x 22.5%)





Example #2:
- Contract has 5 assets
- Scheduled Billable is for 3 assets
- Combined rent amount is $107.00
- Per asset rent amount:
- Asset 1: $25.00 | Percentage of rent amount: 23.36%
- Asset 3: $37.00 | Percentage of rent amount: 34.58%
- Asset 5: $45.00 | Percentage of rent amount: 42.06%
- Scheduled Billable amount is $150.00
- Per asset scheduled billable amount distribution:
- Asset 1: $35.04 ($150 x 23.36%)
- Asset 3: $51.87 ($150 x 34.58%)
- Asset 5: $63.09 ($150 x 42.06%)



















Appendix A
Scenario #1:
Send Scheduled Billable for $100.00 without assets. Match billing to contract main billing. In Aspire, the effect will be a scheduled billable that is not associated with any assets, and will not be prorated.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “6/9/2026”,
“Occurrences”:”1”,
“Frequency”: “Monthly”,
“Amount”: “100”
}
],
“ProrationMethod”:{
“Code”: 1,
“Description”: “EquipmentCost”
},
“RelatedAssets”:[],
“DoNotRelateAssets”: true, //This is the key/value that determines if assets are associated.
“DelinquencyCode”: null,
“InvoiceCode”: null,
“InvoiceLeadDays”: null,
“SuspendInvoicing”: false,
“FollowRent”: false,
“BillingType”: null
}


Scenario #2:
Send Scheduled Billable for $100, assigned to one asset. Match billing to contract main billing.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “6/9/2026”,
“Occurrences”:”1”,
“Frequency”: “Monthly”,
“Amount”: “100”
}
],
“ProrationMethod”:{
“Code”: 1, // Always use this Code/Description unless told otherwise.
“Description”: “EquipmentCost”
},
“RelatedAssets”:[
{
“AssetRecordId”:{
“Value”: “80975”,
“Type”: “transaction”
},
“PaymentAmount”: “100”,
“FirstPaymentDate”: “6/9/2026”
},
],
“DoNotRelateAssets”: false,
“DelinquencyCode”: null
“InvoiceCode”: null,
“InvoiceLeadDays”: null,
“SuspendInvoicing”: false,
“FollowRent”: false,
“BillingType”: null
}


Scenario #3:
Send Scheduled Billable for $100, assign $25 to one asset and $75 to the other. Match billing to contract main billing.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “6/9/2026”,
“Occurrences”:”1”,
“Frequency”: “Monthly”,
“Amount”: “100” // Total Payment Amount
}
],
“ProrationMethod”:{
“Code”: 1, // Always use this Code/Description unless told otherwise.
“Description”: “EquipmentCost”
},
“RelatedAssets”:[ // Asset Payment Amount(s) must equal the Total Payment Amount above.
{
“AssetRecordId”:{
“Value”: “APIE-108297”,
“Type”: “record”
},
“PaymentAmount”: “25”,
“FirstPaymentDate”: “6/9/2026”
},
{
“AssetRecordId”:{
“Value”: “80975”,
“Type”: “transaction”
},
“PaymentAmount”: “75”,
“FirstPaymentDate”: “6/9/2026”
}
],
“DoNotRelateAssets”: false,
“DelinquencyCode”: null,
“InvoiceCode”: null,
“InvoiceLeadDays”: null,
“SuspendInvoicing”: false,
“FollowRent”: false,
“BillingType”: null
}

Scenario #4:
Send Scheduled Billable for $100, assign $33.33 to one asset and $66.67 to the other. Change billing to Advanced billing, 30 day invoice lead, 30 day delinquency at 15%, and consolidated invoice code.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “7/9/2026”, // If specifying the lead days, add to start date.
“Occurrences”:”1”,
“Frequency”: “Monthly”,
“Amount”: “100” // Total Payment Amount
}
],
“ProrationMethod”:{
“Code”: 1, // Always use this Code/Description unless told otherwise.
“Description”: “EquipmentCost”
},
“RelatedAssets”:[ // Asset Payment Amount(s) must equal the Total Payment Amount above.
{
“AssetRecordId”:{
“Value”: “APIE-108297”,
“Type”: “record”
},
“PaymentAmount”: “33.33”,
“FirstPaymentDate”: “6/9/2026”
},
{
“AssetRecordId”:{
“Value”: “80975”,
“Type”: “transaction”
},
“PaymentAmount”: “66.67”,
“FirstPaymentDate”: “6/9/2026”
}
],
“DoNotRelateAssets”: false,
“DelinquencyCode”: {“Code”: “30-15”}
“InvoiceCode”: {“Code”: “BTF”},
“InvoiceLeadDays”: {“Code”: “30”},
“SuspendInvoicing”: false,
“FollowRent”: false,
“BillingType”: “Advanced”
}

Scenario #5:
Send Scheduled Billable for $100, selecting all assets. Match billing to contract main billing. In Aspire, the amount specified will be prorated across all assets, proportionally.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “6/9/2026”,
“Occurrences”:”1”,
“Frequency”: “Monthly”,
“Amount”: “100”
}
],
“ProrationMethod”:{
“Code”: 1,
“Description”: “EquipmentCost”
},
“RelatedAssets”:[],
“DoNotRelateAssets”: false, //This is the key/value that determines if assets are associated.
“DelinquencyCode”: null,
“InvoiceCode”: null,
“InvoiceLeadDays”: null,
“SuspendInvoicing”: false,
“FollowRent”: false,
“BillingType”: null
}


Scenario #6:
Send Schedule Billable for $100, assign to 2 assets proportionally. Change billing to 30 day invoice lead, 15 day delinquency at 15%, not consolidated. End after 3 months.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “7/9/2026”, // If specifying the lead days, add to start date.
“Occurrences”:”3”,
“Frequency”: “Monthly”,
“Amount”: “100”
}
],
“ProrationMethod”:{
“Code”: 1, // Always use this Code/Description unless told otherwise.
“Description”: “EquipmentCost”
},
“RelatedAssets”:[      // Omit PaymentAmount and FirstPaymentDate for proportional amounts.
{
“AssetRecordId”:{
“Value”: “APIE-108297”,
“Type”: “record”
}
},
{
“AssetRecordId”:{
“Value”: “80975”,
“Type”: “transaction”
}
}
],
“DoNotRelateAssets”: false,
“DelinquencyCode”: {“Code”: “15-15”}
“InvoiceCode”: {“Code”: “BTS”},
“InvoiceLeadDays”: {“Code”: “30”},
“SuspendInvoicing”: false,
“FollowRent”: false,
“BillingType”: null
}


Scenario #7:
Send Scheduled Billable for $100 without assets, matching billing to contract main billing, for remainder of contract’s term.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “6/9/2026”,  // When FollowRent = true, omit StartDate, Occurrences and
“Occurrences”:”1”,        // Frequency.
“Frequency”: “Monthly”,
“Amount”: “100”
}
],
“ProrationMethod”:{
“Code”: 1,
“Description”: “EquipmentCost”
},
“RelatedAssets”:[], // Amount will be prorated proportionally across all available assets.
“DoNotRelateAssets”: true, //This is the key/value that determines if assets are associated.
“DelinquencyCode”: null,
“InvoiceCode”: null,
“InvoiceLeadDays”: null,
“SuspendInvoicing”: false,
“FollowRent”: true, // This is the key/value that will match the contracts terms
“BillingType”: null
}


Scenario #8:
Send Scheduled Billable for $100, assign to 2 assets, matching billing to contract main billing, for remainder of contract’s term.
{
“RecordId”: “NewSB_Test”,
“Description”: “New SB Test 1”,
“TransactionCode”: {“Code”: “CLEANING”},
“ContractId”:{
“Value”: “55854”,
“Type”: “transaction”
},
“Payments”:[
{
“StartDate”: “6/9/2026”,  // When FollowRent = true, omit StartDate, Occurrences and
“Occurrences”:”1”,        // Frequency.
“Frequency”: “Monthly”,
“Amount”: “100”
}
],
“ProrationMethod”:{
“Code”: 1,
“Description”: “EquipmentCost”
},
“RelatedAssets”:[           // Amount will be prorated proportionally across selected assets.
{
“AssetRecordId”:{
“Value”: “APIE-108297”,
“Type”: “record”
}
“PaymentAmount”: “33.33”, // Asset pricing is not available when FollowRent = true
“FirstPaymentDate”: “6/9/2026”  // Cannot set date unless PaymentAmount > 0
},
{
“AssetRecordId”:{
“Value”: “80975”,
“Type”: “transaction”
}
“PaymentAmount”: “66.67”,
“FirstPaymentDate”: “6/9/2026”
}
],
“DoNotRelateAssets”: false, //This is the key/value that determines if assets are associated.
“DelinquencyCode”: null,
“InvoiceCode”: null,
“InvoiceLeadDays”: null,
“SuspendInvoicing”: false,
“FollowRent”: true, // This is the key/value that will match the contracts terms
“BillingType”: null
}