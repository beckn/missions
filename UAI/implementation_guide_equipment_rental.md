# UAI Implementation Guide - Equipment Rental Services

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 06-11-2024 | 0.1     | Initial Version                                     |
| 14-11-2024 | 0.2     | Internal Review Comments are incorprated                                     |
| 18-11-2024 | 1.0     | Final Version                                     |

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

![UAI Equipment Rental Outcome Visualization](images/uai_equipment_rental_outcome_visualization.svg)

### Use case - Discovery of Rental Services for Farm Equipment

1.	Vijay, a grape farmer in Nashik, faces a powdery mildew infestation on his crops and needs a sprayer to apply fungicides effectively.
2.	Using the UAI platform, Vijay searches for rental services that offer sprayers and other specialized equipment operated by experienced providers nearby.
3.	He filters the search results by rating, experience, location, and rental cost, finding options like:
a.	Sandip Patil - 4.2 rating, 3 years of experience, sprayer and tractor, 2 km away
b.	Bhagwan Wane - 3.8 rating, 2 years of experience, sprayer and harvester, 3 km away
c.	Ajinkya Sonavane - 3.5 rating, 4 years of experience, sprayer and drones, 2 km away
4.	Vijay chooses Sandip Patil as his preferred service provider and reviews available equipment details and rental costs.
5.	After selecting the sprayer, Vijay receives a rental quote of INR 300 per acre. He inputs his booking date, time, and the required area for spraying.
6.	He reviews the final quote, including any applicable taxes and terms of service, such as cancellation and refund policies.
7.	Vijay accepts the terms and confirms his order by choosing cash on delivery as the payment method.
8.	The service provider, Sandip Patil, confirms the booking and shares the order ID along with the agent’s details and scheduled time for the service.
9.	On the scheduled day, Vijay receives status updates as Sandip Patil arrives and completes the spraying.
10.	After completing the service, Vijay pays Sandip by cash, based on the final amount agreed.
11.	Satisfied with the service, Vijay submits a rating for the equipment quality, service punctuality, and overall support, helping others make informed choices.

## Flow diagrams

### General Beckn message flow and error handling

This section is relevant to all the messages flows illustrated below and discussed further in the document.

Beckn is a aynchronous protocol at its core.

- When a network participant(NP1) sends a message to another participant(NP2), the other participant(NP2) immediately returns back an ACK/NACK(Acknowledgement or Negative Acknowledgement in case of error - usually with wrongly formed messages).
- An ACK is an indicator that the receiving participant(NP2) will process this message and dispatch an on_xxxxxx message to original NP (NP1)
- Subsequently after processing the message NP2 sends back the real response in the corresponding on_xxxxxx message, to which again the first participant(NP1).
- This message can contain a message field (for success) or error field (for failure)
- NP1 when it receives the on_xxxxxx message, sends back an ACK/NACK (Here in both the cases NP1 will not send any subsequent message).
- In the Use case diagrams, this ACK/NACK is not illustrated explicitly to keep the diagrams crisp.
- However when writing software we should be prepared to receive these NACK messages as well as error field in the on_xxxxxx messages
- While this discussion is from a Beckn perspective, Adapters can provide synchronous modes. For example, the Protocol Server which is the reference implementation of the Beckn Adapter provides a synchronous mode by default. So if your software calls the support endpoint on the BAP Protocol Server, the Protocol Server waits till it gets the on_support and returns back that as the response.

![ACK NACK for messages](/docs/assets/images/ack_nack.png)

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

### Use case - Discovery of Rental Services for Farm Equipment

**Search for rental service**

![Search for farm equipment rental service](Images/Rental/uai-rental-discovery.jpg)

**Place an order for the Farm Equipment**

![Select the farm equipment for rental, initialise and confirm the order](Images/Rental/uai-rental-order.jpg)

**Fullfilment of an active order**

![check the order status, update the order details or cancel the order](Images/Rental/uai-rental-fulfilment.jpg)

**Seek support and provide rating**

![Support call to get contact information of Provider platofrm](Images/Rental/uai-rental-post%20fulfilment.jpg)

## API Calls and Schema

### Discovery of Rental Services for Farm Equipment

#### search

**search by equpement/service  type**
```
{
    "context": {
      "domain": "retail:uai",
      "location": {
        "country": {
          "name": "IND"
        }
      },
      "action": "search",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "timestamp": "2023-11-06T09:41:09.673Z",
      "ttl": "PT10M"
    },
    "message": {
      "intent": {
        "category": {
            "descriptor": {
                "code": "rent"
            }
        },
        "item": {
          "descriptor": {
            "name": "agriculture sprayer"
          }
        }
      }
    }
  }
```

**search by equpement/service type, rating and crop stage**
```
{
    "context": {
      "domain": "retail:uai",
      "location": {
        "country": {
          "name": "IND"
        }
      },
      "action": "search",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "timestamp": "2023-11-06T09:41:09.673Z",
      "ttl": "PT10M"
    },
    "message": {
      "intent": {
        "category": {
            "descriptor": {
                "code": "rent"
            }
        },
        "item": {
          "descriptor": {
            "name": "agriculture sprayer"
          },
          "rating": ">4.0",
          "tags": [
                {
                    "descriptor": {
                        "name": "crop stage"
                    },
                    "list": [
                        {
                            "value": "Land Preparation"
                        }
                    ]
                }
            ]
        }
      }
    }
  }
```

**search by equpement/service type, price limit and fulfillment type**
```
{
    "context": {
      "domain": "retail:uai",
      "location": {
        "country": {
          "name": "IND"
        }
      },
      "action": "search",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "timestamp": "2023-11-06T09:41:09.673Z",
      "ttl": "PT10M"
    },
    "message": {
      "intent": {
        "category": {
            "descriptor": {
                "code": "rent"
            }
        },
        "item": {
          "descriptor": {
            "name": "agriculture sprayer"
          },
          "price": {
            "currency": "INR/Acre",
            "maximum_value": "1000.0"
          }
        },
        "fulfillment": {
            "type": "Delivery",
            "stops": [
                {
                    "type": "SITE",
                    "location": {
                        "gps": "23.11355, 24.32344"
                    },
                    "time: {
                        "range": {
                            "start": "2024-12-01T08:30:00Z",
                            "end": "2024-12-10T18:00:00Z"
                        }
                    }
                }
            ]
        }
      }
    }
  }
```

**search by equpement/service type, user location, provider's distance from user location and date of delivery**
```
{
    "context": {
      "domain": "retail:uai",
      "location": {
        "country": {
          "name": "IND"
        }
      },
      "action": "search",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "timestamp": "2023-11-06T09:41:09.673Z",
      "ttl": "PT10M"
    },
    "message": {
      "intent": {
        "category": {
            "descriptor": {
                "code": "rent"
            }
        },
        "item": {
          "descriptor": {
            "name": "agriculture sprayer"
          }
        },
        "fulfillment": {
            "stops": [
                {
                    "type": "SITE",
                    "location": {
                        "gps": "23.11355, 24.32344"
                    },
                    "time: {
                        "range": {
                            "start": "2024-12-01T08:30:00Z",
                            "end": "2024-12-10T18:00:00Z"
                        }
                    }
                }
            ]
        },
        "provider": {
            "locations": [
                {
                    "circle": {
                        "gps": "23.11355, 24.32344",
                        "radius": {
                            "unit": "Killometer",
                            "value": "20"
                        }
                    }
                }
            ]
        }
      }
    }
  }
```

#### on_search

**on_search with catalog of results**
- The catalog that comes back has a list of providers.
- Each provider has a list of items.
- Each item is the catalog listing for a resource.
- The name, short_desc and long_desc fields contain the name and description of the resource.
- Further, if the resource is a video or a pdf, its mimetype and url are specified in the media field.

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_search",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "catalog": {
            "descriptor": {
                "name": "Agricultural Aggregator",
                "short_desc": "",
                "long_desc": "",
                "images": [
                    {
                        "url": "https://image_url"
                    }
                ]
            },
            "providers": [
                {
                    "id": "1",
                    "descriptor": {
                        "name": "Sandip",
                        "short_desc": "A local sprayer provider",
                        "long_desc": "The provider has ....",
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "c1",
                            "descriptor": {
                                "code": "rent",
                                "name": "Rent"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "f1",
                            "type": "Delivery"
                        },
                        {
                            "id": "f2",
                            "type": "Self-Pickup"
                        }
                    ],
                    "locations": [
                        {
                            "id": "l1",
                            "gps": "23.11355, 24.32344"
                        },
                        {
                            "id": "l2",
                            "gps": "24.11355, 23.32344"
                        },
                        {
                            "id": "l3",
                            "gps": "24.24434, 23.24234"
                        }
                    ],
                    "items": [
                        {
                            "id": "1",
                            "descriptor": {
                                "images": [
                                    {
                                        "url": "https://image_url"
                                    }
                                ],
                                "name": "sprayer and tractor",
                                "short_desc": "Our Tractor-Mounted Sprayer Service provides efficient, uniform application of pesticides, herbicides, and fertilizers across large fields.",
                                "long_desc": "Designed to enhance crop health and maximize yield, this service ensures precise chemical distribution, reducing waste and promoting optimal growth. Perfect for large-scale farms looking to protect crops and boost productivity with minimal manual effort."
                            },
                            "price": {
                                "currency": "INR/Acre",
                                "value": "300.0"
                            },
                            "rating": "4.2",
                            "location_ids": [
                                "l1"
                            ],
                            "category_ids": [
                                "c1"
                            ],
                            "fulfillment_ids": [
                                "f1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "name": "stage"
                                    },
                                    "list": [
                                        {
                                            "value": "Land Preparation"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "name": "experience"
                                    },
                                    "list": [
                                        {
                                            "value": "3 Years"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "2",
                    "descriptor": {
                        "name": "Bhagwan Wane",
                        "short_desc": "A modern sprayer provider",
                        "long_desc": "The provider has ....",
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "c1",
                            "descriptor": {
                                "code": "rent",
                                "name": "Rent"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "f1",
                            "type": "Delivery"
                        },
                        {
                            "id": "f2",
                            "type": "Self-Pickup"
                        }
                    ],
                    "locations": [
                        {
                            "id": "l1",
                            "gps": "23.11355, 24.32344"
                        },
                        {
                            "id": "l2",
                            "gps": "24.11355, 23.32344"
                        },
                        {
                            "id": "l3",
                            "gps": "24.24434, 23.24234"
                        }
                    ],
                    "items": [
                        {
                            "id": "2",
                            "descriptor": {
                                "images": [
                                    {
                                        "url": "https://image_url"
                                    }
                                ],
                                "name": "sprayer and harvester,",
                                "short_desc": "Our Tractor-Mounted Sprayer and Harvester Service combines crop protection and harvesting efficiency in one seamless solution.",
                                "long_desc": "With precision spraying for pest, weed, and nutrient management, followed by high-speed harvesting, this service streamlines your crop cycle. Ideal for large-scale farms, it minimizes crop loss, saves time, and ensures maximum yield, enabling healthier, ready-to-market crops with minimal manual labor."
                            },
                            "price": {
                                "currency": "INR/Acre",
                                "value": "400.0"
                            },
                            "rating": "3.8",
                            "location_ids": [
                                "l2"
                            ],
                            "category_ids": [
                                "c1"
                            ],
                            "fulfillment_ids": [
                                "f1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "name": "stage"
                                    },
                                    "list": [
                                        {
                                            "value": "Land Preparation"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "name": "experience"
                                    },
                                    "list": [
                                        {
                                            "value": "2 Years"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": "3",
                    "descriptor": {
                        "name": "Ajinkya Sonavane",
                        "short_desc": "Agricultural parts provider ....",
                        "long_desc": "The provider has ....",
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "c1",
                            "descriptor": {
                                "code": "rent",
                                "name": "Rent"
                            }
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "f1",
                            "type": "Delivery"
                        },
                        {
                            "id": "f2",
                            "type": "Self-Pickup"
                        }
                    ],
                    "locations": [
                        {
                            "id": "l1",
                            "gps": "23.11355, 24.32344"
                        },
                        {
                            "id": "l2",
                            "gps": "24.11355, 23.32344"
                        },
                        {
                            "id": "l3",
                            "gps": "24.24434, 23.24234"
                        }
                    ],
                    "items": [
                        {
                            "id": "3",
                            "descriptor": {
                                "images": [
                                    {
                                        "url": "https://image_url"
                                    }
                                ],
                                "name": "sprayer and drones",
                                "short_desc": "Our Drone-Integrated Sprayer Service offers advanced, precise application of pesticides, herbicides, and fertilizers across fields with minimal ground disruption.",
                                "long_desc": "Equipped for efficient aerial spraying, drones reach areas difficult for traditional machinery, ensuring thorough crop protection and optimal nutrient delivery. Ideal for modern farms seeking to reduce labor, save time, and improve crop health through smart, targeted spraying."
                            },
                            "price": {
                                "currency": "INR/Acre",
                                "value": "600.0"
                            },
                            "rating": "3.5",
                            "location_ids": [
                                "l2"
                            ],
                            "category_ids": [
                                "c1"
                            ],
                            "fulfillment_ids": [
                                "f1"
                            ],
                            "tags": [
                                {
                                    "descriptor": {
                                        "name": "stage"
                                    },
                                    "list": [
                                        {
                                            "value": "Land Preparation"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "name": "experience"
                                    },
                                    "list": [
                                        {
                                            "value": "4 Years"
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

**selecting an item from the catalog**

- provider Contains details of the seller from whom the items are purchased.
- Items  A list of individual items included in the purchase.
- fulfillments[idx] object provides specific fulfillment details for each part of the order.

```
{
    "context": {
      "domain": "retail:uai",
      "location": {
        "country": {
            "name": "IND"
          }
      },
      "action": "select",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "order": {
        "provider": {
          "id": "1"
        },
        "items": [
          {
            "id": "1",
            "quantity": {
              "selected": {
                "measure" : {
                  "unit" : "Acres",
                  "value": "10"
                }
              }
            }
          }
        ],
        "fulfillments": [
          {
            "id": "f1",
            "stops": [
                {
                    "location": {
                        "gps": "23.23234, 24.2424"
                    },
                    "time": {
                        "timestamp": "2023-11-06T09:41:09.708Z"
                    }
                }
            ]
          }
        ]
      }
    }
  }
```

#### on_select

**Getting a quotation from the seller for the selected items from the catalog**

- Contains the pricing, itemized details, and cost breakup for a potential purchase.

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_select",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "provider": {
                "id": "1",
                "descriptor": {
                    "name": "Provider 1",
                    "short_desc": "",
                    "long_desc": "",
                    "images": [
                        {
                            "url": "https://image_url"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "1",
                    "descriptor": {
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ],
                        "name": "sprayer and tractor",
                        "short_desc": "Our Tractor-Mounted Sprayer Service provides efficient, uniform application of pesticides, herbicides, and fertilizers across large fields.",
                        "long_desc": "Designed to enhance crop health and maximize yield, this service ensures precise chemical distribution, reducing waste and promoting optimal growth. Perfect for large-scale farms looking to protect crops and boost productivity with minimal manual effort."
                    },
                    "price": {
                        "currency": "INR/Acre",
                        "value": "300.0"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "unit": "Acres",
                                "value": "10"
                            }
                        }
                    },
                    "rating": "4.2",
                    "creator": {
                        "descriptor": {
                            "name": "Sandip"
                        }
                    },
                    "location_ids": [
                        "l1"
                    ],
                    "category_ids": [
                        "c1"
                    ],
                    "fulfillment_ids": [
                        "f1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "name": "stage"
                            },
                            "list": [
                                {
                                    "value": "Land Preparation"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "experience"
                            },
                            "list": [
                                {
                                    "value": "3 Years"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "Delivery",
                    "stops": [
                        {
                            "location": {
                                "gps": "23.23234, 24.2424"
                            },
                            "time": {
                                "timestamp": "2023-11-06T09:41:09.708Z"
                            }
                        }
                    ]
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3150.0"
                },
                "breakup": [
                    {
                        "title": "base-price",
                        "price": {
                            "currency": "INR",
                            "value": "3000.0"
                        }
                    },
                    {
                        "title": "taxes",
                        "price": {
                            "currency": "INR",
                            "value": "150.0"
                        }
                    }
                ]
            }
        }
    }
}
```

#### init

**initialise an order with billing details**

```
{
    "context": {
      "domain": "retail:uai",
      "location": {
        "country": {
            "name": "IND"
          }
      },
      "action": "init",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "order": {
        "provider": {
          "id": "1"
        },
        "items": [
          {
            "id": "1",
            "quantity": {
              "selected": {
                "measure" : {
                  "unit" : "Acres",
                  "value": "10"
                }
              }
            }
          }
        ],
        "fulfillments": [
          {
            "id": "f1",
            "stops": [
                {
                    "location": {
                        "gps": "23.23234, 24.2424"
                    },
                    "time": {
                        "timestamp": "2023-11-06T09:41:09.708Z"
                    }
                }
            ],
            "customer": {
                "person": {
                    "name": "Madhavan"
                },
                "contact": {
                    "phone": "8130xxxxxx"
                }
            }
          }
        ],
        "billing": {
            "name": "Madhvan",
            "phone" : "8130xxxxxx",
            "email" : "Mad@example.com",
            "address": "123, main street, Nashik"
        }
      }
    }
  }
```

#### on_init

**Receive the payment details from the seller.**

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_init",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "provider": {
                "id": "1",
                "descriptor": {
                    "name": "Provider 1",
                    "short_desc": "",
                    "long_desc": "",
                    "images": [
                        {
                            "url": "https://image_url"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "1",
                    "descriptor": {
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ],
                        "name": "sprayer and tractor",
                        "short_desc": "Our Tractor-Mounted Sprayer Service provides efficient, uniform application of pesticides, herbicides, and fertilizers across large fields.",
                        "long_desc": "Designed to enhance crop health and maximize yield, this service ensures precise chemical distribution, reducing waste and promoting optimal growth. Perfect for large-scale farms looking to protect crops and boost productivity with minimal manual effort."
                    },
                    "price": {
                        "currency": "INR/Acre",
                        "value": "300.0"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "unit": "Acres",
                                "value": "10"
                            }
                        }
                    },
                    "rating": "4.2",
                    "creator": {
                        "descriptor": {
                            "name": "Sandip"
                        }
                    },
                    "location_ids": [
                        "l1"
                    ],
                    "category_ids": [
                        "c1"
                    ],
                    "fulfillment_ids": [
                        "f1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "name": "stage"
                            },
                            "list": [
                                {
                                    "value": "Land Preparation"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "experience"
                            },
                            "list": [
                                {
                                    "value": "3 Years"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "Delivery",
                    "stops": [
                        {
                            "location": {
                                "gps": "23.23234, 24.2424"
                            },
                            "time": {
                                "timestamp": "2023-11-06T09:41:09.708Z"
                            }
                        }
                    ],
                    "customer": {
                        "person": {
                            "name": "Madhavan"
                        },
                        "contact": {
                            "phone": "8130xxxxxx"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3150.0"
                },
                "breakup": [
                    {
                        "title": "base-price",
                        "price": {
                            "currency": "INR",
                            "value": "3000.0"
                        }
                    },
                    {
                        "title": "taxes",
                        "price": {
                            "currency": "INR",
                            "value": "150.0"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Madhvan",
                "phone": "8130xxxxxx",
                "email": "Mad@example.com",
                "address": "123, main street, Nashik"
            },
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "if cancelling before the scheduled day of fulfillment"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                }
            ],
            "payments": [
                {
                    "status": "NOT-PAID",
                    "type": "POST-FULFILLMENT",
                    "collected_by": "AGENT",
                    "params": {
                        "amount": "3150",
                        "currency": "INR"
                    }
                }
            ]
        }
    }
}
```

#### confirm

**Confirm payment terms for post-fulfillment collection**

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "confirm",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "provider": {
                "id": "1"
            },
            "items": [
                {
                    "id": "1",
                    "quantity": {
                        "selected": {
                            "measure": {
                                "unit": "Acres",
                                "value": "10"
                            }
                        }
                    }
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "stops": [
                        {
                            "location": {
                                "gps": "23.23234, 24.2424"
                            },
                            "time": {
                                "timestamp": "2023-11-06T09:41:09.708Z"
                            }
                        }
                    ],
                    "customer": {
                        "person": {
                            "name": "Madhavan"
                        },
                        "contact": {
                            "phone": "8130xxxxxx"
                        }
                    }
                }
            ],
            "billing": {
                "name": "Madhvan",
                "phone": "8130xxxxxx",
                "email": "Mad@example.com"
            },
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "if cancelling before the scheduled day of fulfillment"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                }
            ],
            "payments": [
                {
                    "status": "NOT-PAID",
                    "type": "POST-FULFILLMENT",
                    "collected_by": "AGENT",
                    "params": {
                        "amount": "3150",
                        "currency": "INR"
                    }
                }
            ]
        }
    }
}
```

#### on_confirm

**Confirm order details, including agent assignment and payment terms**

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_init",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "b989c9a9-f603-4d44-b38d-26fd72286b40",
            "provider": {
                "id": "1",
                "descriptor": {
                    "name": "Provider 1",
                    "short_desc": "",
                    "long_desc": "",
                    "images": [
                        {
                            "url": "https://image_url"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "1",
                    "descriptor": {
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ],
                        "name": "sprayer and tractor",
                        "short_desc": "Our Tractor-Mounted Sprayer Service provides efficient, uniform application of pesticides, herbicides, and fertilizers across large fields.",
                        "long_desc": "Designed to enhance crop health and maximize yield, this service ensures precise chemical distribution, reducing waste and promoting optimal growth. Perfect for large-scale farms looking to protect crops and boost productivity with minimal manual effort."
                    },
                    "price": {
                        "currency": "INR/Acre",
                        "value": "300.0"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "unit": "Acres",
                                "value": "10"
                            }
                        }
                    },
                    "rating": "4.2",
                    "creator": {
                        "descriptor": {
                            "name": "Sandip"
                        }
                    },
                    "location_ids": [
                        "l1"
                    ],
                    "category_ids": [
                        "c1"
                    ],
                    "fulfillment_ids": [
                        "f1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "name": "stage"
                            },
                            "list": [
                                {
                                    "value": "Land Preparation"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "experience"
                            },
                            "list": [
                                {
                                    "value": "3 Years"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "Delivery",
                    "stops": [
                        {
                            "location": {
                                "gps": "23.23234, 24.2424"
                            },
                            "time": {
                                "timestamp": "2023-11-06T09:41:09.708Z"
                            }
                        }
                    ],
                    "customer": {
                        "person": {
                            "name": "Madhavan"
                        },
                        "contact": {
                            "phone": "8130xxxxxx"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Tony"
                        },
                        "contact": {
                            "phone": "9230xxxxxx"
                        },
                        "rating": "4.7"
                    },
                    "state": {
                        "descriptor": {
                            "code": "ORDER CONFIRMED",
                            "name": "Your Order is confirmed"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3150.0"
                },
                "breakup": [
                    {
                        "title": "base-price",
                        "price": {
                            "currency": "INR",
                            "value": "3000.0"
                        }
                    },
                    {
                        "title": "taxes",
                        "price": {
                            "currency": "INR",
                            "value": "150.0"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Madhvan",
                "phone": "8130xxxxxx",
                "email": "Mad@example.com"
            },
             "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "if cancelling before the scheduled day of fulfillment"
                    },
                    "cancellation_fee": {
                        "percentage": "10%"
                    }
                }
            ],
            "payments": [
                {
                    "status": "NOT-PAID",
                    "type": "POST-FULFILLMENT",
                    "collected_by": "AGENT",
                    "params": {
                        "amount": "3150",
                        "currency": "INR"
                    }
                }
            ]
        }
    }
}
```

#### status

**Request current status of the order using order ID**

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "status",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b40"
    }
}
```

#### on_status

**Retrieve the current status of the order, including progress and fulfillment details**
```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_status",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "order": {
            "id": "b989c9a9-f603-4d44-b38d-26fd72286b40",
            "provider": {
                "id": "1",
                "descriptor": {
                    "name": "Provider 1",
                    "short_desc": "",
                    "long_desc": "",
                    "images": [
                        {
                            "url": "https://image_url"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "1",
                    "descriptor": {
                        "images": [
                            {
                                "url": "https://image_url"
                            }
                        ],
                        "name": "sprayer and tractor",
                        "short_desc": "Our Tractor-Mounted Sprayer Service provides efficient, uniform application of pesticides, herbicides, and fertilizers across large fields.",
                        "long_desc": "Designed to enhance crop health and maximize yield, this service ensures precise chemical distribution, reducing waste and promoting optimal growth. Perfect for large-scale farms looking to protect crops and boost productivity with minimal manual effort."
                    },
                    "price": {
                        "currency": "INR/Acre",
                        "value": "300.0"
                    },
                    "quantity": {
                        "selected": {
                            "measure": {
                                "unit": "Acres",
                                "value": "10"
                            }
                        }
                    },
                    "rating": "4.2",
                    "creator": {
                        "descriptor": {
                            "name": "Sandip"
                        }
                    },
                    "location_ids": [
                        "l1"
                    ],
                    "category_ids": [
                        "c1"
                    ],
                    "fulfillment_ids": [
                        "f1"
                    ],
                    "tags": [
                        {
                            "descriptor": {
                                "name": "stage"
                            },
                            "list": [
                                {
                                    "value": "Land Preparation"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "name": "experience"
                            },
                            "list": [
                                {
                                    "value": "3 Years"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "Delivery",
                    "stops": [
                        {
                            "location": {
                                "gps": "23.23234, 24.2424"
                            },
                            "time": {
                                "timestamp": "2023-11-06T09:41:09.708Z"
                            }
                        }
                    ],
                    "customer": {
                        "person": {
                            "name": "Madhavan"
                        },
                        "contact": {
                            "phone": "8130xxxxxx"
                        }
                    },
                    "agent": {
                        "person": {
                            "name": "Tony"
                        },
                        "contact": {
                            "phone": "9230xxxxxx"
                        },
                        "rating": "4.7"
                    },
                    "state": {
                        "descriptor": {
                            "code": "IN-PROGRESS",
                            "name": "Service is underway"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "3150.0"
                },
                "breakup": [
                    {
                        "title": "base-price",
                        "price": {
                            "currency": "INR",
                            "value": "3000.0"
                        }
                    },
                    {
                        "title": "taxes",
                        "price": {
                            "currency": "INR",
                            "value": "150.0"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Madhvan",
                "phone": "8130xxxxxx",
                "email": "Mad@example.com"
            },
            "payments": [
                {
                    "status": "NOT-PAID",
                    "type": "POST-FULFILLMENT",
                    "collected_by": "AGENT",
                    "params": {
                        "amount": "3150",
                        "currency": "INR"
                    }
                }
            ]
        }
    }
}
```

#### support

**sending a support request**

- In most cases the support request is a call to get contact details of the provider platform (Phone, Web url etc).
- However in rare cases a reference id might be sent to provide a context to the request.

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "support",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "ref_id": "9e188d26-0b1b-4920-a586-6006b0bcf768"
    }
}
```

#### on_support

**Getting an on_support callback**

- In most cases the returned details are common contact information of the provider platform.
- However it is possible to connect this to other customer management solutions.

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_support",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "support": {
            "ref_id": "9e188d26-0b1b-4920-a586-6006b0bcf768",
            "phone": "18001801551",
            "url": "https://agritech.tnau.ac.in/agriculture/agri_faqs.html"
        }
    }
}
```
#### get_rating_categories

- BAP can fetch a list of categoried for which a BPP accepts rating
- ```message``` field is not needed in the get_rating_categories request.

```
{
  "context": {
    "domain": "retail:uai",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  }
}
```

#### rating_categories

- The BPP responds with a list of categories in which ratings can be provided.

```
{
  "context": {
    "domain": "retail:uai",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "rating_categories": [
      Item,
      Order,
      Fulfillment,
      Provider
    ]
  }
}
```

#### rating

**Rating a provider**

- When the user wants to rate the provider, it happens in two steps.
- The user sends a numerical (0-5) rating in the request.
- The response will have a link to the form where the user can provide text feedback.

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "rating",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
    },
    "message": {
        "ratings": [
            {
                "id": "19a02a67-d2f0-4ea7-b7e1-b2cf4fa57f56",
                "rating_category": "Provider",
                "value": "5"
            }
        ]
    }
}
```

#### on_rating

**Getting on_rating response**

- The on_rating response has a link to the form where the user can specify a text feedback that goes alongside the numerical rating provided earlier.

```
{
    "context": {
        "domain": "retail:uai",
        "location": {
            "country": {
                "name": "IND"
            }
        },
        "action": "on_rating",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z",
        "ttl": "PT10M"
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

## Taxonomy and layer 2 configuration

- Any specific tags, enumerations, and rules we add for the use cases or required by the network, will go here.

## Integrating with your software

This section gives general walkthrough of how you would integrate your software with the Beckn network (say the sandbox environment). Refer to the starter kit for details on how to register with the sandbox and get credentials.

Beckn-ONIX is an initiative to promote easy install and maintenance of a Beckn Network. Apart from the Registry and Gateway components that are required for a network facilitator, Beckn-ONIX provides a Beckn Adapter. A reference implementation of the Beckn-ONIX specificatino is available at [Beckn-ONIX repository](https://github.com/beckn/beckn-onix). The reference implementation of the Beckn Adapter is called the Protocol Server. Based on whether we are writing the seeker platform or the provider platform, we will be installing the BAP Protocol Server or the BPP Protocol Server respectively.

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
   - the remaining components are provided by the sandbox enviornment
8. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

![Seeker platform testing sandbox ](/docs/assets/images/seeker_deployment.png)

### Integrating the provider platform

If you are writing the provider platform software, the following are the steps you can follow to build and integrate your application.

1. Identify the use cases from above section that are close to the functionality you plan for your application.
2. Design and develop the component that accepts the Beckn requests and interacts with your software to do transactions. It has to be a endpoint(it is called as webhook_url in the description below) which receives all the Beckn requests (search, select etc). This endpoint can either exist outside of your marketplace/shop software or within it. That is a design decision that will have to be taken by you based on the design of your existing marketplace/shop software. This component is also responsible for sending back the responses to a the Beckn Adaptor.
3. Install the BPP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL), a subscriber_id (typically the same as subscriber URL without the "https://" prefix) and the webhook_url that you configured in the step above. Also the address of the BPP Protocol Server Client will have to be configured in your component above. This address hosts all the response endpoints (on_search,on_select etc)
4. Install the layer 2 file for the domain (Link is in the last section of this document)
5. Check with your network tech support to enable your BPP Protocol Server in the registry.
6. Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,
   - you write the Provider Platform(dark blue) - Here the component you wrote above in point 2 as well as your marketplace/shop software is together shown as Provider Platform
   - install the BPP Protocol Server (light blue)
   - the remaining components are provided by the sandbox enviornment
   - Use the postman collection to test your Provider Platform
7. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

![Provider platform testing sandbox](/docs/assets/images/provider_deployment.png)

## Links to artefacts


- [Postman collection for UAI](./postman/Uai.postman_collection.json)

## Sandbox Details

### Registry/Gateway:
- **Gateway Sandbox:** gateway-uai.becknprotocol.io
- **Registry Sandbox:** registery-uai.becknprotocol.io

### BPP:
 - **BPP Client Sandbox:** bpp-ps-client-sandbox-uai.becknprotocol.io
 - **BPP Network Sandbox:** bpp-ps-network-sandbox-uai.becknprotocol.io
 - **BPP Sandbox:** bpp-unified-sandbox-uai.becknprotocol.io

### Domain name:
    retail:uai
