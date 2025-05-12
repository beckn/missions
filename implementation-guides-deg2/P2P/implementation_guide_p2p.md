# P2P Energy Trading Implementation Guide

#### Version 1.0

## Version History

| Date       | Version | Description                    |
| ---------- | ------- | ------------------------------ |
| 13-05-2025 | 1.0     | Initial version for P2P energy |

## Introduction

This guide provides implementation instructions for a peer-to-peer (P2P) energy trading use case using the Beckn Protocol. It helps developers and integrators build Beckn-compliant BAPs and BPPs for decentralized energy trading between prosumers and consumers via smart grid infrastructure.

## Structure of the document

1. [Outcome Visualization](#outcome-visualization)
2. [API Calls and Schema](#api-calls-and-schema)
3. [Update & Order Management](#update--order-management)
4. [Taxonomy and Configuration](#taxonomy-and-configuration)
5. [Integration Notes](#integration-notes)
6. [Sandbox Access](#sandbox-access)

## Outcome Visualization

### Use Case - Peer-to-Peer Solar Energy Trade

**Scenario**: Ethan, a solar prosumer in San Francisco, has a surplus of 30kWh energy from 10 AM to 6 PM. John, a neighbor, wishes to purchase 10kWh.

1. **Discovery**: John uses the EnergyTrade app to discover local surplus offers.
2. **Selection**: He selects Ethan’s offer and specifies his required quantity.
3. **Order**: John confirms the order, provides payment, and receives transfer details.
4. **Fulfillment**: PG\&E Grid facilitates the actual transfer between DER meters.
5. **Post-Trade**: Both parties receive confirmation and transaction closure.

## API Calls and Schema

### search

Search request is used by the consumer to discover available energy offers matching location, time, and quantity.

* `intent.descriptor.name`: specifies what energy item is being searched.
* `fulfillment.stops[0].location`: indicates where the energy is needed.

````json
{
  "context": {
    "domain": "trade",
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
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "search-message-001",
    "timestamp": "2025-05-09T10:00:00Z"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {
          "name": "Solar Surplus Energy"
        }
      },
      "fulfillment": {
        "agent": {
          "organization": {
            "descriptor": {
              "name": "PG&E Grid Services"
            }
          }
        },
        "stops": [
          {
            "type": "end",
            "location": {
              "address": "der://ssf.meter/98765456"
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
````
### on_search
on_search returns matched energy catalogs from registered providers. Use the following fields:
- Energy offers and their details are found in `catalog.providers[].items[]`.
- Each offer has price, quantity, source type, and certification tags.

```json
{
  "context": {
    "domain": "trade",
    "action": "on_search",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0",
    "bap_id": "p2pTrading-bap.com",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "on_search-message-001",
    "timestamp": "2025-05-09T10:00:00Z"
  },
  "message": {
    "catalog": {
      "providers": [
        {
          "id": "p1072",
          "descriptor": {
            "name": "Ethan Maxwell",
            "images": [
              { "url": "https://p1072.in/images/logo.png" }
            ]
          },
          "items": [
            {
              "id": "uid_xyz",
              "descriptor": {
                "code": "energy",
                "name": "Solar Surplus Energy",
                "short_desc": "30 kWh surplus available between 10AM-6PM"
              },
              "price": {
                "value": "5",
                "currency": "USD/kWH"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "30",
                    "unit": "kWH"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": { "code": "Energy_Attributes" },
                  "list": [
                    {
                      "descriptor": { "code": "Source_Type" },
                      "value": "Rooftop Solar"
                    },
                    {
                      "descriptor": { "code": "Carbon_Offset_Certified" },
                      "value": "Yes"
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
````

### select
Used to select a specific energy offer and specify required kWh.

```json
{
  "context": {
    "domain": "trade",
    "action": "select",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "select-message-001",
    "timestamp": "2025-05-09T10:01:00Z",
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
        "id": "p1072"
      },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      ],
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          }
        }
      ]
    }
  }
}
````

### on_select
Confirms availability, pricing, and any certifications (e.g., carbon offset, ownership certificates).

```json
{
  "context": {
    "domain": "trade",
    "action": "on_select",
    "bap_id": "p2pTrading-bap.com",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "on_select-message-001",
    "timestamp": "2025-05-09T10:01:15Z",
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
        "id": "p1072"
      },
      "items": [
        {
          "id": "uid_xyz",
          "price": {
            "value": "5",
            "currency": "USD/kWH"
          },
          "quantity": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "50",
          "currency": "USD"
        },
        "breakup": [
          {
            "title": "Energy Cost",
            "price": {
              "value": "50",
              "currency": "USD"
            }
          }
        ]
      },
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          }
        }
      ]
    }
  }
}
````

### init
Initiates the order with consumer billing and meter location.

```json
{
  "context": {
    "domain": "trade",
    "action": "init",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "init-message-001",
    "timestamp": "2025-05-09T10:02:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order": {
      "provider": { "id": "p1072" },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      ],
      "billing": {
        "name": "John Miller",
        "address": "123 Bay St, SF, CA",
        "phone": "+14151112222",
        "email": "john.miller@example.com"
      },
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          }
        }
      ]
    }
  }
}
````

### on_init
Returns final quote, document links, payment instructions, and any regulatory tags.

```json
{
  "context": {
    "domain": "trade",
    "action": "on_init",
    "bap_id": "p2pTrading-bap.com",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "on_init-message-001",
    "timestamp": "2025-05-09T10:02:30Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order": {
      "provider": { "id": "p1072" },
      "items": [
        {
          "id": "uid_xyz",
          "price": {
            "value": "5",
            "currency": "USD/kWH"
          },
          "quantity": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "value": "50",
          "currency": "USD"
        },
        "breakup": [
          {
            "title": "Energy Cost",
            "price": {
              "value": "50",
              "currency": "USD"
            }
          }
        ]
      },
      "billing": {
        "name": "John Miller",
        "address": "123 Bay St, SF, CA",
        "phone": "+14151112222",
        "email": "john.miller@example.com"
      },
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          },
          "customer": {
            "person": {
              "name": "John Miller"
            },
            "contact": {
              "phone": "+14151112222",
              "email": "john.miller@example.com"
            }
          }
        }
      ]
    }
  }
}
````

### confirm
Consumer confirms order and payment details. Includes energy quantity and meter stop locations.

```json
{
  "context": {
    "domain": "trade",
    "action": "confirm",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "confirm-message-001",
    "timestamp": "2025-05-09T10:03:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order": {
      "provider": { "id": "p1072" },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      ],
      "billing": {
        "name": "John Miller",
        "address": "123 Bay St, SF, CA",
        "phone": "+14151112222",
        "email": "john.miller@example.com"
      },
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          }
        }
      ]
    }
  }
}
````

### on_confirm
Confirms the order with final provider metadata, schedule, fulfillment stops, and order ID.

```json
{
  "context": {
    "domain": "trade",
    "action": "on_confirm",
    "bap_id": "p2pTrading-bap.com",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "on_confirm-message-001",
    "timestamp": "2025-05-09T10:03:30Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order": {
      "id": "p2p-order-001",
      "provider": { "id": "p1072" },
      "items": [
        {
          "id": "uid_xyz",
          "quantity": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
        }
      ],
      "billing": {
        "name": "John Miller",
        "address": "123 Bay St, SF, CA",
        "phone": "+14151112222",
        "email": "john.miller@example.com"
      },
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          },
          "customer": {
            "person": {
              "name": "John Miller"
            },
            "contact": {
              "phone": "+14151112222",
              "email": "john.miller@example.com"
            }
          }
        }
      ]
    }
  }
}
````

### status
Used to fetch the current status of an order.

```json
{
  "context": {
    "domain": "trade",
    "action": "status",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "status-message-001",
    "timestamp": "2025-05-09T10:04:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order_id": "p2p-order-001"
  }
}
````

### on\_status

Returns the current status and updated fulfillment info.

````json
{
  "context": {
    "domain": "trade",
    "action": "on_status",
    "bap_id": "p2pTrading-bap.com",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "on_status-message-001",
    "timestamp": "2025-05-09T10:04:15Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order": {
      "id": "p2p-order-001",
      "state": "ACTIVE",
      "fulfillments": [
        {
          "state": {
            "descriptor": {
              "code": "In-Progress",
              "name": "Energy transfer ongoing"
            }
          },
          "end": {
            "location": {
              "address": "der://ssf.meter/65432100"
            }
          }
        }
      ]
    }
  }
}
````


## Update & Order Management

### update
Used to make changes to fulfillment (time slots, end meter, etc.) or to resend details in case of payment retries.

```json
{
  "context": {
    "domain": "trade",
    "action": "update",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "update-message-001",
    "timestamp": "2025-05-09T10:05:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "update_target": "fulfillments",
    "order": {
      "id": "p2p-order-001",
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/000NEWDEST"
            }
          }
        }
      ]
    }
  }
}
````
### on_update
Responds with updated order details or error message. Confirms that change has been acknowledged by provider.

```json
{
  "context": {
    "domain": "trade",
    "action": "on_update",
    "bap_id": "p2pTrading-bap.com",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "on_update-message-001",
    "timestamp": "2025-05-09T10:05:30Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    }
  },
  "message": {
    "order": {
      "id": "p2p-order-001",
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "der://ssf.meter/000NEWDEST"
            }
          }
        }
      ]
    }
  }
}
````

## Taxonomy and Configuration

- `source_type`: Rooftop Solar / Wind / Hydro
- `certifications`: SPOC, P2PTL
- `fulfillment.agent`: Grid operator (e.g., PG&E)
- `location.address`: DER URI format (e.g., der://meter-id)

## Integration Notes

- Each participant must be registered with gateway and registry.
- DER endpoints should support standard Beckn fulfillment interfaces.
- Grid operator must support two-way transfer event logging.

## Sandbox Access

**Registry/Gateway:**
- Gateway: [gateway.becknprotocol.io/bg](https://gateway.becknprotocol.io/bg)
- Registry: [registry.becknprotocol.io](https://registry.becknprotocol.io)

**BAP:**
- EnergyTrade Client: [bap-p2p.dev.becknprotocol.io](https://bap-p2p.dev.becknprotocol.io)

**BPP:**
- SolarProsumer Node: [bpp-p2p.dev.becknprotocol.io](https://bpp-p2p.dev.becknprotocol.io)


