
# Use Case Document - DEG Application

## Use Case Name
Retail Flow - Purchase and Financing of Energy Products

## Objective / Problem Statement
Enable users to search, purchase, finance, and manage energy retail products like batteries, solar panels, and accessories on the DEG-enabled network, with an option to publish purchased assets for renting.

---

## Actors Involved

| Actor | Role Description |
|-------|------------------|
| Consumer | User searching for and purchasing energy products |
| BAP (Buyer App) | User-facing application initiating the search and purchase flow |
| Gateway | Forwards search request to multiple BPPs (Seller Apps) |
| BPP (Seller App) | Provides product catalog in response to search |
| Energy Financing BPP | Provides catalog of financing options |
| Energy Wallet | Stores user transaction history , personal data and physical Assets|

---

## Flow of Events / Steps

| Step No. | Action Description | Actor |
|---------|---------------------|-------|
| 1 | Search for energy products (battery, solar panel, accessories) | Consumer (via BAP) |
| 2 | Forward search request to Gateway | BAP |
| 3 | Broadcast search to different BPPs | Gateway |
| 4 | Send catalog of products | BPPs |
| 5 | Display catalog to user | BAP |
| 6 | Select product and view details | Consumer |
| 7 | Add product to cart | Consumer |
| 8 | View and navigate to cart | Consumer |
| 9 | Proceed with checkout | Consumer |
| 10 | Search for financing options from network | BAP |
| 11 | Fetch financing catalog | BAP |
| 12 | Optionally sync previous transactions from wallet for better interest rates | Consumer |
| 13 | Display updated financing catalog with discounts | BAP |
| 14 | Select financing option and submit EMI application | Consumer |
| 15 | Optionally sync wallet to auto-fill personal details | Consumer |
| 16 | Financing approval and proceed to pay down payment | Energy Financing BPP + Consumer |
| 17 | Complete purchase | Consumer |
| 18 | Add purchased product as asset to wallet | Consumer |


# Use Case Name
Publishing Asset for Renting

---

## Objective / Problem Statement
Enable users to list their purchased physical assets (stored in wallet) for renting in the DEG network.

---

## Actors Involved

| Actor | Role Description |
|-------|------------------|
| Consumer | User listing an asset for renting |
| BAP | User-facing application |
| Energy Wallet | Stores user transaction history , personal data and physical Assets|

---

## Flow of Events / Steps

| Step No. | Action Description | Actor |
|---------|---------------------|-------|
| 1 | Navigate to Provide Rental service | Consumer |
| 2 | Select asset to publish for renting | Consumer |
| 3 | Provide start date, end date, and rental price | Consumer |
| 4 | Publish asset for renting | Consumer |
| 5 | Asset listed in network for potential renters | DEG Platform |

---
## Reference Implementation
Reference Implementation of the above discussed use case is present here.
1. [Retail App](https://exp.energy.becknprotocol.io/retail)
1. [Wallet App](https://exp.energy.becknprotocol.io/wallet)
1. [Finance App](https://exp.energy.becknprotocol.io/finance)
