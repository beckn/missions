# UAI Implementation Guide - Crop Insurance

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 06-11-2024 | 0.1     | Initial Version                                     |
| 14-11-2024 | 0.2     | Internal Review Comments Incorporated                |
| 18-11-2024 | 1.0     | Final Version                                       |

## Introduction

This document provides material that helps network participants build and integrate their application with the Beckn Network. This document is part of the starter kit that provides information about the network, learning resources, network participant checklist etc. This document only focuses on the implementation of the seeker/provider platform. It assumes the reader has a good overview of the Beckn network, its APIs, the overall structure of the schema etc.

## Structure of the document

This document has the following parts:

1. Outcome Visualization - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
2. Flow diagrams - This section provides a pictorial representation of the message flows that happen during the use case.
3. API Calls and Schema - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. Taxonomy and layer 2 configuration - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
5. Notes on writing/integrating with your own software - This section describes ways in which you can integrate (Becknify) your new or existing software
6. Links to downloadable resources - This section contains the downloadable files referenced in this document.

## Outcome Visualisation

### Use case - Discovery and purchase of crop insurance

1. Rajesh, a guava farmer in Nashik district, faces unpredictable weather conditions and crop losses.
2. He wants to purchase an insurance product to protect against these risks.
3. Rajesh uses a Beckn-enabled app connected to the UKI network to search for insurance products.
4. He provides details: Crops (Guava), Farm location (Niphad, Nashik), Duration (2 months), Type of Cover (Rainfall/Temperature), Land area (5 acres).
5. The app displays insurance products from multiple providers with ratings, premiums, and claim settlement ratios.
6. Rajesh selects "Rainfall Cover" from Insurance Bazaar and proceeds with the application.
7. He completes KYC verification, provides nominee details, and bank account information.
8. Rajesh receives a final quote, reviews terms, and confirms the order via UPI payment.
9. The insurance provider shares an order ID and policy cover document (PDF).
10. Rajesh can track order status and submits ratings for the insurance product.
11. The app provides support services for any issues with the service delivery.

## Flow diagrams

### General Beckn message flow and error handling

This section is relevant to all the messages flows illustrated below and discussed further in the document.

Beckn is an asynchronous protocol at its core.

- When a network participant(NP1) sends a message to another participant(NP2), the other participant(NP2) immediately returns back an ACK/NACK(Acknowledgement or Negative Acknowledgement in case of error - usually with wrongly formed messages).
- An ACK is an indicator that the receiving participant(NP2) will process this message and dispatch an on_xxxxxx message to original NP (NP1)
- Subsequently after processing the message NP2 sends back the real response in the corresponding on_xxxxxx message, to which again the first participant(NP1).
- This message can contain a message field (for success) or error field (for failure)
- NP1 when it receives the on_xxxxxx message, sends back an ACK/NACK (Here in both the cases NP1 will not send any subsequent message).
- In the Use case diagrams, this ACK/NACK is not illustrated explicitly to keep the diagrams crisp.
- However when writing software we should be prepared to receive these NACK messages as well as error field in the on_xxxxxx messages
- While this discussion is from a Beckn perspective, Adapters can provide synchronous modes. For example, the Protocol Server which is the reference implementation of the Beckn Adapter provides a synchronous mode by default. So if your software calls the support endpoint on the BAP Protocol Server, the Protocol Server waits till it gets the on_support and returns back that as the response.

**Structure of a message with a NACK**

```
{
    "message": {
        "ack": {
            "status": "NACK"
        }
    },
    "error": {
        "code": 400,
        "message": "OpenApiValidator Error at BAP-CLIENT",
    }
}
```

**Structure of a on_select message with an error**

```
{
    "context": {
        "action": "on_select",
        "version": "1.1.0",
        ...
    },
    "error": {
        "code": 30001,
        "message": "Requested provider is not in the database"
    }
}
```

## API Calls and Schema

### Crop Insurance

#### search

**search for crop insurance products**

- The search includes crop details, farm location, insurance duration, and coverage type.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "search",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "2b159ca2-0d68-43f8-a2cd-5e0e2c649ab2",
        "timestamp": "2025-06-02T09:00:00Z",
        "ttl": "PT10S"
    },
    "message": {
        "intent": {
            "descriptor": {
                "name": "Crop Insurance"
            },
            "category": {
                "descriptor": {
                    "code": "Insurance"
                }
            },
            "fulfillment": {
                "type": "digital"
            },
            "item": {
                "tags": [
                    {
                        "descriptor": {
                            "code": "crop_details"
                        },
                        "list": [
                            {
                                "descriptor": {
                                    "code": "crop_name"
                                },
                                "value": "guava"
                            },
                            {
                                "descriptor": {
                                    "code": "land_area"
                                },
                                "value": "5 Acres"
                            },
                            {
                                "descriptor": {
                                    "code": "farm_location"
                                },
                                "value": "Niphad, Nashik"
                            }
                        ]
                    },
                    {
                        "descriptor": {
                            "code": "financial_terms"
                        },
                        "list": [
                            {
                                "descriptor": {
                                    "code": "insurance_duration"
                                },
                                "value": "2 Months"
                            }
                        ]
                    },
                    {
                        "descriptor": {
                            "code": "risk_coverage"
                        },
                        "list": [
                            {
                                "descriptor": {
                                    "code": "coverage_type"
                                },
                                "value": "Rainfall Cover"
                            }
                        ]
                    }
                ]
            }
        }
    }
}
```

#### on_search

**on_search with catalog of insurance products**

- The catalog contains insurance providers with their products, premiums, and policy details.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_search",
        "version": "1.1.0",
        "location": {
            "country": {
                "code": "IND",
                "name": "India"
            },
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "2b159ca2-0d68-43f8-a2cd-5e0e2c649ab2",
        "timestamp": "2025-06-02T09:00:03Z",
        "ttl": "PT10M"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "p1",
                    "descriptor": {
                        "name": "Pradhan Mantri Fasal Bhima Yojana",
                        "short_desc": "Government of India crop insurance"
                    },
                    "locations": [
                        {
                            "id": "l1",
                            "address": "Nashik District Office",
                            "area_code": "416506" // pin code
                            "city": {
                                "name": "Nashik"
                            },
                            "state": {
                                "name": "Maharashtra"
                            }
                        }
                    ],
                    "categories": [
                        {
                            "id": "c1",
                            "descriptor": {
                                "code": "insurance"
                            }
                        },
                        {
                            "id": "c2",
                            "descriptor": {
                                "code": "crop-insurance"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "f1",
                            "type": "digital"
                        }
                    ],
                    "items": [
                        {
                            "id": "pmfby-2025-1yr",
                            "descriptor": {
                                "name": "PMFBY – Seasonal Crop Cover"
                            },
                            "rating": "4.1",
                            "category_ids": [
                                "c1",
                                "c2"
                            ],
                            "fulfillment_ids": [
                                "f1"
                            ],
                            "location_ids": [
                                "l1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "policy_details"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "sum_insured"
                                            },
                                            "value": "10000 INR/acre"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "premium_payable"
                                            },
                                            "value": "250 INR/acre"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "policy_duration"
                                            },
                                            "value": "1 Year"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "start_date"
                                            },
                                            "value": "2025-05-01"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "end_date"
                                            },
                                            "value": "2026-04-30"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "performance_metrics"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "claims_settlement_ratio"
                                            },
                                            "value": "90%"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "p2",
                    "descriptor": {
                        "name": "Risk Free Insurers",
                        "short_desc": "High Temperature Crop Cover"
                    },
                    "locations": [
                        {
                            "id": "l1",
                            "address": "Nashik District Office",
                            "area_code": "416506" // pin code
                            "city": {
                                "name": "Nashik"
                            },
                            "state": {
                                "name": "Maharashtra"
                            }
                        }
                    ],
                    "categories": [
                        {
                            "id": "c1",
                            "descriptor": {
                                "code": "insurance"
                            }
                        },
                        {
                            "id": "c2",
                            "descriptor": {
                                "code": "crop-insurance"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "f1",
                            "type": "digital"
                        }
                    ],
                    "items": [
                        {
                            "id": "ht-cover-30d",
                            "descriptor": {
                                "name": "High Temperature Cover"
                            },
                            "rating": "4.5",
                            "category_ids": [
                                "c1",
                                "c2"
                            ],
                            "fulfillment_ids": [
                                "f1"
                            ],
                            "location_ids": [
                                "l1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "policy_details"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "sum_insured"
                                            },
                                            "value": "10000 INR/acre"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "premium_payable"
                                            },
                                            "value": "500 INR/acre"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "policy_duration"
                                            },
                                            "value": "30 Days"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "start_date"
                                            },
                                            "value": "2025-05-01"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "end_date"
                                            },
                                            "value": "2025-05-31"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "performance_metrics"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "claims_settlement_ratio"
                                            },
                                            "value": "98%"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "p3",
                    "descriptor": {
                        "name": "Insurance Bazaar",
                        "short_desc": "Rainfall Risk Crop Cover"
                    },
                    "locations": [
                        {
                            "id": "l1",
                            "address": "Nashik District Office",
                            "area_code": "416506" // pin code
                            "city": {
                                "name": "Nashik"
                            },
                            "state": {
                                "name": "Maharashtra"
                            }
                        }
                    ],
                    "categories": [
                        {
                            "id": "c1",
                            "descriptor": {
                                "code": "insurance"
                            }
                        },
                        {
                            "id": "c2",
                            "descriptor": {
                                "code": "crop-insurance"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "f1",
                            "type": "digital"
                        }
                    ],
                    "items": [
                        {
                            "id": "rainfall-cover-30d",
                            "descriptor": {
                                "name": "Rainfall Cover"
                            },
                            "rating": "4.2",
                            "category_ids": [
                                "c1",
                                "c2"
                            ],
                            "fulfillment_ids": [
                                "f1"
                            ],
                            "location_ids": [
                                "l1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "policy_details"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "sum_insured"
                                            },
                                            "value": "15000 INR/acre"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "premium_payable"
                                            },
                                            "value": "500 INR/acre"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "policy_duration"
                                            },
                                            "value": "90 Days"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "start_date"
                                            },
                                            "value": "2025-05-01"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "end_date"
                                            },
                                            "value": "2025-07-31"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "performance_metrics"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "claims_settlement_ratio"
                                            },
                                            "value": "95%"
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

#### select

**select insurance product**

- Rajesh selects a specific insurance product and proceeds with his intent to purchase it.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "select",
        "version": "1.1.0",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "4d5e6f7g-8h9i-0j1k-2l3m-4n5o6p7q8r9",
        "timestamp": "2025-06-02T09:06:00Z",
        "ttl": "PT10M",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "location": {
            "country": {
                "code": "IND",
                "name": "India"
            },
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            }
        }
    },
    "message": {
        "order": {
            "provider": {
                "id": "p3"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital"
                }
            ]
        }
    }
}
```

#### on_select

**on_select response with detailed quote**

- The provider responds by validating initial eligibility and provides a detailed quote.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_select",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3000"
                },
                "breakup": [
                    {
                        "title": "Insurance Fee",
                        "price": {
                            "currency": "INR",
                            "value": "2500"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500"
                        }
                    }
                ]
            }
        }
    }
}
```

#### init

**init order with personal details**

- Rajesh confirms his eligibility and proceeds to submit personal details, billing and shipping addresses.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "init",
        "version": "1.1.0",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "4d5e6f7g-8h9i-0j1k-2l3m-4n5o6p7q8r9",
        "timestamp": "2025-06-02T09:06:00Z",
        "ttl": "PT10M",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "location": {
            "country": {
                "code": "IND",
                "name": "India"
            },
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            }
        }
    },
    "message": {
        "order": {
            "provider": {
                "id": "p3"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    }
                }
            ],
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            }
        }
    }
}
```

#### on_init

**on_init response with locked quote**

- The BPP responds by locking the quote and validating the final user inputs.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_init",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ],
                    "xinput": {
                        "required": false,
                        "head": {
                            "descriptor": {
                                "name": "Requirements Form"
                            },
                            "index": {
                                "min": 0,
                                "cur": 0,
                                "max": 4
                            },
                            "headings": [
                                "Land‐holding proof",
                                "Identity proof",
                                "Bank account details",
                                "Sowing/planting declaration",
                                "Application form"
                            ]
                        },
                        "form": {
                            "mime_type": "text/html",
                            "url": "https://6vs8xnx5i7.loan-finder.co.in/schems/xinput/formid/a23f2fdfbbb8ac402bfd54f",
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
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3000"
                },
                "breakup": [
                    {
                        "title": "Insurance Fee",
                        "price": {
                            "currency": "INR",
                            "value": "2500"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500"
                        }
                    }
                ]
            }
        }
    }
}
```


#### confirm

**confirm order with payment**

- Rajesh confirms the order by initiating payment via UPI.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "confirm",
        "version": "1.1.0",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "4d5e6f7g-8h9i-0j1k-2l3m-4n5o6p7q8r9",
        "timestamp": "2025-06-02T09:06:00Z",
        "ttl": "PT10M",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "location": {
            "country": {
                "code": "IND",
                "name": "India"
            },
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            }
        }
    },
    "message": {
        "order": {
            "provider": {
                "id": "p3"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    }
                }
            ],
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3000.00"
                    }
                }
            ]
        }
    }
}
```

#### on_confirm

**on_confirm response with order ID and policy document**

- The BPP confirms receipt of payment and issues a unique Order ID along with the digital policy document.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_confirm",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "APPLICATION_SUCCESSFUL"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3000"
                },
                "breakup": [
                    {
                        "title": "Insurance Fee",
                        "price": {
                            "currency": "INR",
                            "value": "2500"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3000.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "time": "2025-04-30T23:59:59Z",
                        "label": "Full refund window"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                },
                {
                    "cancel_by": {
                        "time": "2025-05-31T23:59:59Z",
                        "label": "Partial refund window (within 30 days)"
                    },
                    "cancellation_fee": {
                        "percentage": "50%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "No cancellation allowed after 30 days"
                    },
                    "cancellation_fee": {
                        "percentage": "100%"
                    }
                }
            ]
        }
    }
}
```

#### status

**check order status**

- Rajesh wishes to check the status of his insurance policy after purchase.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:30Z",
        "ttl": "PT10S"
    },
    "message": {
        "order_id": "ord0123"
    }
}
```

#### on_status

**on_status response with policy status**

- The BPP returns the current policy status and any fulfillment updates.

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "state": "Active",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "state": "Active"
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "state": {
                        "descriptor": {
                            "code": "ACTIVE"
                        }
                    }
                }
            ]
        }
    }
}
```


#### on_status2
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "KYC_SUCCESSFUL"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3000"
                },
                "breakup": [
                    {
                        "title": "Insurance Fee",
                        "price": {
                            "currency": "INR",
                            "value": "2500"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3000.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "time": "2025-04-30T23:59:59Z",
                        "label": "Full refund window"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                },
                {
                    "cancel_by": {
                        "time": "2025-05-31T23:59:59Z",
                        "label": "Partial refund window (within 30 days)"
                    },
                    "cancellation_fee": {
                        "percentage": "50%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "No cancellation allowed after 30 days"
                    },
                    "cancellation_fee": {
                        "percentage": "100%"
                    }
                }
            ]
        }
    }
}
```

#### on_status3
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        },
                        "person": {
                            "name": "Suhana Khan",
                            "languages": [
                                {
                                    "code": "Hindi"
                                },
                                {
                                    "code": "English"
                                },
                                {
                                    "code": "Marathi"
                                }
                            ]
                        },
                        "contact": {
                            "phone": "+9876******"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "AGENT_ASSIGNED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3000"
                },
                "breakup": [
                    {
                        "title": "Insurance Fee",
                        "price": {
                            "currency": "INR",
                            "value": "2500"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3000.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "time": "2025-04-30T23:59:59Z",
                        "label": "Full refund window"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                },
                {
                    "cancel_by": {
                        "time": "2025-05-31T23:59:59Z",
                        "label": "Partial refund window (within 30 days)"
                    },
                    "cancellation_fee": {
                        "percentage": "50%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "No cancellation allowed after 30 days"
                    },
                    "cancellation_fee": {
                        "percentage": "100%"
                    }
                }
            ]
        }
    }
}
```

#### on_status4
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        },
                        "person": {
                            "name": "Suhana Khan",
                            "languages": [
                                {
                                    "code": "Hindi"
                                },
                                {
                                    "code": "English"
                                },
                                {
                                    "code": "Marathi"
                                }
                            ]
                        },
                        "contact": {
                            "phone": "+9876******"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "FINAL_QUOTE_SHARED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3481.00"
                },
                "breakup": [
                    {
                        "title": "Base Insurance Premium",
                        "price": {
                            "currency": "INR",
                            "value": "2500.00"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500.00"
                        }
                    },
                    {
                        "title": "KYC Verification Fee",
                        "price": {
                            "currency": "INR",
                            "value": "100.00"
                        }
                    },
                    {
                        "title": "Underwriting Fee",
                        "price": {
                            "currency": "INR",
                            "value": "50.00"
                        }
                    },
                    {
                        "title": "No-Claims Discount",
                        "price": {
                            "currency": "INR",
                            "value": "-200.00"
                        }
                    },
                    {
                        "title": "GST (18%)",
                        "price": {
                            "currency": "INR",
                            "value": "531.00"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3481.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "time": "2025-04-30T23:59:59Z",
                        "label": "Full refund window"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                },
                {
                    "cancel_by": {
                        "time": "2025-05-31T23:59:59Z",
                        "label": "Partial refund window (within 30 days)"
                    },
                    "cancellation_fee": {
                        "percentage": "50%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "No cancellation allowed after 30 days"
                    },
                    "cancellation_fee": {
                        "percentage": "100%"
                    }
                }
            ]
        }
    }
}
```

#### on_status5
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        },
                        "person": {
                            "name": "Suhana Khan",
                            "languages": [
                                {
                                    "code": "Hindi"
                                },
                                {
                                    "code": "English"
                                },
                                {
                                    "code": "Marathi"
                                }
                            ]
                        },
                        "contact": {
                            "phone": "+9876******"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "ORDER_FULFILLED"
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "time": {
                                "timestamp": "2025-06-05T10:00:00Z"
                            }
                        },
                        {
                            "type": "end",
                            "time": {
                                "timestamp": "2025-08-05T10:00:00Z"
                            },
                            "instructions": {
                                "short_desc": "Access the insurance docs at the below provided links",
                                "media": [
                                    {
                                        "mimetype": "application/pdf",
                                        "url": "https://bpp.example.com/policies/PMFBY-0001234.pdf"
                                    }
                                ]
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3481.00"
                },
                "breakup": [
                    {
                        "title": "Base Insurance Premium",
                        "price": {
                            "currency": "INR",
                            "value": "2500.00"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500.00"
                        }
                    },
                    {
                        "title": "KYC Verification Fee",
                        "price": {
                            "currency": "INR",
                            "value": "100.00"
                        }
                    },
                    {
                        "title": "Underwriting Fee",
                        "price": {
                            "currency": "INR",
                            "value": "50.00"
                        }
                    },
                    {
                        "title": "No-Claims Discount",
                        "price": {
                            "currency": "INR",
                            "value": "-200.00"
                        }
                    },
                    {
                        "title": "GST (18%)",
                        "price": {
                            "currency": "INR",
                            "value": "531.00"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3481.00",
                        "transaction_id": "tran0123"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "time": "2025-04-30T23:59:59Z",
                        "label": "Full refund window"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                },
                {
                    "cancel_by": {
                        "time": "2025-05-31T23:59:59Z",
                        "label": "Partial refund window (within 30 days)"
                    },
                    "cancellation_fee": {
                        "percentage": "50%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "No cancellation allowed after 30 days"
                    },
                    "cancellation_fee": {
                        "percentage": "100%"
                    }
                }
            ]
        }
    }
}
```

#### rating
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "rating",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "ratings": [
            {
                "id": "p3",
                "rating_category": "Provider",
                "value": "5"
            }
        ]
    }
}
```

#### on_rating
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_rating",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "feedback_form": {
            "form": {
                "url": "https://agri_acad.example.org/feedback"
            }
        }
    }
}
```

#### support
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "support",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "ref_id": "ord0123"
    }
}
```

#### on_support
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_support",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network"
    },
    "message": {
        "support": {
            "ref_id": "ord0123",
            "phone": "+14155550123",
            "email": "support@service-insurance-bazar.com"
        }
    }
}
```

#### update
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "update",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:30Z",
        "ttl": "PT10S"
    },
    "message": {
        "update_target": "order.items",
        "order": {
            "id": "ord0123",
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "xinput": {
                        "required": false,
                        "head": {
                            "descriptor": {
                                "name": "Complementary docs"
                            },
                            "index": {
                                "min": 0,
                                "cur": 0,
                                "max": 0
                            },
                            "headings": [
                                "Complementary documents"
                            ]
                        },
                        "form": {
                            "mime_type": "text/html",
                            "url": "https://6vs8xnx5i7.loan-finder.co.in/schems/xinput/formid/a23f2fdfbbb8ac402bfd54f",
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
            ]
        }
    }
}
```

#### on_update
```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_update",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "acd6d8c9-4f6f-4a5d-b705-3fd5ec9feaa9",
        "timestamp": "2025-06-02T09:01:33Z",
        "ttl": "PT10S"
    },
    "message": {
        "order": {
            "id": "ord0123",
            "provider": {
                "id": "p3",
                "name": "Insurance Bazaar",
                "short_desc": "Rainfall Risk Crop Cover"
            },
            "items": [
                {
                    "id": "rainfall-cover-30d",
                    "descriptor": {
                        "name": "Rainfall Cover"
                    },
                    "rating": "4.2",
                    "tags": [
                        {
                            "descriptor": {
                                "code": "policy_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "sum_insured"
                                    },
                                    "value": "15000 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "premium_payable"
                                    },
                                    "value": "500 INR/acre"
                                },
                                {
                                    "descriptor": {
                                        "code": "policy_duration"
                                    },
                                    "value": "90 Days"
                                },
                                {
                                    "descriptor": {
                                        "code": "start_date"
                                    },
                                    "value": "2025-05-01"
                                },
                                {
                                    "descriptor": {
                                        "code": "end_date"
                                    },
                                    "value": "2025-07-31"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "performance_metrics"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "claims_settlement_ratio"
                                    },
                                    "value": "95%"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "crop_details"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "crop_name"
                                    },
                                    "value": "guava"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "farm_location"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "address"
                                    },
                                    "value": "Niphad, Nashik"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "land_area"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "area"
                                    },
                                    "value": "5 Acres"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Rajesh"
                        },
                        "contact": {
                            "email": "Rajesh.Mahavir@mail.com",
                            "phone": "+9876******"
                        }
                    },
                    "agent": {
                        "organization": {
                            "descriptor": {
                                "name": "Insurance Bazaar"
                            }
                        },
                        "person": {
                            "name": "Suhana Khan",
                            "languages": [
                                {
                                    "code": "Hindi"
                                },
                                {
                                    "code": "English"
                                },
                                {
                                    "code": "Marathi"
                                }
                            ]
                        },
                        "contact": {
                            "phone": "+9876******"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "COMPLEMENTARY_DOCS_UPLOADED"
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "time": {
                                "timestamp": "2025-06-05T10:00:00Z"
                            }
                        },
                        {
                            "type": "end",
                            "time": {
                                "timestamp": "2025-08-05T10:00:00Z"
                            },
                            "instructions": {
                                "short_desc": "Access the insurance docs at the below provided links",
                                "media": [
                                    {
                                        "mimetype": "application/pdf",
                                        "url": "https://bpp.example.com/policies/PMFBY-0001234.pdf"
                                    }
                                ]
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3481.00"
                },
                "breakup": [
                    {
                        "title": "Base Insurance Premium",
                        "price": {
                            "currency": "INR",
                            "value": "2500.00"
                        }
                    },
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "500.00"
                        }
                    },
                    {
                        "title": "KYC Verification Fee",
                        "price": {
                            "currency": "INR",
                            "value": "100.00"
                        }
                    },
                    {
                        "title": "Underwriting Fee",
                        "price": {
                            "currency": "INR",
                            "value": "50.00"
                        }
                    },
                    {
                        "title": "No-Claims Discount",
                        "price": {
                            "currency": "INR",
                            "value": "-200.00"
                        }
                    },
                    {
                        "title": "GST (18%)",
                        "price": {
                            "currency": "INR",
                            "value": "531.00"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Ravi",
                "phone": "+9876543210",
                "email": "ravi.shinde@mail.com",
                "address": "Vill- Niphad, ishtar, Nashik"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "PRE_FULFILLMENT",
                    "status": "PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "3481.00",
                        "transaction_id": "tran0123"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "time": "2025-04-30T23:59:59Z",
                        "label": "Full refund window"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                },
                {
                    "cancel_by": {
                        "time": "2025-05-31T23:59:59Z",
                        "label": "Partial refund window (within 30 days)"
                    },
                    "cancellation_fee": {
                        "percentage": "50%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "No cancellation allowed after 30 days"
                    },
                    "cancellation_fee": {
                        "percentage": "100%"
                    }
                }
            ]
        }
    }
}
```


## Taxonomy and layer 2 configuration

- Any specific tags, enumerations, and rules we add for the use cases or required by the network, will go here.

## Integrating with your software

This section gives general walkthrough of how you would integrate your software with the Beckn network (say the sandbox environment). Refer to the starter kit for details on how to register with the sandbox and get credentials.

Beckn-ONIX is an initiative to promote easy install and maintenance of a Beckn Network. Apart from the Registry and Gateway components that are required for a network facilitator, Beckn-ONIX provides a Beckn Adapter. A reference implementation of the Beckn-ONIX specification is available at [Beckn-ONIX repository](https://github.com/beckn/beckn-onix). The reference implementation of the Beckn Adapter is called the Protocol Server. Based on whether we are writing the seeker platform or the provider platform, we will be installing the BAP Protocol Server or the BPP Protocol Server respectively.

### Integrating the seeker platform

If you are writing the seeker platform software, the following are the steps you can follow to build and integrate your application.

1. Identify the use cases from above section that are close to the functionality you plan for your application.
2. Design and develop the UI that implements the flow you need. Typically you will have a API server that this UI talks to and it is called the Seeker Platform in the diagram below.
3. The API server should construct the required JSON message packets required for the different endpoints shown in the API section above.
4. Install the BAP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL) and a subscriber_id (typically the same as subscriber URL without the "https://" prefix)
5. Install the layer 2 file for the domain (Link is in the last section of this document)
6. Check with your network tech support to enable your BAP Protocol Server in the registry.
7. Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,
   - you write the Seeker Platform(dark blue)
   - install the BAP Protocol Server (light blue)
   - the remaining components are provided by the sandbox environment
8. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

### Integrating the provider platform

If you are writing the provider platform software, the following are the steps you can follow to build and integrate your application.

1. Identify the use cases from above section that are close to the functionality you plan for your application.
2. Design and develop the component that accepts the Beckn requests and interacts with your software to do transactions. It has to be a endpoint(it is called as webhook_url in the description below) which receives all the Beckn requests (search, select etc). This endpoint can either exist outside of your marketplace/shop software or within it. That is a design decision that will have to be taken by you based on the design of your existing marketplace/shop software. This component is also responsible for sending back the responses to a the Beckn Adapter.
3. Install the BPP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL), a subscriber_id (typically the same as subscriber URL without the "https://" prefix) and the webhook_url that you configured in the step above. Also the address of the BPP Protocol Server Client will have to be configured in your component above. This address hosts all the response endpoints (on_search,on_select etc)
4. Install the layer 2 file for the domain (Link is in the last section of this document)
5. Check with your network tech support to enable your BPP Protocol Server in the registry.
6. Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,
   - you write the Provider Platform(dark blue) - Here the component you wrote above in point 2 as well as your marketplace/shop software is together shown as Provider Platform
   - install the BPP Protocol Server (light blue)
   - the remaining components are provided by the sandbox environment
   - Use the postman collection to test your Provider Platform
7. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

## Schema Details

| SN | Use Case                  | Input Details                        | Values                           | Data Types        |
|----|---------------------------|--------------------------------------|----------------------------------|-------------------|
| 1  | Crop Insurance            | Crop Name                            | guava                            | varchar           |
| 2  | Crop Insurance            | Farm Location                        | Niphad, Nashik                   | varchar           |
| 3  | Crop Insurance            | Land Area                            | 5 Acres                          | varchar           |
| 4  | Crop Insurance            | Insurance Duration                   | 2 Months                         | varchar           |
| 5  | Crop Insurance            | Coverage Type                        | Rainfall Cover                   | varchar           |
| 6  | Crop Insurance            | Sum Insured                          | 10000 INR/acre                  | varchar           |
| 7  | Crop Insurance            | Premium Payable                      | 250 INR/acre                     | varchar           |

## Links to artefacts

- [Postman collection for UAI](../../postman/Uai.postman_collection.json)

## Sandbox Details

### Registry/Gateway:
- **Gateway Sandbox:** gateway-uai.becknprotocol.io
- **Registry Sandbox:** registery-uai.becknprotocol.io

### BPP:
 - **BPP Client Sandbox:** bpp-ps-client-sandbox-uai.becknprotocol.io
 - **BPP Network Sandbox:** bpp-ps-network-sandbox-uai.becknprotocol.io
 - **BPP Sandbox:** bpp-unified-sandbox-uai.becknprotocol.io

### Domain name:
    financial-services:uki 