# DEG Implementation Guide - Connections

#### Version 1.1

## Version History

| Date       | Version | Description                     |
| ---------- | ------- | ------------------------------- |
| 12-05-2025 | 1.0     | Initial version                 |
| 13-05-2025 | 1.1     | Completed and revised structure |

## Introduction

This guide provides implementation instructions for electricity connection services using the Beckn Protocol in an open digital energy network. It is intended for developers and integrators building BAPs or BPPs for licensed utility providers such as San Francisco Electric Authority. The document aligns the sample APIs and JSON message structures for seamless interoperability across the network.

## Structure of the document

- [DEG Implementation Guide - Connections](#deg-implementation-guide---connections)
      - [Version 1.1](#version-11)
  - [Version History](#version-history)
  - [Introduction](#introduction)
  - [Structure of the document](#structure-of-the-document)
  - [Outcome Visualization](#outcome-visualization)
    - [Use Case - Electricity Connection](#use-case---electricity-connection)
  - [General Flow Diagrams](#general-flow-diagrams)
  - [API Calls and Schema](#api-calls-and-schema)
    - [search](#search)
    - [on\_search](#on_search)
    - [select](#select)
    - [on\_select](#on_select)
    - [init](#init)
    - [on\_init](#on_init)
    - [confirm](#confirm)
    - [on\_confirm](#on_confirm)
  - [Taxonomy and Layer 2 Configuration](#taxonomy-and-layer-2-configuration)
  - [Notes on Software Integration](#notes-on-software-integration)
  - [Links to Artefacts](#links-to-artefacts)
  - [Sandbox Details](#sandbox-details)

## Outcome Visualization

### Use Case - Electricity Connection

**Scenario**: Maria, a homeowner in San Francisco, needs a new residential electricity connection.

1. **Discovery**: Uses Energykart app to find available electricity service providers.
2. **Order**: Selects "Residential Electricity Connection", uploads required documents, receives order ID.
3. **Fulfillment**: Technician installs meter onsite and activates power.
4. **Post Fulfillment**: Receives invoice and gives feedback.

## General Flow Diagrams

Refer to the [Generic Implementation Guide - Flow Diagrams](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#general-flow-diagrams).

## API Calls and Schema

### search

Search request can contain one or more search criterion within it. Use the following list on how to specify the criterion:

- The location to search around is specified in message->intent->location.

- The service type is indicated in message->intent->category->descriptor->code.

- Free text can be specified in message->intent->descriptor->name.

- Additional filters like load or connection phase can be passed in message->intent->tags.



```json
{
  "context": {
    "domain": "service",
    "action": "search",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10001",
    "timestamp": "2025-05-07T13:59:00Z",
    "ttl": "PT10M",
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
    "intent": {
      "descriptor": {
        "name": "Request for a new electricity connection"
      },
      "category": {
        "descriptor": {
          "code": "electricity-connection"
        }
      }
    }
  }
}
```

### on\_search
on_search returns catalogs of services from matched providers. Use the following to extract details:

- Providers are listed in message->catalog->providers.

- Each provider lists services in provider->items.

- Each item includes pricing, description, and tags for filtering.

- The bpp_id and bpp_uri are used for follow-up API calls.

```json
{
  "context": {
    "domain": "service",
    "action": "on_search",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://example-bpp.com",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10002",
    "timestamp": "2025-05-07T14:00:10Z",
    "ttl": "PT10M",
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
    "catalog": {
      "providers": [
        {
          "id": "sf_electric_authority",
          "descriptor": {
            "name": "San Francisco Electric Authority",
            "short_desc": "Official provider for new electricity connections",
            "long_desc": "Government utility responsible for delivering new residential and commercial electricity connections in San Francisco.",
            "images": [
              {
                "url": "https://sfea.gov/logo.png"
              }
            ]
          },
          "items": [
            {
              "id": "new-resi-connection",
              "descriptor": {
                "name": "Residential Electricity Connection"
              },
              "price": {
                "estimated_value": "200",
                "currency": "USD"
              },
              "tags": [
                {
                  "descriptor": {
                    "code": "connection-type"
                  },
                  "list": [
                    {
                      "descriptor": {
                        "code": "property-type"
                      },
                      "value": "Residential"
                    },
                    {
                      "descriptor": {
                        "code": "phase"
                      },
                      "value": "Single Phase"
                    },
                    {
                      "descriptor": {
                        "code": "user_type"
                      },
                      "value": "Consumer"
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
Select is used to choose a specific service item returned in on_search. The following fields are used:

- The selected item is passed under message->order->items.

- Any additional parameters (like load or documents) are added as tags under the item.

- Fulfillment details are listed in message->order->fulfillments.
```json
{
  "context": {
    "domain": "service",
    "action": "select",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10002",
    "timestamp": "2025-05-07T14:00:15Z",
    "ttl": "PT10M",
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
        "id": "sf_electric_authority"
      },
      "items": [
        {
          "id": "new-resi-connection",
          "tags": [
            {
              "descriptor": {
                "code": "connection-type"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "user_type"
                  },
                  "value": "Consumer"
                }
              ]
            },
            {
              "descriptor": {
                "code": "requested_connection_details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "requested_load_kw"
                  },
                  "value": "5"
                },
                {
                  "descriptor": {
                    "code": "estimated_monthly_consumption_kwh"
                  },
                  "value": "300"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1"
        }
      ]
    }
  }
}
```

### on\_select
on_select confirms availability and details of the selected service. Look at these key fields:

- Service name and pricing are in message->order->items.

- Provider info is present in message->order->provider.

- Fulfillment and quote details are included with the item for confirmation.
```json
{
  "context": {
    "domain": "service",
    "action": "on_select",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://example-bpp.com",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10003",
    "timestamp": "2025-05-07T14:00:20Z",
    "ttl": "PT10M",
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
        "id": "sf_electric_authority",
        "descriptor": {
          "name": "San Francisco Electric Authority",
          "short_desc": "Official provider for new electricity connections",
          "long_desc": "Government utility responsible for delivering new residential and commercial electricity connections in San Francisco.",
          "images": [
            {
              "url": "https://sfea.gov/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "new-resi-connection",
          "descriptor": {
            "name": "Residential Electricity Connection"
          },
          "price": {
            "estimated_value": "200",
            "currency": "USD"
          },
          "tags": [
            {
              "descriptor": {
                "code": "connection-type"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "property-type"
                  },
                  "value": "Residential"
                },
                {
                  "descriptor": {
                    "code": "phase"
                  },
                  "value": "Single Phase"
                },
                {
                  "descriptor": {
                    "code": "user_type"
                  },
                  "value": "Consumer"
                }
              ]
            },
            {
              "descriptor": {
                "code": "documentation"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "required-docs"
                  },
                  "value": "ID proof, ownership certificate, wiring diagram"
                }
              ]
            },
            {
              "descriptor": {
                "code": "requested_connection_details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "requested_load_kw"
                  },
                  "value": "5"
                },
                {
                  "descriptor": {
                    "code": "estimated_monthly_consumption_kwh"
                  },
                  "value": "300"
                }
              ]
            }
          ]
        }
      ],
      "fulfillments": [
        {
          "id": "1",
          "type": "ONSITE"
        }
      ],
      "quote": {
        "price": {
          "value": "200",
          "currency": "USD"
        },
        "breakup": [
          {
            "title": "Connection Setup Fee",
            "price": {
              "value": "200",
              "currency": "USD"
            }
          }
        ]
      }
    }
  }
}
```

### init
init is used to submit customer, billing, and location details before placing the order. The following fields are important:

- Consumer details are added in billing and fulfillments.

- Location and GPS for connection are specified in fulfillments->stops.

- Any uploaded documents can be passed as tags or xinput if needed.


```json
{
  "context": {
    "domain": "service",
    "action": "init",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10004",
    "timestamp": "2025-05-07T14:00:30Z",
    "ttl": "PT10M",
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
        "id": "sf_electric_authority"
      },
      "items": [
        {
          "id": "new-resi-connection",
          "tags": [
            {
              "descriptor": {
                "code": "connection-type"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "user_type"
                  },
                  "value": "Consumer"
                }
              ]
            },
            {
              "descriptor": {
                "code": "requested_connection_details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "requested_load_kw"
                  },
                  "value": "5"
                },
                {
                  "descriptor": {
                    "code": "estimated_monthly_consumption_kwh"
                  },
                  "value": "300"
                }
              ]
            }
          ]
        }
      ],
      "billing": {
        "name": "Jonathan Park",
        "email": "jonathan.park@example.com",
        "phone": "+14155553333",
        "address": "910 Pinecrest Blvd, San Francisco, CA"
      },
      "fulfillments": [
        {
          "id": "1",
          "type": "ONSITE",
          "customer": {
            "person": {
              "name": "Jonathan Park"
            },
            "contact": {
              "phone": "+14155553333",
              "email": "jonathan.park@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "910 Pinecrest Blvd, San Francisco, CA",
                "gps": "37.774921,122.419424"
              }
            }
          ]
        }
      ]
    }
  }
}
```

### on\_init
on_init returns additional information needed for order confirmation. Refer to the fields below:

- Required documents and form link are in xinput under item tags.

- Payment terms and pricing breakup are found in quote.

- Provider’s contact info and tracking preferences are shared in fulfillments.


```json
{
  "context": {
    "domain": "service",
    "action": "on_init",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://example-bpp.com",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10005",
    "timestamp": "2025-05-07T14:00:35Z",
    "ttl": "PT10M",
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
        "id": "sf_electric_authority",
        "descriptor": {
          "name": "San Francisco Electric Authority",
          "short_desc": "Official provider for new electricity connections",
          "long_desc": "Government utility responsible for delivering new residential and commercial electricity connections in San Francisco.",
          "images": [
            {
              "url": "https://sfea.gov/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "new-resi-connection",
          "descriptor": {
            "name": "Residential Electricity Connection"
          },
          "price": {
            "estimated_value": "200",
            "currency": "USD"
          },
          "tags": [
            {
              "descriptor": {
                "code": "connection-type"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "property-type"
                  },
                  "value": "Residential"
                },
                {
                  "descriptor": {
                    "code": "phase"
                  },
                  "value": "Single Phase"
                },
                {
                  "descriptor": {
                    "code": "user_type"
                  },
                  "value": "Consumer"
                }
              ]
            },
            {
              "descriptor": {
                "code": "documentation"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "required-docs"
                  },
                  "value": "ID proof, ownership certificate, wiring diagram"
                }
              ]
            },
            {
              "descriptor": {
                "code": "requested_connection_details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "requested_load_kw"
                  },
                  "value": "5"
                },
                {
                  "descriptor": {
                    "code": "estimated_monthly_consumption_kwh"
                  },
                  "value": "300"
                }
              ]
            }
          ],
          "xinput": {
            "required": false,
            "head": {
              "descriptor": {
                "name": "Document Form"
              },
              "index": {
                "min": 0,
                "cur": 0,
                "max": 2
              },
              "headings": [
                "ID proof",
                "ownership certificate",
                "wiring diagram"
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
      "billing": {
        "name": "Jonathan Park",
        "email": "jonathan.park@example.com",
        "phone": "+14155553333",
        "address": "910 Pinecrest Blvd, San Francisco, CA"
      },
      "fulfillments": [
        {
          "id": "1",
          "type": "ONSITE",
          "tracking": true,
          "customer": {
            "person": {
              "name": "Jonathan Park"
            },
            "contact": {
              "phone": "+14155553333",
              "email": "jonathan.park@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "910 Pinecrest Blvd, San Francisco, CA",
                "gps": "37.774921,122.419424"
              }
            }
          ],
          "contact": {
            "phone": "+18004321234",
            "email": "support@sfea.gov"
          }
        }
      ],
      "quote": {
        "price": {
          "value": "200",
          "currency": "USD"
        },
        "breakup": [
          {
            "title": "Connection Setup Fee",
            "price": {
              "value": "200",
              "currency": "USD"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "ON-FULFILLMENT",
          "collected_by": "BPP",
          "status": "NOT-PAID",
          "params": {
            "amount": "150",
            "currency": "USD"
          }
        }
      ]
    }
  }
}
```

### confirm
confirm is used to finalize the order. You must send the following:

- Full details of the selected item and consumer in order.

- Payment instructions are specified under payments.

- Final location and service details must match those returned in on_init.
```json
{
  "context": {
    "domain": "service",
    "action": "confirm",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bap_uri": "https://api.example-bap.com/pilot/bap/energy/v1",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10006",
    "timestamp": "2025-05-07T14:00:45Z",
    "ttl": "PT10M",
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
        "id": "sf_electric_authority"
      },
      "items": [
        {
          "id": "new-resi-connection",
          "tags": [
            {
              "descriptor": {
                "code": "connection-type"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "user_type"
                  },
                  "value": "Consumer"
                }
              ]
            },
            {
              "descriptor": {
                "code": "requested_connection_details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "requested_load_kw"
                  },
                  "value": "5"
                },
                {
                  "descriptor": {
                    "code": "estimated_monthly_consumption_kwh"
                  },
                  "value": "300"
                }
              ]
            }
          ]
        }
      ],
      "billing": {
        "name": "Jonathan Park",
        "email": "jonathan.park@example.com",
        "phone": "+14155553333",
        "address": "910 Pinecrest Blvd, San Francisco, CA"
      },
      "fulfillments": [
        {
          "id": "1",
          "type": "ONSITE",
          "customer": {
            "person": {
              "name": "Jonathan Park"
            },
            "contact": {
              "phone": "+14155553333",
              "email": "jonathan.park@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "910 Pinecrest Blvd, San Francisco, CA",
                "gps": "37.774921,122.419424"
              }
            }
          ]
        }
      ],
      "payments": [
        {
          "type": "ON-FULFILLMENT",
          "collected_by": "BPP",
          "status": "NOT-PAID",
          "params": {
            "amount": "150",
            "currency": "USD"
          }
        }
      ]
    }
  }
}
```

### on\_confirm
on_confirm confirms that the order has been successfully placed. Look at these fields for final confirmation:

- Order ID is found in message->order->id.

- Scheduled state, tracking info, and support contact are inside fulfillments.

- Final payment, quote, and item details are echoed back for verification.


```json
{
  "context": {
    "domain": "service",
    "action": "on_confirm",
    "version": "1.1.0",
    "bap_id": "example-bap.com",
    "bpp_id": "example-bpp.com",
    "bpp_uri": "https://example-bpp.com",
    "transaction_id": "connection-txn-8001",
    "message_id": "connection-msg-10007",
    "timestamp": "2025-05-07T14:00:50Z",
    "ttl": "PT10M",
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
      "id": "conn-order-001",
      "provider": {
        "id": "sf_electric_authority",
        "descriptor": {
          "name": "San Francisco Electric Authority",
          "short_desc": "Official provider for new electricity connections",
          "long_desc": "Government utility responsible for delivering new residential and commercial electricity connections in San Francisco.",
          "images": [
            {
              "url": "https://sfea.gov/logo.png"
            }
          ]
        }
      },
      "items": [
        {
          "id": "new-resi-connection",
          "descriptor": {
            "name": "Residential Electricity Connection"
          },
          "price": {
            "estimated_value": "200",
            "currency": "USD"
          },
          "tags": [
            {
              "descriptor": {
                "code": "connection-type"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "property-type"
                  },
                  "value": "Residential"
                },
                {
                  "descriptor": {
                    "code": "phase"
                  },
                  "value": "Single Phase"
                },
                {
                  "descriptor": {
                    "code": "user_type"
                  },
                  "value": "Consumer"
                }
              ]
            },
            {
              "descriptor": {
                "code": "documentation"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "required-docs"
                  },
                  "value": "ID proof, ownership certificate, wiring diagram"
                }
              ]
            },
            {
              "descriptor": {
                "code": "requested_connection_details"
              },
              "list": [
                {
                  "descriptor": {
                    "code": "requested_load_kw"
                  },
                  "value": "5"
                },
                {
                  "descriptor": {
                    "code": "estimated_monthly_consumption_kwh"
                  },
                  "value": "300"
                }
              ]
            }
          ]
        }
      ],
      "billing": {
        "name": "Jonathan Park",
        "email": "jonathan.park@example.com",
        "phone": "+14155553333",
        "address": "910 Pinecrest Blvd, San Francisco, CA"
      },
      "fulfillments": [
        {
          "id": "1",
          "type": "ONSITE",
          "tracking": true,
          "state": {
            "descriptor": {
              "code": "Scheduled",
              "name": "Connection Visit Scheduled"
            }
          },
          "customer": {
            "person": {
              "name": "Jonathan Park"
            },
            "contact": {
              "phone": "+14155553333",
              "email": "jonathan.park@example.com"
            }
          },
          "stops": [
            {
              "type": "end",
              "location": {
                "address": "910 Pinecrest Blvd, San Francisco, CA",
                "gps": "37.774921,122.419424"
              }
            }
          ],
          "contact": {
            "phone": "+18004321234",
            "email": "support@sfea.gov"
          }
        }
      ],
      "quote": {
        "price": {
          "value": "200",
          "currency": "USD"
        },
        "breakup": [
          {
            "title": "Connection Setup Fee",
            "price": {
              "value": "200",
              "currency": "USD"
            }
          }
        ]
      },
      "payments": [
        {
          "type": "ON-FULFILLMENT",
          "collected_by": "BPP",
          "status": "NOT-PAID",
          "params": {
            "amount": "150",
            "currency": "USD"
          }
        }
      ]
    }
  }
}
```

## Taxonomy and Layer 2 Configuration

|Property Name|Enums|
|-------------|-----|
|connection category|ELECTRICITY_CONNECTION|
|property-type|RESIDENTIAL<br>COMMERCIAL<br>INDUSTRIAL<br>AGRICULTURAL|
|connection-status|PENDING<br>IN_PROGRESS<br>COMPLETED<br>CANCELLED<br>FAILED<br>ON_HOLD|
|connection-phase|SINGLE_PHASE<br>THREE_PHASE<br>SPLIT_PHASE|
|user-type|CONSUMER<br>PRODUCER|
|fulfillment-type|ON_SITE|
|meter-type|DIGITAL<br>ANALOG<br>SMART_METER<br>PREPAID_METER<br>POSTPAID_METER|
|documentation-status|PENDING<br>SUBMITTED<br>VERIFIED<br>REJECTED<br>EXPIRED|
|payment-status|PENDING<br>PAID<br>FAILED<br>REFUNDED<br>PARTIALLY_PAID|
|billing cycle|WEEKLY<br>MONTHLY<br>QUARTERLY|
|fulfillment state codes| TECHNICIAN_ASSIGNED<br>ASSESSMENT_COMPLETED<br>APPROVED<br>CONNECTION_ESTABLISHED|
|payment type| PRE_ORDER<br>POST_ORDER<br>PRE_FULFILLMENT<br>POST_FULFILLMENT|
|payment status| PAID<br>NOT_PAID
|payment collected by| BAP<br>BPP<br>AGENT
|fulfillment stop types| START<br>END




## Notes on Software Integration

Refer: [Integrating with Your Software](https://github.com/beckn/missions/blob/main/Generic-Implementation-Guide/generic_implementation_guide.md#integrating-with-your-software)

## Links to Artefacts

* [Postman Collection for DEG Connection APIs]()
* [Layer 2 Configuration File]()

## Sandbox Details

**Registry/Gateway:**

* Gateway: []()
* Registry: []()

**BAP:**

* BAP Client: []()

**BPP:**

* BPP Client: []()
* BPP Network: []()