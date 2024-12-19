# UEI Implementation Guide - P2P energy trade

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 16-12-2024 | 1.0     | Initial Version                                     |

## Introduction

This document provides material that helps network participants build and integrate their application with a Beck network for Peer to Peer(P2P) energy trade. This document is part of the starter kit that provides information about the network, learning resources, network participant checklist etc. This document only focuses on the implementation of the seeker/provider platform. It assumes the reader has a good overview of the Beckn network, its APIs, the overall structure of the schema etc.

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
Peer-to-peer Energy Trading on a virtual network-based model for P2P energy trading involving the decentralised exchange of electricity between producers and consumers within a localised grid network, under the distribution coverage of DISCOMs. It allows individuals and groups to buy and sell surplus renewable energy directly to each other, bypassing traditional energy suppliers. The aim is to reduce energy costs, promote renewable energy use and enhance the security and resilience of the energy grid.

![UEI Charging Outcome Visualization](images/uei_p2p_energy_trade.png)

### Use case - Discovery, order and fulfillment of EV Charging

**User Journey**
- The prosumer (e.g., Rakesh) creates a new listing on their app for surplus energy, specifying details such as availability (15 kWh), time slot (11 am to 3 pm), and price (Rs. 3/kWh). The listing is stored on the BPP (Buyer-Provider Platform) for visibility.
- The consumer (e.g., Mr. Raj) searches for energy via their app on Day T-1. A catalog of available energy options is displayed, showing details like quantity, time slots, and price.
- The consumer selects a listing, initiates quote negotiation, and confirms (agreement) an order if terms align (e.g., up to Rs. 3.25/kWh). Orders finalized before the gate close time are stored on the Observability Layer for tracking. 
- On the transaction day (Day T), reminders are sent to both parties.Both the prosumer and consumer have the option to make last-minute adjustments (e.g., quantity or timing) up to a cutoff time (e.g. 1.5 hours before transfer).
- The energy transfer is carried out between the specified time slots (e.g., 11 am to 3 pm) and monitored via smart meters. Meter data for both prosumer and consumer is recorded periodically on the Observability Layer.
- The Allocation Engine fetches confirmed (agreed) order details and meter data from the Observability Layer after a cutoff time. Energy units are allocated to consumers, and the confirmed order records are updated in the Observability Layer accordingly.
- The BAP (Buyer Aggregator Platform) and BPP read updated order data and confirm fulfilment. Payments are settled between BAP and BPP, including platform fees and wheeling charges.
- Both users confirm the successful transaction and receive a summary of energy transferred, payment details and platform and wheeling fees paid


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

### Use case - Discovery, order and fulfillment of surplus solar energy

- This use case focuses exclusively on peer-to-peer (P2P) trading of solar energy. The trading of other energy types is currently out of scope.
- An additional component, referred to as the Observability Layer, is used to store confirmed orders (i.e., P2P energy trade contracts) between consumers and prosumers.
- Meter data for both consumers and prosumers is periodically stored in the Observability Layer. In this use case, this task is performed by the Discom; however, other entities may also carry out this responsibility when applicable.
- The records in the Observability Layer serve as the single source of truth.
- An entity called the Allocation Engine reads contracts and meter data from the Observability Layer. Based on this information, it allocates solar energy units to consumers.
- The Allocation Engine updates the order objects in the Observability Layer with the calculated allocations. Any changes in the actual allocation may also result in updates to the quote breakdown.
- The BAP (Beckn Application Platform) and BPP (Beckn Provider Platform) read the updated orders from the Observability Layer and synchronize their local copies. At this stage, the order is considered fulfilled.


**Search for surplus solar energy**

## API Calls and Schema

### search

A search request can include one or more search criteria. Below are examples of how a search request can be constructed.
A general rule is:
if searching by free text, it is specified in message->intent->descriptor->name
If searching by category, it is specified in message->intent->category->descriptor->code

**Search by the type of energy, units of energy required and the fulfillment time**


```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "solar-energy"
        }
      },
      "item": {
        "quantity": {
          "available": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      },
      "fulfillment": {
        "stops": [
          {
            "type": "end",
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
          }
        ]
      }
    }
  }
}
```

**Search by the type of energy, units of energy required, fulfillment time, and maximum purchase price**


```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "solar-energy"
        }
      },
      "item": {
        "quantity": {
          "available": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        },
        "price": {
          "maximum_value": "7",
          "currency": "INR/kWH"
        }
      },
      "fulfillment": {
        "stops": [
          {
            "type": "end",
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
          }
        ]
      }
    }
  }
}
```

**Search by the type of energy, units of energy required, fulfillment time, maximum purchase price and grid name**


```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "solar-energy"
        }
      },
      "item": {
        "quantity": {
          "available": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        },
        "price": {
          "maximum_value": "7",
          "currency": "INR/kWH"
        }
      },
      "fulfillment": {
        "agent": {
          "organization": {
            "descriptor": {
              "name": "UPPCL"
            }
          }
        },
        "stops": [
          {
            "type": "end",
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
          }
        ]
      }
    }
  }
}
```

**Search by the type of energy, units of energy required, fulfillment time, maximum purchase price, grid name and meter id**


```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "solar-energy"
        }
      },
      "item": {
        "quantity": {
          "available": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        },
        "price": {
          "maximum_value": "7",
          "currency": "INR/kWH"
        }
      },
      "fulfillment": {
        "agent": {
          "organization": {
            "descriptor": {
              "name": "UPPCL"
            }
          }
        },
        "stops": [
          {
            "type": "end",
            "id": "MTR3210",
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
          }
        ]
      }
    }
  }
}
```

**Search by the type of energy, units of energy required, fulfillment time, maximum purchase price, grid name, meter id and geographical location of the consumer**


```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "solar-energy"
        }
      },
      "item": {
        "quantity": {
          "available": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        },
        "price": {
          "maximum_value": "7",
          "currency": "INR/kWH"
        }
      },
      "fulfillment": {
        "agent": {
          "organization": {
            "descriptor": {
              "name": "UPPCL"
            }
          }
        },
        "stops": [
          {
            "type": "end",
            "id": "MTR3210",
            "location": {
              "circle": {
                "gps": "12.423423,77.325647",
                "radius": {
                  "type": "CONSTANT",
                  "value": "5",
                  "unit": "km"
                }
              }
            },
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
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
- Each item is the catalog listing solar energy.

```
{
    "context": {
        "domain": "uei:p2p-trading",
        "action": "on_search",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "example-bap.com",
        "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
        "bpp_id": "example-bpp.com",
        "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "provider1",
                    "descriptor": {
                        "name": "Mukesh Shankar",
                        "short_desc": "Household rooftop solar energy",
                        "images": [
                            {
                                "url": "https://provider1.in/images/logo.png"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "1",
                            "descriptor": {
                                "code": "solar-energy",
                                "name": "solar energy"
                            }
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "gps": "12.345345,77.389754"
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "1",
                            "agent": {
                                "organization": {
                                    "descriptor": {
                                        "name": "UPPCL"
                                    }
                                }
                            },
                            "stops": [
                                {
                                    "type": "start",
                                    "id": "MTR0123",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                },
                                {
                                    "type": "end",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                }
                            ]
                        }
                    ],
                    "items": [
                        {
                            "id": "uid_xyz",
                            "price": {
                                "value": "7",
                                "currency": "INR/kWH"
                            },
                            "quantity": {
                                "available": {
                                    "measure": {
                                        "value": "30",
                                        "unit": "kWH"
                                    }
                                }
                            },
                            "category_ids": [
                                "1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "1"
                            ]
                        }
                    ]
                },
                {
                    "id": "provider2",
                    "descriptor": {
                        "name": "ABC Solar farms",
                        "short_desc": "Utility scale solar farm",
                        "images": [
                            {
                                "url": "https://provider2.in/images/logo.png"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "1",
                            "descriptor": {
                                "code": "solar-energy",
                                "name": "solar energy"
                            }
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "gps": "12.297668,77.276467"
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "1",
                            "agent": {
                                "organization": {
                                    "descriptor": {
                                        "name": "UPPCL"
                                    }
                                }
                            },
                            "stops": [
                                {
                                    "type": "start",
                                    "id": "MTR0124",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                },
                                {
                                    "type": "end",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                }
                            ]
                        }
                    ],
                    "items": [
                        {
                            "id": "uid_abc",
                            "price": {
                                "value": "7",
                                "currency": "INR/kWH"
                            },
                            "quantity": {
                                "available": {
                                    "measure": {
                                        "value": "25000",
                                        "unit": "kWH"
                                    }
                                }
                            },
                            "category_ids": [
                                "1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "1"
                            ]
                        }
                    ]
                },
                {
                    "id": "provider3",
                    "descriptor": {
                        "name": "Sharda Desai",
                        "short_desc": "Rooftop solar energy",
                        "images": [
                            {
                                "url": "https://provider3.in/images/logo.png"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "1",
                            "descriptor": {
                                "code": "solar-energy",
                                "name": "solar energy"
                            }
                        }
                    ],
                    "locations": [
                        {
                            "id": "1",
                            "gps": "12.187363,77.137474"
                        }
                    ],
                    "fulfillments": [
                        {
                            "id": "1",
                            "agent": {
                                "organization": {
                                    "descriptor": {
                                        "name": "UPPCL"
                                    }
                                }
                            },
                            "stops": [
                                {
                                    "type": "start",
                                    "id": "MTR0125",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                },
                                {
                                    "type": "end",
                                    "time": {
                                        "range": {
                                            "start": "2024-10-04T10:00:00",
                                            "end": "2024-10-04T18:00:00"
                                        }
                                    }
                                }
                            ]
                        }
                    ],
                    "items": [
                        {
                            "id": "uid_mnq",
                            "price": {
                                "value": "7.3",
                                "currency": "INR/kWH"
                            },
                            "quantity": {
                                "available": {
                                    "measure": {
                                        "value": "15",
                                        "unit": "kWH"
                                    }
                                }
                            },
                            "category_ids": [
                                "1"
                            ],
                            "location_ids": [
                                "1"
                            ],
                            "fulfillment_ids": [
                                "1"
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

The examples from this stage onwards will be based on a scenario where consumer chooses item with `"id": "uid_xyz"` from the on_search response.

**sending a select request**

- Choose the item(s) from the list from the on_search response and request a quote
- The chosen item is in message->order->item_id

```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "select",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider1"
      },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "selected": {
              "measure": {
                "value": "10",
                "unit": "KWH"
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
                    "type": "end",
                    "id":"MTR3210",
                    "time": {
                        "range": {
                            "start": "2024-10-04T10:00:00",
                            "end": "2024-10-04T18:00:00"
                        }
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
    "domain": "uei:p2p-trading",
    "action": "on_select",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider1",
        "descriptor": {
          "name": "Mukesh Shankar",
          "short_desc": "Household rooftop solar energy",
          "images": [
            {
              "url": "https://provider1.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "uid_xyz",
          "price": {
            "value": "5",
            "currency": "INR/kWH"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "10",
                "unit": "KWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "agent": {
            "organization": {
              "descriptor": {
                "name": "UPPCL"
              }
            }
          },
          "stops": [
            {
              "type": "start",
              "id": "MTR0123",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            },
            {
              "type": "end",
              "id": "MTR3210",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "53.5",
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
                    "value": "10",
                    "unit": "kWh"
                  }
                }
              },
              "price": {
                "value": "5",
                "currency": "INR/kWH"
              }
            },
            "price": {
              "value": "50",
              "currency": "INR"
            }
          },
          {
            "title": "wheeling charge",
            "price": {
              "value": "2.5",
              "currency": "INR"
            }
          },
          {
            "title": "platform charge",
            "price": {
              "value": "1",
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
    "domain": "uei:p2p-trading",
    "action": "init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider1"
      },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "selected": {
              "measure": {
                "value": "10",
                "unit": "KWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "customer": {
            "person": {
              "name": "Raj"
            },
            "contact": {
              "phone": "+91-1276522222"
            }
          },
          "stops": [
            {
              "type": "end",
              "id": "MTR3210",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Raj",
        "email": "raj@example.com",
        "phone": "+91-1276522222"
      }
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
    "domain": "uei:p2p-trading",
    "action": "on_init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider1",
        "descriptor": {
          "name": "Mukesh Shankar",
          "short_desc": "Household rooftop solar energy",
          "images": [
            {
              "url": "https://provider1.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "uid_xyz",
          "price": {
            "value": "5",
            "currency": "INR/kWH"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "10",
                "unit": "KWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "agent": {
            "organization": {
              "descriptor": {
                "name": "UPPCL"
              }
            }
          },
          "customer": {
            "person": {
              "name": "Raj"
            },
            "contact": {
              "phone": "+91-1276522222"
            }
          },
          "stops": [
            {
              "type": "start",
              "id": "MTR0123",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            },
            {
              "type": "end",
              "id": "MTR3210",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            }
          ]
        }
      ],
      "quote": {
        "price": {
          "value": "53.5",
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
                    "value": "10",
                    "unit": "kWh"
                  }
                }
              },
              "price": {
                "value": "5",
                "currency": "INR/kWH"
              }
            },
            "price": {
              "value": "50",
              "currency": "INR"
            }
          },
          {
            "title": "wheeling charge",
            "price": {
              "value": "2.5",
              "currency": "INR"
            }
          },
          {
            "title": "platform charge",
            "price": {
              "value": "1",
              "currency": "INR"
            }
          }
        ]
      },
      "billing": {
        "name": "Raj",
        "email": "raj@example.com",
        "phone": "+91-1276522222"
      },
      "payments": [
        {
          "type": "POST-FULFILLMENT",
          "status": "NOT-PAID",
          "params": {
            "amount": "53.50",
            "currency": "INR"
          }
        }
      ],
      "cancellation_terms": [
        {
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://mvvnl.in/cancellation_terms.html"
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
    "domain": "uei:p2p-trading",
    "action": "confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider1"
      },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "selected": {
              "measure": {
                "value": "10",
                "unit": "KWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "customer": {
            "person": {
              "name": "Raj"
            },
            "contact": {
              "phone": "+91-1276522222"
            }
          },
          "stops": [
            {
              "type": "end",
              "id": "MTR3210",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "Raj",
        "email": "raj@example.com",
        "phone": "+91-1276522222"
      },
      "payments": [
        {
          "type": "POST-FULFILLMENT",
          "status": "NOT-PAID",
          "params": {
            "amount": "53.50",
            "currency": "INR"
          }
        }
      ],
      "cancellation_terms": [
        {
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://mvvnl.in/cancellation_terms.html"
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
    "domain": "uei:p2p-trading",
    "action": "on_confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "provider1",
        "descriptor": {
          "name": "Mukesh Shankar",
          "short_desc": "Household rooftop solar energy",
          "images": [
            {
              "url": "https://provider1.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "uid_xyz",
          "price": {
            "value": "5",
            "currency": "INR/kWH"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "10",
                "unit": "KWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "agent": {
            "organization": {
              "descriptor": {
                "name": "UPPCL"
              }
            }
          },
          "customer": {
            "person": {
              "name": "Raj"
            },
            "contact": {
              "phone": "+91-1276522222"
            }
          },
          "stops": [
            {
              "type": "start",
              "id": "MTR0123",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            },
            {
              "type": "end",
              "id": "MTR3210",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "Order-Placed"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "53.5",
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
                    "value": "10",
                    "unit": "kWh"
                  }
                }
              },
              "price": {
                "value": "5",
                "currency": "INR/kWH"
              }
            },
            "price": {
              "value": "50",
              "currency": "INR"
            }
          },
          {
            "title": "wheeling charge",
            "price": {
              "value": "2.5",
              "currency": "INR"
            }
          },
          {
            "title": "platform charge",
            "price": {
              "value": "1",
              "currency": "INR"
            }
          }
        ]
      },
      "billing": {
        "name": "Raj",
        "email": "raj@example.com",
        "phone": "+91-1276522222"
      },
      "payments": [
        {
          "type": "POST-FULFILLMENT",
          "status": "NOT-PAID",
          "params": {
            "amount": "53.50",
            "currency": "INR"
          }
        }
      ],
      "cancellation_terms": [
        {
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://mvvnl.in/cancellation_terms.html"
          }
        }
      ],
      "Created_at": "2024-10-03T14:30:00"
    }
  }
}
```

### Record a confirmed order on Observability Layer.

- BAP and BPP record a new Order on the Observability Layer.
- Observability Layers allows recoding a new Order object between a fixed window only. In this use case it is between the trade start time and a gate closer time.
- Observability Layer validates the BAP and BPP writing on the Observability Layer with their public key stored in the network registry.
- Optionally: A new Order object recorded on the Observability Layer can be signed by both BAP and BPP involved in that contract. Implementation can use http Authorization header to store BAP and BPP signatures. Signing will ensure the authenticity and integrity of the contract gettig saved on the Observability Layer. We can refer [this](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-006-Signing-Beckn-APIs-In-HTTP-Draft-01.md) document to understand how to use it.

**How to encode signatures of BAP and BPP both in the Authorization header**

 Authorization header in the request sent ot Observability Layer can contain 2 signatures.
 - BPP returns an on_confirm with BPP's signature in the header.
 - BAP takes the returned Order object and computes its own signature.
 - BAP creates an /Order request for Observability layer.
 - BAP adds BPP's signature in the new request.
 - BAP adds its own signature in the new request.
 - BAP sends the request to the Observability Layer.

 How does the header look

```
Authorization : Signature keyId="{BAP_subscriber_id}|{BAP_unique_key_id}|{algorithm}",algorithm="ed25519",created="1606970629",expires="1607030629",headers="(created) (expires) digest",signature="Base64(ed25519_sign_of_bap(signing string))"

keyId="{BPP_subscriber_id}|{BPP_unique_key_id}|{algorithm}",algorithm="ed25519",created="1606970629",expires="1607030629",headers="(created) (expires) digest",signature="Base64(ed25519_sign_of_bpp(signing string))
```

How does the Observability Layer verify the integrity of the message
- Observability Layer parses BAP's subscriber id and unique Key ID from the header.
- It looks up the registry for the public key of the BAP by sending the subscriber_id and the unique_key_id
- It uses the BAP's public key to decrypt the BAP signature, parses the decrypted signature to get (created), (expires) and digest fields. it calculates the hash of the order payload from the /order request. It compares the decrypted (created), (expires) and digest with the value in the request header and the locally calculated digest.
- Observability Layer repeats the same process with the BPP's signature.

**Use Observability Layer PUT order/ API to store a confirmed Order object in the Observability Layer**

```
  {
    "id": "6743e9e2-4fb5-487c-92b7",
    "provider": {
      "id": "provider1",
      "descriptor": {
        "name": "Mukesh Shankar",
        "short_desc": "Household rooftop solar energy",
        "images": [
          {
            "url": "https://provider1.in/images/logo.png"
          }
        ]
      }
    },
    "items": [
      {
        "id": "uid_xyz",
        "price": {
          "value": "5",
          "currency": "INR/kWH"
        },
        "quantity": {
          "selected": {
            "measure": {
              "value": "10",
              "unit": "KWH"
            }
          }
        }
      }
    ],
    "fulfillments": [
      {
        "id": "1",
        "agent": {
          "organization": {
            "descriptor": {
              "name": "UPPCL"
            }
          }
        },
        "customer": {
          "person": {
            "name": "Raj"
          },
          "contact": {
            "phone": "+91-1276522222"
          }
        },
        "stops": [
          {
            "type": "start",
            "id": "MTR0123",
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
          },
          {
            "type": "end",
            "id": "MTR3210",
            "time": {
              "range": {
                "start": "2024-10-04T10:00:00",
                "end": "2024-10-04T18:00:00"
              }
            }
          }
        ],
        "state": {
          "descriptor": {
            "code": "Order-Placed"
          }
        }
      }
    ],
    "quote": {
      "price": {
        "value": "53.5",
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
                  "value": "10",
                  "unit": "kWh"
                }
              }
            },
            "price": {
              "value": "5",
              "currency": "INR/kWH"
            }
          },
          "price": {
            "value": "50",
            "currency": "INR"
          }
        },
        {
          "title": "wheeling charge",
          "price": {
            "value": "2.5",
            "currency": "INR"
          }
        },
        {
          "title": "platform charge",
          "price": {
            "value": "1",
            "currency": "INR"
          }
        }
      ]
    },
    "billing": {
      "name": "Raj",
      "email": "raj@example.com",
      "phone": "+91-1276522222"
    },
    "payments": [
      {
        "type": "POST-FULFILLMENT",
        "status": "NOT-PAID",
        "params": {
          "amount": "53.50",
          "currency": "INR"
        }
      }
    ],
    "cancellation_terms": [
      {
        "external_ref": {
          "mimetype": "text/html",
          "url": "https://mvvnl.in/cancellation_terms.html"
        }
      }
    ],
    "Created_at": "2024-10-03T14:30:00"
  }
```

**Use Observability Layer PUT fulfillment_event/ API to store time series meter data in the Observability Layer**

```
{
   "items": [
       {
           "Descriptor": {
               "name": "Active Energy"
           },
           "quantity": {
               "utilized": {
                   "measure": {
                       "value": "2",
                       "unit": "kWh"
                   }
               }
           },
           "fulfillment_ids": [
               "f1"
           ]
       },
       {
           "Descriptor": {
               "name": "Active Energy"
           },
           "quantity": {
               "utilized": {
                   "measure": {
                       "value": "3",
                       "unit": "kWh"
                   }
               }
           },
           "fulfillment_ids": [
               "f2"
           ]
       },
       {
           "Descriptor": {
               "name": "Apparent Energy"
           },
           "quantity": {
               "utilized": {
                   "measure": {
                       "value": "1",
                       "unit": "kVAh"
                   }
               }
           },
           "fulfillment_ids": [
               "f1"
           ]
       },
       {
           "Descriptor": {
               "name": "Apparent Energy"
           },
           "quantity": {
               "utilized": {
                   "measure": {
                       "value": "2",
                       "unit": "kVAh"
                   }
               }
           },
           "fulfillment_ids": [
               "f2"
           ]
       }
   ],
   "fulfillments": [
       {
           "id": "f1",
           "type": "EnergyImported",
           "stops": [
               {
                   "id": "MTR1",
                   "type": "3-Ph",
                   "time": {
                       "range": {
                           "start": "2014-10-04T10:00:00",
                           "end": "2014-10-04T11:00:00"
                       }
                   }
               }
           ]
       },
       {
           "id": "f2",
           "type": "EnergyExported",
           "stops": [
               {
                   "id": "MTR1",
                   "type": "3-Ph",
                   "time": {
                       "range": {
                           "start": "2014-10-04T10:00:00",
                           "end": "2014-10-04T11:00:00"
                       }
                   }
               }
           ]
       }
   ],
   "tags": [
    {
      "list": [
        {
          "descriptor": {
            "code": "avg_current"
          },
          "value": "1.05"
        },
        {
          "descriptor": {
            "code": "avg_voltage"
          },
          "value": "260.5"
        } 
      ]
    }
  ],
  "created_at": "2014-10-04T11:00:00",
}
```

### Allocation Engine fetches Order details and Meter data from the Observability Layer to calculate the allocation of energy units to consumers.

**Use Observability Layer GET order/ API to fetch Order data**

```
GET /order?order_id=order0123 HTTP/1.1
Host: api.example.com
Authorization: Signature keyId="example-bap.com|ae3ea24b-cfec-495e-81f8-044aaef164ac|ed25519",algorithm="ed25519",created="1641287875",expires="1641291475",headers="(created) (expires) digest",signature="cjbhP0PFyrlSCNszJM1F/YmHDVAWsZqJUPzojnE/7TJU3fJ/rmIlgaUHEr5E0/2PIyf0tpSnWtT6cyNNlpmoAQ=="
```

**Use Observability Layer GET fulfillment_events/ API to fetch meter data**

```
{
   "fulfillments": [
       {
           "stops": [
               {
                   "id": "MTR1",
                   "time": {
                       "range": {
                           "start": "2014-10-04T10:00:00",
                           "end": "2014-10-04T12:00:00"
                       }
                   }
               },
           ]
       }
   ]
}
```

**Use Observability Layer PATCH order/ API to update the details of an existing order object**

```
{
  "id": "6743e9e2-4fb5-487c-92b7",
  "items": [
    {
      "id": "uid_xyz",
      "price": {
        "value": "5",
        "currency": "INR/kWH"
      },
      "quantity": {
        "selected": {
          "measure": {
            "value": "8",
            "unit": "kWh"
          }
        }
      }
    }
  ],
  "quote": {
    "price": {
      "value": "43.5",
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
                "value": "8",
                "unit": "kWh"
              }
            }
          },
          "price": {
            "value": "5",
            "currency": "INR/kWH"
          }
        },
        "price": {
          "value": "40",
          "currency": "INR"
        }
      },
      {
        "title": "wheeling charge",
        "price": {
          "value": "2.5",
          "currency": "INR"
        }
      },
      {
        "title": "transaction charge",
        "price": {
          "value": "1",
          "currency": "INR"
        }
      }
    ]
  },
  "payments": [
    {
      "type": "POST-FULFILLMENT",
      "status": "NOT-PAID",
      "params": {
        "amount": "43.50",
        "currency": "INR"
      }
    }
  ]
}
```

BPP can use the GET order/ API endpoint of the Observability Layer to fetch the updated Order objects and update their local copy of the Orders.

### status

- BAP checks the status of the Order by calling the /status endpoint of BPP.
- Request for status of an active order. Order_id is used to fetch the order status. order_id is specified in message->order_id field of the status request.

```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "status",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-energy-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
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
- A payment link can be generated and sent if BPP does the payment.

```
{
  "context": {
    "domain": "uei:p2p-trading",
    "action": "on_status",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://api.example-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "provider1",
        "descriptor": {
          "name": "Mukesh Shankar",
          "short_desc": "Household rooftop solar energy",
          "images": [
            {
              "url": "https://provider1.in/images/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "uid_xyz",
          "price": {
            "value": "5",
            "currency": "INR/kWH"
          },
          "quantity": {
            "selected": {
              "measure": {
                "value": "8",
                "unit": "KWH"
              }
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "agent": {
            "organization": {
              "descriptor": {
                "name": "UPPCL"
              }
            }
          },
          "customer": {
            "person": {
              "name": "Raj"
            },
            "contact": {
              "phone": "+91-1276522222"
            }
          },
          "stops": [
            {
              "type": "start",
              "id": "MTR0123",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            },
            {
              "type": "end",
              "id": "MTR3210",
              "time": {
                "range": {
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "Order-fulfilled"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "43.5",
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
                    "value": "8",
                    "unit": "kWh"
                  }
                }
              },
              "price": {
                "value": "5",
                "currency": "INR/kWH"
              }
            },
            "price": {
              "value": "40",
              "currency": "INR"
            }
          },
          {
            "title": "wheeling charge",
            "price": {
              "value": "2.5",
              "currency": "INR"
            }
          },
          {
            "title": "transaction charge",
            "price": {
              "value": "1",
              "currency": "INR"
            }
          }
        ]
      },
      "billing": {
        "name": "Raj",
        "email": "raj@example.com",
        "phone": "+91-1276522222"
      },
      "payments": [
        {
          "url": "https://paymentgateway.example.com/pay?merchant_id=12345&amount=100.00&currency=INR&order_id=67890&customer_email=user@example.com&callback_url=https://example.com/payment-status",
          "type": "POST-FULFILLMENT",
          "status": "NOT-PAID",
          "params": {
            "amount": "53.50",
            "currency": "INR"
          }
        }
      ],
      "cancellation_terms": [
        {
          "external_ref": {
            "mimetype": "text/html",
            "url": "https://mvvnl.in/cancellation_terms.html"
          }
        }
      ],
      "created_at": "2024-10-03T14:30:00",
      "updated_at": "2014-10-04T12:00:00"
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

