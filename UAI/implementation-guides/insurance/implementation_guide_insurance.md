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