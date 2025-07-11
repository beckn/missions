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

1. Rajesh is a guava farmer in Nashik district. Due to unpredictable weather conditions, he is unable to cope with crop losses. He wants to purchase an insurance product which can solve his issues.
2. Rajesh uses an app enabled by the UKI network to search for insurance products.
3. The platform sends the search query asynchronously to Service providers selling insurance.
4. He searches for insurance products and provides the following details:
   - Crops - Guava
   - Farm location (Not the farmer location), explicitly state the same: Niphad, Nashik
   - Duration of cover or [Sowing Date and Estimated Harvest Date]: 2 months
   - Type of Cover - Rainfall Cover/Temperature Cover
   - Number of units of land - 5 acres
5. He can filter by service provider, agent, premium, sum insured and rating.
6. He receives a list of service providers along with available insurance products.
7. He sees a list of insurance products from 3 providers:
   - Pradhan Mantri Fasal Bhima Yojana, GoI, Sum Insured: INR 10000 per acre, Premium Payable: INR 250, Policy Duration: 1 Year, Start Date: 1 May 2025, End Date: 30 April 2026 | Insurance Kart | Rating: 4.1 | Claims settlement ratio: 90%
   - High Temperature Cover, Sum Insured: INR 10000 per acre, Premium Payable: INR 500, Policy Duration: 30 days, Start Date: 1 May 2025, End Date: 31 May 2025 | Risk Free Insurers | Rating: 4.5 | Claims settlement ratio: 98%
   - Rainfall Cover, Sum Insured: INR 15000 per acre, Premium Payable: INR 500, Policy Duration: 30 days, Start Date: 1 May 2025, End Date: 31 May 2025 | Insurance Bazaar | Rating: 4.2 | Claims settlement ratio: 95%
8. Rajesh selects the preferred insurance cover, "Rainfall Cover" by Insurance Bazaar.
9. Rajesh then clicks on Check for eligibility. An order ID is generated for KYC check.
10. Rajesh can check the status of KYC using the order ID.
11. If Rajesh is eligible for the cover, he will receive a quote with the following details:
    - Base Cover
    - Max units he is eligible for
    - TnC
12. The UKI enabled app sends a purchase request to the BPP (insurance Bazaar). This request includes:
    - Selected product (Rainfall Cover)
    - Number of units of land - 5 acres
    - Number of units of insurance (to be limited by max units he is eligible for)
    - Farm Location
    - Contact information, billing address, shipping address for invoicing
    - Nominee Details (Name, Relationship, Date of Birth, Phone Number, Percent nomination) - more than one nominee details to be allowed
    - Bank account details
13. Rajesh receives the following:
    - Final quote with any additional charges (e.g., Rs 1000)
    - Terms of purchase (Historical date (Rainfall Index, Payout), Payoff Matrix, cancellation policy, returns/refunds)
14. He can review the final quote, terms of purchase.
15. Rajesh chooses to pay via UPI option and confirms the order.
16. The seller (BPP) shares an order ID along with the policy cover document (PDF).
17. Rajesh can check the status of his order and the BPP shares the status of the order.
18. Rajesh submits a rating for the insurance product that he has purchased. The seller (BPP) receives the rating and can request for further feedback.
19. This Rating can be segmented into:
    - Product/service quality
    - Fulfilment quality (time, ease, etc)
    - Support
20. Rajesh can reach out to support services in case of any issues with the service delivery.

## Flow diagrams

### General Beckn message flow and error handling

This section is relevant to all the messages flows illustrated below and discussed further in the document.

Beckn is a synchronous protocol at its core.

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

**search for crop insurance**

- The search request includes parameters like crop type, farm location, and filters for premium, sum insured, agent rating, and claim settlement ratio.

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

- The catalog that comes back has a list of providers.
- Each provider has a list of items.
- Each item is the catalog listing for an insurance product.
- The name, short_desc and long_desc fields contain the name and description of the insurance product.

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
                }
            ]
        }
    }
}
```

#### select

**select an insurance product**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "select",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "3c159ca2-0d68-43f8-a2cd-5e0e2c649ab3",
        "timestamp": "2025-06-02T09:05:00Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "items": [
                {
                    "id": "pmfby-2025-1yr",
                    "quantity": {
                        "count": 5
                    }
                }
            ]
        }
    }
}
```

#### on_select

**on_select response with quote and terms**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_select",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "3c159ca2-0d68-43f8-a2cd-5e0e2c649ab3",
        "timestamp": "2025-06-02T09:05:03Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "order-12345",
            "state": "Created",
            "provider": {
                "id": "p1",
                "descriptor": {
                    "name": "Pradhan Mantri Fasal Bhima Yojana"
                }
            },
            "items": [
                {
                    "id": "pmfby-2025-1yr",
                    "quantity": {
                        "count": 5
                    },
                    "price": {
                        "currency": "INR",
                        "value": "1250"
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "1250"
                },
                "breakup": [
                    {
                        "title": "Premium",
                        "price": {
                            "currency": "INR",
                            "value": "1250"
                        }
                    }
                ]
            },
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

#### init

**initiate order with customer details**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "init",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "4d159ca2-0d68-43f8-a2cd-5e0e2c649ab4",
        "timestamp": "2025-06-02T09:10:00Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "order-12345",
            "billing": {
                "name": "Rajesh Kumar",
                "address": {
                    "locality": "Niphad",
                    "city": "Nashik",
                    "state": "Maharashtra"
                },
                "email": "rajesh@example.com",
                "phone": "+91-9876543210"
            },
            "fulfillment": {
                "type": "digital",
                "customer": {
                    "person": {
                        "name": "Rajesh Kumar"
                    }
                }
            },
            "payment": {
                "uri": "upi://pay?pa=insurance@bazaar&pn=Insurance%20Bazaar&tn=Order%2012345&am=1250&cu=INR"
            }
        }
    }
}
```

#### on_init

**on_init response with order confirmation**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_init",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "4d159ca2-0d68-43f8-a2cd-5e0e2c649ab4",
        "timestamp": "2025-06-02T09:10:03Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "order-12345",
            "state": "Created",
            "provider": {
                "id": "p1",
                "descriptor": {
                    "name": "Pradhan Mantri Fasal Bhima Yojana"
                }
            },
            "items": [
                {
                    "id": "pmfby-2025-1yr",
                    "quantity": {
                        "count": 5
                    },
                    "price": {
                        "currency": "INR",
                        "value": "1250"
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "1250"
                },
                "breakup": [
                    {
                        "title": "Premium",
                        "price": {
                            "currency": "INR",
                            "value": "1250"
                        }
                    }
                ]
            },
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

#### confirm

**confirm the order**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "confirm",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "5e159ca2-0d68-43f8-a2cd-5e0e2c649ab5",
        "timestamp": "2025-06-02T09:15:00Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "order-12345"
        }
    }
}
```

#### on_confirm

**on_confirm with policy details**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_confirm",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "5e159ca2-0d68-43f8-a2cd-5e0e2c649ab5",
        "timestamp": "2025-06-02T09:15:03Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "order-12345",
            "state": "Confirmed",
            "provider": {
                "id": "p1",
                "descriptor": {
                    "name": "Pradhan Mantri Fasal Bhima Yojana"
                }
            },
            "items": [
                {
                    "id": "pmfby-2025-1yr",
                    "quantity": {
                        "count": 5
                    },
                    "price": {
                        "currency": "INR",
                        "value": "1250"
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "1250"
                },
                "breakup": [
                    {
                        "title": "Premium",
                        "price": {
                            "currency": "INR",
                            "value": "1250"
                        }
                    }
                ]
            },
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "state": {
                        "descriptor": {
                            "code": "Completed"
                        }
                    },
                    "tracking": false
                }
            ],
            "payment": {
                "status": "PAID"
            },
            "documents": [
                {
                    "url": "https://insurance-bazaar.uki.network/policy/order-12345.pdf",
                    "label": "Policy Document"
                }
            ]
        }
    }
}
```

#### status

**check order status**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "status",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "6f159ca2-0d68-43f8-a2cd-5e0e2c649ab6",
        "timestamp": "2025-06-02T09:20:00Z",
        "ttl": "PT10M"
    },
    "message": {
        "order_id": "order-12345"
    }
}
```

#### on_status

**on_status response**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "6f159ca2-0d68-43f8-a2cd-5e0e2c649ab6",
        "timestamp": "2025-06-02T09:20:03Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "order-12345",
            "state": "Confirmed",
            "provider": {
                "id": "p1",
                "descriptor": {
                    "name": "Pradhan Mantri Fasal Bhima Yojana"
                }
            },
            "items": [
                {
                    "id": "pmfby-2025-1yr",
                    "quantity": {
                        "count": 5
                    },
                    "price": {
                        "currency": "INR",
                        "value": "1250"
                    }
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "state": {
                        "descriptor": {
                            "code": "Completed"
                        }
                    },
                    "tracking": false
                }
            ],
            "payment": {
                "status": "PAID"
            }
        }
    }
}
```

#### support

**sending a support request**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "support",
        "version": "1.1.0",
        "bap_id": "bap-client.uki.network",
        "bap_uri": "https://bap-client.uki.network/protocol",
        "bpp_id": "insurance-bazaar.uki.network",
        "bpp_uri": "https://insurance-bazaar.uki.network/credit",
        "transaction_id": "f75f8973-3261-4931-902f-6a44a7a1ea75",
        "message_id": "7g159ca2-0d68-43f8-a2cd-5e0e2c649ab7",
        "timestamp": "2025-06-02T09:25:00Z",
        "ttl": "PT10M"
    },
    "message": {
        "ref_id": "order-12345"
    }
}
```

#### on_support

**Getting an on_support callback**

```
{
    "context": {
        "domain": "financial-services:uki",
        "action": "on_support",
        "version": "1.1.0",
        "bap_id": "bap-