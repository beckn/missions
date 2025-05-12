# DEG Implementation Guide - Connections

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 12-05-2025 | 1.0     | Initial Version                                     |

## Introduction

<>

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

### Use case - Discovery, order and fulfillment of Connections

<>

## API Calls and Schema

### search

Search request can contain one or more search criterion within it. Use the following list on how to specify the criterion.

- The location to search around is specified in the message->intent->fulfillment->stops[0]->location field.
- The connector type required is specified in message->intent->fulfillment->tags[0]->list[0]. Refer to the taxonomy section below for the list of connector types supported.
- If searching by free text, it is specified in message->intent->descriptor->name
- If searching by category, it is specified in message->intent->category->descriptor->code

```
{
    "context": {
        "domain": "service",
        "action": "search",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "transaction_id": "connection-txn-8001",
        "message_id": "connection-msg-10001",
        "timestamp": "2025-05-07T13:59:00Z",
        "ttl": "PT10M",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        }
    },
    "message": {
        "intent": {
            "descriptor": {
                "name": "Request for a new electricity connection"
            },
            "category": {
                "descriptor": {
                    "code": "electricity-connection"
                }
            }
        }
    }
}
```

### on_search

**on_search with catalog of results**

- The catalog that comes back has a list of providers.
- Each provider has a list of items.
- Each item is the catalog listing for a charging station.


```
{
    "context": {
        "domain": "service",
        "action": "on_search",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "bpp_id": "example-bpp.com",
        "bpp_uri": "https://example-bpp.com",
        "transaction_id": "connection-txn-8001",
        "message_id": "connection-msg-9001",
        "timestamp": "2025-05-07T14:00:00Z"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "sf_electric_authority",
                    "descriptor": {
                        "name": "San Francisco Electric Authority",
                        "short_desc": "Official provider for new electricity connections",
                        "long_desc": "Government utility responsible for delivering new residential and commercial electricity connections in San Francisco.",
                        "images": [
                            {
                                "url": "https://sfea.gov/logo.png"
                            }
                        ]
                    },
                    "rating": "4.8/5",
                    "categories": [
                        {
                            "id": "1",
                            "descriptor": {
                                "code": "electricity-connection",
                                "name": "Electricity Connection"
                            }
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "city": {
                                "name": "San Francisco"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "1",
                            "type": "ONSITE"
                        }
                    ],
                    "items": [
                        {
                            "id": "new-resi-connection",
                            "descriptor": {
                                "name": "Residential Electricity Connection"
                            },
                            "price": {
                                "estimated_value": "200",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "connection-type"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "property-type"
                                            },
                                            "value": "Residential"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "phase"
                                            },
                                            "value": "Single Phase"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "documentation"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "required-docs"
                                            },
                                            "value": "ID proof, ownership certificate, wiring diagram"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "new-comm-connection",
                            "descriptor": {
                                "name": "Commercial Electricity Connection"
                            },
                            "price": {
                                "estimated_value": "500",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "connection-type"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "property-type"
                                            },
                                            "value": "Commercial"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "phase"
                                            },
                                            "value": "Three Phase"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "documentation"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "required-docs"
                                            },
                                            "value": "Business license, wiring plan, ID proof"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    }
}
```

### init

**send init request**

- The draft order including billing details.
- Billing details specified in message->order->billing

```

```

### on_init

- Contains payment terms. Payment terms specified in message->order->payments
- Cancellation terms specified in message->order->cancellation_terms
- Here we show the BPP as payment collector. In case the BAP specifies that it collects the payment in the init, the url field within payments will be empty

```

```

### confirm

- Confirm order including payment paid info (when applicable).
- It is in message->order->payments

```

```

### on_confirm

- Order confirmed. Charging can start.

```

```

### status

- Request for status on order. order_id is specifiedin message->order_id

```

```

### on_status

- Status of requested order.
- Primarily the fulfillment status is specified in message->order->fulfillments[]->state
- The message->order->fulfillments[]->state->descriptor->long_desc can be used to specify the OCPP status of the charge point. This can help the BAP to construct a custom detailed UI for charging status.

```

```

### support

- Request for support information.
- If regarding a specific order, specify the order_id in message->support->ref_id

```

  
```

### on_support

- Contains support information. If integrated with a CMS, the URL could contain order specific support info
- In all other cases, contains general support info (message->support)

```

```

### cancel

- Request to cancel an order. order_id to be specified in message->order_id
- In the case where there is no advance booking of the charging slot, this does not have much relevance

```

```

### on_cancel

- Confirmation of cancelled order. The cancelled order will be in message->order
- In cases where advanced booking of charging station is not supported, this message is not relevant.

```

```

## Taxonomy and layer 2 configuration

|Property Name|Enums|
|-------------|-----|
|connection-type|RESIDENTIAL<br>COMMERCIAL<br>INDUSTRIAL|
|phase|SINGLE_PHASE<br>THREE_PHASE<br>SPLIT_PHASE|
|property-type|APARTMENT<br>HOUSE<br>OFFICE<br>FACTORY<br>SHOP|
|documentation|ID_PROOF<br>OWNERSHIP_CERTIFICATE<br>WIRING_DIAGRAM<br>BUSINESS_LICENSE<br>WIRING_PLAN|
|connection-status|PENDING<br>IN_PROGRESS<br>COMPLETED<br>CANCELLED<br>FAILED|
|payment-status|PENDING<br>PAID<br>FAILED<br>REFUNDED|
|meter-type|DIGITAL<br>ANALOG<br>SMART_METER|
|voltage-level|LOW_VOLTAGE_230V<br>MEDIUM_VOLTAGE_11KV<br>HIGH_VOLTAGE_33KV|
|connection-load|LIGHT_LOAD<br>MEDIUM_LOAD<br>HEAVY_LOAD|
|connection-phase|SINGLE_PHASE<br>THREE_PHASE|
|connection-category|NEW_CONNECTION<br>TEMPORARY_CONNECTION<br>PERMANENT_CONNECTION<br>TEMP_TO_PERM|
|connection-purpose|DOMESTIC<br>COMMERCIAL<br>INDUSTRIAL<br>AGRICULTURAL|
|connection-tariff|RESIDENTIAL<br>COMMERCIAL<br>INDUSTRIAL<br>AGRICULTURAL|
|connection-metering|SINGLE_POINT<br>MULTI_POINT|
|connection-billing|PREPAID<br>POSTPAID|

## Links to artefacts

- [Postman collection for Connections]()
- [Layer2 config for UEI Connections]()

## Sandbox Details

### Registry/Gateway:

- **Gateway Sandbox:** []()
- **Registry Sandbox:** []()

### BPP:

- **BPP Client Sandbox:** []()
- **BPP Network Sandbox:** []()
- **BPP Playground Sandbox:** []()