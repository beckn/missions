# DEG Implementation Guide: Peer To Peer Energy Trade

## Version History

| Date       | Version | Description                        |
| ---------- | ------- | ---------------------------------- |
| 13-05-2025 | 1.0     | Initial version for Peer To Peer Energy Trade |


## Introduction

This guide outlines the end-to-end implementation for a Peer To Peer Energy Trade use case using the Beckn Protocol. It is intended for developers and system integrators creating Beckn-compliant BAP and BPP systems for renting energy storage devices.

## Structure of the Document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualization) - A pictorial or descriptive representation of the different use cases that are supported by the network.
2. [General Flow Diagrams](#general-flow-diagrams) - This section is relevant to all the message flows illustrated below and discussed further in the document.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the messages that are sent in the form of sample schemas.
4. [Taxonomy and Layer 2 Configuration](#taxonomy-and-layer2-configuration) - This section provides details on the taxonomy, enumerations, and any rules defined for either the use case or by the network.
5. [Integration Notes](#notes-on-software-integration) - Notes on writing/integrating with your own software.
6. [Links to Artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Registry/Gateway, and BPP.

## Outcome Visualization

### Use Case - Peer To Peer Energy Trade for Residential Backup

<>

## General Flow Diagrams

Refer to the [Generic Implementation Guide - Flow Diagrams](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams).

## API Calls and Schema

### search
Search is used to discover available Peer To Peer Energy Trade options.

```json
{
    "context": {
        "domain": "trade",
        "action": "search",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "search-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "intent": {
            "item": {
                "descriptor": {
                    "name": "Solar Surplus Energy"
                }
            },
            "fulfillment": {
                "agent": {
                    "organization": {
                        "descriptor": {
                            "name": "PG&E Grid Services"
                        }
                    }
                },
                "stops": [
                    {
                        "type": "end",
                        "location": {
                            "address": "der://ssf.meter/98765456"
                        },
                        "time": {
                            "range": {
                                "start": "2024-10-04T10:00:00",
                                "end": "2024-10-04T18:00:00"
                            }
                        }
                    }
                ]
            }
        }
    }
}
````

### on_search
Responds with available Peer To Peer Energy Trade options from different providers.
- Product options listed under `catalog.providers[].items[]`.
- Pricing, quantity, rental terms and images are included.

```json
{
    "context": {
        "domain": "trade",
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
        "bap_id": "p2pTrading-bap.com",
        "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
        "bpp_id": "p2pTrading-bpp.com",
        "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "p1072",
                    "descriptor": {
                        "name": "Ethan Maxwell",
                        "images": [
                            {
                                "url": "https://p1072.in/images/logo.png"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "1",
                            "descriptor": {
                                "code": "solar-energy",
                                "name": "Solar Energy"
                            }
                        },
                        {
                            "id": "2",
                            "descriptor": {
                                "code": "EV",
                                "name": "Electric Vehicle"
                            }
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "gps": "37.7749,-122.4194",
                            "descriptor": {
                                "code": "home"
                            },
                            "address": "der://ethan.home"
                        },
                        {
                            "id": "2",
                            "gps": "37.7749,-122.4194",
                            "descriptor": {
                                "code": "ev"
                            },
                            "address": "der://ethan.ev"
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "1",
                            "agent": {
                                "organization": {
                                    "descriptor": {
                                        "name": "PG&E Grid Services"
                                    }
                                }
                            },
                            "stops": [
                                {
                                    "type": "start",
                                    "location": {
                                        "address": "der://pge.meter/100200300"
                                    },
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                },
                                {
                                    "type": "end",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                }
                            ]
                        }
                    ],
                    "items": [
                        {
                            "id": "uid_xyz",
                            "descriptor": {
                                "code": "energy",
                                "name": "Solar Surplus Energy",
                                "short_desc": "30 kWh surplus available between 10AM–6PM"
                            },
                            "price": {
                                "value": "5",
                                "currency": "INR/kWH"
                            },
                            "quantity": {
                                "available": {
                                    "measure": {
                                        "value": "30",
                                        "unit": "kWH"
                                    }
                                }
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
                                        "name": "Energy Attributes"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "name": "Source Type"
                                            },
                                            "value": "Rooftop Solar"
                                        },
                                        {
                                            "descriptor": {
                                                "name": "Carbon Offset Certified"
                                            },
                                            "value": "Yes"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "name": "Certifications"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "name": "Solar Panel Ownership Certificate",
                                                "code": "SPOC"
                                            },
                                            "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
                                        },
                                        {
                                            "descriptor": {
                                                "name": "P2P Trading Licence",
                                                "code": "P2PTL"
                                            },
                                            "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
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
````

### select

Used to choose a specific energy trade listing and indicate quantity

* The item is chosen using `order.items.id`.
* Delivery preferences go in `fulfillments.stops[].location`.

````json
{
    "context": {
        "domain": "trade",
        "action": "select",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "select-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p1072"
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
                    }
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ]
                }
            ]
        }
    }
}
````

### on_select
Returns confirmation of selection with pricing and fulfillment info.

```json
{
    "context": {
        "domain": "trade",
        "action": "on_select",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bpp_id": "p2pTrading-bpp.com",
        "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "on_select-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p1072",
                "descriptor": {
                    "name": "Ethan Maxwell",
                    "images": [
                        {
                            "url": "https://p1072.in/images/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "descriptor": {
                        "code": "energy",
                        "name": "Solar Surplus Energy",
                        "short_desc": "30 kWh surplus available between 10AM-6PM"
                    },
                    "price": {
                        "value": "5",
                        "currency": "USD/kWH"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
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
                                "code": "Energy_Attributes"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "Source_Type"
                                    },
                                    "value": "Rooftop Solar"
                                },
                                {
                                    "descriptor": {
                                        "code": "Carbon_Offset_Certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "location": {
                                "address": "der://pge.meter/100200300"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        },
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "value": "50",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Energy Cost",
                        "price": {
                            "value": "50",
                            "currency": "USD"
                        }
                    }
                ]
            }
        }
    }
}
````

### init

Initializes the order by sending billing and fulfillment details.

```json
{
    "context": {
        "domain": "trade",
        "action": "init",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "init-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p1072"
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
                    }
                }
            ],
            "billing": {
                "name": "John Doe",
                "email": "john@example.com",
                "phone": "+11234567890"
            },
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ]
                }
            ]
        }
    }
}
````

### on_init

- Returns quote, payment info, and order summary for confirmation.

```json
{
    "context": {
        "domain": "trade",
        "action": "on_init",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bpp_id": "p2pTrading-bpp.com",
        "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "on_select-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p1072",
                "descriptor": {
                    "name": "Ethan Maxwell",
                    "images": [
                        {
                            "url": "https://p1072.in/images/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "descriptor": {
                        "code": "energy",
                        "name": "Solar Surplus Energy",
                        "short_desc": "30 kWh surplus available between 10AM-6PM"
                    },
                    "price": {
                        "value": "5",
                        "currency": "USD/kWH"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
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
                                "code": "Energy_Attributes"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "Source_Type"
                                    },
                                    "value": "Rooftop Solar"
                                },
                                {
                                    "descriptor": {
                                        "code": "Carbon_Offset_Certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Certifications"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Solar Panel Ownership Certificate",
                                        "code": "SPOC"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
                                },
                                {
                                    "descriptor": {
                                        "name": "P2P Trading Licence",
                                        "code": "P2PTL"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "location": {
                                "address": "der://pge.meter/100200300"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        },
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "value": "50",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Energy Cost",
                        "price": {
                            "value": "50",
                            "currency": "USD"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE-ORDER",
                    "params": {
                        "amount": "50",
                        "currency": "USD"
                    },
                    "url": "https://payment-gateway.com/checkout?amount=1500&currency=USD&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback",
                    "status": "NOT-PAID"
                }
            ]
        }
    }
}
````

### confirm

- Used to finalize the Peer To Peer Energy Trade request including selected item and delivery.

```json
{
    "context": {
        "domain": "trade",
        "action": "confirm",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "confirm-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p1072"
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
                    }
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ]
                }
            ],
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE-ORDER",
                    "params": {
                        "amount": "50",
                        "currency": "USD"
                    },
                    "url": "https://payment-gateway.com/checkout?amount=1500&currency=USD&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback",
                    "status": "NOT-PAID"
                }
            ]
        }
    }
}
````

### on_confirm

- Confirmation of successful trade order placement.

```json
{
    "context": {
        "domain": "trade",
        "action": "on_confirm",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bpp_id": "p2pTrading-bpp.com",
        "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "on_select-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "id": "ord/12",
            "provider": {
                "id": "p1072",
                "descriptor": {
                    "name": "Ethan Maxwell",
                    "images": [
                        {
                            "url": "https://p1072.in/images/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "descriptor": {
                        "code": "energy",
                        "name": "Solar Surplus Energy",
                        "short_desc": "30 kWh surplus available between 10AM-6PM"
                    },
                    "price": {
                        "value": "5",
                        "currency": "USD/kWH"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
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
                                "name": "Energy Attributes"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Source Type"
                                    },
                                    "value": "Rooftop Solar"
                                },
                                {
                                    "descriptor": {
                                        "name": "Carbon Offset Certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Certifications"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Solar Panel Ownership Certificate",
                                        "code": "SPOC"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
                                },
                                {
                                    "descriptor": {
                                        "name": "P2P Trading Licence",
                                        "code": "P2PTL"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "location": {
                                "address": "der://pge.meter/100200300"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        },
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "status": "ORDER_PLACED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "value": "50",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Energy Cost",
                        "price": {
                            "value": "50",
                            "currency": "USD"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE-ORDER",
                    "params": {
                        "amount": "50",
                        "currency": "USD",
                        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176"
                    },
                    "url": "https://payment-gateway.com/checkout?amount=1500&currency=USD&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback",
                    "status": "PAID"
                }
            ]
        }
    }
}
````

### status

- Used to fetch the status of a tarde order.

```json
{
    "context": {
      "domain": "trade",
      "action": "status",
      "location": {
        "country": {
          "code": "USA"
        },
        "city": {
          "code": "NANP:628"
        }
      },
      "version": "1.1.0",
      "bap_id": "p2pTrading-bap.com",
      "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "status-message-001",
      "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
      "order_id": "ord/12"
    }
  }
````

### on_status

```json
{
    "context": {
      "domain": "trade",
      "action": "on_status",
      "location": {
        "country": {
          "code": "USA"
        },
        "city": {
          "code": "NANP:628"
        }
      },
      "version": "1.1.0",
      "bap_id": "p2pTrading-bap.com",
      "bpp_id": "p2pTrading-bpp.com",
      "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
      "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
      "message_id": "on_status-message-001",
      "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "id": "ord/12",
            "provider": {
                "id": "p1072",
                "descriptor": {
                    "name": "Ethan Maxwell",
                    "images": [
                        {
                            "url": "https://p1072.in/images/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "descriptor": {
                        "code": "energy",
                        "name": "Solar Surplus Energy",
                        "short_desc": "30 kWh surplus available between 10AM-6PM"
                    },
                    "price": {
                        "value": "5",
                        "currency": "USD/kWH"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
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
                                "name": "Energy Attributes"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Source Type"
                                    },
                                    "value": "Rooftop Solar"
                                },
                                {
                                    "descriptor": {
                                        "name": "Carbon Offset Certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Certifications"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Solar Panel Ownership Certificate",
                                        "code": "SPOC"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
                                },
                                {
                                    "descriptor": {
                                        "name": "P2P Trading Licence",
                                        "code": "P2PTL"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "location": {
                                "address": "der://pge.meter/100200300"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        },
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "status": "ORDER_CONFIRMED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "value": "50",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Energy Cost",
                        "price": {
                            "value": "50",
                            "currency": "USD"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE-ORDER",
                    "params": {
                        "amount": "50",
                        "currency": "USD",
                        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176"
                    },
                    "url": "https://payment-gateway.com/checkout?amount=1500&currency=USD&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback",
                    "status": "PAID"
                }
            ]
        }
    }
  }
````

### on_update


```json
{
    "context": {
        "domain": "trade",
        "action": "on_update",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bpp_id": "p2pTrading-bpp.com",
        "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "on_status-message-001",
        "timestamp": "2025-05-09T10:00:00Z"
    },
    "message": {
        "order": {
            "id": "ord/12",
            "provider": {
                "id": "p1072",
                "descriptor": {
                    "name": "Ethan Maxwell",
                    "images": [
                        {
                            "url": "https://p1072.in/images/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "uid_xyz",
                    "descriptor": {
                        "code": "energy",
                        "name": "Solar Surplus Energy",
                        "short_desc": "30 kWh surplus available between 10AM-6PM"
                    },
                    "price": {
                        "value": "5",
                        "currency": "USD/kWH"
                    },
                    "quantity": {
                        "available": {
                            "measure": {
                                "value": "30",
                                "unit": "kWH"
                            }
                        },
                        "selected": {
                            "measure": {
                                "value": "10",
                                "unit": "kWH"
                            }
                        }
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
                                "name": "Energy Attributes"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Source Type"
                                    },
                                    "value": "Rooftop Solar"
                                },
                                {
                                    "descriptor": {
                                        "name": "Carbon Offset Certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Certifications"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "name": "Solar Panel Ownership Certificate",
                                        "code": "SPOC"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
                                },
                                {
                                    "descriptor": {
                                        "name": "P2P Trading Licence",
                                        "code": "P2PTL"
                                    },
                                    "value": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "PG&E Grid Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "location": {
                                "address": "der://pge.meter/100200300"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        },
                        {
                            "type": "end",
                            "location": {
                                "address": "der://ssf.meter/98765456"
                            },
                            "time": {
                                "range": {
                                    "start": "2024-10-04T10:00:00",
                                    "end": "2024-10-04T18:00:00"
                                }
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "status": "ORDER_PARTIALLY_FULFILLED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "value": "40",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Energy Cost",
                        "price": {
                            "value": "40",
                            "currency": "USD"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE-ORDER",
                    "params": {
                        "amount": "50",
                        "currency": "USD",
                        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176"
                    },
                    "url": "https://payment-gateway.com/checkout?amount=1500&currency=USD&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback",
                    "status": "PAID"
                }
            ]
        }
    }
}
````

## Taxonomy and Layer2 Configuration

|Property Name|Enums|
|-------------|-----|
|p2p-type|ENERGY_TRADE<br>POWER_EXCHANGE<br>GRID_SERVICES<br>COMMUNITY_SHARING|
|trade-status|PENDING<br>ACTIVE<br>COMPLETED<br>CANCELLED<br>FAILED<br>DISPUTED|
|trade-mode|AUTOMATED<br>MANUAL<br>HYBRID<br>SCHEDULED|
|energy-source|SOLAR<br>BATTERY<br>GRID<br>HYBRID<br>RENEWABLE|
|trade-direction|BUY<br>SELL<br>BIDIRECTIONAL|
|payment-status|PENDING<br>COMPLETED<br>FAILED<br>REFUNDED<br>DISPUTED|
|settlement-type|REAL_TIME<br>HOURLY<br>DAILY<br>WEEKLY<br>MONTHLY|
|trade-currency|USD<br>EUR<br>GBP<br>INR<br>CRYPTO|
|verification-status|PENDING<br>VERIFIED<br>REJECTED<br>NEEDS_REVISION|
|trade-priority|HIGH<br>MEDIUM<br>LOW<br>EMERGENCY|
|trade-volume|SMALL<br>MEDIUM<br>LARGE<br>CUSTOM|
|trade-duration|INSTANT<br>HOURLY<br>DAILY<br>WEEKLY<br>MONTHLY|
|trade-participant|INDIVIDUAL<br>COMMERCIAL<br>UTILITY<br>AGGREGATOR|
|trade-platform|CENTRALIZED<br>DECENTRALIZED<br>HYBRID|
|trade-compliance|COMPLIANT<br>NON_COMPLIANT<br>PENDING_VERIFICATION|
|trade-security|SECURE<br>PENDING_VERIFICATION<br>FLAGGED|
|trade-settlement|AUTOMATED<br>MANUAL<br>HYBRID|
|trade-notification|PUSH<br>EMAIL<br>SMS<br>IN_APP|
|trade-documentation|PENDING<br>COMPLETED<br>VERIFIED<br>REJECTED|
|trade-dispute|NONE<br>PENDING<br>RESOLVED<br>ESCALATED|


## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to artefacts

- [Postman collection for Demand Flexibility]()
- [Layer2 config for UEI Demand Flexibility]()

## Sandbox Details

### Registry/Gateway:

- **Gateway Sandbox:** []()
- **Registry Sandbox:** []()

### BPP:

- **BPP Client Sandbox:** []()
- **BPP Network Sandbox:** []()
- **BPP Playground Sandbox:** []()