# UAI Implementation Guide - Price Listing

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 06-11-2024 | 0.1     | Initial Version                                     |
| 14-11-2024 | 0.2     | Internal Review Comments Incorprated                |
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

![UAI Price Listing Outcome Visualization](images/uai_knowledge_advisory_outcome_visualization.svg)


### Use case - Price Listing
**Farmer searching for prices of commodities from service providers on the UKI Network**

Prashanth is a grapes farmer in Nashik. He wants to know the price of grapes in nearby mandis to plan the harvest of his crops and to plan the logistics to take his crops to the market. He uses a beckn enabled app which is integrated to the UKI network to search for commodity prices in mandis

#### Discovery
- Prashanth searches on the BAP app the price of grapes. He enters the following parameters in the search parameters - 
  - Name of commodity: e.g. Grapes
  - Variety: Thompson Seedless
  - Size: 18 to 25 mm
  - Grade: Grade A
  - Date range: From Date, to date
  - Location - GPS (Latitude/Longitude) or Pin Code
  - Range from the location: e.g. - 100 kms
- Prashanth receives multiple search results from different service providers on the price of grapes.
- The search result can be a pdf, audio file, text, video
- Prashanth chooses Agri-Dham service provider to view the price of grapes.
  - Commodity: Grapes
  - Grade: Grade A
  - Variety: Thompson Seedless
  - Size: 18 mm to 25 mm
  - Date: 12 Feb to 17 Feb
  - Mandi Name: Nashik: 
    - Min Price: INR 250
    - Max Price: INR 300
    - Average Price: INR 275
  - Mandi Name: Pune
    - Min Price: INR 250
    - Max Price: INR 500
    - Average Price: INR 375

#### Post Fulfilment
- Prashanth submits a rating for the price listing services post consuming the service. BPP can request for a rating or a feedback form
- Rating: 
  - Based on the content and delivery of the course
- Prashanth can reach out to support services in case of any issues with the service delivery


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

### Use case - Price Listing

**Search for resources on topic**

![Search for resources on infested disease](Images/Advisory/free/UAI%20advisory-Discovery.jpg)

**Seek support and provide rating**

![Support call to get contact information of Provider platofrm](./Images/Advisory/free/uai-Post%20Fulfilment.jpg)

### Use case - Discovery and consumption of paid knowledge advisory

**Search for resources on topic**

![Search for resources on infested disease](./Images/Advisory/paid/UAI%20advisory-Discovery.jpg)

**Place an order for the advisory**

![Select the advisory, initialise and confirm the order ](./Images/Advisory/paid/UAI%20advisory-Order.jpg)

**fulfillment of an active order**

![check the order status, update the order details or cancel the order ](./Images/Advisory/paid/UAI%20advisory-Fulfilment.jpg)

**Seek support and provide rating**

![Support call to get contact information of Provider platofrm](./Images/Advisory/paid/UAI%20advisory-Post%20Fulfilment.jpg)

## API Calls and Schema

### Price Listing

#### search

**search by a commodity and a date range for a price discovery**

- The desired language is specified in a tag named languages.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
    "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "price-discovery"
        }
      },
      "item": {
        "descriptor": {
          "code": "Basmati-Rice"
        }.
        "tags": [
          {
            "descriptor": {
              "name": "languages"
            },
            "list": [
              {
                "value": "mr"
              },
              {
                "value": "en"
              }
            ]
          }
        ]
      },
      "fulfillment": {
        "stops": [
          {
            "type": "price-location",
            "location": {
              "area_code": "416506" // pin code
            },
            "time": {
              "range": {
                "start": "2024-12-01T08:30:00Z",
                "end": "2024-19-10T18:00:00Z"
              }
            }
          }
        ]
      }
    }
  }
}
```

**search by variety, size, **

- The desired language is specified in a tag named languages.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
    "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "price-discovery"
        }
      },
      "item": {
        "descriptor": {
          "code": "grapes",
          "name": "Grapes"
        }.
        "tags": [
          {
            "descriptor": {
              "name": "Languages"
              "code": "languages"
            },
            "list": [
              {
                "value": "mr"
              },
              {
                "value": "en"
              }
            ]
          },
          {
            "descriptor": {
              "name": "Item Details"
              "name": "item-details"
            },
            "list": [
              {
                "descriptor": {
                  "code": "variety"
                },
                "value": "Thompson Seedless"
              },
              {
                "descriptor": {
                  "code": "size"
                },
                "value": "18 to 25 mm"
              },
              {
                "descriptor": {
                  "code": "grade"
                },
                "value": "A"
              }
            ]
          }
        ]
      },
      "fulfillment": {
        "stops": [
          {
            "type": "price-location",
            "location": {
              "circle": {
                "gps": "12.243245,24.344234",
                "radius": {
                  "unit": "KM",
                  "value": "100"
                }
              }
            },
            "time": {
              "range": {
                "start": "2024-12-01T08:30:00Z",
                "end": "2024-19-10T18:00:00Z"
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
        "domain": "advisory:uai",
        "action": "on_search",
        "location": {
          "country": {
            "name": "India",
            "code": "IND"
          },
          "city": {
            "name": "Nashik",
            "code": "std:95253"
          }
        },
        "version": "1.1.0",
        "bap_id": "example-bap.becknprotocol.io",
        "bap_uri": "https://example-bap-client.becknprotocol.io",
        "bpp_id": "example-bpp.becknprotocol.io",
        "bpp_uri": "https://example-bpp-client.becknprotocol.io",
        "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
        "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
        "timestamp": "2024-07-02T09:15:30Z"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "privider012",
                    "descriptor": {
                        "name": "ExampleAgri",
                        "short_desc": "ExampleAgri Services",
                        "images": [
                            {
                                "url": "https://ExampleAgri.info/img/logo.jpg"
                            }
                        ]
                    },
                    "time": {
                        "range": {
                            "start": "2024-12-19T00:00:00Z",
                            "end": "2024-12-19T00:00:00Z"
                        }
                    },
                    "locations": [
                        {
                            "id": "1",
                            "city": "Yewla"
                        },
                        {
                            "id": "2",
                            "city": "Laslgaon"
                        },
                        {
                            "id": "3",
                            "city": "Nashik"
                        },
                        {
                            "id": "4",
                            "city": "Satara"
                        },
                        {
                            "id": "5",
                            "city": "Malegaon"
                        },
                        {
                            "id": "6",
                            "city": "Manmad"
                        }
                    ],
                    "items": [
                        {
                            "id": "1",
                            "descriptor": {
                                "name": "Grapes"
                            },
                            "location_ids": [
                                "1"
                            ],
                            "price": {
                                "minimum_value": "301",
                                "maximum_value": "2190",
                                "estimated_value": "1400"
                            }
                        },
                        {
                            "id": "2",
                            "descriptor": {
                                "name": "Grapes"
                            },
                            "location_ids": [
                                "2"
                            ],
                            "price": {
                                "minimum_value": "800",
                                "maximum_value": "2900",
                                "estimated_value": "1900"
                            }
                        },
                        {
                            "id": "3",
                            "descriptor": {
                                "name": "Grapes"
                            },
                            "location_ids": [
                                "3"
                            ],
                            "price": {
                                "minimum_value": "1000",
                                "maximum_value": "2305",
                                "estimated_value": "1600"
                            }
                        },
                        {
                            "id": "4",
                            "descriptor": {
                                "name": "Grapes"
                            },
                            "location_ids": [
                                "4"
                            ],
                            "price": {
                                "minimum_value": "300",
                                "maximum_value": "2380",
                                "estimated_value": "2000"
                            }
                        },
                        {
                            "id": "5",
                            "descriptor": {
                                "name": "Grapes"
                            },
                            "location_ids": [
                                "5"
                            ],
                            "price": {
                                "minimum_value": "4010",
                                "maximum_value": "4010",
                                "estimated_value": "4010"
                            }
                        },
                        {
                            "id": "6",
                            "descriptor": {
                                "name": "Grapes"
                            },
                            "location_ids": [
                                "6"
                            ],
                            "price": {
                                "minimum_value": "1100",
                                "maximum_value": "2200",
                                "estimated_value": "1800"
                            }
                        }
                    ]
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
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
    "action": "support",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "d8b23543-24b4-48eb-ae8a-4a5db68f8d09",
    "transaction_id": "fa2c9c8b-ba24-4d2b-bd9c-3e03d7f6b193",
    "timestamp": "2024-07-02T09:18:30Z"
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
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
    "action": "on_support",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "d8b23543-24b4-48eb-ae8a-4a5db68f8d09",
    "transaction_id": "fa2c9c8b-ba24-4d2b-bd9c-3e03d7f6b193",
    "timestamp": "2024-07-02T09:18:30Z",
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
    "domain": "advisory:uai",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
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
    "domain": "advisory:uai",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
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

**Rating a resource**

- When the user wants to rate the resource, it happens in two steps.
- The user sends a numerical (0-5) rating in the request.
- The response will have a link to the form where the user can provide text feedback.

```
{
  "context": {
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
    "action": "rating",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "e8c50b1e-6512-42b3-b0b4-8f8a703a5c66",
    "transaction_id": "b7204c3a-9f5e-418f-80a3-ae5dd4e5b97a",
    "timestamp": "2024-07-02T09:15:30Z"
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
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Nashik",
        "code": "std:95253"
      }
    },
    "action": "on_rating",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "e8c50b1e-6512-42b3-b0b4-8f8a703a5c66",
    "transaction_id": "b7204c3a-9f5e-418f-80a3-ae5dd4e5b97a",
    "timestamp": "2024-07-02T09:15:30Z",
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

## Schema Details

| SN | Use Case                  | Input Details                        | Values                           | Data Types        |
|----|---------------------------|--------------------------------------|----------------------------------|-------------------|

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
    advisory:uai
