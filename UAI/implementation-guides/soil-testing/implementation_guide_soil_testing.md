# UAI Implementation Guide - Soil Testing

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

### Use case - Discovery and booking of soil testing services

1. Smita, a farmer from Nashik, wants to test the quality of her soil to plan agri inputs and choose the right crop for better yield.
2. She uses a Beckn-enabled app integrated with the UKI Network to search for soil testing centres and receive detailed soil analysis.
3. Smita searches for soil testing centres via the BAP app, supporting multilingual input.
4. The app displays a list of soil testing service providers (BPPs) and allows Smita to filter results by rating, location, cost, and preferred date/time.
5. She can choose collection type: Pick-up from farm (by provider) or Delivery to testing centre (by farmer).
6. Smita specifies required test types: NPK, pH, OC, micronutrients, soil texture, moisture, contaminants, CEC, or all parameters.
7. She selects Krishi Kendra Soil Services as the provider and receives available time slots, service quote, and prerequisites.
8. The provider may request crop-related details: current crop info, previous crop info, and previous yield.
9. Smita accepts the service terms, selects Cash on Delivery as payment mode, and confirms the order.
10. The provider confirms booking, shares Order ID, and details of agent and scheduled time.
11. For farm collection: Service provider visits the farm, collects the sample, and Smita pays in cash.
12. For lab collection: Smita delivers the sample to the centre and pays in cash.
13. Smita receives ongoing status updates and a final report (PDF) with intervention recommendations.
14. She rates the service provider on product quality, provider behavior, and support received.

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

### Soil Testing

#### search

**search for soil testing services**

- The search includes test types, fulfillment type, and location details.

```
{
  "context": {
    "domain": "agri-services:uki",
    "action": "search",
    "version": "1.1.0",
    "bap_id": "soiltest.bap.uki.in",
    "bap_uri": "https://soiltest.bap.uki.in",
    "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
    "message_id": "ffccbc41-782b-4cc9-9822-3ca2d3f87df0",
    "ttl": "PT20S",
    "timestamp": "2025-04-29T09:00:00Z",
    "location": {
      "city": {
        "code": "std:95253",
        "name": "Nashik"
      },
      "country": {
        "code": "IND",
        "name": "India"
      }
    }
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {
          "name": "Soil Testing Service"
        },
        "tags": [
          {
            "descriptor": {
              "name": "test-types"
            },
            "list": [
              {
                "value": "NPK"
              },
              {
                "value": "pH"
              },
              {
                "value": "OC"
              }
            ]
          }
        ]
      },
      "fulfillment": {
        "type": "Sample-PickUp",
        "stops": [
          {
            "type": "start",
            "location": {
              "gps": "19.9975,73.7898"
            }
          }
        ]
      }
    }
  }
}
```

#### on_search

**on_search with catalog of soil testing services**

- The catalog contains soil testing providers with their services, pricing, and fulfillment options.

```
{
  "context": {
    "domain": "agri.uki",
    "action": "on_search",
    "version": "1.1.0",
    "location": {
      "city": {
        "code": "std:95253",
        "name": "Nashik"
      },
      "country": {
        "code": "IND",
        "name": "India"
      }
    },
    "bap_id": "soiltest.bap.uki.in",
    "bap_uri": "https://soiltest.bap.uki.in",
    "bpp_id": "krishi-kendra.uki.in",
    "bpp_uri": "https://krishi-kendra.uki.in",
    "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
    "message_id": "ffccbc41-782b-4cc9-9822-3ca2d3f87df0",
    "timestamp": "2025-04-29T09:00:03Z",
    "ttl": "PT10M"
  },
  "message": {
    "catalog": {
      "providers": [
        {
          "id": "krishi-kendra-001",
          "descriptor": {
            "name": "Krishi Kendra Soil Services",
            "short_desc": "Comprehensive soil testing solutions"
          },
          "items": [
            {
              "id": "soil-test-comprehensive",
              "descriptor": {
                "name": "Comprehensive Soil Analysis",
                "short_desc": "Complete soil testing including NPK, pH, OC, and micronutrients"
              },
              "tags": [
                {
                  "descriptor": {
                    "code": "service_details"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "test_types"
                      },
                      "value": "NPK, pH, OC, Micronutrients"
                    },
                    {
                      "descriptor": {
                        "code": "turnaround_time"
                      },
                      "value": "7 days"
                    },
                    {
                      "descriptor": {
                        "code": "price"
                      },
                      "value": "1500 INR"
                    }
                  ]
                },
                {
                  "descriptor": {
                    "code": "fulfillment_options"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "collection_type"
                      },
                      "value": "Farm Pickup"
                    },
                    {
                      "descriptor": {
                        "code": "lab_dropoff"
                      },
                      "value": "Available"
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
```
{
    "context": {
        "domain": "agri.uki",
        "action": "select",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "41a7eb31-09a4-4fc7-9845-8cb65cbdf13c",
        "timestamp": "2025-04-29T09:01:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "krishikendra.bpp.uki.in"
            },
            "items": [
                {
                    "id": "soil-npk-test"
                }
            ],
            "fulfillments": [
                {
                    "id": "f1"
                }
            ]
        }
    }
}
```

#### on_select
```
{
    "context": {
        "domain": "agri.uki",
        "action": "on_select",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Nashik"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            }
        }
    }
}
```

#### init
```
{
    "context": {
        "domain": "agri.uki",
        "action": "init",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "41a7eb31-09a4-4fc7-9845-8cb65cbdf13c",
        "timestamp": "2025-04-29T09:01:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "krishikendra.bpp.uki.in"
            },
            "items": [
                {
                    "id": "soil-npk-test"
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210",
                                "email": "smita@example.com"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Smita",
                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003",
                "phone": "9876543210",
                "email": "smita@example.com"
            }
        }
    }
}
```

#### on_init
```
{
    "context": {
        "domain": "agri.uki",
        "action": "on_init",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            },
            "payment": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
                    }
                }
            ]
        }
    }
}
```

#### confirm
```
{
    "context": {
        "domain": "agri.uki",
        "action": "confirm",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "41a7eb31-09a4-4fc7-9845-8cb65cbdf13c",
        "timestamp": "2025-04-29T09:01:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "krishikendra.bpp.uki.in"
            },
            "items": [
                {
                    "id": "soil-npk-test"
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210",
                                "email": "smita@example.com"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Smita",
                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003",
                "phone": "9876543210",
                "email": "smita@example.com"
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "status": "NOT_PAID",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
                    }
                }
            ]
        }
    }
}
```

#### on_confirm
```
{
    "context": {
        "domain": "agri.uki",
        "action": "on_confirm",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "id": "order-873652",
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "code": "ORDER_PLACED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "status": "NOT_PAID",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
                    }
                }
            ]
        }
    }
}
```

#### status
```
{
    "context": {
        "domain": "agri.uki",
        "action": "status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order_id": "order-873652"
    }
}
```

#### on_status1
```
{
    "context": {
        "domain": "agri.uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "id": "order-873652",
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "code": "AGENT_ASSIGNED"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Heera"
                        },
                        "contact": {
                            "phone": "9876543210"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
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
        "domain": "agri.uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "id": "order-873652",
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "code": "SAMPLE_COLLECTED"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Heera"
                        },
                        "contact": {
                            "phone": "9876543210"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "status": "PAID",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
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
        "domain": "agri.uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "id": "order-873652",
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "code": "SAMPLE_TESTS_IN_PROGRESS"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Heera"
                        },
                        "contact": {
                            "phone": "9876543210"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "status": "PAID",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
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
        "domain": "agri.uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "e2e56f14-72fd-4cb5-b8c4-6e269ce87e68",
        "timestamp": "2025-04-29T09:01:03Z"
    },
    "message": {
        "order": {
            "id": "order-873652",
            "provider": {
                "id": "krishikendra.bpp.uki.in",
                "descriptor": {
                    "name": "Krishi Kendra Soil Services",
                    "short_desc": "Comprehensive Soil Testing at your farm or lab"
                }
            },
            "items": [
                {
                    "id": "soil-npk-test",
                    "descriptor": {
                        "name": "NPK Soil Test"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "400.00"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "name": "test-types"
                            },
                            "list": [
                                {
                                    "value": "NPK"
                                },
                                {
                                    "value": "pH"
                                },
                                {
                                    "value": "OC"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "Soil Collection Guidelines"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "document-link"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/docs/soil-collection-guidelines.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "type": "PickUp",
                            "location": {
                                "gps": "19.9975,73.7898",
                                "address": "Smita's Farm, Village Patole, Nashik District, Maharashtra 422003"
                            },
                            "contact": {
                                "phone": "9876543210"
                            },
                            "time": {
                                "timestamp": "2025-04-30T07:30:00Z"
                            }
                        },
                        {
                            "type": "DropOff",
                            "location": {
                                "gps": "19.9910,73.7769",
                                "address": "Krishi Kendra Soil Lab, Nashik"
                            }
                        }
                    ],
                    "state": {
                        "descriptor": {
                            "code": "REPORT_GENERATED"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Heera"
                        },
                        "contact": {
                            "phone": "9876543210"
                        }
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "reports"
                            },
                            "list":[
                                {
                                    "descriptor": {
                                        "code": "SOIL_TEST_REPORT"
                                    },
                                    "value": "https://krishikendra.bpp.uki.in/reports/soiltest-873652.pdf"
                                }
                            ]
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "400.00"
                },
                "breakup": [
                    {
                        "title": "NPK Soil Test",
                        "price": {
                            "currency": "INR",
                            "value": "400.00"
                        }
                    }
                ]
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "ON-FULFILLMENT",
                    "status": "PAID",
                    "params": {
                        "currency": "INR",
                        "value": "400.00"
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
        "domain": "agri.uki",
        "action": "rating",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "4215a2e5-2ec1-4a55-90c8-b046cbfbd3a5",
        "timestamp": "2025-05-01T10:00:00Z"
    },
    "message": {
        "ratings": [
            {
                "id": "order-873652",
                "rating_category": "ORDER",
                "value": "4.8"
            }
        ]
    }
}
```

#### on_rating
```
{
    "context": {
        "domain": "agri.uki",
        "action": "on_rating",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "4215a2e5-2ec1-4a55-90c8-b046cbfbd3a5",
        "timestamp": "2025-05-01T10:00:00Z"
    },
    "message": {
        "feedback_form": {
            "form": {
                "url": "https://link-to-the-form.html"
            },
            "required": "false"
        }
    }
}
```

#### support
```
{
    "context": {
        "domain": "agri.uki",
        "action": "support",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "a53200c2-267f-49b5-aef0-3a2e607aa6c0",
        "timestamp": "2025-05-01T10:20:00Z"
    },
    "message": {
        "support": {
            "ref_id": "order-873652",
            "url": "https://soiltest.bap.uki.in/uploads/delay-proof.png",
            "phone": "86XX564567"
        }
    }
}
```

#### on_support
```
{
    "context": {
        "domain": "agri.uki",
        "action": "on_support",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253",
                "name": "Nashik"
            },
            "country": {
                "code": "IND",
                "name": "India"
            }
        },
        "bap_id": "soiltest.bap.uki.in",
        "bap_uri": "https://soiltest.bap.uki.in",
        "bpp_id": "krishikendra.bpp.uki.in",
        "bpp_uri": "https://krishikendra.bpp.uki.in",
        "transaction_id": "7e92a1d4-3fa2-48e7-944d-01a6c02fa98b",
        "message_id": "a53200c2-267f-49b5-aef0-3a2e607aa6c0",
        "timestamp": "2025-05-01T10:20:00Z"
    },
    "message": {
        "support": {
            "ref_id": "order-873652",
            "url": "https://soiltest.bap.uki.in/uploads/delay-proof.png",
            "phone": "86XX564567",
            "callback_phone": "897389XX87"
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
| 1  | Soil Testing              | Test Types                           | NPK, pH, OC, Micronutrients      | varchar           |
| 2  | Soil Testing              | Collection Type                      | Farm Pickup                      | varchar           |
| 3  | Soil Testing              | Turnaround Time                      | 7 days                           | varchar           |
| 4  | Soil Testing              | Price                                | 1500 INR                         | varchar           |
| 5  | Soil Testing              | Location                             | Nashik                            | varchar           |
| 6  | Soil Testing              | Fulfillment Type                     | Sample-PickUp                    | varchar           |
| 7  | Soil Testing              | GPS Coordinates                      | 19.9975,73.7898                 | varchar           |

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
    agri-services:uki 