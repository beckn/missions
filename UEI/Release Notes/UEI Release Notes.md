# UEI Implementation Guide Release Notes

#### Version 1.0

1. Released the initial version of the document, providing a baseline framework and structure for the content. This version lays the foundation for subsequent updates and improvements.

#### Version 1.1

1. Incorporated valuable input and suggestions received from participants during the Winroom discussions.
2. The feedback addressed key aspects of the document, helping to refine the content and ensure alignment with stakeholders' expectations and requirements.

#### Version 1.2

1. Further refined the document by integrating additional feedback received from participants after broader reviews.
2. These updates focused on improving clarity, consistency, and usability based on practical insights and collaborative discussions.

#### Version 1.3

1. Updated the document to include a significant structural change by moving the connector specification tag groups from the "fulfillments" section to the "items" section.
2. This change was made to better reflect the logical grouping of these tags, enhancing the document's accuracy, usability, and alignment with the intended design.

#### Version 1.4

1. Added `ON-FULFILLMENT` postpaid metered charging for DC chargers (CCS2/CHAdeMO) with cable-lock enforcement. Cable stays physically locked after charging ends; BPP sends a UPI collect request to the driver's VPA and issues OCPP `UnlockConnector` only after UPI approval.
2. Added `confirm (ON-FULFILLMENT)`, `on_confirm (ON-FULFILLMENT)`, `on_status (LIVE-METER)`, `on_update (SESSION-SUMMARY — amount due)`, and `on_update (Payment Confirmed — cable released)` examples.
3. Added `LIVE-METER` tag group (used in `on_status`) with `energy-delivered`, `cost-so-far`, and `rate` fields for real-time kWh/cost display during active sessions.
4. Added `SESSION-SUMMARY` tag group (used in `on_update`) with `total-energy`, `total-cost`, `session-start`, and `session-end` fields for post-session settlement.
5. Added new fulfillment state codes: `vehicle-getting-charged`, `cable-released`.
6. Updated taxonomy table (rows 16, 19, 29–37): added `ON-FULFILLMENT` to payment type enum; documented `LIVE-METER` tags (rows 29–31), `SESSION-SUMMARY` tags (rows 32–35), and new payment params `virtual_payment_address`, `transaction_id` (rows 36–37).
