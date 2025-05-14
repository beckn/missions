# DEG Implementation Guide - Subsidy

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 12-05-2025 | 1.0     | Initial Version                                     |


## Introduction

This document provides integration guidelines for implementing energy subsidy distribution programs over a Beckn-enabled open energy network. It outlines how governments and utility providers in San Francisco can publish targeted subsidy schemes, enabling eligible users to discover, apply for, and redeem financial support for electricity usage or renewable energy adoption.

This guide assumes the reader is familiar with the Beckn protocol, message flow, and schema standards. It maps the JSON-based catalog provided by participating BPPs to the Beckn on_search response pattern, facilitating seamless discovery and access to subsidy programs within the network.

## Structure of the Document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualization) - A pictorial or descriptive representation of the use case.
2. [General Flow Diagrams](#general-flow-diagrams) - This section is relevant to all the message flows illustrated below and discussed further in the document.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the messages that are sent in the form of sample schemas.
4. [Taxonomy and Layer 2 Configuration](#taxonomy-and-layer-2-configuration) - This section provides details on the taxonomy, enumerations, and any rules defined for either the use case or by the network.
5. [Integration Notes](#notes-on-software-integration) - Notes on writing/integrating with your own software.
6. [Links to Artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Registry/Gateway, and BPP.

## Outcome Visualization

### Use case - Discovery and Application of Subsidy programs

Jordan Smith, a resident of San Francisco, wants to save on energy costs and contribute to the city’s clean energy transition. He is particularly interested in installing rooftop solar panels but is looking for financial support to reduce the upfront cost.

He installs a Beckn-enabled energy app (powered by [example-bap.com](https://www.example-bap.com)) and discovers active subsidy programs from utility providers in the San Francisco region. Through the app, he finds the following program:

- **Residential Solar Panel Rebate – Up to $2500**  
  _Offered by the SF Department of Energy Support_  
  _One-time subsidy on approved rooftop solar purchases_  
  _Application deadline: 2025-12-31_

### Jordan’s Journey:
1. Uses the app to:
   - Discover the rebate program from **SF Department of Energy Support** (`sf-green-energy-office.gov`)
   - Review details like:
     - **Subsidy Type**: Direct Purchase Rebate  
     - **Max Amount**: $2500 USD  
     - **Eligibility**:
       - Property Type: Residential  
       - System Capacity: Minimum 2 kW  
       - Certified Installer required (SF-approved)  
2. Applies digitally via the Beckn-enabled platform:
   - Submits the application form
   - Uploads supporting documents (e.g., proof of property ownership, installation quote from certified vendor)
3. Tracks the application status:
   - Receives acknowledgment with Transaction ID: `subsidy-txn-002`  
   - Status moves from `INITIATED` to `Approved` through updates in the app  
4. Once approved:
   - Receives confirmation and sees the subsidy reflected in the app dashboard  
   - Coordinates with the certified installer to complete installation


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
        "domain": "scheme",
        "action": "search",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "www.example-bap.com",
        "transaction_id": "subsidy-txn-002",
        "message_id": "search-msg-002",
        "timestamp": "2025-05-08T15:59:30Z",
        "ttl": "PT10S",
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
            "category": {
                "descriptor": {
                    "code": "subsidy-scheme"
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
        "domain": "scheme",
        "action": "on_search",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-002",
        "timestamp": "2025-05-08T16:00:00Z",
        "ttl": "PT10S",
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
        "catalog": {
            "descriptor": {
                "name": "San Francisco Solar Panel Subsidy Programs",
                "short_desc": "Explore subsidies for solar panel purchases in the Bay Area"
            },
            "providers": [
                {
                    "id": "sf_energy_office",
                    "descriptor": {
                        "name": "SF Department of Energy Support",
                        "short_desc": "State incentives for clean energy adoption",
                        "long_desc": "Provides incentives, rebates, and support programs for residents installing solar panels, as part of San Francisco's clean energy transition goals."
                    },
                    "categories": [
                        {
                            "id": "solar_purchase_subsidy",
                            "descriptor": {
                                "code": "solar_purchase_subsidy",
                                "name": "Solar Panel Purchase Subsidy"
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
                            "id": "subsidy_enrollment_digital",
                            "type": "DIGITAL"
                        }
                    ],
                    "items": [
                        {
                            "id": "purchase-rebate-basic",
                            "descriptor": {
                                "name": "Residential Solar Panel Rebate - Up to $2500",
                                "short_desc": "One-time subsidy on approved rooftop solar purchases"
                            },
                            "price": {
                                "estimated_value": "0",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "solar_purchase_subsidy"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "subsidy_enrollment_digital"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_program_details"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "subsidy_type"
                                            },
                                            "value": "Direct Purchase Rebate"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "max_subsidy_amount_usd"
                                            },
                                            "value": "2500"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "application_deadline"
                                            },
                                            "value": "2025-12-31"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "administered_by"
                                            },
                                            "value": "SF Energy Department"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "eligibility_criteria"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "property_type"
                                            },
                                            "value": "Residential"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "system_capacity_min_kw"
                                            },
                                            "value": "2"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "system_certified"
                                            },
                                            "value": "Yes - by SF Certified Installer"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "ev-load-balance-incentive",
                            "descriptor": {
                                "name": "Smart EV Charger Load-Balancing Incentive",
                                "short_desc": "Incentive for smart charger installations that support grid load-balancing"
                            },
                            "price": {
                                "estimated_value": "0",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "solar_purchase_subsidy"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "subsidy_enrollment_digital"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_program_details"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "subsidy_type"
                                            },
                                            "value": "Load-Balancing Rebate"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "max_subsidy_amount_usd"
                                            },
                                            "value": "500"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "application_mode"
                                            },
                                            "value": "Auto-applied upon installation"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "administered_by"
                                            },
                                            "value": "San Francisco Grid Authority"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "eligibility_criteria"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "required_device_type"
                                            },
                                            "value": "Smart EV Charger"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "grid_responsive"
                                            },
                                            "value": "Yes"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "installation_certified"
                                            },
                                            "value": "Yes"
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
                                                "code": "required_documents"
                                            },
                                            "value": "Installation Report, Smart Charger ID, Utility Consent"
                                        }
                                    ]
                                }
                            ]
                        },
                        {
                            "id": "carbon-grid-enrollment-incentive",
                            "descriptor": {
                                "name": "Carbon Savings & Grid Enrollment Incentive",
                                "short_desc": "Auto-enrollment incentive for carbon savings or grid-participation programs"
                            },
                            "price": {
                                "estimated_value": "0",
                                "currency": "USD"
                            },
                            "category_ids": [
                                "solar_purchase_subsidy"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "subsidy_enrollment_digital"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_program_details"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "subsidy_type"
                                            },
                                            "value": "Performance-Based Incentive"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "program_name"
                                            },
                                            "value": "SF Carbon Efficiency Rebate"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "administered_by"
                                            },
                                            "value": "SF Carbon & Grid Response Unit"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "application_mode"
                                            },
                                            "value": "Auto-enrollment via device agent"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "eligibility_criteria"
                                    },
                                    "display": true,
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "device_agent_required"
                                            },
                                            "value": "Yes"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "participation_type"
                                            },
                                            "value": "Carbon Tracking or Grid Load Response"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "minimum_duration_months"
                                            },
                                            "value": "6"
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
                                                "code": "required_documents"
                                            },
                                            "value": "Carbon tracking reports, device telemetry consent, participation certificate (if any)"
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
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-005",
        "timestamp": "2025-05-08T16:01:20Z",
        "ttl": "PT10S",
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
                "id": "sf_energy_office"
            },
            "items": [
                {
                    "id": "purchase-rebate-basic"
                }
            ],
            "fulfillments": [
                {
                    "id": "subsidy_enrollment_digital",
                    "type": "DIGITAL",
                    "customer": {
                        "person": {
                            "name": "Jordan Smith"
                        },
                        "contact": {
                            "email": "jordan.smith@example.com",
                            "phone": "+14155556789"
                        }
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
        "domain": "service",
        "action": "on_confirm",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-005",
        "timestamp": "2025-05-08T16:01:20Z",
        "ttl": "PT10S",
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
            "id": "ord/id/01",
            "provider": {
                "id": "sf_energy_office",
                "descriptor": {
                    "name": "SF Department of Energy Support",
                    "short_desc": "State incentives for clean energy adoption",
                    "long_desc": "Provides incentives, rebates, and support programs for residents installing solar panels, as part of San Francisco's clean energy transition goals."
                }
            },
            "items": [
                {
                    "id": "purchase-rebate-basic",
                    "descriptor": {
                        "name": "Residential Solar Panel Rebate - Up to $2500",
                        "short_desc": "One-time subsidy on approved rooftop solar purchases"
                    },
                    "price": {
                        "estimated_value": "0",
                        "currency": "USD"
                    },
                    "category_ids": [
                        "solar_purchase_subsidy"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "fulfillment_ids": [
                        "subsidy_enrollment_digital"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "subsidy_program_details"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_type"
                                    },
                                    "value": "Direct Purchase Rebate"
                                },
                                {
                                    "descriptor": {
                                        "code": "max_subsidy_amount_usd"
                                    },
                                    "value": "2500"
                                },
                                {
                                    "descriptor": {
                                        "code": "application_deadline"
                                    },
                                    "value": "2025-12-31"
                                },
                                {
                                    "descriptor": {
                                        "code": "administered_by"
                                    },
                                    "value": "SF Energy Department"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility_criteria"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_min_kw"
                                    },
                                    "value": "2"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_certified"
                                    },
                                    "value": "Yes - by SF Certified Installer"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "subsidy_enrollment_digital",
                    "type": "DIGITAL",
                    "customer": {
                        "person": {
                            "name": "Jordan Smith"
                        },
                        "contact": {
                            "email": "jordan.smith@example.com",
                            "phone": "+14155556789"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "INITIATED"
                        }
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
        "domain": "service",
        "action": "status",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-005",
        "timestamp": "2025-05-08T16:01:20Z",
        "ttl": "PT10S",
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
        "order_id": "ord/id/01"
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
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-005",
        "timestamp": "2025-05-08T16:01:20Z",
        "ttl": "PT10S",
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
            "id": "ord/id/01",
            "provider": {
                "id": "sf_energy_office",
                "descriptor": {
                    "name": "SF Department of Energy Support",
                    "short_desc": "State incentives for clean energy adoption",
                    "long_desc": "Provides incentives, rebates, and support programs for residents installing solar panels, as part of San Francisco's clean energy transition goals."
                }
            },
            "items": [
                {
                    "id": "purchase-rebate-basic",
                    "descriptor": {
                        "name": "Residential Solar Panel Rebate - Up to $2500",
                        "short_desc": "One-time subsidy on approved rooftop solar purchases"
                    },
                    "price": {
                        "estimated_value": "0",
                        "currency": "USD"
                    },
                    "category_ids": [
                        "solar_purchase_subsidy"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "fulfillment_ids": [
                        "subsidy_enrollment_digital"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "subsidy_program_details"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_type"
                                    },
                                    "value": "Direct Purchase Rebate"
                                },
                                {
                                    "descriptor": {
                                        "code": "max_subsidy_amount_usd"
                                    },
                                    "value": "2500"
                                },
                                {
                                    "descriptor": {
                                        "code": "application_deadline"
                                    },
                                    "value": "2025-12-31"
                                },
                                {
                                    "descriptor": {
                                        "code": "administered_by"
                                    },
                                    "value": "SF Energy Department"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility_criteria"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_min_kw"
                                    },
                                    "value": "2"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_certified"
                                    },
                                    "value": "Yes - by SF Certified Installer"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "subsidy_enrollment_digital",
                    "type": "DIGITAL",
                    "customer": {
                        "person": {
                            "name": "Jordan Smith"
                        },
                        "contact": {
                            "email": "jordan.smith@example.com",
                            "phone": "+14155556789"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "AWAITING_APPROVAL"
                        }
                    }
                }
            ]
        }
    }
}
```

### on_status 2

```
{
    "context": {
        "domain": "service",
        "action": "on_status",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-005",
        "timestamp": "2025-05-08T16:01:20Z",
        "ttl": "PT10S",
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
            "id": "ord/id/01",
            "provider": {
                "id": "sf_energy_office",
                "descriptor": {
                    "name": "SF Department of Energy Support",
                    "short_desc": "State incentives for clean energy adoption",
                    "long_desc": "Provides incentives, rebates, and support programs for residents installing solar panels, as part of San Francisco's clean energy transition goals."
                }
            },
            "items": [
                {
                    "id": "purchase-rebate-basic",
                    "descriptor": {
                        "name": "Residential Solar Panel Rebate - Up to $2500",
                        "short_desc": "One-time subsidy on approved rooftop solar purchases"
                    },
                    "price": {
                        "estimated_value": "0",
                        "currency": "USD"
                    },
                    "category_ids": [
                        "solar_purchase_subsidy"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "fulfillment_ids": [
                        "subsidy_enrollment_digital"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "subsidy_program_details"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_type"
                                    },
                                    "value": "Direct Purchase Rebate"
                                },
                                {
                                    "descriptor": {
                                        "code": "max_subsidy_amount_usd"
                                    },
                                    "value": "2500"
                                },
                                {
                                    "descriptor": {
                                        "code": "application_deadline"
                                    },
                                    "value": "2025-12-31"
                                },
                                {
                                    "descriptor": {
                                        "code": "administered_by"
                                    },
                                    "value": "SF Energy Department"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility_criteria"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_min_kw"
                                    },
                                    "value": "2"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_certified"
                                    },
                                    "value": "Yes - by SF Certified Installer"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "subsidy_enrollment_digital",
                    "type": "DIGITAL",
                    "customer": {
                        "person": {
                            "name": "Jordan Smith"
                        },
                        "contact": {
                            "email": "jordan.smith@example.com",
                            "phone": "+14155556789"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "APPROVED"
                        }
                    }
                }
            ]
        }
    }
}
```

### on_status 3

```
{
    "context": {
        "domain": "service",
        "action": "on_status",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "www.example-bap.com",
        "bpp_id": "sf-green-energy-office.gov",
        "bpp_uri": "https://api.sf-green-energy-office.gov",
        "transaction_id": "subsidy-txn-002",
        "message_id": "subsidy-msg-005",
        "timestamp": "2025-05-08T16:01:20Z",
        "ttl": "PT10S",
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
            "id": "ord/id/01",
            "provider": {
                "id": "sf_energy_office",
                "descriptor": {
                    "name": "SF Department of Energy Support",
                    "short_desc": "State incentives for clean energy adoption",
                    "long_desc": "Provides incentives, rebates, and support programs for residents installing solar panels, as part of San Francisco's clean energy transition goals."
                }
            },
            "items": [
                {
                    "id": "purchase-rebate-basic",
                    "descriptor": {
                        "name": "Residential Solar Panel Rebate - Up to $2500",
                        "short_desc": "One-time subsidy on approved rooftop solar purchases"
                    },
                    "price": {
                        "estimated_value": "0",
                        "currency": "USD"
                    },
                    "category_ids": [
                        "solar_purchase_subsidy"
                    ],
                    "location_ids": [
                        "1"
                    ],
                    "fulfillment_ids": [
                        "subsidy_enrollment_digital"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "code": "subsidy_program_details"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "subsidy_type"
                                    },
                                    "value": "Direct Purchase Rebate"
                                },
                                {
                                    "descriptor": {
                                        "code": "max_subsidy_amount_usd"
                                    },
                                    "value": "2500"
                                },
                                {
                                    "descriptor": {
                                        "code": "application_deadline"
                                    },
                                    "value": "2025-12-31"
                                },
                                {
                                    "descriptor": {
                                        "code": "administered_by"
                                    },
                                    "value": "SF Energy Department"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility_criteria"
                            },
                            "display": true,
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "property_type"
                                    },
                                    "value": "Residential"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_capacity_min_kw"
                                    },
                                    "value": "2"
                                },
                                {
                                    "descriptor": {
                                        "code": "system_certified"
                                    },
                                    "value": "Yes - by SF Certified Installer"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "subsidy_enrollment_digital",
                    "type": "DIGITAL",
                    "customer": {
                        "person": {
                            "name": "Jordan Smith"
                        },
                        "contact": {
                            "email": "jordan.smith@example.com",
                            "phone": "+14155556789"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "CANCELLED"
                        }
                    }
                }
            ]
        }
    }
}
```

## Taxonomy and layer 2 configuration

|Property Name|Enums|
|-------------|-----|
|Subsidy categories|SOLAR_PURCHASE_SUBSIDY|
|fulfillment_type|DIGITAL|
|property-type|RESIDENTIAL<br>COMMERCIAL<br>INDUSTRIAL<br>AGRICULTURAL|
|fulfillment state|APPLICATION_SUCCESSFUL<br>AWAITING_APPROVAL<br>CANCELLED<br>APPROVED|
|stops types|END|
|payment-status|PENDING<br>PROCESSED<br>FAILED<br>REFUNDED|
|payment type| POST_ORDER<br>PRE_FULFILLMENT<br>POST_FULFILLMENT|
|payment collected by| BAP<br>BPP<br>AGENT
|fulfillment stop types| START<br>END

## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to artefacts

- [Postman collection for Subsidy](https://documenter.getpostman.com/view/23690031/2sB2qUmQAU#5c3ebe79-8c10-4546-8de8-4657cdf62068)
- [Layer2 config for Subsidy]()

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