
# Use Case Document - DEG Application

## Use Case Name
P2P Energy Trading Flow - Trading Energy Between Consumers and Prosumers

---

## Objective / Problem Statement
Enable peer-to-peer energy trading between consumers (high energy demand users) and prosumers (users with surplus energy production) within the DEG-enabled network.

---

## Actors Involved

| Actor | Role Description |
|-------|------------------|
| Consumer | User submitting a buy trade due to high energy consumption or demand |
| Prosumer | User listing surplus energy for sale (typically rooftop solar owners with low consumption) |
| P2P Energy Trading Platform | Facilitates matching of buy and sell trades, and billing adjustments |

---

## Flow of Events / Steps

| Step No. | Action Description | Actor |
|---------|---------------------|-------|
| 1 | Open the P2P Energy Trading App | Consumer / Prosumer |
| 2 | Submit a buy trade request for energy | Consumer |
| 3 | List surplus energy available for sale | Prosumer |
| 4 | Collect all buy trades submitted by consumers | P2P Energy Trading Platform |
| 5 | Collate and match buy trades with available sell orders after cut-off time | P2P Energy Trading Platform |
| 6 | Finalize trade execution between consumers and prosumers | P2P Energy Trading Platform |
| 7 | Adjust the traded energy and billing in the monthly meter bill of respective users | P2P Energy Trading Platform |

