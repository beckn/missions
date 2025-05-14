# DEG Implementation Guide - Demand Flexibility

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 12-05-2025 | 1.0     | Initial Version                                     |


## Introduction

This document provides integration guidelines for implementing demand-side flexibility programs over a Beckn-enabled open energy network. It showcases how energy providers and aggregators in San Francisco can publish flexible load reduction programs, enabling users to participate in grid-responsive activities and earn monetary rewards for reducing electricity usage during peak hours.

This guide assumes the reader is familiar with the Beckn protocol, message flow, and schema standards. It maps the JSON-based catalog provided by participating BPPs to the Beckn on_search response pattern.

## Structure of the Document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualization) - A pictorial or descriptive representation of the different use cases that are supported by the network.
2. [General Flow Diagrams](#general-flow-diagrams) - This section is relevant to all the message flows illustrated below and discussed further in the document.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the messages that are sent in the form of sample schemas.
4. [Taxonomy and Layer 2 Configuration](#taxonomy-and-layer-2-configuration) - This section provides details on the taxonomy, enumerations, and any rules defined for either the use case or by the network.
5. [Integration Notes](#notes-on-software-integration) - Notes on writing/integrating with your own software.
6. [Links to Artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Registry/Gateway, and BPP.

## Outcome Visualization

### Use case - Discovery, order and fulfillment of Demand Flexibility

Emily, a tech-savvy resident of San Francisco, wants to save on her electricity bill and support the grid during peak hours.

She installs a Beckn-enabled energy app and opts into flexible energy-saving programs. She sees programs from:

- **SF Grid LoadFlex** offering $0.20/kWh for manual reductions during 5–8 PM
- **FlexiGrid Aggregator** automating her smart devices to reduce usage and pay $0.15/kWh

Emily's journey:
1. Links her Nest thermostat with FlexiGrid and accepts LoadFlex's manual opt-in alerts
2. During a Flex Alert:
   - Receives a push notification to reduce usage
   - Or lets FlexiGrid control her thermostat automatically
3. Post-event:
   - Views energy saved and incentive earned within the app dashboard

## General Flow Diagrams

Refer to the [Generic Implementation Guide - Flow Diagrams](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams).

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
    "domain": "flexibility",
    "action": "search",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2000",
    "timestamp": "2025-05-07T10:44:00Z"
  },
  "message": {
    "intent": {
      "descriptor": {
        "name": "Energy Incentives"
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
    "domain": "flexibility",
    "action": "on_search",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2001",
    "timestamp": "2025-05-07T10:45:00Z"
  },
  "message": {
    "catalog": {
      "descriptor": {
        "name": "Energy Incentives and Subsidies"
      },
      "providers": [
        {
          "id": "provider_demand_flex_001",
          "descriptor": {
            "name": "SF Grid LoadFlex",
            "short_desc": "San Francisco Demand Flexibility Programs",
            "images": [
              {
                "url": "https://sfeu.org/assets/loadflex_logo.png"
              }
            ]
          },
          "locations": [
            {
              "id": "1",
              "city": {
                "name": "San Francisco"
              }
            }
          ],
          "fulfillments": [
            {
              "id": "manual_optin",
              "type": "DIGITAL",
              "tags": [
                {
                  "descriptor": {
                    "code": "participation_mode"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "acceptance_type"
                      },
                      "value": "Manual via App Notification"
                    }
                  ]
                }
              ]
            }
          ],
          "categories": [
            {
              "id": "flex_manual",
              "descriptor": {
                "code": "MANUAL_FLEX",
                "name": "Manual Demand Response"
              }
            },
            {
              "id": "flex_auto",
              "descriptor": {
                "code": "AUTO_FLEX",
                "name": "Automated Load Management"
              }
            }
          ],
          "items": [
            {
              "id": "manual_optin_demand_flex",
              "descriptor": {
                "name": "Evening Peak Saver Manual Opt-in",
                "short_desc": "$0.20/kWh for 5-8 PM reductions (manual acceptance)",
                "long_desc": "Users will receive app notifications during grid peak hours (5–8 PM). Upon manual acceptance, they can reduce usage and earn $0.20 for every kWh saved during the window.",
                "additional_desc": {
                  "url": "https://sfeu.org/flexibility/peak-saver"
                }
              },
              "location_ids": [
                "1"
              ],
              "category_ids": [
                "flex_manual"
              ],
              "fulfillment_ids": [
                "manual_optin"
              ],
              "tags": [
                {
                  "descriptor": {
                    "code": "Flexibility_Event_Details"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "Window"
                      },
                      "value": "5:00 PM to 8:00 PM daily during alerts"
                    },
                    {
                      "descriptor": {
                        "code": "Reward"
                      },
                      "value": "$0.20 per reduced kWh"
                    },
                    {
                      "descriptor": {
                        "code": "Enrollment_Required"
                      },
                      "value": "Yes"
                    },
                    {
                      "descriptor": {
                        "code": "Acceptance_Mode"
                      },
                      "value": "Manual (via app notification)"
                    }
                  ]
                }
              ]
            }
          ]
        },
        {
          "id": "provider_flex_aggregator_001",
          "descriptor": {
            "name": "FlexiGrid Aggregator",
            "short_desc": "Third-party aggregator for flexible load management",
            "images": [
              {
                "url": "https://flexigrid.com/assets/logo.png"
              }
            ]
          },
          "locations": [
            {
              "id": "1",
              "city": {
                "name": "San Francisco"
              }
            }
          ],
          "fulfillments": [
            {
              "id": "automated_control",
              "type": "DIGITAL",
              "tags": [
                {
                  "descriptor": {
                    "code": "participation_mode"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "acceptance_type"
                      },
                      "value": "Automated via Device API"
                    }
                  ]
                }
              ]
            }
          ],
          "categories": [
            {
              "id": "flex_manual",
              "descriptor": {
                "code": "MANUAL_FLEX",
                "name": "Manual Demand Response"
              }
            },
            {
              "id": "flex_auto",
              "descriptor": {
                "code": "AUTO_FLEX",
                "name": "Automated Load Management"
              }
            }
          ],
          "items": [
            {
              "id": "aggregator_optin_flex",
              "descriptor": {
                "name": "Smart Appliance Flex Program",
                "short_desc": "$0.15/kWh reduction via smart device control",
                "long_desc": "FlexiGrid can adjust your smart appliances like thermostats or EV chargers during grid alerts. You'll earn $0.15 for each kWh avoided.",
                "additional_desc": {
                  "url": "https://flexigrid.com/programs/smart-load"
                }
              },
              "location_ids": [
                "1"
              ],
              "category_ids": [
                "flex_auto"
              ],
              "fulfillment_ids": [
                "automated_control"
              ],
              "tags": [
                {
                  "descriptor": {
                    "code": "Flexibility_Event_Details"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "Appliance_Control_Mode"
                      },
                      "value": "Automated (via device integration)"
                    },
                    {
                      "descriptor": {
                        "code": "Reward"
                      },
                      "value": "$0.15 per reduced kWh"
                    },
                    {
                      "descriptor": {
                        "code": "Compatible_Devices"
                      },
                      "value": "Nest, Tesla Wall Connector, Ecobee"
                    },
                    {
                      "descriptor": {
                        "code": "Enrollment_Required"
                      },
                      "value": "Yes, with device linking"
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

### init

**send init request**

- The draft order including billing details.
- Billing details specified in message->order->billing

```
{
  "context": {
    "domain": "deg:flexibility",
    "action": "init",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2003",
    "timestamp": "2025-05-07T10:47:00Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider_demand_flex_001"
      },
      "items": [
        {
          "id": "manual_optin_demand_flex"
        }
      ],
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Alex Morgan"
            },
            "contact": {
              "phone": "+14155552626",
              "email": "alex.morgan@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "der://ssf.meter/98765456"
              }
            }
          ]
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
    "domain": "flexibility",
    "action": "on_init",
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2003",
    "timestamp": "2025-05-07T10:47:00Z",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    }
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider_demand_flex_001",
        "descriptor": {
          "name": "SF Grid LoadFlex",
          "short_desc": "San Francisco Demand Flexibility Programs",
          "images": [
            {
              "url": "https://sfeu.org/assets/loadflex_logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "manual_optin_demand_flex",
          "descriptor": {
            "name": "Evening Peak Saver Manual Opt-in",
            "short_desc": "$0.20/kWh for 5-8 PM reductions (manual acceptance)",
            "long_desc": "Users will receive app notifications during grid peak hours (5–8 PM). Upon manual acceptance, they can reduce usage and earn $0.20 for every kWh saved during the window.",
            "additional_desc": {
              "url": "https://sfeu.org/flexibility/peak-saver"
            }
          },
          "tags": [
            {
              "descriptor": {
                "code": "Flexibility_Event_Details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "Window"
                  },
                  "value": "5:00 PM to 8:00 PM daily during alerts"
                },
                {
                  "descriptor": {
                    "code": "Reward"
                  },
                  "value": "$0.20 per reduced kWh"
                },
                {
                  "descriptor": {
                    "code": "Enrollment_Required"
                  },
                  "value": "Yes"
                },
                {
                  "descriptor": {
                    "code": "Acceptance_Mode"
                  },
                  "value": "Manual (via app notification)"
                }
              ]
            }
          ],
          "xinput": {
            "required": false,
            "head": {
              "descriptor": {
                "name": "Details Form"
              },
              "index": {
                "min": 0,
                "cur": 0,
                "max": 0
              },
              "headings": [
                "Customer Details"
              ]
            },
            "form": {
              "mime_type": "text/html",
              "url": "https://6vs8xnx5i7.scheme-finder.co.in/schems/xinput/formid/a23f2fdfbbb8ac402bfd54f",
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
          "customer": {
            "person": {
              "name": "Alex Morgan"
            },
            "contact": {
              "phone": "+14155552626",
              "email": "alex.morgan@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "der://ssf.meter/98765456"
              }
            }
          ]
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
    "domain": "flexibility",
    "action": "confirm",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2004",
    "timestamp": "2025-05-07T10:48:00Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider_demand_flex_001"
      },
      "items": [
        {
          "id": "manual_optin_demand_flex"
        }
      ],
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Alex Morgan"
            },
            "contact": {
              "phone": "+14155552626",
              "email": "alex.morgan@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "der://ssf.meter/98765456"
              }
            }
          ]
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
    "domain": "flexibility",
    "action": "on_confirm",
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2004",
    "timestamp": "2025-05-07T10:48:00Z",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    }
  },
  "message": {
    "order": {
      "id": "order-flex-001",
      "provider": {
        "id": "provider_demand_flex_001",
        "descriptor": {
          "name": "SF Grid LoadFlex",
          "short_desc": "San Francisco Demand Flexibility Programs",
          "images": [
            {
              "url": "https://sfeu.org/assets/loadflex_logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "manual_optin_demand_flex",
          "descriptor": {
            "name": "Evening Peak Saver Manual Opt-in",
            "short_desc": "$0.20/kWh for 5-8 PM reductions (manual acceptance)",
            "long_desc": "Users will receive app notifications during grid peak hours (5–8 PM). Upon manual acceptance, they can reduce usage and earn $0.20 for every kWh saved during the window.",
            "additional_desc": {
              "url": "https://sfeu.org/flexibility/peak-saver"
            }
          },
          "tags": [
            {
              "descriptor": {
                "code": "Flexibility_Event_Details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "Window"
                  },
                  "value": "5:00 PM to 8:00 PM daily during alerts"
                },
                {
                  "descriptor": {
                    "code": "Reward"
                  },
                  "value": "$0.20 per reduced kWh"
                },
                {
                  "descriptor": {
                    "code": "Enrollment_Required"
                  },
                  "value": "Yes"
                },
                {
                  "descriptor": {
                    "code": "Acceptance_Mode"
                  },
                  "value": "Manual (via app notification)"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Alex Morgan"
            },
            "contact": {
              "phone": "+14155552626",
              "email": "alex.morgan@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "der://ssf.meter/98765456"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "APPLICATION_SUBMITTED"
            }
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
    "domain": "flexibility",
    "action": "status",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2005",
    "timestamp": "2025-05-07T10:50:00Z"
  },
  "message": {
    "order_id": "order-flex-001"
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
    "domain": "flexibility",
    "action": "on_status",
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2005",
    "timestamp": "2025-05-07T10:50:00Z",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    }
  },
  "message": {
    "order": {
      "id": "order-flex-001",
      "provider": {
        "id": "provider_demand_flex_001",
        "descriptor": {
          "name": "SF Grid LoadFlex",
          "short_desc": "San Francisco Demand Flexibility Programs",
          "images": [
            {
              "url": "https://sfeu.org/assets/loadflex_logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "manual_optin_demand_flex",
          "descriptor": {
            "name": "Evening Peak Saver Manual Opt-in",
            "short_desc": "$0.20/kWh for 5-8 PM reductions (manual acceptance)",
            "long_desc": "Users will receive app notifications during grid peak hours (5–8 PM). Upon manual acceptance, they can reduce usage and earn $0.20 for every kWh saved during the window.",
            "additional_desc": {
              "url": "https://sfeu.org/flexibility/peak-saver"
            }
          },
          "tags": [
            {
              "descriptor": {
                "code": "Flexibility_Event_Details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "Window"
                  },
                  "value": "5:00 PM to 8:00 PM daily during alerts"
                },
                {
                  "descriptor": {
                    "code": "Reward"
                  },
                  "value": "$0.20 per reduced kWh"
                },
                {
                  "descriptor": {
                    "code": "Enrollment_Required"
                  },
                  "value": "Yes"
                },
                {
                  "descriptor": {
                    "code": "Acceptance_Mode"
                  },
                  "value": "Manual (via app notification)"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Alex Morgan"
            },
            "contact": {
              "phone": "+14155552626",
              "email": "alex.morgan@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "der://ssf.meter/98765456"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "APPLICATION_SUCCESSFUL"
            }
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
      "domain": "flexibility",
      "action": "support",
      "location": {
        "country": { "code": "USA" },
        "city": { "code": "NANP:628" }
      },
      "version": "1.1.0",
      "bap_id": "energy-bap-network.example.com",
      "bap_uri": "https://energy-bap-network.example.com",
      "bpp_id": "energy-incentive-bpp.example.com",
      "bpp_uri": "https://energy-incentive-bpp.example.com",
      "transaction_id": "incentive-txn-1001",
      "message_id": "incentive-msg-2009",
      "timestamp": "2025-05-07T10:54:00Z"
    },
    "message": {
      "ref_id": "order-flex-001"
    }
  }
  
```

### on_support

- Contains support information. If integrated with a CMS, the URL could contain order specific support info
- In all other cases, contains general support info (message->support)

```
{
  "context": {
    "domain": "flexibility",
    "action": "on_support",
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2009",
    "timestamp": "2025-05-07T10:54:00Z",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    }
  },
  "message": {
    "support": {
      "ref_id": "support-flex-001",
      "phone": "+14155550123",
      "email": "support@flexibility-program.com"
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
    "domain": "flexibility",
    "action": "cancel",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2006",
    "timestamp": "2025-05-07T10:51:00Z"
  },
  "message": {
    "order_id": "order-flex-001",
    "descriptor": {
      "name": "Moving Out"
    }
  }
}
```

### on_cancel

- Confirmation of cancelled order. The cancelled order will be in message->order
- In cases where advanced booking of charging station is not supported, this message is not relevant.

```
{
  "context": {
    "domain": "flexibility",
    "action": "on_cancel",
    "location": {
      "country": {
        "code": "USA"
      },
      "city": {
        "code": "NANP:628"
      }
    },
    "version": "1.1.0",
    "bap_id": "energy-bap-network.example.com",
    "bap_uri": "https://energy-bap-network.example.com",
    "bpp_id": "energy-incentive-bpp.example.com",
    "bpp_uri": "https://energy-incentive-bpp.example.com",
    "transaction_id": "incentive-txn-1001",
    "message_id": "incentive-msg-2006",
    "timestamp": "2025-05-07T10:51:00Z"
  },
  "message": {
    "order": {
      "id": "order-flex-001",
      "provider": {
        "id": "provider_demand_flex_001",
        "descriptor": {
          "name": "SF Grid LoadFlex",
          "short_desc": "San Francisco Demand Flexibility Programs",
          "images": [
            {
              "url": "https://sfeu.org/assets/loadflex_logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "manual_optin_demand_flex",
          "descriptor": {
            "name": "Evening Peak Saver Manual Opt-in",
            "short_desc": "$0.20/kWh for 5-8 PM reductions (manual acceptance)",
            "long_desc": "Users will receive app notifications during grid peak hours (5–8 PM). Upon manual acceptance, they can reduce usage and earn $0.20 for every kWh saved during the window.",
            "additional_desc": {
              "url": "https://sfeu.org/flexibility/peak-saver"
            }
          },
          "tags": [
            {
              "descriptor": {
                "code": "Flexibility_Event_Details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "Window"
                  },
                  "value": "5:00 PM to 8:00 PM daily during alerts"
                },
                {
                  "descriptor": {
                    "code": "Reward"
                  },
                  "value": "$0.20 per reduced kWh"
                },
                {
                  "descriptor": {
                    "code": "Enrollment_Required"
                  },
                  "value": "Yes"
                },
                {
                  "descriptor": {
                    "code": "Acceptance_Mode"
                  },
                  "value": "Manual (via app notification)"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "customer": {
            "person": {
              "name": "Alex Morgan"
            },
            "contact": {
              "phone": "+14155552626",
              "email": "alex.morgan@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "der://ssf.meter/98765456"
              }
            }
          ],
          "state": {
            "descriptor": {
              "code": "APPLICATION_CANCELLED"
            }
          }
        }
      ]
    }
  }
}
```

## Taxonomy and layer 2 configuration

|Property Name|Enums|
|-------------|-----|
|Demand flexibility categories|MANUAL_FLEX<br>AUTO_FLEX<br>HYBRID_FLEX|
|acceptance_type|MANUAL_OPTIN<br>AUTO_OPTIN<br>SCHEDULED|
|fulfillment_type|DIGITAL|
|fulfillment state|APPLICATION_SUCCESSFUL<br>APPLICATION_PROCESSING<br>SMS<br>APPLICATION_FAILED|
|stops types|END|
|payment-status|PENDING<br>PROCESSED<br>FAILED<br>REFUNDED|
|payment type| POST_ORDER<br>PRE_FULFILLMENT<br>POST_FULFILLMENT|
|payment collected by| BAP<br>BPP<br>AGENT
|fulfillment stop types| START<br>END

## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to artefacts

- [Postman collection for Demand Flexibility]()
- [Layer2 config for Demand Flexibility]()

## Sandbox Details

**Registry/Gateway:**

* Gateway: [gateway-deg.becknprotocol.io](gateway-deg.becknprotocol.io	)
* Registry: [https://dedi-registry.becknprotocol.io](https://dedi-registry.becknprotocol.io)

**BAP:**

* BAP Client: [bap-ps-client-deg.becknprotocol.io](bap-ps-client-deg.becknprotocol.io)
* BAP Network: [bap-ps-network-deg.becknprotocol.io](bap-ps-network-deg.becknprotocol.io)

**BPP:**

* BPP Client: [bpp-ps-client-deg.becknprotocol.io](bpp-ps-client-deg.becknprotocol.io)
* BPP Network: [bpp-ps-network-deg.becknprotocol.io	](bpp-ps-network-deg.becknprotocol.io	)