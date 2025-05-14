# Battery Rental Implementation Guide

## Version History

| Date       | Version | Description                        |
| ---------- | ------- | ---------------------------------- |
| 13-05-2025 | 1.0     | Initial version for battery rental |


## Introduction

This guide outlines the end-to-end implementation for a battery rental use case using the Beckn Protocol. It is intended for developers and system integrators creating Beckn-compliant BAP and BPP systems for renting energy storage devices.

## Structure of the Document

This document has the following parts:

1. [Outcome Visualization](#outcome-visualization) - A pictorial or descriptive representation of the different use cases that are supported by the network.
2. [General Flow Diagrams](#general-flow-diagrams) - This section is relevant to all the message flows illustrated below and discussed further in the document.
3. [API Calls and Schema](#api-calls-and-schema) - This section provides details on the API calls and the schema of the messages that are sent in the form of sample schemas.
4. [Taxonomy and Layer 2 Configuration](#taxonomy-and-layer2-configuration) - This section provides details on the taxonomy, enumerations, and any rules defined for either the use case or by the network.
5. [Integration Notes](#notes-on-software-integration) - Notes on writing/integrating with your own software.
6. [Links to Artefacts](#links-to-artefacts) - This section contains the downloadable files referenced in this document.
7. [Sandbox Details](#sandbox-details) - Sandbox links to BAP, Registry/Gateway, and BPP.


## Outcome Visualization

### Use Case - Battery Rental for Residential Backup

**Scenario**: Jane Doe wants to rent a STARMAX 6-Volt Deep Cycle Battery for solar backup. She selects the product through Energykart, confirms rental dates, and opts for home delivery.

1. **Search**: Jane searches for batteries available for rental in San Francisco.
2. **Discovery**: Energykart shows multiple battery products from vendors like SF Energy Depot.
3. **Selection**: Jane selects the STARMAX battery with a 60-day rental period.
4. **Order**: Billing and delivery details are confirmed.
5. **Fulfillment**: The battery is delivered and picked up at the end of the term.
6. **Post-Rental**: Status is updated, and return policies apply.

## General Flow Diagrams

Refer to the [Generic Implementation Guide - Flow Diagrams](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams).

## API Calls and Schema

### search
Search is used to discover available battery rental options.
- `intent.descriptor.name`: battery product or type.
- `fulfillment.stops[0].location`: location of delivery or pickup.

```json
{
  "context": {
    "domain": "rental",
    "action": "search",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-search-3001",
    "timestamp": "2025-05-12T09:00:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": { "name": "Battery" }
      },
      "fulfillment": {
        "stops": [
          {
            "type": "end",
            "location": {
              "address": "456 Battery St, San Francisco, CA"
            },
            "time": {
              "range": {
                "start": "2025-05-14T09:00:00Z",
                "end": "2025-07-13T17:00:00Z"
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
Responds with available battery rental options from different providers.
- Product options listed under `catalog.providers[].items[]`.
- Pricing, quantity, rental terms and images are included.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_search",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-onsearch-3001",
    "timestamp": "2025-05-12T09:00:03Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "catalog": {
      "providers": [
        {
          "id": "sfenergystore",
          "descriptor": {
            "name": "SF Energy Depot",
            "images": [ { "url": "https://sfenergystore.com/images/battery1.png" } ]
          },
          "items": [
            {
              "id": "starmax-6v",
              "descriptor": {
                "name": "STARMAX 6-Volt Deep Cycle Battery",
                "short_desc": "Ideal for solar backup."
              },
              "price": {
                "currency": "USD",
                "value": "50"
              },
              "quantity": {
                "available": {
                  "measure": {
                    "value": "10",
                    "unit": "units"
                  }
                }
              },
              "tags": [
                {
                  "descriptor": { "code": "rental_terms" },
                  "list": [
                    {
                      "descriptor": { "code": "duration" },
                      "value": "60 days"
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

Used to choose a specific battery rental product and indicate quantity or rental preferences.

* The item is chosen using `order.items.id`.
* Delivery preferences go in `fulfillments.stops[].location`.

````json
{
  "context": {
    "domain": "rental",
    "action": "select",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-select-3001",
    "timestamp": "2025-05-12T09:01:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "provider": { "id": "sfenergystore" },
      "items": [
        {
          "id": "starmax-6v",
          "quantity": {
            "measure": {
              "value": 1,
              "unit": "unit"
            }
          }
        }
      ],
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "456 Battery St, San Francisco, CA"
            }
          }
        }
      ]
    }
  }
}
````

### on_select
Returns confirmation of selection with pricing and fulfillment info.
- Check item pricing and rental period.
- Validate final address and available quantity.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_select",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-onselect-3001",
    "timestamp": "2025-05-12T09:01:02Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "items": [
        {
          "id": "starmax-6v",
          "price": {
            "currency": "USD",
            "value": "50"
          },
          "quantity": {
            "measure": {
              "value": 1,
              "unit": "unit"
            }
          }
        }
      ],
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "456 Battery St, San Francisco, CA"
            }
          }
        }
      ]
    }
  }
}
````

### init

Initializes the order by sending billing and fulfillment details.

* Contains renter details and location for battery delivery.

```json
{
  "context": {
    "domain": "rental",
    "action": "init",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-init-3001",
    "timestamp": "2025-05-12T09:02:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "provider": { "id": "sfenergystore" },
      "items": [
        {
          "id": "starmax-6v"
        }
      ],
      "billing": {
        "name": "Jane Doe",
        "address": "456 Battery St, SF, CA",
        "email": "jane@example.com",
        "phone": "+14155551234"
      },
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "456 Battery St, SF, CA"
            }
          }
        }
      ]
    }
  }
}
````

### on_init

- Returns quote, payment info, and order summary for confirmation.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_init",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-oninit-3001",
    "timestamp": "2025-05-12T09:02:05Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "quote": {
        "price": {
          "currency": "USD",
          "value": "50"
        }
      },
      "payment": {
        "collected_by": "BAP",
        "type": "PRE-FULFILLMENT",
        "status": "NOT-PAID"
      },
      "items": [
        {
          "id": "starmax-6v"
        }
      ]
    }
  }
}
````

### confirm

- Used to finalize the battery rental request including selected item and delivery.

```json
{
  "context": {
    "domain": "rental",
    "action": "confirm",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-confirm-3001",
    "timestamp": "2025-05-12T09:03:00Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "id": "battery-order-001",
      "provider": { "id": "sfenergystore" },
      "items": [ { "id": "starmax-6v" } ],
      "billing": {
        "name": "Jane Doe",
        "email": "jane@example.com",
        "phone": "+14155551234"
      },
      "fulfillments": [
        {
          "end": {
            "location": { "address": "456 Battery St, SF, CA" }
          }
        }
      ]
    }
  }
}
````

### on_confirm

- Confirmation of successful rental placement.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_confirm",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-onconfirm-3001",
    "timestamp": "2025-05-12T09:03:03Z",
    "location": {
      "country": { "code": "USA" },
      "city": { "code": "NANP:628" }
    },
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "id": "battery-order-001",
      "state": "CONFIRMED",
      "items": [
        {
          "id": "starmax-6v"
        }
      ],
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "456 Battery St, SF, CA"
            }
          }
        }
      ]
    }
  }
}
````

### status

- Used to fetch the status of the rental fulfillment.

```json
{
  "context": {
    "domain": "rental",
    "action": "status",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-status-3001",
    "timestamp": "2025-05-12T09:04:00Z",
    "version": "1.1.0"
  },
  "message": {
    "order_id": "battery-order-001"
  }
}
````

### on_status

- Gives latest state of delivery or pickup for the rental order.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_status",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-onstatus-3001",
    "timestamp": "2025-05-12T09:04:02Z",
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "id": "battery-order-001",
      "state": "ACTIVE",
      "fulfillments": [
        {
          "state": {
            "descriptor": {
              "code": "In-Transit",
              "name": "Battery out for delivery"
            }
          }
        }
      ]
    }
  }
}
````

### update

- Used to update order fulfillment like new delivery window.

```json
{
  "context": {
    "domain": "rental",
    "action": "update",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-update-3001",
    "timestamp": "2025-05-12T09:05:00Z",
    "version": "1.1.0"
  },
  "message": {
    "update_target": "fulfillments",
    "order": {
      "id": "battery-order-001",
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "789 New Delivery Lane, SF, CA"
            }
          }
        }
      ]
    }
  }
}
````

### on_update

- Acknowledges update to the order details.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_update",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-onupdate-3001",
    "timestamp": "2025-05-12T09:05:03Z",
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "id": "battery-order-001",
      "fulfillments": [
        {
          "end": {
            "location": {
              "address": "789 New Delivery Lane, SF, CA"
            }
          }
        }
      ]
    }
  }
}
````

### cancel

- Used to cancel a placed order before fulfillment.

```json
{
  "context": {
    "domain": "rental",
    "action": "cancel",
    "bap_id": "battery-bap.com",
    "bap_uri": "https://battery-bap.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-cancel-3001",
    "timestamp": "2025-05-12T09:06:00Z",
    "version": "1.1.0"
  },
  "message": {
    "order_id": "battery-order-001",
    "cancellation_reason_id": "buyer_changed_mind"
  }
}
````

### on_cancel

- Confirms that the order has been successfully cancelled.

```json
{
  "context": {
    "domain": "rental",
    "action": "on_cancel",
    "bap_id": "battery-bap.com",
    "bpp_id": "sfenergystore.com",
    "bpp_uri": "https://api.sfenergystore.com",
    "transaction_id": "txn-rental-3001",
    "message_id": "msg-oncancel-3001",
    "timestamp": "2025-05-12T09:06:02Z",
    "version": "1.1.0"
  },
  "message": {
    "order": {
      "id": "battery-order-001",
      "state": "CANCELLED"
    }
  }
}
````


## Taxonomy and layer2 configuration

|Property Name|Enums|
|-------------|-----|
|battery-type|LITHIUM_ION<br>LEAD_ACID<br>FLOW_BATTERY<br>SOLID_STATE|
|rental-status|AVAILABLE<br>RENTED<br>MAINTENANCE<br>RETIRED<br>RESERVED|
|battery-capacity|SMALL_1KWH<br>MEDIUM_5KWH<br>LARGE_10KWH<br>XL_20KWH<br>CUSTOM|
|battery-condition|NEW<br>LIKE_NEW<br>GOOD<br>FAIR<br>POOR|
|rental-duration|HOURLY<br>DAILY<br>WEEKLY<br>MONTHLY<br>LONG_TERM|
|payment-status|PENDING<br>PAID<br>FAILED<br>REFUNDED<br>PARTIALLY_PAID|
|battery-usage|BACKUP<br>GRID_SERVICES<br>EV_CHARGING<br>SOLAR_STORAGE<br>MIXED_USE|
|battery-health|EXCELLENT<br>GOOD<br>FAIR<br>POOR<br>CRITICAL|
|battery-location|FIXED<br>MOBILE<br>PORTABLE<br>STATIONARY|
|battery-connectivity|WIRED<br>WIRELESS<br>HYBRID<br>MANUAL|
|battery-monitoring|REAL_TIME<br>PERIODIC<br>MANUAL<br>OFFLINE|
|battery-maintenance|REGULAR<br>PREVENTIVE<br>CORRECTIVE<br>EMERGENCY|
|battery-warranty|ACTIVE<br>EXPIRED<br>VOID<br>PENDING|
|battery-certification|CERTIFIED<br>PENDING<br>EXPIRED<br>REVOKED|
|battery-safety|COMPLIANT<br>NON_COMPLIANT<br>PENDING_VERIFICATION|
|battery-availability|AVAILABLE<br>RESERVED<br>UNAVAILABLE<br>MAINTENANCE|
|battery-charging|FAST_CHARGE<br>NORMAL_CHARGE<br>TRICKLE_CHARGE<br>OFFLINE|
|battery-discharging|ACTIVE<br>STANDBY<br>OFFLINE<br>PROTECTED|
|battery-temperature|NORMAL<br>WARNING<br>CRITICAL<br>SHUTDOWN|
|battery-cycles|LOW<br>MEDIUM<br>HIGH<br>CRITICAL|


## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to artefacts

- [Postman collection for Demand Flexibility](https://documenter.getpostman.com/view/23690031/2sB2qUmQAU#7f196f71-6ff6-457d-b813-fdca59faf12f)
- [Layer2 config for UEI Demand Flexibility]()

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