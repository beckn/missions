
# Use Case Document - DEG Application

## Use Case Name
EV Charging Flow - Searching and Booking EV Charging Stations
---

## Objective / Problem Statement
Enable users to search, view, and book EV charging stations in the DEG-enabled network, with real-time charging status updates.

---

## Actors Involved

| Actor | Role Description |
|-------|------------------|
| Consumer | User searching and booking EV charging stations |
| BAP (Buyer App) | User-facing application initiating the search and booking flow |
| Gateway | Forwards search request to multiple BPPs (Charging Station Providers) |
| BPP (Seller App) | Provides catalog of available EV charging stations and charging quotes |


## Flow of Events / Steps

| Step No. | Action Description | Actor |
|---------|---------------------|-------|
| 1 | Provide current location and search for nearby EV charging stations | Consumer (via BAP) |
| 2 | Forward search request to Gateway | BAP |
| 3 | Broadcast search to different BPPs | Gateway |
| 4 | BPPs respond with /on_search containing list of charging stations | BPPs |
| 5 | Render search results on the map view | BAP |
| 6 | Select a particular EV charging station | Consumer |
| 7 | Provide either amount or required kWh for charging | Consumer |
| 8 | Receive charging quote based on input | BPP |
| 9 | Proceed with checkout and confirm the order | Consumer |
| 10 | On the order detail page, view the status of charging | Consumer |
| 11 | Receive real-time charging status updates via unsolicited /on_status calls | BPP |

---