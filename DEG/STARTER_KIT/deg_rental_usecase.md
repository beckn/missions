# Use Case Document - DEG Application

## Use Case Name

Rental Flow - Renting Energy Sources

---

## Objective / Problem Statement

Enable users to search, view, and rent energy sources (such as batteries ) available for renting in the DEG-enabled network.

---

## Actors Involved

| Actor            | Role Description                                              |
| ---------------- | ------------------------------------------------------------- |
| Consumer         | User searching for renting energy sources                     |
| BAP (Buyer App)  | User-facing application initiating the search and rental flow |
| Gateway          | Forwards search request to multiple BPPs (Seller Apps)        |
| BPP (Seller App) | Provides catalog of energy sources available for rent         |

---

## Flow of Events / Steps

| Step No. | Action Description                                   | Actor              |
| -------- | ---------------------------------------------------- | ------------------ |
| 1        | Search for energy sources available for renting      | Consumer (via BAP) |
| 2        | Forward search request to Gateway                    | BAP                |
| 3        | Broadcast search to different BPPs                   | Gateway            |
| 4        | Send catalog of available energy sources             | BPPs               |
| 5        | Display catalog to user                              | BAP                |
| 6        | Select energy source and view details                | Consumer           |
| 7        | Select start time and end time for rental            | Consumer           |
| 8        | Proceed with checkout                                | Consumer           |
| 9        | Complete payment and confirm rental                  | Consumer           |
| 10       | Use the rented energy source for the selected period | Consumer           |

## Reference Implementation

Reference Implementation of the above discussed use case is present [here](https://deg-experience-sandbox.becknprotocol.io/rental)
