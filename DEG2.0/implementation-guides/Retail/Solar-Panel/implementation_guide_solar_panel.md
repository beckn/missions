# DEG Implementation Guide - Solar Panel Purchase

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 14-05-2025 | 1.0     | Initial Version                                     |


## Introduction

This document provides integration guidelines for implementing solar energy product and service discovery, purchase, and fulfillment using a Beckn-enabled open retail network. It showcases how solar energy providers and service aggregators can publish rooftop panel offerings and installation services, enabling residents to seamlessly transition to clean energy solutions.

Using the Beckn protocol, retail platforms in cities like San Jose can offer users end-to-end access to certified solar hardware, professional installation services, and transparent pricing—all from within a single digital interface. This use case demonstrates how a user can discover, select, confirm, and fulfill a solar panel purchase and its associated services through a decentralized ecosystem.

This guide assumes the reader is familiar with the Beckn protocol, message flow, and schema standards.

## Structure of the Document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualization) - A pictorial or descriptive representation of the different use cases that are supported by the network.
2. [General Flow Diagrams](#general-flow-diagrams) - This section is relevant to all the message flows illustrated below and discussed further in the document.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the messages that are sent in the form of sample schemas.
4. [Taxonomy and Layer 2 Configuration](#taxonomy-and-layer-2-configuration) - This section provides details on the taxonomy, enumerations, and any rules defined for either the use case or by the network.
5. [Integration Notes](#notes-on-software-integration) - Notes on writing/integrating with your own software.
6. [Links to Artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Registry/Gateway, and BPP.

## Outcome Visualization

### Use case - Discovery, order and fulfillment of Solar Panel Purchase

Renee Lopez, a homeowner in San Jose, CA, is planning to transition to solar energy to reduce her electricity bills and environmental footprint. She wants a convenient way to discover, purchase, and install a solar panel system for her rooftop.

She opens a Beckn-enabled marketplace app and browses offerings in the solar category. She selects a complete solution offered by **SunPro Energy Systems**:

- **Product**: *SunPro Mono 450W Panel*  
  - Price: $320 USD  
  - Type: Monocrystalline  
  - Efficiency: 21.4%  
  - Warranty: 25 years  
  - Certified (SEC, UL Safety)  
- **Service**: *Solar Panel Installation Service*  
  - Price: $250 USD  
  - Scope: Rooftop/Ground-mount setup, inverter wiring, mounting, testing  
  - Duration: 4–6 hours  
  - Rated 4.8 stars  
  - Certified and empaneled with nodal agency  

### Renee’s Journey:
1. Adds the solar panel and installation service to her cart.
2. Confirms the price and the terms of the order.
3. Schedules delivery and installation.
4. Receives order confirmation with the terms of the order. 
5. Chooses to pay $150 on fulfillment (Pay on Delivery option)  
6. Check the regular status of the order.

By using the Beckn-enabled solar store, Renee is able to transparently compare specifications, certifications, and installation offerings, ensuring a smooth end-to-end experience in her clean energy journey.

## General Flow Diagrams

Refer to the [Generic Implementation Guide - Flow Diagrams](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams).

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
        "domain": "retail",
        "action": "search",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bap_uri": "https://bap-consumer.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-search-001",
        "timestamp": "2025-05-07T12:29:00Z",
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
                "name": "Solar panels for rooftop installation"
            },
            "category": {
                "descriptor": {
                    "code": "mono_panels"
                }
            },
            "fulfillment": {
                "type": "HOME_DELIVERY"
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

### on_search

**on_search with catalog of results**

- The catalog that comes back has a list of providers.
- Each provider has a list of items.
- Each item is the catalog listing for a charging station.

```
{
    "context": {
        "domain": "retail",
        "action": "on_search",
        "version": "1.1.0",
        "bpp_id": "bpp-solar-store.example.com",
        "bpp_uri": "https://bpp-solar-store.example.com",
        "location": {
            "country": {
                "code": "USA"
            },
            "city": {
                "code": "NANP:628"
            }
        },
        "bap_id": "bap-consumer.example.com",
        "bap_uri": "https://bap-consumer.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-solar-panel-001",
        "ttl": "PT10M",
        "timestamp": "2025-05-07T12:30:00Z"
    },
    "message": {
        "catalog": {
            "descriptor": {
                "name": "Energykart Solar",
                "short_desc": "Shop solar panels by efficiency, budget, or portability"
            },
            "providers": [
                {
                    "id": "sunpro_energy",
                    "descriptor": {
                        "name": "SunPro Energy Systems",
                        "short_desc": "Premium mono and polycrystalline solar panels",
                        "long_desc": "SunPro manufactures high-efficiency solar panels for residential and industrial installations. Known for reliability and long warranties.",
                        "images": [
                            {
                                "url": "https://example.com/images/sunpro_logo.png",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "premium_panels",
                            "descriptor": {
                                "name": "High Efficiency Panels",
                                "code": "mono_panels"
                            }
                        },
                        {
                            "id": "budget_panels",
                            "descriptor": {
                                "name": "Economy Panels",
                                "code": "poly_panels"
                            }
                        },
                        {
                            "id": "solar_installation",
                            "descriptor": {
                                "name": "Solar installation service",
                                "code": "Solar_installation_service"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "hd1",
                            "type": "HOME_DELIVERY"
                        },
                        {
                            "id": "sp1",
                            "type": "STORE_PICKUP"
                        },
                        {
                            "id": "sv1",
                            "type": "SITE_VISIT"
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "city": {
                                "name": "San Jose"
                            }
                        },
                        {
                            "id": "2",
                            "city": {
                                "name": " OakLand"
                            }
                        }
                    ],
                    "rateable": true,
                    "items": [
                        {
                            "id": "sp_mono_450w",
                            "descriptor": {
                                "name": "SunPro Mono 450W Panel",
                                "short_desc": "High-efficiency 450W panel for rooftop systems",
                                "long_desc": "450W monocrystalline panel with superior power output and compact size. Built for long-lasting rooftop installations with 25-year warranty."
                            },
                            "price": {
                                "value": "320",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "premium_panels"
                            ],
                            "fulfillment_ids": [
                                "hd1",
                                "sp1"
                            ],
                            "location_ids": [
                                "1",
                                "2"
                            ],
                            "rating": "4.5",
                            "rateable": true,
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "Panel_Specifications"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "Type"
                                            },
                                            "value": "Monocrystalline"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Wattage"
                                            },
                                            "value": "450W"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Efficiency"
                                            },
                                            "value": "21.4%"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Warranty"
                                            },
                                            "value": "25 years"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "IP_Rating"
                                            },
                                            "value": "IP68"
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
                                                "name": "solar equipment certificate",
                                                "code": "SEC"
                                            },
                                            "value": "https://www.example.com/certificates/solar/CEC_LISTED/SunPower_X21_345_Module_CEC_Certification.pdf"
                                        },
                                        {
                                            "descriptor": {
                                                "name": "pv module safety certificate",
                                                "code": "PMSC"
                                            },
                                            "value": "https://www.example.com/certificates/pv_module_safety_certificate/JinkoTigerNeo_UL61730_Cert.pdf"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "sp_poly_325w",
                            "descriptor": {
                                "name": "SunPro Poly 325W Panel",
                                "short_desc": "Affordable 325W solar panel for home kits",
                                "long_desc": "Entry-level 325W polycrystalline panel. Suitable for small rooftop or DIY solar applications. Offers good performance at lower cost."
                            },
                            "price": {
                                "value": "200",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "budget_panels"
                            ],
                            "fulfillment_ids": [
                                "sp1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "rating": "4.2",
                            "rateable": true,
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "Panel_Specifications"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "Type"
                                            },
                                            "value": "Polycrystalline"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Wattage"
                                            },
                                            "value": "325W"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Efficiency"
                                            },
                                            "value": "18.3%"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Warranty"
                                            },
                                            "value": "10 years"
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
                                                "name": "solar equipment certificate",
                                                "code": "SEC"
                                            },
                                            "value": "https://www.example.com/certificates/solar/CEC_LISTED/SunPower_X21_345_Module_CEC_Certification.pdf"
                                        },
                                        {
                                            "descriptor": {
                                                "name": "pv module safety certificate",
                                                "code": "PMSC"
                                            },
                                            "value": "https://www.example.com/certificates/pv_module_safety_certificate/JinkoTigerNeo_UL61730_Cert.pdf"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "it/panel/12",
                            "descriptor": {
                                "name": "Solar Panel Installation Service",
                                "short_desc": "On-site installation for rooftop or ground-mounted solar panels",
                                "long_desc": "Includes mounting structure setup, panel alignment, inverter integration, grid connection readiness, and post-installation testing. Covers both residential and commercial rooftop systems."
                            },
                            "price": {
                                "value": "250",
                                "currency": "USD"
                            },
                            "rateable": true,
                            "rating": "4.8",
                            "category_ids": [
                                "solar_installation"
                            ],
                            "fulfillment_ids": [
                                "sv1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "installation_specification"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "installation_type"
                                            },
                                            "value": "Rooftop / Ground-mount"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "supported_panel_types"
                                            },
                                            "value": "Mono PERC, Polycrystalline, Thin Film"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "system_capacity_limit"
                                            },
                                            "value": "Up to 10kW"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "included_services"
                                            },
                                            "value": "Mounting, inverter wiring, earthing, net-meter interface"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "installation_duration"
                                            },
                                            "value": "4–6 hours"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "installer_qualification"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "empaneled_with_nodal_agency"
                                            },
                                            "value": "Yes"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "safety_certified"
                                            },
                                            "value": "Yes"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "voltedge_solar",
                    "descriptor": {
                        "name": "VoltEdge Solar",
                        "short_desc": "Flexible and mobile solar panel systems",
                        "long_desc": "VoltEdge provides bendable and lightweight solar panels ideal for portable energy needs, RVs, and boats. Built for rugged environments.",
                        "images": [
                            {
                                "url": "https://example.com/images/voltedge_logo.png",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "portable_panels",
                            "descriptor": {
                                "name": "Portable Panels",
                                "code": "flex_panels"
                            }
                        },
                        {
                            "id": "solar_installation",
                            "descriptor": {
                                "name": "Solar installation service",
                                "code": "Solar_installation_service"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "hd2",
                            "type": "HOME_DELIVERY"
                        },
                        {
                            "id": "sv1",
                            "type": "SITE_VISIT"
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "city": {
                                "name": "San Jose"
                            }
                        },
                        {
                            "id": "2",
                            "city": {
                                "name": " OakLand"
                            }
                        }
                    ],
                    "rateable": true,
                    "items": [
                        {
                            "id": "ve_flex_100w",
                            "descriptor": {
                                "name": "VoltEdge Flex 100W Panel",
                                "short_desc": "Bendable solar panel for mobile energy use",
                                "long_desc": "Flexible 100W solar panel built for curved surfaces and off-grid portability. Ideal for campers, boats, or remote setups. Durable and water-resistant."
                            },
                            "price": {
                                "value": "145",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "portable_panels"
                            ],
                            "fulfillment_ids": [
                                "hd2"
                            ],
                            "location_ids": [
                                "2"
                            ],
                            "rating": "4.6",
                            "rateable": true,
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "Panel_Specifications"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "Type"
                                            },
                                            "value": "Flexible PET"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Wattage"
                                            },
                                            "value": "100W"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Weight"
                                            },
                                            "value": "2.5 kg"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "Use_Case"
                                            },
                                            "value": "Camping, RV, Off-grid"
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
                                                "name": "solar equipment certificate",
                                                "code": "SEC"
                                            },
                                            "value": "https://www.example.com/certificates/solar/CEC_LISTED/SunPower_X21_345_Module_CEC_Certification.pdf"
                                        },
                                        {
                                            "descriptor": {
                                                "name": "pv module safety certificate",
                                                "code": "PMSC"
                                            },
                                            "value": "https://www.example.com/certificates/pv_module_safety_certificate/JinkoTigerNeo_UL61730_Cert.pdf"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "it/panel/12",
                            "descriptor": {
                                "name": "Solar Panel Installation Service",
                                "short_desc": "On-site installation for rooftop or ground-mounted solar panels",
                                "long_desc": "Includes mounting structure setup, panel alignment, inverter integration, grid connection readiness, and post-installation testing. Covers both residential and commercial rooftop systems."
                            },
                            "price": {
                                "value": "250",
                                "currency": "USD"
                            },
                            "rateable": true,
                            "rating": "4.8",
                            "category_ids": [
                                "solar_installation"
                            ],
                            "fulfillment_ids": [
                                "sv1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "installation_specification"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "installation_type"
                                            },
                                            "value": "Rooftop / Ground-mount"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "supported_panel_types"
                                            },
                                            "value": "Mono PERC, Polycrystalline, Thin Film"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "system_capacity_limit"
                                            },
                                            "value": "Up to 10kW"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "included_services"
                                            },
                                            "value": "Mounting, inverter wiring, earthing, net-meter interface"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "installation_duration"
                                            },
                                            "value": "4–6 hours"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "installer_qualification"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "empaneled_with_nodal_agency"
                                            },
                                            "value": "Yes"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "safety_certified"
                                            },
                                            "value": "Yes"
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
        "domain": "retail",
        "action": "init",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bap_uri": "https://bap-consumer.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-init-001",
        "timestamp": "2025-05-07T12:30:15Z",
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
                "id": "sunpro_energy"
            },
            "items": [
                {
                    "id": "sp_mono_450w",
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    }
                },
                {
                    "id": "it/panel/12"
                }
            ],
            "billing": {
                "name": "Evan Smith",
                "email": "evan.smith@example.com",
                "phone": "+14155558888",
                "address": "456 Solar Valley Rd, San Jose, CA"
            },
            "fulfillments": [
                {
                    "id": "hd1",
                    "type": "HOME_DELIVERY",
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Solar Valley Rd, San Jose, CA"
                            },
                            "time": {
                                "range": {
                                    "start": "2025-05-08T09:00:00Z",
                                    "end": "2025-05-08T17:00:00Z"
                                }
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
        "domain": "retail",
        "action": "on_init",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bpp_id": "bpp-solar-store.example.com",
        "bpp_uri": "https://bpp-solar-store.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-on-init-001",
        "timestamp": "2025-05-07T12:30:20Z",
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
                "id": "sunpro_energy",
                "descriptor": {
                    "name": "SunPro Energy Systems",
                    "short_desc": "Premium mono and polycrystalline solar panels"
                }
            },
            "items": [
                {
                    "id": "sp_mono_450w",
                    "descriptor": {
                        "name": "SunPro Mono 450W Panel",
                        "short_desc": "High-efficiency 450W panel for rooftop systems"
                    },
                    "price": {
                        "value": "320",
                        "currency": "USD"
                    },
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "Panel_Specifications"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "Type"
                                    },
                                    "value": "Monocrystalline"
                                },
                                {
                                    "descriptor": {
                                        "code": "Wattage"
                                    },
                                    "value": "450W"
                                },
                                {
                                    "descriptor": {
                                        "code": "Efficiency"
                                    },
                                    "value": "21.4%"
                                },
                                {
                                    "descriptor": {
                                        "code": "Warranty"
                                    },
                                    "value": "25 years"
                                },
                                {
                                    "descriptor": {
                                        "code": "IP_Rating"
                                    },
                                    "value": "IP68"
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
                                        "name": "solar equipment certificate",
                                        "code": "SEC"
                                    },
                                    "value": "https://www.example.com/certificates/solar/CEC_LISTED/SunPower_X21_345_Module_CEC_Certification.pdf"
                                },
                                {
                                    "descriptor": {
                                        "name": "pv module safety certificate",
                                        "code": "PMSC"
                                    },
                                    "value": "https://www.example.com/certificates/pv_module_safety_certificate/JinkoTigerNeo_UL61730_Cert.pdf"
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "it/panel/12",
                    "descriptor": {
                        "name": "Solar Panel Installation Service",
                        "short_desc": "On-site installation for rooftop or ground-mounted solar panels",
                        "long_desc": "Includes mounting structure setup, panel alignment, inverter integration, grid connection readiness, and post-installation testing. Covers both residential and commercial rooftop systems."
                    },
                    "price": {
                        "value": "250",
                        "currency": "USD"
                    },
                    "rateable": true,
                    "rating": "4.8",
                    "category_ids": [
                        "solar_installation"
                    ],
                    "fulfillment_ids": [
                        "sv1"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "installation_specification"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "installation_type"
                                    },
                                    "value": "Rooftop / Ground-mount"
                                },
                                {
                                    "descriptor": {
                                        "code": "supported_panel_types"
                                    },
                                    "value": "Mono PERC, Polycrystalline, Thin Film"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_limit"
                                    },
                                    "value": "Up to 10kW"
                                },
                                {
                                    "descriptor": {
                                        "code": "included_services"
                                    },
                                    "value": "Mounting, inverter wiring, earthing, net-meter interface"
                                },
                                {
                                    "descriptor": {
                                        "code": "installation_duration"
                                    },
                                    "value": "4–6 hours"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installer_qualification"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "empaneled_with_nodal_agency"
                                    },
                                    "value": "Yes"
                                },
                                {
                                    "descriptor": {
                                        "code": "safety_certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Evan Smith",
                "email": "evan.smith@example.com",
                "phone": "+14155558888",
                "address": "456 Solar Valley Rd, San Jose, CA"
            },
            "fulfillments": [
                {
                    "id": "hd1",
                    "type": "HOME_DELIVERY",
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Solar Valley Rd, San Jose, CA"
                            },
                            "time": {
                                "range": {
                                    "start": "2025-05-08T09:00:00Z",
                                    "end": "2025-05-08T17:00:00Z"
                                }
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "value": "345",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Panel Price",
                        "price": {
                            "value": "320",
                            "currency": "USD"
                        }
                    },
                    {
                        "title": "Home Delivery",
                        "price": {
                            "value": "25",
                            "currency": "USD"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "type": "PRE-FULFILLMENT",
                    "collected_by": "BPP",
                    "status": "NOT-PAID",
                    "params": {
                        "amount": "345",
                        "currency": "USD",
                        "transaction_id": "txn-sunpro-001"
                    },
                    "url": "https://bpp-solar-store.example.com/payments/txn-sunpro-001"
                }
            ],
            "cancellation_terms": [
                {
                    "fulfillment_state": {
                        "descriptor": {
                            "code": "order_placed",
                            "name": "Order Placed"
                        }
                    },
                    "reason_required": true,
                    "cancel_by": {
                        "timestamp": "2025-05-07T20:00:00Z"
                    },
                    "cancellation_fee": {
                        "percentage": 5,
                        "minimum_value": {
                            "currency": "USD",
                            "value": "10"
                        }
                    },
                    "xinput": {
                        "required": true,
                        "form": {
                            "url": "https://example.com/cancel"
                        }
                    },
                    "external_ref": {
                        "url": "https://example.com/docs/cancellation-policy.pdf",
                        "mime_type": "application/pdf"
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
        "domain": "retail",
        "action": "confirm",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bap_uri": "https://bap-consumer.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-confirm-001",
        "timestamp": "2025-05-07T12:30:30Z",
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
                "id": "sunpro_energy"
            },
            "items": [
                {
                    "id": "sp_mono_450w",
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    }
                },
                {
                    "id": "it/panel/12"
                }
            ],
            "billing": {
                "name": "Evan Smith",
                "email": "evan.smith@example.com",
                "phone": "+14155558888",
                "address": "456 Solar Valley Rd, San Jose, CA"
            },
            "fulfillments": [
                {
                    "id": "hd1",
                    "type": "HOME_DELIVERY",
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Solar Valley Rd, San Jose, CA"
                            },
                            "time": {
                                "range": {
                                    "start": "2025-05-08T09:00:00Z",
                                    "end": "2025-05-08T17:00:00Z"
                                }
                            }
                        }
                    ]
                }
            ],
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

### on_confirm

- Order confirmed. Charging can start.

```
{
    "context": {
        "domain": "retail",
        "action": "on_confirm",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bpp_id": "bpp-solar-store.example.com",
        "bpp_uri": "https://bpp-solar-store.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-on-confirm-001",
        "timestamp": "2025-05-07T12:30:35Z",
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
            "id": "order-sunpro-001",
            "provider": {
                "id": "sunpro_energy",
                "descriptor": {
                    "name": "SunPro Energy Systems",
                    "short_desc": "Premium mono and polycrystalline solar panels"
                }
            },
            "items": [
                {
                    "id": "sp_mono_450w",
                    "descriptor": {
                        "name": "SunPro Mono 450W Panel",
                        "short_desc": "High-efficiency 450W panel for rooftop systems"
                    },
                    "price": {
                        "value": "320",
                        "currency": "USD"
                    },
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "Panel_Specifications"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "Type"
                                    },
                                    "value": "Monocrystalline"
                                },
                                {
                                    "descriptor": {
                                        "code": "Wattage"
                                    },
                                    "value": "450W"
                                },
                                {
                                    "descriptor": {
                                        "code": "Efficiency"
                                    },
                                    "value": "21.4%"
                                },
                                {
                                    "descriptor": {
                                        "code": "Warranty"
                                    },
                                    "value": "25 years"
                                },
                                {
                                    "descriptor": {
                                        "code": "IP_Rating"
                                    },
                                    "value": "IP68"
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
                                        "name": "solar equipment certificate",
                                        "code": "SEC"
                                    },
                                    "value": "https://www.example.com/certificates/solar/CEC_LISTED/SunPower_X21_345_Module_CEC_Certification.pdf"
                                },
                                {
                                    "descriptor": {
                                        "name": "pv module safety certificate",
                                        "code": "PMSC"
                                    },
                                    "value": "https://www.example.com/certificates/pv_module_safety_certificate/JinkoTigerNeo_UL61730_Cert.pdf"
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "it/panel/12",
                    "descriptor": {
                        "name": "Solar Panel Installation Service",
                        "short_desc": "On-site installation for rooftop or ground-mounted solar panels",
                        "long_desc": "Includes mounting structure setup, panel alignment, inverter integration, grid connection readiness, and post-installation testing. Covers both residential and commercial rooftop systems."
                    },
                    "price": {
                        "value": "250",
                        "currency": "USD"
                    },
                    "rateable": true,
                    "rating": "4.8",
                    "category_ids": [
                        "solar_installation"
                    ],
                    "fulfillment_ids": [
                        "sv1"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "installation_specification"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "installation_type"
                                    },
                                    "value": "Rooftop / Ground-mount"
                                },
                                {
                                    "descriptor": {
                                        "code": "supported_panel_types"
                                    },
                                    "value": "Mono PERC, Polycrystalline, Thin Film"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_limit"
                                    },
                                    "value": "Up to 10kW"
                                },
                                {
                                    "descriptor": {
                                        "code": "included_services"
                                    },
                                    "value": "Mounting, inverter wiring, earthing, net-meter interface"
                                },
                                {
                                    "descriptor": {
                                        "code": "installation_duration"
                                    },
                                    "value": "4–6 hours"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installer_qualification"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "empaneled_with_nodal_agency"
                                    },
                                    "value": "Yes"
                                },
                                {
                                    "descriptor": {
                                        "code": "safety_certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Evan Smith",
                "email": "evan.smith@example.com",
                "phone": "+14155558888",
                "address": "456 Solar Valley Rd, San Jose, CA"
            },
            "fulfillments": [
                {
                    "id": "hd1",
                    "type": "HOME_DELIVERY",
                    "state": {
                        "descriptor": {
                            "code": "Order-confirmed",
                            "name": "Order Confirmed"
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Solar Valley Rd, San Jose, CA"
                            },
                            "time": {
                                "range": {
                                    "start": "2025-05-08T09:00:00Z",
                                    "end": "2025-05-08T17:00:00Z"
                                }
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "value": "345",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Panel Price",
                        "price": {
                            "value": "320",
                            "currency": "USD"
                        }
                    },
                    {
                        "title": "Home Delivery",
                        "price": {
                            "value": "25",
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
            ],
            "cancellation_terms": [
                {
                    "fulfillment_state": {
                        "descriptor": {
                            "code": "order_placed",
                            "name": "Order Placed"
                        }
                    },
                    "reason_required": true,
                    "cancel_by": {
                        "timestamp": "2025-05-07T20:00:00Z"
                    },
                    "cancellation_fee": {
                        "percentage": 5,
                        "minimum_value": {
                            "currency": "USD",
                            "value": "10"
                        }
                    },
                    "xinput": {
                        "required": true,
                        "form": {
                            "url": "https://example.com/cancel"
                        }
                    },
                    "external_ref": {
                        "url": "https://example.com/docs/cancellation-policy.pdf",
                        "mime_type": "application/pdf"
                    }
                }
            ]
        }
    }
}
```

### status

- Request for status on order. order_id is specifiedin message->order_id

```
{
    "context": {
        "domain": "retail",
        "action": "status",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bap_uri": "https://bap-consumer.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-status-001",
        "timestamp": "2025-05-07T12:30:40Z",
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
        "order_id": "order-sunpro-001"
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
        "domain": "retail",
        "action": "on_status",
        "version": "1.1.0",
        "bap_id": "bap-consumer.example.com",
        "bpp_id": "bpp-solar-store.example.com",
        "bpp_uri": "https://bpp-solar-store.example.com",
        "transaction_id": "tx-solar-panel-001",
        "message_id": "msg-on-status-001",
        "timestamp": "2025-05-07T12:30:45Z",
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
            "id": "order-sunpro-001",
            "provider": {
                "id": "sunpro_energy",
                "descriptor": {
                    "name": "SunPro Energy Systems",
                    "short_desc": "Premium mono and polycrystalline solar panels"
                }
            },
            "items": [
                {
                    "id": "sp_mono_450w",
                    "descriptor": {
                        "name": "SunPro Mono 450W Panel",
                        "short_desc": "High-efficiency 450W panel for rooftop systems"
                    },
                    "price": {
                        "value": "320",
                        "currency": "USD"
                    },
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "Panel_Specifications"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "Type"
                                    },
                                    "value": "Monocrystalline"
                                },
                                {
                                    "descriptor": {
                                        "code": "Wattage"
                                    },
                                    "value": "450W"
                                },
                                {
                                    "descriptor": {
                                        "code": "Efficiency"
                                    },
                                    "value": "21.4%"
                                },
                                {
                                    "descriptor": {
                                        "code": "Warranty"
                                    },
                                    "value": "25 years"
                                },
                                {
                                    "descriptor": {
                                        "code": "IP_Rating"
                                    },
                                    "value": "IP68"
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
                                        "name": "solar equipment certificate",
                                        "code": "SEC"
                                    },
                                    "value": "https://www.example.com/certificates/solar/CEC_LISTED/SunPower_X21_345_Module_CEC_Certification.pdf"
                                },
                                {
                                    "descriptor": {
                                        "name": "pv module safety certificate",
                                        "code": "PMSC"
                                    },
                                    "value": "https://www.example.com/certificates/pv_module_safety_certificate/JinkoTigerNeo_UL61730_Cert.pdf"
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "it/panel/12",
                    "descriptor": {
                        "name": "Solar Panel Installation Service",
                        "short_desc": "On-site installation for rooftop or ground-mounted solar panels",
                        "long_desc": "Includes mounting structure setup, panel alignment, inverter integration, grid connection readiness, and post-installation testing. Covers both residential and commercial rooftop systems."
                    },
                    "price": {
                        "value": "250",
                        "currency": "USD"
                    },
                    "rateable": true,
                    "rating": "4.8",
                    "category_ids": [
                        "solar_installation"
                    ],
                    "fulfillment_ids": [
                        "sv1"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "installation_specification"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "installation_type"
                                    },
                                    "value": "Rooftop / Ground-mount"
                                },
                                {
                                    "descriptor": {
                                        "code": "supported_panel_types"
                                    },
                                    "value": "Mono PERC, Polycrystalline, Thin Film"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_limit"
                                    },
                                    "value": "Up to 10kW"
                                },
                                {
                                    "descriptor": {
                                        "code": "included_services"
                                    },
                                    "value": "Mounting, inverter wiring, earthing, net-meter interface"
                                },
                                {
                                    "descriptor": {
                                        "code": "installation_duration"
                                    },
                                    "value": "4–6 hours"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "installer_qualification"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "empaneled_with_nodal_agency"
                                    },
                                    "value": "Yes"
                                },
                                {
                                    "descriptor": {
                                        "code": "safety_certified"
                                    },
                                    "value": "Yes"
                                }
                            ]
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Evan Smith",
                "email": "evan.smith@example.com",
                "phone": "+14155558888",
                "address": "456 Solar Valley Rd, San Jose, CA"
            },
            "fulfillments": [
                {
                    "id": "hd1",
                    "type": "HOME_DELIVERY",
                    "state": {
                        "descriptor": {
                            "code": "Out-for-delivery",
                            "name": "Out for Delivery"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Carlos Green",
                            "phone": "+14155551234"
                        },
                        "organization": {
                            "descriptor": {
                                "name": "SunPro Delivery Services"
                            }
                        }
                    },
                    "stops": [
                        {
                            "type": "end",
                            "location": {
                                "address": "456 Solar Valley Rd, San Jose, CA"
                            },
                            "time": {
                                "range": {
                                    "start": "2025-05-08T09:00:00Z",
                                    "end": "2025-05-08T17:00:00Z"
                                }
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "value": "345",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Panel Price",
                        "price": {
                            "value": "320",
                            "currency": "USD"
                        }
                    },
                    {
                        "title": "Home Delivery",
                        "price": {
                            "value": "25",
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
            ],
            "cancellation_terms": [
                {
                    "fulfillment_state": {
                        "descriptor": {
                            "code": "order_placed",
                            "name": "Order Placed"
                        }
                    },
                    "reason_required": true,
                    "cancel_by": {
                        "timestamp": "2025-05-07T20:00:00Z"
                    },
                    "cancellation_fee": {
                        "percentage": 5,
                        "minimum_value": {
                            "currency": "USD",
                            "value": "10"
                        }
                    },
                    "xinput": {
                        "required": true,
                        "form": {
                            "url": "https://example.com/cancel"
                        }
                    },
                    "external_ref": {
                        "url": "https://example.com/docs/cancellation-policy.pdf",
                        "mime_type": "application/pdf"
                    }
                }
            ]
        }
    }
}
```


## Taxonomy and layer 2 configuration

Property Name|Enums|
|-------------|-----|
|Panel Categories|MONO_PANELS<br>POLY_PANELS<br>FLEX_PANELS<br>SOLAR_INSTALLATION_SERVICE|
|Fulfillment Types|HOME_DELIVERY<br>STORE_PICKUP<br>SITE_VISIT|
|Payment Types|ON-FULFILLMENT|
|Payment Collected By|BPP|
|Payment Status|NOT-PAID|
|Panel Specification Types|TYPE<br>WATTAGE<br>EFFICIENCY<br>WARRANTY<br>IP_RATING<br>WEIGHT<br>USE_CASE|
|Certification Types|SEC<br>PMSC|
|Installation Specification Types|INSTALLATION_TYPE<br>SUPPORTED_PANEL_TYPES<br>SYSTEM_CAPACITY_LIMIT<br>INCLUDED_SERVICES<br>INSTALLATION_DURATION|
|Installer Qualification Types|EMPANELED_WITH_NODAL_AGENCY<br>SAFETY_CERTIFIED|
|payment type| PRE_ORDER<br>POST_ORDER<br>PRE_FULFILLMENT<br>POST_FULFILLMENT|
|payment status| PAID<br>NOT_PAID<br>FAILED<br>PROCESSING
|payment collected by| BAP<br>BPP<br>AGENT
|fulfillment stop types| START<br>END

## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to artefacts

- [Postman collection for Solar Panel Purchase](https://documenter.getpostman.com/view/23690031/2sB2qUmQAU#7f196f71-6ff6-457d-b813-fdca59faf12f)
- [Layer2 config for Solar Panel Purchase]()

## Sandbox Details

**Registry/Gateway:**

* Gateway: [gateway-deg.becknprotocol.io](gateway-deg.becknprotocol.io	)
* Registry: [https://dedi-registry.becknprotocol.io](https://dedi-registry.becknprotocol.io)

**BAP:**

* BAP Client: [bap-ps-client-deg.becknprotocol.io](bap-ps-client-deg.becknprotocol.io)
* BAP Network: [bap-ps-network-deg.becknprotocol.io](bap-ps-network-deg.becknprotocol.io)

**BPP:**

* BPP Client: [bpp-ps-client-deg.becknprotocol.io](bpp-ps-client-deg.becknprotocol.io)
* BPP Network: [bpp-ps-network-deg.becknprotocol.io	](bpp-ps-network-deg.becknprotocol.io	)