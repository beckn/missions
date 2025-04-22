# UAI Implementation Guide - Weather Data

#### Version 1.1

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 06-11-2024 | 0.1     | Initial Version                                     |
| 14-11-2024 | 0.2     | Internal Review Comments Incorprated                |
| 18-11-2024 | 1.0     | Final Version                                       |
| 04-12-2024 | 1.1     | A new section created for Schema Details and added details for Weather forecast      |

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

![UAI Knowledge Advisory Outcome Visualization](../../Images/uai_knowledge_advisory_outcome_visualization.svg)

### Use case - Discovery and consumption of paid knowledge advisory

1. Monisha, a tomato farmer in Nashik, seeks to understand upcoming weather changes to plan her fertilization, irrigation, pest control, and other crop management practices.
2. Using a beckn-enabled app connected to the UAI network, she searches for weather data and forecasts specific to her farm's location.
3. She discovers a variety of weather content providers on the network, each offering different forecasts, ratings, and pricing options.
4. Monisha chooses a 14-day hyperlocal weather forecast from News14, priced at INR 30, based on the service's details and rating.
5. After selecting her forecast, she provides her contact information and billing address, receiving a final quote and terms of purchase, including any cancellation or refund policies.
5. Reviewing the terms, Monisha makes her payment via UPI and confirms her order, receiving an order ID from the service provider.
6. Shortly after, Monisha receives her detailed weather report from News14, formatted as a PDF or in a user-friendly display on the app.
7. Satisfied with the service, she submits a rating based on the quality and usefulness of the weather information.
8. The app allows Monisha to reach out to support services in case she encounters any issues with the advisory service.

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

### Use case - Discovery and consumption weather forecast report

**Search for resources on topic**

![Search for resources on infested disease](../../Images/Advisory/paid/UAI%20advisory-Discovery.jpg)

**Place an order for the advisory**

![Select the advisory, initialise and confirm the order ](../../Images/Advisory/paid/UAI%20advisory-Order.jpg)

**fulfillment of an active order**

![check the order status, update the order details or cancel the order ](../../Images/Advisory/paid/UAI%20advisory-Fulfilment.jpg)

**Seek support and provide rating**

![Support call to get contact information of Provider platofrm](../../Images/Advisory/paid/UAI%20advisory-Post%20Fulfilment.jpg)

## API Calls and Schema

### Discovery and consumption of weather forecast reports

#### search

**search by topic, time range and fulfillment location**
```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "7b3d0c62-7c1b-4c6b-b768-14f81b6c3c90",
    "message_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
     "intent": {
        "category": {
            "descriptor": {
              "code": "Weather-Forecast"
          }
        },
        "item": {
          "time" : {
            "range" : {
              "start" : "2024-03-01T00:00:00.000Z",
              "end" : "2024-03-15T00:00:00.000Z"
            }
          }
        },
        "fulfillment": {
          "stops": [
            {
              "location": {
                "gps": ""12.9716, 77.5946""
              }
            }
          ]
        }
      }
  }
}
```

**search by topic, provider licence, item datapoints**
```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "7b3d0c62-7c1b-4c6b-b768-14f81b6c3c90",
    "message_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
     "intent": {
      "provider": {
        "tags": [
          {
            "descriptor": {
              "name": "Provider's Additional Information"
            },
            "list": [
              {
                "descriptor": {
                  "name": "License"
                },
                "value": "Proprietary"
              }
            ]
          }
        ]
      },
      "category": {
          "descriptor": {
            "code": "Weather-Forecast"
        }
      },
      "item": {
        "time" : {
          "range" : {
            "start" : "2024-03-01T00:00:00.000Z",
            "end" : "2024-03-15T00:00:00.000Z"
          }
        },
        "tags": [
          {
            "descriptor": {
              "name": "Weather datapoints"
            },
            "list": [
              {
                "value": "Temperature"
              },
              {
                "value": "Vertical wind speed"
              }
            ]
          }
        ]
      },
      "fulfillment": {
        "stops": [
          {
            "location": {
              "gps": ""12.9716, 77.5946""
            }
          }
        ]
      }
    }
  }
}
```

**search by topic, duration, language and fulfillment location**
```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "7b3d0c62-7c1b-4c6b-b768-14f81b6c3c90",
    "message_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
     "intent": {
      "category": {
          "descriptor": {
            "code": "Weather-Forecast"
        }
      },
      "item": {
        "time" : {
          "duration" : "P7D" // 7 days in ISO8601
        },
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
            "type": "Forecast-Region"
            "location": {
              "gps": ""12.9716, 77.5946""
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
      "location": {
        "country": {
          "code": "INDIA"
        }
      },
      "action": "on_search",
      "version": "1.1.0",
      "bap_id": "example-bap.becknprotocol.io",
      "bap_url": "https://example-bap-client.becknprotocol.io",
      "bpp_id": "example-bpp.becknprotocol.io",
      "bpp_uri": "https://example-bpp-network.becknprotocol.io",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "catalog": {
        "descriptor": {
          "name": "Data source platform ltd"
        },
        "providers": [
          {
            "id": "1",
            "descriptor": {
              "name": "Amicus",
              "short_desc": "Amicus Agri Services",
              "images": [
                  {
                      "url": "https://amicusagro.info/img/amicus_logo.jpg"
                  }
              ]
            },
            "tags": [
              {
                "descriptor": {
                  "name": "Provider's Additional Information"
                },
                "list": [
                  {
                    "descriptor": {
                      "name": "License"
                    },
                    "value": "Proprietary"
                  },
                  {
                    "descriptor": {
                      "name": "Provider's years in operation"
                    },
                    "value": "3"
                  }
                ]
              }
            ],
            "rating": "4.9",
            "categories": [
              {
                "id": "c1",
                "descriptor": {
                  "code": "Weather-forecast",
                  "name": "Weather Forecast"
                }
              }
            ],
            "fulfillments": [
              {
                "id": "f1",
                "stops": [
                  "time" : {
                    "range" : {
                      "start" : "2024-12-20T00:00:00.000Z",
                      "end" : "2024-12-26T00:00:00.000Z"
                    }
                  }
                ]
              }
            ],
            "items": [
              {
                "id": "1",
                "descriptor": {
                  "images": [
                    {
                      "url": "https://www.analyticssteps.com/backend/media/thumbnail/6006173/6278986_1571298721_Weather_Forecoast_Graphics.jpg"
                    }
                  ],
                  "name": "Hyperlocal weather forecast",
                  "short_desc": "7 दिवसाचा हवामान अंदाज",
                  "long_desc": "Our Weather Forecast Data delivers reliable and timely forecasts tailored specifically for regions. Stay informed about current and future weather conditions, including temperature variations, precipitation levels, wind speed patterns, and more."
                },
                "matched": true,
                "recommended": true,
                "category_ids": [
                  "c1"
                ],
                "fulfillment_ids": [
                  "f1"
                ],
                "tags": [
                  {
                    "descriptor": {
                      "code": "2024-12-20"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "16.7°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "29.4°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "0mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "11-12 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "N"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "3mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "6"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "code": "2024-12-21"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "14.7°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "29.9°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "0mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "4-8 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "NNW"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "4mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "7"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "code": "2024-12-22"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "15.6°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "30.3°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "0mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "7-9 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "NE"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "4mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "7"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "code": "2024-12-23"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "17.6°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "22.2°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "1.5mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "6-8 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "SE"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "4mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "7"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "code": "2024-12-24"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "17.1°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "28.4°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "0.3mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "4-8 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "SE"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "4mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "7"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "code": "2024-12-25"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "16.2°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "29.6°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "0mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "6-9 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "ENE"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "3mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "4"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "code": "2024-12-26"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "code": "min-temp"
                        }
                        "value": "14.4°C"
                      },
                      {
                        "descriptor": {
                          "code": "max-temp"
                        }
                        "value": "29.4°C"
                      },
                      {
                        "descriptor": {
                          "code": "precipitation"
                        }
                        "value": "0mm"
                      },
                      {
                        "descriptor": {
                          "code": "sunlight"
                        }
                        "value": "11h"
                      },
                      {
                        "descriptor": {
                          "code": "wind-speed"
                        }
                        "value": "6-11 km/hr"
                      },
                      {
                        "descriptor": {
                          "code": "wind-dir"
                        }
                        "value": "ENE"
                      },
                      {
                        "descriptor": {
                          "code": "eto"
                        }
                        "value": "3mm"
                      },
                      {
                        "descriptor": {
                          "code": "uv"
                        }
                        "value": "6"
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
**on_search: Example catalog in case the result is a video**
```
{
  "context": {
    "domain": "advisory:uai",
    "action": "on_search",
    "version": "1.1.0",
    "bpp_id": "onix-bpp.fasal.co",
    "bpp_uri": "https://onix-bpp.fasal.co",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "bap_id": "wa-bap",
    "bap_uri": "https://wa-bap.freecustomer.in",
    "transaction_id": "9504db39-a7cd-4d6e-b935-2d6fd33c96eb",
    "message_id": "3cddbac5-b9fa-41a9-aab0-f59781bb1f46",
    "ttl": "PT10M",
    "timestamp": "2024-12-03T12:15:30.660Z"
  },
  "message": {
    "catalog": {
      "descriptor": {
        "name": "Fasal",
        "images": [
          {
            "url": "https://fasal.co/images/logo.png"
          }
        ]
      },
      "providers": [
        {
          "id": "2d867cbe-a362-4cb5-81ec-0e969bb5695f",
          "descriptor": {
            "name": "Fasal"
          },
          "items": [
            {
              "id": "f590e9b2-9421-4ff7-851e-469b096a34e4",
              "descriptor": {
                "name": "FasalOne",
                "short_desc": "Fasal One is your ultimate smart farming solution, designed to help you increase your crop produce by 40% , decrease spend on inputs by 30% and  double your profits. By combining precision farming techniques with advanced automation, FasalOne puts you in complete control of your farm. Whether you cultivate a small plot or vast farmlands, this IoT-powered system adapts to your needs, helping you eliminate guesswork, boost productivity, and achieve higher profitability with ease.",
                "long_desc": "FasalOne is a flagship technology offering from Fasal. This future ready intelligent system puts precision farming at your fingertips helping you scale with your ambitions in your way. It empowers the farmers with super-smart information about their crops, offering farm-level, crop-specific, and crop-stage-specific intelligence like Irrigation Advisory, Pest and Disease Forewarnings, Weather forecast and Data Analytics. It is designed to help you Double your Profits, Reduce your input costs by upto 30%, Increase your yields by upto 40% and save Water by upto 60%. The Technological Advancements enable you to seamlessly integrate and monitor over 100 sensors across your entire farm, giving you unprecedented actionable insights into crop health, soil conditions, and environmental factors. It offers Affordable scalability where you can upgrade your smart farming journey with our flexible subscription model, beginning at just Rs. 49. The lightning-fast Over-The-Air (OTA) updates ensure that your system always remains at the forefront of agricultural innovation. With 24x7 data access through Bluetooth Low Energy (BLE) technology you get uninterrupted intelligence flow keeping you connected to your farm's critical information. Experience complete farm management with our smart, automated systems that reduce manual effort and maximize efficiency which is now available with Customization options as per your requirement.",
                "images": [
                  {
                    "url": "https://fasal-public-bucket-mumbai.s3.ap-south-1.amazonaws.com/fasal-one.png"
                  }
                ],
                "media": [
                  {
                    "mimetype": "video/mp4",
                    "url": "https://youtu.be/aJtcBCl8Cl4"
                  }
                ]
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
        "name": "IND"
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
        "code": "IND"
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
        "code": "IND"
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
        "code": "IND"
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
        "name": "IND"
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
        "name": "IND"
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
| 1  | Weather Advisory          | Location (Pin Code, Lat/Long)        | 416506 or coordinates            | int or point      |
| 2  | Weather Advisory          | Language                             | mr, en                           | varchar           |
| 3  | Weather Advisory          | Duration ( 1 day, 3 day or 7 days)   | 1 or 3 or 7                      | int               |

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
    advisory:uai
