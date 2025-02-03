# Implementation Guide: UEI - Demand Note Sale and Purchase

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 30-01-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides material that helps network participants build and integrate their application with a Beck network for energy deamd note sale and purchase. This document is part of the starter kit that provides information about the network, learning resources, network participant checklist etc. This document only focuses on the implementation of the seeker/provider platform. It assumes the reader has a good overview of the Beckn network, its APIs, the overall structure of the schema etc.

## Structure of the document

This document has the following parts:

1. Outcome Visualization - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
2. Flow diagrams - This section provides a pictorial representation of the message flows that happen during the use case.
3. API Calls and Schema - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. Taxonomy and layer 2 configuration - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
5. Notes on writing/integrating with your own software - This section describes ways in which you can integrate (Becknify) your new or existing software
6. Links to downloadable resources - This section contains the downloadable files referenced in this document.
7. Sandbox Details - Sandbox links to BAP, Regitry/Gateway and BPP.

## Outcome Visualisation

**Context**

<TBP>

![UEI: Demand Note Sale and Purchase Outcome Visualization](images/<link to image>.png)

### Use case - Discovery, order and fulfillment of EV Charging

**User Journey**
- The user searches for available energy demand notes such as green energy demand notes (DNs) or other energy offerings in the network.
- The system provides a list of available energy demand notes with key details like: Energy type (e.g., Green or Normal), Cost per kWh, location, and energy provider (BPP).
- Users can compare and evaluate energy offerings from different providers to select the best fit for their needs.
- The user selects specific energy demand notes (DNs) they wish to purchase.
- They receive a detailed quote, including: Cost of energy (kWh), taxes, and delivery/transfer charges.
- The user adds their energy discharge location, billing details, and payment information to finalize the purchase.
- The system confirms the order with a unique ID and processes it for energy discharge or fulfillment to the grid or other locations.



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

![ACK NACK for messages](../docs/assets/images/ack_nack.png)

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


**Discovery of Demand Notes**

## API Calls and Schema

### search

A search request can include one or more search criteria. Below are examples of how a search request can be constructed.
A general rule is:
- if searching by free text, it is specified in message->intent->descriptor->name
- If searching by category, it is specified in message->intent->category->descriptor->code

Search requests are sent to the gateway url.

**Search by the category of energy**


```
{
  "context": {
    "domain": "energy",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "demand-note"
        }
      }
    }
  }
}
```

### on_search

BAP receives a response with a catalog.

**on_search with catalog of results**

- The catalog contains a list of providers.
- Each provider offers a list of items.
- Each item in the catalog represents a demand note listing, including the number of units available and the price per unit.

```
{
  "context": {
    "domain": "energy",
    "action": "on_search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "catalog": {
      "providers": [
        {
          "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6",
          "descriptor": {
            "name": "Pulse Energy",
            "short_desc": "Pulse Energy offers a state-of-the-art EV charging station designed for fast, efficient, and reliable electric vehicle charging. Equipped with advanced technology, our stations provide high-speed charging capabilities to get you back on the road quickly",
            "images": [
              {
                "url": "https://pulse.energy/images/logo.png"
              }
            ]
          },
          "categories": [
            {
              "id": "2",
              "descriptor": {
                "code": "demand-note",
                "name": "Demand Note"
              }
            }
          ],
          "locations": [
            {
              "id": "1",
              "gps": "12.345345,77.389754"
            },
            {
              "id": "2",
              "gps": "12.247934,77.876987"
            }
          ],
          "items": [
            {
              "id": "2",
              "descriptor": {
                "code": "demand-note",
                "name": "Demand Note"
              },
              "price": {
                "value": "2",
                "currency": "INR/kWH"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "4",
                    "unit": "kWH"
                  }
                }
              },
              "category_ids": [
                "2"
              ],
              "tags": [
                {
                  "descriptor": {
                    "name": "energy-specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Energy type",
                        "code": "energy-type"
                      },
                      "value": "Green Energy"
                    }
                  ],
                  "display": true
                }
              ]
            }
          ]
        },        {
          "id": "8y43894v23-8r9yr3-234i9u3-4344",
          "descriptor": {
            "name": "XYZ Energy",
            "short_desc": "XYZ is a ....",
            "images": [
              {
                "url": "https://xyz.energy/images/logo.png"
              }
            ]
          },
          "categories": [
            {
              "id": "2",
              "descriptor": {
                "code": "demand-note",
                "name": "Demand Note"
              }
            }
          ],
          "locations": [
            {
              "id": "1",
              "gps": "12.564445,77.5434455"
            },
            {
              "id": "2",
              "gps": "12.455443,77.234444"
            }
          ],
          "items": [
            {
              "id": "2",
              "descriptor": {
                "code": "demand-note",
                "name": "Demand Note"
              },
              "price": {
                "value": "3",
                "currency": "INR/kWH"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "20",
                    "unit": "kWH"
                  }
                }
              },
              "category_ids": [
                "2"
              ],
              "tags": [
                {
                  "descriptor": {
                    "name": "energy-specifications"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "name": "Energy type",
                        "code": "energy-type"
                      },
                      "value": "Green Energy"
                    }
                  ],
                  "display": true
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

- Choose item(s) from the list from the on_search response. The chosen item are identified by message->order->item[i].id
- Choose a fulfillment type, fill in the required fulfillment details.
- Send a select request to BPP. BPP endpoint can be fetched from the on_search payload context.bpp_uri field.

```
{
  "context": {
    "domain": "energy",
    "action": "select",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6"
      },
      "items": [
        {
          "id": "2"
        }
      ]
    }
  }
}
```

### on_select

- The BPP creates an on_select payload.
- The BPP generates a quote for the selected items, includes the quote in on_select payload.
- The quote is included in message->order->quote
- BPP sends an on_select request to BAP. 
- Send an on_select request to BAP. BAP endpoint can be fetched from the select payload context.bap_uri field.

```
{
  "context": {
    "domain": "energy",
    "action": "on_select",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "providers": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6",
        "descriptor": {
          "name": "Pulse Energy",
          "short_desc": "Pulse Energy offers a state-of-the-art EV charging station designed for fast, efficient, and reliable electric vehicle charging. Equipped with advanced technology, our stations provide high-speed charging capabilities to get you back on the road quickly",
          "images": [
            {
              "url": "https://pulse.energy/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "2",
          "descriptor": {
            "code": "demand-note",
            "name": "Demand Note"
          },
          "price": {
            "value": "2",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "4",
                "unit": "kWH"
              }
            }
          },
          "tags": [
            {
              "descriptor": {
                "name": "energy-specifications"
              },
              "list": [
                {
                  "descriptor": {
                    "name": "Energy type",
                    "code": "energy-type"
                  },
                  "value": "Green Energy"
                }
              ],
              "display": true
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "8",
          "currency": "INR"
        },
        "breakup": [
          {
            "item": {
              "descriptor": {
                "name": "Energy Demand cost"
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
              "value": "8",
              "currency": "INR"
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
    "domain": "energy",
    "action": "init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6"
      },
      "items": [
        {
          "id": "2"
        }
      ],
      "billing": {
        "organization": {
          "descriptor": {
            "name": "Sheru Energy Ltd"
          }
        },
        "name": "Ramesh Dsouza",
        "email": "ramesh@sheru.com",
        "phone": "+91-9876555555"
      },
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Ramesh Dsouza"
            },
            "contact": {
              "phone": "+91-9887755555"
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

```
{
  "context": {
    "domain": "energy",
    "action": "on_init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "providers": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6",
        "descriptor": {
          "name": "Pulse Energy",
          "short_desc": "Pulse Energy offers a state-of-the-art EV charging station designed for fast, efficient, and reliable electric vehicle charging. Equipped with advanced technology, our stations provide high-speed charging capabilities to get you back on the road quickly",
          "images": [
            {
              "url": "https://pulse.energy/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "2",
          "descriptor": {
            "code": "demand-note",
            "name": "Demand Note"
          },
          "price": {
            "value": "2",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "4",
                "unit": "kWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Ramesh Dsouza"
            },
            "contact": {
              "phone": "+91-9887755555"
            }
          }
        }
      ],
      "billing": {
        "organization": {
          "descriptor": {
            "name": "Sheru Energy Ltd"
          }
        },
        "name": "Ramesh Dsouza",
        "email": "ramesh@sheru.com",
        "phone": "+91-9876555555"
      },
      "quote": {
        "price": {
          "value": "8",
          "currency": "INR"
        },
        "breakup": [
          {
            "item": {
              "descriptor": {
                "name": "Energy Demand cost"
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
              "value": "8",
              "currency": "INR"
            }
          }
        ]
      },
      "payments": [
        {
          "url": "https://payment.gateway.in",
          "type": "PRE-ORDER",
          "status": "NOT-PAID",
          "params": {
            "amount": "8",
            "currency": "INR"
          },
          "time": {
            "range": {
              "start": "2023-08-10T10:00:00Z",
              "end": "2023-08-10T10:30:00Z"
            }
          }
        }
      ]
    }
  }
}
```

### confirm

- Consumer confirming the terms of the order.

```
{
  "context": {
    "domain": "energy",
    "action": "confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "providers": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6"
      },
      "items": [
        {
          "id": "2"
        }
      ],
      "billing": {
        "organization": {
          "descriptor": {
            "name": "Sheru Energy Ltd"
          }
        },
        "name": "Ramesh Dsouza",
        "email": "ramesh@sheru.com",
        "phone": "+91-9876555555"
      },
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Ramesh Dsouza"
            },
            "contact": {
              "phone": "+91-9887755555"
            }
          }
        }
      ],
      "payments": [
        {
          "url": "https://payment.gateway.in",
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "transaction_id": "paytxn_131414231",
            "amount": "8",
            "currency": "INR"
          }
        }
      ]
    }
  }
}
```

### on_confirm

- Order confirmed and an order id is generated to the order.

```
{
  "context": {
    "domain": "energy",
    "action": "on_confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "c8570489-0189-42bc-81fd-130d73ea16c4",
      "providers": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6",
        "descriptor": {
          "name": "Pulse Energy",
          "short_desc": "Pulse Energy offers a state-of-the-art EV charging station designed for fast, efficient, and reliable electric vehicle charging. Equipped with advanced technology, our stations provide high-speed charging capabilities to get you back on the road quickly",
          "images": [
            {
              "url": "https://pulse.energy/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "2",
          "descriptor": {
            "code": "demand-note",
            "name": "Demand Note"
          },
          "price": {
            "value": "2",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "4",
                "unit": "kWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "state": {
            "descriptor": {
              "code": "order-confirmed",
              "name" : "Energy Order has been confirmed"
            }
          },
          "customer": {
            "person": {
              "name": "Ramesh Dsouza"
            },
            "contact": {
              "phone": "+91-9887755555"
            }
          }
        }
      ],
      "billing": {
        "organization" : {
          "descriptor" : {
            "name" : "Sheru Energy Ltd"
          }
        },
        "name": "Ramesh Dsouza",
        "email": "ramesh@sheru.com",
        "phone": "+91-9876555555"
      },
      "quote": {
        "price": {
          "value": "8",
          "currency": "INR"
        },
        "breakup": [
          {
            "item": {
              "descriptor": {
                "name": "Energy Demand cost"
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
              "value": "8",
              "currency": "INR"
            }
          }
        ]
      },
      "payments": [
        {
          "url": "https://payment.gateway.in",
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "transaction_id": "paytxn_131414231",
            "amount": "8",
            "currency": "INR"
          },
          "time": {
            "range": {
              "start": "2023-08-10T10:00:00Z",
              "end": "2023-08-10T10:30:00Z"
            }
          }
        }
      ]
    }
  }
}
```


### status

- BAP checks the status of the Order by calling the /status endpoint of BPP.
- Request for status of an active order. Order_id is used to fetch the order status. order_id is specified in message->order_id field of the status request.

```
{
  "context": {
    "domain": "energy",
    "action": "status",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order_id": "c8570489-0189-42bc-81fd-130d73ea16c4"
  }
}
```

### on_status

- Status of requested order.
- Primarily the fulfillment status is specified in message->order->fulfillments[]->state
- A payment link can be generated and sent if BPP does the payment.

```
{
  "context": {
    "domain": "energy",
    "action": "on_status",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "sheru-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "pulse-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "c8570489-0189-42bc-81fd-130d73ea16c4",
      "providers": {
        "id": "da327407-7e41-4d1d-9462-8ecb7bfccca6",
        "descriptor": {
          "name": "Pulse Energy",
          "short_desc": "Pulse Energy offers a state-of-the-art EV charging station designed for fast, efficient, and reliable electric vehicle charging. Equipped with advanced technology, our stations provide high-speed charging capabilities to get you back on the road quickly",
          "images": [
            {
              "url": "https://pulse.energy/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "2",
          "descriptor": {
            "code": "demand-note",
            "name": "Demand Note"
          },
          "price": {
            "value": "2",
            "currency": "INR/kWH"
          },
          "quantity": {
            "available": {
              "measure": {
                "value": "4",
                "unit": "kWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "state": {
            "descriptor": {
              "code": "energy-transferred",
              "name": "The Energy has been transferred"
            }
          },
          "customer": {
            "person": {
              "name": "Ramesh Dsouza"
            },
            "contact": {
              "phone": "+91-9887755555"
            }
          }
        }
      ],
      "billing": {
        "organization": {
          "descriptor": {
            "name": "Sheru Energy Ltd"
          }
        },
        "name": "Ramesh Dsouza",
        "email": "ramesh@sheru.com",
        "phone": "+91-9876555555"
      },
      "quote": {
        "price": {
          "value": "8",
          "currency": "INR"
        },
        "breakup": [
          {
            "item": {
              "descriptor": {
                "name": "Energy Demand cost"
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
              "value": "8",
              "currency": "INR"
            }
          }
        ]
      },
      "payments": [
        {
          "url": "https://payment.gateway.in",
          "type": "PRE-ORDER",
          "status": "PAID",
          "params": {
            "transaction_id": "paytxn_131414231",
            "amount": "8",
            "currency": "INR"
          },
          "time": {
            "range": {
              "start": "2023-08-10T10:00:00Z",
              "end": "2023-08-10T10:30:00Z"
            }
          }
        }
      ]
    }
  }
}
```


## Taxonomy and layer 2 configuration

TBP


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

![Seeker platform testing sandbox ](../docs/assets/images/seeker_deployment.png)

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

![Provider platform testing sandbox](../docs/assets/images/provider_deployment.png)

## Links to artefacts

TBP

## Sandbox Details

TBP