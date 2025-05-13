# DEG Implementation Guide - Installation

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 13-05-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides integration guidelines for implementing solar panel installation services over a Beckn-enabled open energy network. It outlines how licensed solar installers, such as SunPower Installers, can publish installation offerings—residential or commercial—through standardized digital catalogs accessible to users via Beckn-compliant platforms.

The guide is intended for developers and service integrators familiar with the Beckn protocol, message flows, and schema structures. It maps the JSON-based catalog responses from installation service providers (BPPs) to the standard Beckn `on_search` message format to ensure interoperability and seamless service discovery.

## Structure of the document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualization) - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
2. [General Flow diagrams](#general-flow-diagrams) - This section is relevant to all the messages flows illustrated below and discussed further in the document.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. [Taxonomy and layer 2 configuration](#taxonomy-and-layer-2-configuration) - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
5. [Integration Notes](#notes-on-software-integration) - Notes on writing/integrating with your own software.
6. [Links to artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Registry/Gateway and BPP.

## Outcome Visualization

### Use case - Discovery, order and fulfillment of Solar Panel Installation

### Solar Panel Installation in San Francisco

Emily, a homeowner in San Francisco, wants to install solar panels on her residence to reduce her carbon footprint and electricity bills.

### 1. Discovery

* Emily opens the **Energykart app**, which lists licensed solar installation providers.
* The app shows:
  * **SunPower Installers**
  * Service offered: **Residential and Commercial Solar Panel Installation**
  * Fulfillment: **Onsite visit**
* She selects the **Residential Solar Installation** option.

### 2. Order

* Emily chooses the **2kW Rooftop Solar Setup**.
* Price displayed: **$85,000 USD (estimated)**
* Required documents listed:
  * **ID proof**
  * **Ownership certificate**
  * **Electricity bill**
  * **Net metering form**
  * **Wiring diagram**
* Emily uploads the documents, agrees to the terms, and selects **card payment**.
* She receives an **Installation Request ID** and an appointment for a site survey.

### 3. Fulfillment

* On the scheduled day, a certified technician visits Emily's home.
* The technician conducts a site survey and installs the solar panels.
* System is commissioned and connected to the grid.
* A digital installation certificate is issued.

### 4. Post Fulfillment

* Emily receives a **digital confirmation of installation** and a formal invoice.
* She is asked to **rate the service** on a 0–5 scale.
* Emily gives a **5-star rating** for professional installation and system performance.

## General Flow Diagrams

Refer to the [Generic Implementation Guide - Flow Diagrams](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams).

## API Calls and Schema

### search

Search request can contain one or more search criterion within it. Use the following list on how to specify the criterion.

- The location to search around is specified in the message->intent->fulfillment->stops[0]->location field.
- The installation type required is specified in message->intent->category->descriptor->code
- If searching by free text, it is specified in message->intent->descriptor->name
- If searching by provider, it is specified in message->intent->provider->descriptor->name

```
{
    "context": {
        "domain": "service",
        "action": "search",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:00Z",
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
                "name": "Solar Installation Request"
            },
            "provider": {
                "descriptor": {
                    "name": "SunPower Installers"
                }
            },
            "category": {
                "descriptor": {
                    "code": "solar_panel_installation"
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
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "sunpower_sf",
                    "descriptor": {
                        "code": "SunPower_Installers",
                        "short_desc": "Residential and commercial solar panel installation",
                        "long_desc": "Leading provider of rooftop and industrial solar panel setup services in San Francisco.",
                        "images": [
                            {
                                "url": "https://sunpower.com/logo.png"
                            },
                            {
                                "url": "https://sunpower.com/gallery.jpg"
                            }
                        ]
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "provider-details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "govt_subsidy_empaneled"
                                    },
                                    "value": "true"
                                }
                            ]
                        }
                    ],
                    "rating": "4.6/5",
                    "categories": [
                        {
                            "id": "1",
                            "descriptor": {
                                "code": "solar_panel_installation",
                                "name": "Solar Panel Installation"
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
                            "id": "sp-resi-001",
                            "descriptor": {
                                "code": "2kW_Rooftop_Solar_Setup"
                            },
                            "price": {
                                "estimated_value": "85000",
                                "currency": "USD"
                            },
                            "category_ids": ["1"],
                            "location_ids": ["1"],
                            "fulfillment_ids": ["1"],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "system_specification"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "system_capacity"
                                            },
                                            "value": "2KW"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "installation_context"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "property_type"
                                            },
                                            "value": "Residential"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "installer_details"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "empaneled"
                                            },
                                            "value": "true"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "financial_details"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "financial_assistance"
                                            },
                                            "value": "true"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "documentation"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "required_docs"
                                            },
                                            "value": "ID proof, Ownership certificate, Electricity bill, Net metering form, Wiring diagram"
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
{
    "context": {
        "domain": "service",
        "action": "init",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10004",
        "timestamp": "2023-07-16T04:41:30Z",
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
        "order": {
            "provider": {
                "id": "sunpower_sf"
            },
            "items": [
                {
                    "id": "sp-resi-001"
                }
            ],
            "billing": {
                "name": "Emily Tran",
                "email": "emily.tran@example.com",
                "phone": "+14155556666",
                "address": "456 Glenview St, San Francisco, CA"
            },
            "fulfillments": [
                {
                    "id": "1",
                    "type": "ONSITE",
                    "customer": {
                        "person": {
                            "name": "Emily Tran"
                        },
                        "contact": {
                            "phone": "+14155556666",
                            "email": "emily.tran@example.com"
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Glenview St, San Francisco, CA"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
```

### on_init

- Contains payment terms. Payment terms specified in message->order->payments
- Cancellation terms specified in message->order->cancellation_terms
- Here we show the BPP as payment collector. In case the BAP specifies that it collects the payment in the init, the url field within payments will be empty

```
{
    "context": {
        "domain": "service",
        "action": "on_init",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bpp_id": "example-bpp.com",
        "bpp_uri": "https://example-bpp.com",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10005",
        "timestamp": "2023-07-16T04:41:35Z",
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
        "order": {
            "provider": {
                "id": "sunpower_sf",
                "descriptor": {
                    "code": "SunPower_Installers",
                    "short_desc": "Residential and commercial solar panel installation",
                    "long_desc": "Leading provider of rooftop and industrial solar panel setup services in San Francisco.",
                    "images": [
                        {
                            "url": "https://sunpower.com/logo.png"
                        },
                        {
                            "url": "https://sunpower.com/gallery.jpg"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "sp-resi-001",
                    "descriptor": {
                        "name": "2kW Rooftop Solar Setup"
                    },
                    "price": {
                        "estimated_value": "85000",
                        "currency": "USD"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "system_specification"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "system_capacity"
                                    },
                                    "value": "2KW"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installation_context"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installer_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "empaneled"
                                    },
                                    "value": "true"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "financial_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "financial_assistance"
                                    },
                                    "value": "true"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documentation"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "required_docs"
                                    },
                                    "value": "ID proof, Ownership certificate, Electricity bill, Net metering form, Wiring diagram"
                                }
                            ]
                        }
                    ],
                    "xinput": {
                        "required": false,
                        "head": {
                            "descriptor": {
                                "name": "Details Form"
                            },
                            "index": {
                                "min": 0,
                                "cur": 0,
                                "max": 1
                            },
                            "headings": [
                                "Customer Details",
                                "Technical Details"
                            ]
                        },
                        "form": {
                            "mime_type": "text/html",
                            "url": "https://6vs8xnx5i7.scheme-finder.co.in/schems/xinput/formid/a23f2fdfbbb8ac402bfd54f",
                            "resubmit": false,
                            "auth": {
                                "descriptor": {
                                    "code": "jwt"
                                },
                                "value": "eyJhbGciOiJIUzI.eyJzdWIiOiIxMjM0NTY3O.SflKxwRJSMeKKF2QT4"
                            }
                        }
                    }
                }
            ],
            "billing": {
                "name": "Emily Tran",
                "email": "emily.tran@example.com",
                "phone": "+14155556666",
                "address": "456 Glenview St, San Francisco, CA"
            },
            "fulfillments": [
                {
                    "id": "1",
                    "type": "ONSITE",
                    "tracking": true,
                    "state": {
                        "descriptor": {
                            "code": "Scheduled",
                            "name": "Installation Scheduled"
                        }
                    },
                    "customer": {
                        "person": {
                            "name": "Emily Tran"
                        },
                        "contact": {
                            "phone": "+14155556666",
                            "email": "emily.tran@example.com"
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Glenview St, San Francisco, CA"
                            }
                        }
                    ],
                    "contact": {
                        "phone": "+18005559999",
                        "email": "install@sunpower.com"
                    }
                }
            ],
            "quote": {
                "price": {
                    "value": "85000",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Installation Charge",
                        "price": {
                            "value": "85000",
                            "currency": "USD"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "type": "ON-FULFILLMENT",
                    "collected_by": "BPP",
                    "status": "NOT-PAID",
                    "params": {
                        "amount": "150",
                        "currency": "USD"
                    }
                }
            ]
        }
    }
}
```

### confirm

- Confirm order including payment paid info (when applicable).
- It is in message->order->payments

```
{
    "context": {
        "domain": "service",
        "action": "confirm",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10006",
        "timestamp": "2023-07-16T04:41:40Z",
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
        "order": {
            "provider": {
                "id": "sunpower_sf"
            },
            "items": [
                {
                    "id": "sp-resi-001"
                }
            ],
            "billing": {
                "name": "Emily Tran",
                "email": "emily.tran@example.com",
                "phone": "+14155556666",
                "address": "456 Glenview St, San Francisco, CA"
            },
            "fulfillments": [
                {
                    "id": "1",
                    "type": "ONSITE",
                    "customer": {
                        "person": {
                            "name": "Emily Tran"
                        },
                        "contact": {
                            "phone": "+14155556666",
                            "email": "emily.tran@example.com"
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Glenview St, San Francisco, CA"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
```

### on_confirm

- Order confirmed. Charging can start.

```
{
    "context": {
        "domain": "service",
        "action": "on_confirm",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bpp_id": "example-bpp.com",
        "bpp_uri": "https://example-bpp.com",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10007",
        "timestamp": "2023-07-16T04:41:45Z",
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
        "order": {
            "id": "sunpower-install-order-001",
            "provider": {
                "id": "sunpower_sf",
                "descriptor": {
                    "code": "SunPower_Installers",
                    "short_desc": "Residential and commercial solar panel installation",
                    "long_desc": "Leading provider of rooftop and industrial solar panel setup services in San Francisco.",
                    "images": [
                        {
                            "url": "https://sunpower.com/logo.png"
                        },
                        {
                            "url": "https://sunpower.com/gallery.jpg"
                        }
                    ]
                }
            }
        }
    }
}
```

### status

- Request for status on order. order_id is specifiedin message->order_id

```
{
    "context": {
        "domain": "service",
        "action": "status",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10008",
        "timestamp": "2023-07-16T04:41:50Z",
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
        "order_id": "sunpower-install-order-001"
    }
}
```

### on_status

- Status of requested order.
- Primarily the fulfillment status is specified in message->order->fulfillments[]->state
- The message->order->fulfillments[]->state->descriptor->long_desc can be used to specify the OCPP status of the charge point. This can help the BAP to construct a custom detailed UI for charging status.

```
{
    "context": {
        "domain": "service",
        "action": "on_status",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bpp_id": "example-bpp.com",
        "bpp_uri": "https://example-bpp.com",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10009",
        "timestamp": "2023-07-16T04:41:55Z",
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
        "order": {
            "id": "sunpower-install-order-001",
            "provider": {
                "id": "sunpower_sf",
                "descriptor": {
                    "name": "SunPower Installers",
                    "short_desc": "Residential and commercial solar panel installation"
                }
            },
            "items": [
                {
                    "id": "sp-resi-001",
                    "descriptor": {
                        "name": "2kW Rooftop Solar Setup"
                    },
                    "price": {
                        "estimated_value": "85000",
                        "currency": "USD"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "system_specification"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "system_capacity"
                                    },
                                    "value": "2KW"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installation_context"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installer_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "empaneled"
                                    },
                                    "value": "true"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "financial_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "financial_assistance"
                                    },
                                    "value": "true"
                                }
                            ]
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Emily Tran",
                "email": "emily.tran@example.com",
                "phone": "+14155556666",
                "address": "456 Glenview St, San Francisco, CA"
            },
            "fulfillments": [
                {
                    "id": "1",
                    "type": "ONSITE",
                    "tracking": true,
                    "state": {
                        "descriptor": {
                            "code": "Technician-dispatched",
                            "name": "Technician Dispatched"
                        }
                    },
                    "customer": {
                        "person": {
                            "name": "Emily Tran"
                        },
                        "contact": {
                            "phone": "+14155556666",
                            "email": "emily.tran@example.com"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "David C"
                        },
                        "contact": {
                            "phone": "+1423452345"
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Glenview St, San Francisco, CA"
                            }
                        }
                    ],
                    "contact": {
                        "phone": "+18005559999",
                        "email": "install@sunpower.com"
                    }
                }
            ]
        }
    }
}
```

### select

- Used by the BAP (platform) to indicate the user's intent to proceed with a specific service item (e.g., a residential or commercial electricity connection).

- Contains the selected item ID, fulfillment method (e.g., onsite visit), and any additional user inputs such as preferred appointment date/time or uploaded documents.

- May include preliminary consent or acknowledgment of service terms before confirming the connection request.

```
{
    "context": {
        "domain": "service",
        "action": "select",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10002",
        "timestamp": "2023-07-16T04:41:20Z",
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
        "order": {
            "provider": {
                "id": "sunpower_sf"
            },
            "items": [
                {
                    "id": "sp-resi-001"
                }
            ],
            "fulfillments": [
                {
                    "id": "1"
                }
            ]
        }
    }
}
  
```

### on_select

- Sent by the BPP (provider) in response to the select call to confirm item availability and fulfillability.

- May return refined details such as estimated installation date, required documentation, or steps for scheduling a technician visit.

- Can include a payment breakdown or token for subsequent actions like init or confirm.

- Also serves to validate inputs (e.g., property type, phase selection) and determine readiness to proceed.


```
{
    "context": {
        "domain": "service",
        "action": "on_select",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bpp_id": "example-bpp.com",
        "bpp_uri": "https://example-bpp.com",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "certification-msg-10003",
        "timestamp": "2023-07-16T04:41:25Z",
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
        "order": {
            "provider": {
                "id": "sunpower_sf",
                "descriptor": {
                    "code": "SunPower_Installers",
                    "short_desc": "Residential and commercial solar panel installation",
                    "long_desc": "Leading provider of rooftop and industrial solar panel setup services in San Francisco.",
                    "images": [
                        {
                            "url": "https://sunpower.com/logo.png"
                        },
                        {
                            "url": "https://sunpower.com/gallery.jpg"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "sp-resi-001",
                    "descriptor": {
                        "name": "2kW Rooftop Solar Setup"
                    },
                    "price": {
                        "estimated_value": "85000",
                        "currency": "USD"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "system_specification"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "system_capacity"
                                    },
                                    "value": "2KW"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installation_context"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
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


## Taxonomy and layer 2 configuration

|Property Name|Enums|
|-------------|-----|
|installation-type|METER_INSTALLATION<br>SOLAR_PANEL<br>BATTERY_SYSTEM<br>EV_CHARGER<br>SMART_DEVICE|
|installation-status|PENDING<br>SCHEDULED<br>IN_PROGRESS<br>COMPLETED<br>CANCELLED<br>FAILED|
|technician-type|ELECTRICIAN<br>SOLAR_TECHNICIAN<br>EV_SPECIALIST<br>GENERAL_TECHNICIAN|
|scheduling-status|PENDING<br>CONFIRMED<br>RESCHEDULED<br>CANCELLED|
|installation-priority|HIGH<br>MEDIUM<br>LOW<br>EMERGENCY|
|verification-status|PENDING<br>VERIFIED<br>REJECTED<br>NEEDS_REVISION|
|payment-status|PENDING<br>PAID<br>FAILED<br>REFUNDED|
|installation-category|NEW_INSTALLATION<br>UPGRADE<br>REPLACEMENT<br>MAINTENANCE|
|equipment-type|METER<br>SOLAR_PANEL<br>BATTERY<br>CHARGER<br>INVERTER<br>CONTROLLER|
|installation-complexity|SIMPLE<br>MODERATE<br>COMPLEX<br>SPECIALIZED|
|location-type|RESIDENTIAL<br>COMMERCIAL<br>INDUSTRIAL<br>UTILITY|
|access-type|EASY_ACCESS<br>RESTRICTED_ACCESS<br>SPECIAL_PERMISSION|
|installation-phase|SITE_SURVEY<br>PREPARATION<br>INSTALLATION<br>TESTING<br>COMMISSIONING|
|documentation-status|PENDING<br>COMPLETED<br>VERIFIED<br>REJECTED|
|safety-status|PENDING<br>VERIFIED<br>ISSUES_FOUND<br>RESOLVED|


## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to artefacts

- [Postman collection for Connections]()
- [Layer2 config for Connections]()

## Sandbox Details

### Registry/Gateway:

- **Gateway Sandbox:** []()
- **Registry Sandbox:** []()

### BPP:

- **BPP Client Sandbox:** []()
- **BPP Network Sandbox:** []()
- **BPP Playground Sandbox:** []()