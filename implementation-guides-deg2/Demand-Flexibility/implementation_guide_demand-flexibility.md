# DEG Implementation Guide - Demand Flexibility

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 12-05-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides integration guidelines for implementing demand-side flexibility programs over a Beckn-enabled open energy network. It showcases how energy providers and aggregators in San Francisco can publish flexible load reduction programs, enabling users to participate in grid-responsive activities and earn monetary rewards for reducing electricity usage during peak hours.

This guide assumes the reader is familiar with the Beckn protocol, message flow, and schema standards. It maps the JSON-based catalog provided by participating BPPs to the Beckn on_search response pattern.

## Structure of the document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualisation) - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
2. General Flow diagrams - This section is relevant to all the messages flows illustrated below and discussed further in the document. We can refer [this](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams) section in the generic implementation guide to understand the flow.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. [Taxonomy and layer 2 configuration](#taxonomy-and-layer-2-configuration) - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
5. Notes on writing/integrating with your own software - We can refer [this](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software) section in the generic implementation guide.
6. [Links to artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Regitry/Gateway and BPP.

## Outcome Visualisation

### Use case - Discovery, order and fulfillment of Demand Flexibility

This use cases uses the names "Pulse Energy" and "Kazam" as examples for illustration.

- Srilekha is an IT professional who travels for work five days a week. She uses her four-wheeler electric vehicle (EV) for her commute. She prefers to charge her vehicle during her work hours and looks for suitable charging stations near her workplace.

**Discovery**:

- Srilekha begins to browse the Pulse Energy app to search for nearby EV chargers for four-wheelers.

- She receives a catalogue of 4 available chargers provided by Pulse energy & Kazam. Among them, she finds one which is located at a distance of 500 metres, which was in the catalogue of Kazam.

**Order**:

- Srilekha selects the charger, opting for a charging session at the cost of Rs. 13/kWh for a duration of 1 hour.

- She accepts the terms of order and is prompted to choose a payment method (card or link).

- She chooses to make payment through the link and confirms the order.

- The order is confirmed by Kazam’s charger provider and verifies the payment and generates an order ID

**Fulfillment**:

- Srilekha plugs the kazam’s charger into her vehicle and initiates the charging process. After an hour, the charging stops, and she is prompted to either remove the charger or continue charging. Srilekha removes the charger from her vehicle, thus ending the charging process.

**Post Fulfilment**: 

- Srilekha rates her experience using a 0-5 star rating.

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
    "ttl": "PT10M",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "timestamp": "2024-08-05T09:21:12.618Z",
    "message_id": "e138f204-ec0b-415d-9c9a-7b5bafe10bfe",
    "transaction_id": "2ad735b9-e190-457f-98e5-9702fd895996",
    "domain": "ev-charging:uei",
    "version": "1.1.0",
    "bap_id": "example-bap-id",
    "bap_uri": "https://example-bap-url.com"
  },
  "message": {
    "intent": {
      "descriptor": {
        "name": "charging stations for ioniq 5"
      },
      "category": {
        "descriptor": {
          "code": "green-tariff"
        }
      },
      "fulfillment": {
        "stops": [
          {
            "location": {
              "circle": {
                "gps": "12.423423,77.325647",
                "radius": {
                  "type": "CONSTANT",
                  "value": "5",
                  "unit": "km"
                }
              }
            }
          }
        ],
        "tags": [
          {
            "list": [
              {
                "descriptor": {
                  "code": "connector-type"
                },
                "value": "CCS2"
              }
            ]
          }
        ]
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
    "domain": "ev-charging:uei",
    "action": "on_search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
          "id": "example_provider_id",
          "descriptor": {
            "name": "Example Company",
            "short_desc": "Example Company Pvt Ltd",
            "images": [
              {
                "url": "https://example-company.com/images/logo.png"
              }
            ]
          },
          "categories": [
            {
              "id": "1",
              "descriptor": {
                "code": "green-tariff",
                "name": "green tariff"
              }
            }
          ],
          "locations": [
            {
              "id": "1",
              "gps": "12.345345,77.389754",
              "descriptor": {
                "name": "ABC Charging station, sector 47"
              },
              "address": "123, Plaza house, street 23"
            },
            {
              "id": "2",
              "gps": "12.247934,77.876987",
              "descriptor": {
                "name": "ABC Charging station, sector 81"
              },
              "address": "824, peter lane, street 8"
            }
          ],
          "items": [
            {
              "id": "pe-charging-01",
              "descriptor": {
                "code": "energy"
              },
              "price": {
                "value": "8",
                "currency": "INR/kWH"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "100",
                    "unit": "kWH"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": {
                      "name": "Connector Specifications"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "connector Id",
                            "code": "connector-id"
                        },
                        "value": "con1"
                    },
                    {
                        "descriptor": {
                            "name": "Charger Type",
                            "code": "charger-type"
                        },
                        "value": "DC"
                    },
                    {
                        "descriptor": {
                            "name": "Connector Type",
                            "code": "connector-type"
                        },
                        "value": "CCS2"
                    },
                    {
                        "descriptor": {
                            "name": "Power Rating",
                            "code": "power-rating"
                        },
                        "value": "30kW"
                    },
                    {
                        "descriptor": {
                            "name": "Availability",
                            "code": "availability"
                        },
                        "value": "Available"
                    }
                  ]
                }
              ],
              "category_ids": [
                "1"
              ],
              "location_ids": [
                "1"
              ],
              "fulfillment_ids": [
                "1"
              ],
              "add_ons": [
                {
                  "id": "pe-charging-01-addon-1",
                  "descriptor": {
                    "name": "Free car wash"
                  },
                  "price": {
                    "value": "0",
                    "currency": "INR"
                  }
                }
              ]
            },
            {
              "id": "pe-charging-02",
              "descriptor": {
                "code": "energy"
              },
              "price": {
                "value": "8",
                "currency": "INR/kWH"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "100",
                    "unit": "kWH"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": {
                      "name": "Connector Specifications"
                  },
                  "list": [
                    {
                        "descriptor": {
                            "name": "connector 1",
                            "code": "connector-id"
                        },
                        "value": "con4"
                    },
                    {
                        "descriptor": {
                            "name": "Charger Type",
                            "code": "charger-type"
                        },
                        "value": "AC"
                    },
                    {
                        "descriptor": {
                            "name": "Connector Type",
                            "code": "connector-type"
                        },
                        "value": "CCS2"
                    },
                    {
                        "descriptor": {
                            "name": "Power Rating",
                            "code": "power-rating"
                        },
                        "value": "40kW"
                    },
                    {
                        "descriptor": {
                            "name": "Availability",
                            "code": "unavailability"
                        },
                        "value": "Available"
                    }
                  ]
                }
              ]
              "category_ids": [
                "1"
              ],
              "location_ids": [
                "1"
              ],
              "fulfillment_ids": [
                "3"
              ],
              "add_ons": [
                {
                  "id": "pe-charging-01-addon-1",
                  "descriptor": {
                    "name": "Free car wash"
                  },
                  "price": {
                    "value": "0",
                    "currency": "INR"
                  }
                }
              ]
            }
          ],
          "fulfillments": [
            {
              "id": "1",
              "type": "CHARGING",
              "tags": [
                {
                  "descriptor": {
                    "name": "Charging Point Specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Pillar Number 4",
                        "code": "charger-id"
                      },
                      "value": "charg1"
                    },
                    {
                      "descriptor": {
                          "code": "Availability"
                      },
                      "value": "Available"
                    }
                  ]
                }
              ]
            },
            {
              "id": "3",
              "type": "CHARGING",
              "tags": [
                {
                  "descriptor": {
                    "name": "Charging Point Specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Pillar Number 3",
                        "code": "charger-id"
                      },
                      "value": "charg3"
                    },
                    {
                      "descriptor": {
                          "code": "Availability"
                      },
                      "value": "Unvailable"
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

### select

**sending a select request**

- Choose the item(s) from the list from on_search and request quote
- The chosen item is in message->order->item_id

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "select",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "provider": {
        "id": "example_provider_id"
      },
      "items": [
        {
          "id": "pe-charging-01",
          "quantity": {
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ]
        }
      ]
    }
  }
}
```

### on_select

- The BPP returns back with a quote for the selection
- It is in message->order->quote

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_select",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector Id",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    },
                    "value": "Available"
                }
              ]
            }
          ]
        },
        {
          "id": "pe-charging-01-addon-1",
          "descriptor": {
            "code": "add-on-item",
            "name": "Free car wash"
          },
          "price": {
            "value": "0",
            "currency": "INR"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "type": "CHARGING",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ]
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "39.7",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "charging session cost",
            "item": {
              "descriptor": {
                "name": "Estimated units consumed"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "32",
              "currency": "INR"
            }
          },
          {
            "item": {
              "descriptor": {
                "name": "Free car wash"
              }
            },
            "price": {
              "value": "0",
              "currency": "INR"
            }
          },
          {
            "title": "gst",
            "price": {
                "currency": "INR",
                "value": "7.20"
            }
          }
        ]
      }
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
    "domain": "ev-charging:uei",
    "action": "init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "provider": {
        "id": "example_provider_id"
      },
      "items": [
        {
          "id": "pe-charging-01"
        }
      ],
      "billing": {
        "name": "John Doe",
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "fulfillments": [
        {
          "id": "1",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          }
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
    "domain": "ev-charging:uei",
    "action": "on_init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector 1",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    },
                    "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "type": "CHARGING",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          }
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "billing": {
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "quote": {
        "price": {
          "value": "39.7",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "charging session cost",
            "item": {
              "descriptor": {
                "name": "Estimated units consumed"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "32",
              "currency": "INR"
            }
          },
          {
            "item": {
              "descriptor": {
                "name": "Free car wash"
              }
            },
            "price": {
              "value": "0",
              "currency": "INR"
            }
          },
          {
            "title": "gst",
            "price": {
                "currency": "INR",
                "value": "7.20"
            }
          }
        ]
      }
      "payments": [
        {
          "url": "https://payment.gateway.in",
          "collected_by": "bpp",
          "type": "PRE-ORDER",
          "status": "NOT-PAID",
          "params": {
            "amount": "40",
            "currency": "INR"
          }
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
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
    "domain": "ev-charging:uei",
    "action": "confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "provider": {
        "id": "example_provider_id"
      },
      "items": [
        {
          "id": "pe-charging-01"
        }
      ],
      "billing": {
        "name": "John Doe",
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "fulfillments": [
        {
          "id": "1",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          }
        }
      ],
      "payments": [
        {
          "collected_by": "BPP",
          "params": {
            "amount": "40",
            "currency": "INR",
            "transaction_id": "tans1"
          },
          "status": "PAID",
          "type": "PRE-ORDER"
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
    "domain": "ev-charging:uei",
    "action": "on_confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector 1",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    },
                    "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "type": "CHARGING",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              },
              "instructions": {
                "name": "Charging instructions",
                "short_desc": "To start your charging, go to charger number 987, and click on 'start' on your app"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          },
          "state": {
            "descriptor": {
              "code": "payment-completed"
            }
          },
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "billing": {
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "quote": {
        "price": {
          "value": "39.7",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "charging session cost",
            "item": {
              "descriptor": {
                "name": "Estimated units consumed"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "32",
              "currency": "INR"
            }
          },
          {
            "item": {
              "descriptor": {
                "name": "Free car wash"
              }
            },
            "price": {
              "value": "0",
              "currency": "INR"
            }
          },
          {
            "title": "gst",
            "price": {
                "currency": "INR",
                "value": "7.20"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "transaction_id": "trans1",
            "amount": "40",
            "currency": "INR"
          },
          "time": {
            "timestamp": "2023-07-16T09:30:00+05:30"
          }
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
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
    "domain": "ev-charging:uei",
    "action": "status",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order_id": "6743e9e2-4fb5-487c-92b7"
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
    "domain": "ev-charging:uei",
    "action": "on_status",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector 1",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    },
                    "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "type": "CHARGING",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          },
          "state": {
            "descriptor": {
              "code": "pehicle getting charged"
            }
          },
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            },
            {
              "descriptor": {
                  "name": "Charging Details",
                  "code": "charging-details"
              },
              "list": [
                {
                    "descriptor": {
                      "name": "Energy Delivered",  
                      "code": "energy-delivered"
                    },
                    "value": "2.3kWh"
                },
                {
                    "descriptor": {
                      "name": "State of Charge",
                      "code": "soc"
                    },
                    "value": "80%"
                },
                {
                  "descriptor": {
                      "name": "Start Time",
                      "code": "start-time"
                    },
                    "value": "2023-07-16T10:30:00.000Z"
                },
                {
                  "descriptor": {
                      "name": "Stop Time",
                      "code": "stop-time"
                    },
                    "value": "2023-07-16T11:30:00.000Z"
                },
                {
                  "descriptor": {
                      "name": "Meter Start",
                      "code": "meter-start"
                    },
                    "value": "12345"
                },
                {
                  "descriptor": {
                      "name": "Meter Stop",
                      "code": "meter-stop"
                    },
                    "value": "14345"
                },
                {
                  "descriptor": {
                      "name": "Current",
                      "code": "current"
                    },
                    "value": "10A"
                },
                {
                  "descriptor": {
                      "name": "Power",
                      "code": "power"
                    },
                    "value": "3.3kW"
                },
                {
                  "descriptor": {
                      "name": "Voltage",
                      "code": "voltage"
                    },
                    "value": "330V"
                }
              ],
              "display": true
            }
          ]
        }
      ],
      "billing": {
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "quote": {
        "price": {
          "value": "39.7",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "charging session cost",
            "item": {
              "descriptor": {
                "name": "Estimated units consumed"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "32",
              "currency": "INR"
            }
          },
          {
            "item": {
              "descriptor": {
                "name": "Free car wash"
              }
            },
            "price": {
              "value": "0",
              "currency": "INR"
            }
          },
          {
            "title": "gst",
            "price": {
                "currency": "INR",
                "value": "7.20"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "transaction_id": "trans1",
            "amount": "40",
            "currency": "INR"
          },
          "time": {
            "timestamp": "2023-07-16T09:30:00+05:30"
          }
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### update (start charging)

- Update the state of the fulfillment to start-charging.
- This will trigger charging start.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "update",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "update_target": "order.fulfillments[0].state",
    "order": {
      "fulfillments": [
        {
          "id": "1",
          "type": "CHARGING",
          "state": {
            "descriptor": {
              "code": "start-charging"
            }
          }
        }
      ]
    }
  }
}
```

### on_update (start charging)

- Confirmation of successful start of charging operation
- State in message->order->fulfillments[]->state

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_update",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector 1",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    }
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "type": "CHARGING",
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30"
              },
              "instructions": {
                "name": "Charging instructions",
                "short_desc": "To start your charging, go to charger number 987, and click on 'start' on your app"
              }
            },
            {
              "type": "finish",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          },
          "state": {
            "descriptor": {
              "code": "charging started"
            }
          }
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "billing": {
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "quote": {
        "price": {
          "value": "39.7",
          "currency": "INR"
        },
        "breakup": [
          {
            "title": "charging session cost",
            "item": {
              "descriptor": {
                "name": "Estimated units consumed"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "32",
              "currency": "INR"
            }
          },
          {
            "item": {
              "descriptor": {
                "name": "Free car wash"
              }
            },
            "price": {
              "value": "0",
              "currency": "INR"
            }
          },
          {
            "title": "gst",
            "price": {
                "currency": "INR",
                "value": "7.20"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "transaction_id": "trans1",
            "amount": "40",
            "currency": "INR"
          },
          "time": {
            "timestamp": "2023-07-16T09:30:00+05:30"
          }
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### update (end charging)

- Request to change fulfillment state to end-charging

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "update",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "update_target": "order.fulfillments[0].state",
    "order": {
      "fulfillments": [
        {
          "id": "1",
          "state": {
            "descriptor": {
              "code": "end-charging"
            }
          }
        }
      ]
    }
  }
}
```

### on_update (end charging)

- Confirmation of successful operation to end charging
- The state will be in message->order->fulfillments[]->state

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_update",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector Id",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    },
                    "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          },
          "type": "CHARGING",
          "state": {
            "descriptor": {
              "code": "charging-ended"
            }
          },
          "stops": [
            {
              "type": "start",
              "location": {
                "gps": "12.423423,77.325647"
              },
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30",
              }
            },
            {
              "type": "end",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30",
              }
            }
          ],
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            },
            {
              "descriptor": {
                  "name": "Charging Details",
                  "code": "charging-details"
              },
              "list": [
                {
                    "descriptor": {
                      "name": "Energy Delivered",  
                      "code": "energy-delivered"
                    },
                    "value": "2.3kWh"
                },
                {
                    "descriptor": {
                      "name": "State of Charge",
                      "code": "soc"
                    },
                    "value": "80%"
                },
                {
                  "descriptor": {
                      "name": "Start Time",
                      "code": "start-time"
                    },
                    "value": "2023-07-16T10:30:00.000Z"
                },
                {
                  "descriptor": {
                      "name": "Stop Time",
                      "code": "stop-time"
                    },
                    "value": "2023-07-16T11:30:00.000Z"
                },
                {
                  "descriptor": {
                      "name": "Meter Start",
                      "code": "meter-start"
                    },
                    "value": "12345"
                },
                {
                  "descriptor": {
                      "name": "Meter Stop",
                      "code": "meter-stop"
                    },
                    "value": "14345"
                },
                {
                  "descriptor": {
                      "name": "Current",
                      "code": "current"
                    },
                    "value": "10A"
                },
                {
                  "descriptor": {
                      "name": "Power",
                      "code": "power"
                    },
                    "value": "3.3kW"
                },
                {
                  "descriptor": {
                      "name": "Voltage",
                      "code": "voltage"
                    },
                    "value": "330V"
                }
              ],
              "display": true
            }
          ]
        }
      ],
      "billing": {
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "quote": {
        "price": {
          "value": "40",
          "currency": "INR"
        },
        "breakup": [
          {
            "item": {
              "descriptor": {
                "name": "Estimated units consumed"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "32",
              "currency": "INR"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "amount": "40",
            "currency": "INR"
          },
          "time": {
            "timestamp": "2023-07-16T09:30:00+05:30"
          }
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```

### support

- Request for support information.
- If regarding a specific order, specify the order_id in message->support->ref_id

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "support",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "support": {
      "ref_id": "6743e9e2-4fb5-487c-92b7"
    }
  }
}
```

### on_support

- Contains support information. If integrated with a CMS, the URL could contain order specific support info
- In all other cases, contains general support info (message->support)

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_support",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "support": {
      "ref_id": "6743e9e2-4fb5-487c-92b7",
      "phone": "1800 1080",
      "email": "customer.care@example-company.com",
      "url": "https://www.example-company.com/helpdesk"
    }
  }
}
```

### track

- Request for a tracking URL for the order. Order Id must be specified in message->order_id

```
{
    "context": {
        "domain": "ev-charging:uei",
        "action": "track",
        "location": {
            "city": {
                "code": "std:080"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "example-bap-id",
        "bap_uri": "https://example-bap-url.com",
        "bpp_id": "example-bpp-id",
        "bpp_uri": "https://example-bpp-url.com",
        "transaction_id": "e0a38442-69b7-4698-aa94-a1b6b5d244c2",
        "message_id": "6ace310b-6440-4421-a2ed-b484c7548bd5",
        "timestamp": "2023-02-18T17:00:40.065Z",
        "version": "1.0.0",
        "ttl": "PT10M"
    },
    "message": {
        "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38"
    }
}
```

### on_track

- Tracking URL with custom UI on charging process.
- The tracking URL will be in message->tracking->url

```
{
    "context": {
        "domain": "ev-charging:uei",
        "action": "on_track",
        "location": {
            "city": {
                "code": "std:080"
            },
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "bap_id": "example-bap-id",
        "bap_uri": "https://example-bap-url.com",
        "bpp_id": "example-bpp-id",
        "bpp_uri": "https://example-bpp-url.com",
        "transaction_id": "e0a38442-69b7-4698-aa94-a1b6b5d244c2",
        "message_id": "6ace310b-6440-4421-a2ed-b484c7548bd5",
        "timestamp": "2023-02-18T17:00:40.065Z",
        "version": "1.0.0",
        "ttl": "PT10M"
    },
    "message": {
        "tracking": {
            "id": "38473fhjd7sriyr8",
            "url": "https://xyz/tracking/201f6fa2-a2f7-42e7-a2e5-8947398747",
            "status": "active"
        }
    }
}
```

### cancel

- Request to cancel an order. order_id to be specified in message->order_id
- In the case where there is no advance booking of the charging slot, this does not have much relevance

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "cancel",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "cancellation_reason_id": "5",
    "descriptor": {
      "short_desc": "can't attend booking"
    },
    "order_id": "6743e9e2-4fb5-487c-92b7"
  }
}
```

### on_cancel

- Confirmation of cancelled order. The cancelled order will be in message->order
- In cases where advanced booking of charging station is not supported, this message is not relevant.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_cancel",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "status": "CANCELLED",
      "provider": {
        "id": "example_provider_id",
        "descriptor": {
          "name": "Example Company",
          "short_desc": "Example Company Pvt Ltd",
          "images": [
            {
              "url": "https://example-company.com/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "pe-charging-01",
          "descriptor": {
            "code": "energy"
          },
          "price": {
            "value": "8",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "100",
                "unit": "kWh"
              }
            },
            "selected": {
              "measure": {
                "value": "4",
                "unit": "kWh"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                  "name": "Connector Specifications"
              },
              "list": [
                {
                    "descriptor": {
                        "name": "connector Id",
                        "code": "connector-id"
                    },
                    "value": "con1"
                },
                {
                    "descriptor": {
                        "name": "Charger Type",
                        "code": "charger-type"
                    },
                    "value": "DC"
                },
                {
                    "descriptor": {
                        "name": "Connector Type",
                        "code": "connector-type"
                    },
                    "value": "CCS2"
                },
                {
                    "descriptor": {
                        "name": "Power Rating",
                        "code": "power-rating"
                    },
                    "value": "30kW"
                },
                {
                    "descriptor": {
                        "name": "Availability",
                        "code": "availability"
                    },
                    "value": "Available"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "customer": {
            "person": {
              "name": "John Doe"
            },
            "contact": {
              "phone": "+91-9887766554"
            }
          },
          "type": "CHARGING",
          "state": {
            "descriptor": {
              "code": "order-cancelled"
            }
          },
          "stops": [
            {
              "type": "start",
              "time": {
                "timestamp": "2023-07-16T10:00:00+05:30",
              }
            },
            {
              "type": "end",
              "time": {
                "timestamp": "2023-07-16T10:30:00+05:30",
              }
            }
          ],
          "tags": [
            {
              "descriptor": {
                "name": "Charging Point Specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Pillar Number 4",
                    "code": "charger-id"
                  },
                  "value": "charg1"
                },
                {
                  "descriptor": {
                      "code": "Availability"
                  },
                  "value": "Available"
                }
              ]
            },
            {
              "descriptor": {
                  "name": "Charging Details",
                  "code": "charging-details"
              },
              "list": [
                {
                    "descriptor": {
                      "name": "Energy Delivered",  
                      "code": "energy-delivered"
                    },
                    "value": "2.3kWh"
                },
                {
                    "descriptor": {
                      "name": "State of Charge",
                      "code": "soc"
                    },
                    "value": "80%"
                },
                {
                  "descriptor": {
                      "name": "Start Time",
                      "code": "start-time"
                    },
                    "value": "2023-07-16T10:30:00.000Z"
                },
                {
                  "descriptor": {
                      "name": "Stop Time",
                      "code": "stop-time"
                    },
                    "value": "2023-07-16T11:30:00.000Z"
                },
                {
                  "descriptor": {
                      "name": "Meter Start",
                      "code": "meter-start"
                    },
                    "value": "12345"
                },
                {
                  "descriptor": {
                      "name": "Meter Stop",
                      "code": "meter-stop"
                    },
                    "value": "14345"
                },
                {
                  "descriptor": {
                      "name": "Current",
                      "code": "current"
                    },
                    "value": "10A"
                },
                {
                  "descriptor": {
                      "name": "Power",
                      "code": "power"
                    },
                    "value": "3.3kW"
                },
                {
                  "descriptor": {
                      "name": "Voltage",
                      "code": "voltage"
                    },
                    "value": "330V"
                }
              ],
              "display": true
            }
          ]
        }
      ],
      "billing": {
        "email": "abc@example.com",
        "phone": "+91-9876522222"
      },
      "quote": {
        "price": {
          "value": "-32",
          "currency": "INR"
        },
        "breakup": [
          {
            "item": {
              "descriptor": {
                "name": "payment refund"
              },
              "quantity": {
                "selected": {
                  "measure": {
                    "value": "4",
                    "unit": "kWh"
                  }
                }
              }
            },
            "price": {
              "value": "-32",
              "currency": "INR"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "amount": "40",
            "currency": "INR"
          },
          "time": {
            "timestamp": "2023-07-16T09:30:00+05:30"
          }
        }
      ],
      "cancellation_terms": [
        {
          "fulfillment_state": {
            "descriptor": {
              "code": "charging-start"
            }
          },
          "cancellation_fee": {
            "percentage": "30%"
          },
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://example-company.com/charge/tnc.html"
          }
        }
      ]
    }
  }
}
```
### get_rating_categories
- BAP can fetch a list of categoried for which a BPP supports prviding rating
- BAP does not need to send anything in the message field.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
  }
}
```
### rating_categories

- The BPP responds with a list of categories in which ratings can be provided.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "rating_categories": [
      Item,
      Order,
      Fulfillment,
      Provider
    ]
  }
}
```

### rating

- Rate different categories including fulfillment. Specify in message->ratings
- Specify a rating category from the list recieved in the rating_categories callback from the BPP.
- Value should be a decimal between 0 and 5.
- Id can be the order id.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "rating",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "ratings": [
      {
        "id": "6743e9e2-4fb5-487c-92b7",
        "rating_category": "Fulfillment",
        "value": "5"
      }
    ]
  }
}
```

### on_rating

- Confirmation of rating.
- Optionally send a form for additional input (message->feedback_form). Empty message otherwise.

```
{
  "context": {
    "domain": "ev-charging:uei",
    "action": "on_rating",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "code": "std:080"
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
    "feedback_form": {
      "form": {
        "url": "https://example-bpp.comfeedback/portal"
      },
      "required": false
    }
  }
}
```

## Taxonomy and layer 2 configuration

## Links to artefacts

- [Postman collection for Demand Flexibility]()
- [Layer2 config for UEI Demand Flexibility]()

## Sandbox Details

![<Sandbox Name>]()

### Registry/Gateway:

- **Gateway Sandbox:** []()
- **Registry Sandbox:** []()

### BPP:

- **BPP Client Sandbox:** []()
- **BPP Network Sandbox:** []()
- **BPP Playground Sandbox:** []()
