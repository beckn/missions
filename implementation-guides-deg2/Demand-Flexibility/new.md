# DEG Implementation Guide - Demand Flexibility

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 12-05-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides integration guidelines for implementing demand-side flexibility programs over a Beckn-enabled open energy network. It showcases how energy providers and aggregators in San Francisco can publish flexible load reduction programs, enabling users to participate in grid-responsive activities and earn monetary rewards for reducing electricity usage during peak hours.

This guide assumes the reader is familiar with the Beckn protocol, message flow, and schema standards. It maps the JSON-based catalog provided by participating BPPs to the Beckn on_search response pattern.

## Structure of the document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualisation) - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
2. General Flow diagrams - This section is relevant to all the messages flows illustrated below and discussed further in the document. We can refer [this](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams) section in the generic implementation guide to understand the flow.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. [Taxonomy and layer 2 configuration](#taxonomy-and-layer-2-configuration) - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
5. Notes on writing/integrating with your own software - We can refer [this](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software) section in the generic implementation guide.
6. [Links to artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Regitry/Gateway and BPP.

## Outcome Visualisation

### Use case - Discovery, order and fulfillment of Demand Flexibility

This use cases uses the names "Pulse Energy" and "Kazam" as examples for illustration.

- Srilekha is an IT professional who travels for work five days a week. She uses her four-wheeler electric vehicle (EV) for her commute. She prefers to charge her vehicle during her work hours and looks for suitable charging stations near her workplace.

**Discovery**:

- Srilekha begins to browse the Pulse Energy app to search for nearby EV chargers for four-wheelers.

- She receives a catalogue of 4 available chargers provided by Pulse energy & Kazam. Among them, she finds one which is located at a distance of 500 metres, which was in the catalogue of Kazam.

**Order**:

- Srilekha selects the charger, opting for a charging session at the cost of Rs. 13/kWh for a duration of 1 hour.

- She accepts the terms of order and is prompted to choose a payment method (card or link).

- She chooses to make payment through the link and confirms the order.

- The order is confirmed by Kazam's charger provider and verifies the payment and generates an order ID

**Fulfillment**:

- Srilekha plugs the kazam's charger into her vehicle and initiates the charging process. After an hour, the charging stops, and she is prompted to either remove the charger or continue charging. Srilekha removes the charger from her vehicle, thus ending the charging process.

**Post Fulfilment**: 

- Srilekha rates her experience using a 0-5 star rating.

## API Calls and Schema

### search

Search request can contain one or more search criterion within it. Use the following list on how to specify the criterion.

- The location to search around is specified in the message->intent->fulfillment->stops[0]->location field.
- The connector type required is specified in message->intent->fulfillment->tags[0]->list[0]. Refer to the taxonomy section below for the list of connector types supported.
- If searching by free text, it is specified in message->intent->descriptor->name
- If searching by category, it is specified in message->intent->category->descriptor->code

### on_search

**on_search with catalog of results**

- The catalog that comes back has a list of providers.
- Each provider has a list of items.
- Each item is the catalog listing for a charging station.

### select

**sending a select request**

- Choose the item(s) from the list from on_search and request quote
- The chosen item is in message->order->item_id

### on_select

- The BPP returns back with a quote for the selection
- It is in message->order->quote

### init

**send init request**

- The draft order including billing details.
- Billing details specified in message->order->billing

### on_init

- Contains payment terms. Payment terms specified in message->order->payments
- Cancellation terms specified in message->order->cancellation_terms
- Here we show the BPP as payment collector. In case the BAP specifies that it collects the payment in the init, the url field within payments will be empty

### confirm

- Confirm order including payment paid info (when applicable).
- It is in message->order->payments

### on_confirm

- Order confirmed. Charging can start.

### status

- Request for status on order. order_id is specifiedin message->order_id

### on_status

- Status of requested order.
- Primarily the fulfillment status is specified in message->order->fulfillments[]->state
- The message->order->fulfillments[]->state->descriptor->long_desc can be used to specify the OCPP status of the charge point. This can help the BAP to construct a custom detailed UI for charging status.

### update (start charging)

- Update the state of the fulfillment to start-charging.
- This will trigger charging start.

### on_update (start charging)

- Confirmation of successful start of charging operation
- State in message->order->fulfillments[]->state

### update (end charging)

- Request to change fulfillment state to end-charging

### on_update (end charging)

- Confirmation of successful operation to end charging
- The state will be in message->order->fulfillments[]->state

### support

- Request for support information.
- If regarding a specific order, specify the order_id in message->support->ref_id

### on_support

- Contains support information. If integrated with a CMS, the URL could contain order specific support info
- In all other cases, contains general support info (message->support)

### track

- Request for a tracking URL for the order. Order Id must be specified in message->order_id

### on_track

- Tracking URL with custom UI on charging process.
- The tracking URL will be in message->tracking->url

### cancel

- Request to cancel an order. order_id to be specified in message->order_id
- In the case where there is no advance booking of the charging slot, this does not have much relevance

### on_cancel

- Confirmation of cancelled order. The cancelled order will be in message->order
- In cases where advanced booking of charging station is not supported, this message is not relevant.

### get_rating_categories
- BAP can fetch a list of categoried for which a BPP supports prviding rating
- BAP does not need to send anything in the message field.

### rating_categories

- The BPP responds with a list of categories in which ratings can be provided.

### rating

- Rate different categories including fulfillment. Specify in message->ratings
- Specify a rating category from the list recieved in the rating_categories callback from the BPP.
- Value should be a decimal between 0 and 5.
- Id can be the order id.

### on_rating

- Confirmation of rating.
- Optionally send a form for additional input (message->feedback_form). Empty message otherwise.

## Taxonomy and layer 2 configuration

## Links to artefacts

- [Postman collection for Demand Flexibility]()
- [Layer2 config for UEI Demand Flexibility]()

## Sandbox Details

![<Sandbox Name>]()

### Registry/Gateway:

- **Gateway Sandbox:** []()
- **Registry Sandbox:** []()

### BPP:

- **BPP Client Sandbox:** []()
- **BPP Network Sandbox:** []()
- **BPP Playground Sandbox:** []()