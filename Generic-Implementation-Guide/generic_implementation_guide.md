# Generic Implementation Guide

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 01-04-2025 | 1.0     | Initial Version                                     |

## Introduction

This implementation guide provides a structured approach to integrating the Beckn protocol across various domains. Rather than focusing on a specific industry, this guide presents scenario-based examples that demonstrate how to implement key interactions using Beckn’s open network principles. These examples serve as a reference to help developers design and build interoperable systems while maintaining flexibility for domain-specific adaptations. Whether you're working in mobility, retail, healthcare, or any other sector, this guide will assist in understanding and applying Beckn's decentralized framework effectively.

## Structure of the document

This document has the following parts:
  - [General Flow diagrams](#general-flow-diagrams)
  - [Specific flows](#specific-flows)
    - [Discovery of catalog](#discovery-of-catalogs)
    - [Placing an Order](#placing-an-order)
    - [Payment](#payment-in-a-transaction)
    - [Payment collected by BAP](#payment-collected-by-a-bap)
    - [Payment collected by BPP](#payment-collected-by-a-bpp)
    - [Confirming an Order](#confirming-an-order)
    - [Checking status of an order](#checking-the-status-of-an-order)
    - [Tracking and order](#tracking-the-order-delivery)
    - [Updating an order](#updating-part-of-an-order)
    - [Support](#request-for-a-support)
    - [Cancellation](#cancelling-an-order)
    - [Rating](#providing-rating-for-an-order)
  - [Integrating with your software](#integrating-with-your-software)
  - [Links to Domain specific Implementation guides](#link-to-the-domain-specific-implementation-guide)

## General Flow diagrams

### General Beckn message flow and error handling

This section is relevant to all the messages flows illustrated below and discussed further in the document.

Beckn is a aynchronous protocol at its core.

- When a network participant(NP1) sends a message to another participant(NP2), the other participant(NP2) immediately returns back an ACK/NACK(Acknowledgement or Negative Acknowledgement in case of error - usually with wrongly formed messages).
- An ACK is an indicator that the receiving participant(NP2) will process this message and dispatch an on_xxxxxx message to original NP (NP1)
- Subsequently after processing the message NP2 sends back the real response in the corresponding on_xxxxxx message, to the first participant(NP1).
- The participant(NP2) uses the context.bap_uri from the request payload to send back the on_xxxxxx response.
- This message can contain a message field (for success) or error field (for failure)
- NP1 when it receives the on_xxxxxx message, sends back an ACK/NACK (Here in both the cases NP1 will not send any subsequent message).
- In the Use case diagrams, this ACK/NACK is not illustrated explicitly to keep the diagrams crisp.
- However when writing software we should be prepared to receive these NACK messages as well as error field in the on_xxxxxx messages
- While this discussion is from a Beckn perspective, Adapters can provide synchronous modes. For example, the Protocol Server which is the reference implementation of the Beckn Adapter provides a synchronous mode by default. So if your software calls the support endpoint on the BAP Protocol Server, the Protocol Server waits till it gets the on_support and returns back that as the response.

![ACK NACK for messages](https://github.com/beckn/missions/blob/main/docs/assets/images/ack_nack.png)

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

## Specific flows

Note: In the API payload, context field is almost similar in all the APIs. Context used in a callback is same as the context sent in the corresponding request.

### Key Fields in Context and How to Implement

| Field             | Description & Implementation |
|------------------|--------------------------------|
| `domain`         | Set according to your domain (e.g., `retail`, `mobility`, `ev-charging:uei`). |
| `country.code`   | Use ISO 3166-1 alpha-3 code (e.g., `IND` for India). |
| `action`         | Reflect the current API action (All Possible actions as per the current specification, `search`, `select`, `init`, `confirm`, `status`, `track`, `update`, `support`, `rating`, `cancel`, `on_search`, `on_select`, `on_init`, `on_confirm`, `on_status`, `on_track`, `on_update`, `on_support`, `on_rating`, `on_cancel`, `get_rating_categories`, `rating_categories`, `get_cancellation_reasons`, `cancellation_reasons`). |
| `version`        | Use the supported Protocol Specification version (commonly `1.1.0` or `1.0.0`). |
| `bap_id` / `bap_uri` | Set to your BAP's ID and public callback URL, as per registry. |
| `bpp_id` / `bpp_uri` | Reflect the responding BPP's details. |
| `transaction_id` | Generate a **UUID** for the full transaction flow; reuse across related APIs. |
| `message_id`     | Generate a new **UUID** for each API call and its callbacks. |
| `timestamp`      | Use ISO 8601 UTC format (e.g., `2025-04-23T14:30:00Z`). Must reflect current server time. |
| `ttl`            | Set expiry using ISO 8601 Duration format (e.g., `PT10M` = 10 minutes). |

---

#### Best Practices

- Ensure IDs and URIs match those in the **Beckn Registry**.
- Validate **timestamp** and **ttl** on receiving side to reject stale messages.
- Use `transaction_id` to trace an end-to-end flow.
- Use `message_id` to trace a single request/response pair.

#### Example `context` Block

```json
"context": {
  "domain": "retail",
  "location": {
    "country": {
      "code": "IND"
    },
    "city": {
      "code": "std:080"
    },
  }
  "action": "select",
  "version": "1.1.0",
  "bap_id": "buyer-app.example.com",
  "bap_uri": "https://buyer-app.example.com",
  "bpp_id": "seller-app.example.com",
  "bpp_uri": "https://seller-app.example.com",
  "transaction_id": "12345678-abcd-efgh-ijkl-1234567890ab",
  "message_id": "abcd1234-5678-90ef-ghij-klmnopqrstuv",
  "timestamp": "2025-04-23T14:30:00Z",
  "ttl": "PT10M"
}
```

### Discovery of catalogs

Discovery is the process of finding what’s available — it helps a consumer (on a BAP) discover products, services, or providers (on one or more BPPs) that match their need.

In Beckn, this is done through two core APIs:
  - search – Sent by the BAP to declare an intent.
  - on_search – Sent by BPPs in response, containing matching catalogs.

Discovery in a Beckn network happens generally through an intermediate entity like a Beckn Gateway (BG) present inside the network.

There are 2 ways of discovery:
- If the address of the BPP is **not** specified in the context of the API call

  ```
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
  }
  ```
  The BAP calls the BG and the BG may multicast this API to multiple BPPs based on the context.domain. The BPPs synchronously respond with ACKs. The BPPs then asynchronously call the on_search API to send the catalogs to the BAP.


- If the address of the BPP is specified in the context of the API call.
  
  The details of the BPP is specified in the context.bpp_id and context.bpp_uri fields.

  ```
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
    "bap_uri": "https://example-bap-url.com",
    "bpp_id": "example-bpp-id",
    "bpp_uri": "https://example-bpp-url.com"
  }
  ```
  The BAP calls the targeted BPP directly. The BPP synchronously responds with ACK. The BPP then asynchronously calls the on_search API of the BAP.

  In a callback in response to the search/ call, BPPs send their catalog to the BAP who initiated the search. A catalog generally contains the following things:
    - List of items (products/services)
    - Their fulfillment options (location, agent, time)
    - Payment options
    - Offers
    - Categories and tags

  An example Catalog from a BPP could look like

  ```
      {
    "context": {
      "domain": "local-retail",
      "location": {
        "country": {
          "code": "IND"
        },
        "city": {
          "code": "std:080"
        }
      },
      "action": "on_search",
      "version": "1.1.0",
      "bap_id": "farm-fresh-bap-id",
      "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
      "bpp_id": "farm-fresh-bpp-subId",
      "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "catalog": {
        "descriptor": {
          "name": "HBO"
        },
        "providers": [
          {
            "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
            "descriptor": {
              "name": "Venky.Mahadevan@Bazaar"
            },
            "locations": [
              {
                "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location",
                "gps": "12.909955,77.596316"
              }
            ],
            "categories": [
              {
                "id": "c1",
                "descriptor": {
                  "code": "grocery",
                  "name": "grocery"
                }
              },
              {
                "id": "c2",
                "descriptor": {
                  "code": "electronics",
                  "name": "electronics"
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
            "items": [
              {
                "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
                "descriptor": {
                  "images": [
                    {
                      "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
                    }
                  ],
                  "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
                  "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
                  "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that’s free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won’t spill a drop even when it’s tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
                },
                "matched": true,
                "price": {
                  "listed_value": "1200.0",
                  "currency": "INR",
                  "value": "1200.0"
                },
                "recommended": true,
                "location_ids": [
                    "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
                ],
                "category_ids": ["c1"],
                "fulfillment_ids": ["f1"],
                "tags": [
                  {
                    "descriptor": {
                      "name": "item-cataegory"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "name": "category"
                        },
                        "value": "retail"
                      }
                    ]
                  },
                  {
                    "descriptor": {
                      "name": "item-properties"
                    },
                    "list": [
                      {
                        "descriptor": {
                          "name": "waterbottle"
                        },
                        "value": "y"
                      },
                      {
                        "descriptor": {
                          "name": "Trekking"
                        },
                        "value": "y"
                      },
                      {
                        "descriptor": {
                          "name": "Sipper"
                        },
                        "value": "y"
                      },
                      {
                        "descriptor": {
                          "name": "Hiking"
                        },
                        "value": "y"
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

### Placing an order
  - Order phase generally involves a subset of the below actions wherever applicable.
      - finalizing the quotation
      - providing the billing information of the customer
      - providing additional information needed to confirm the order
      - sharing payment details
      - completing the payment
  - There are 3 APIs(and callbacks) involved in this stage:
      - select, on_select
      - init, on_init
      - confirm, on_confirm
  - If there is a payment involved in a transaction, BAP can use select API call to request a quotation from the seller. BPP uses on_Select Beckn APIs to send back the quotation for the transaction. Multiple select and on_select can be used to finalize the quotation, depending on the use case.
  - If there is a payment involved in a transaction, BAP can use init API call to provide the billing info of the customer to the seller. The BPP uses the on_init callback to send back the payment terms to the BAP. Return, refund, replacement terms can also be sent as part of the on_init callback.
  - Customer verifies the terms of the order, completes the payment if it is required and sends a confirmation to the BPP using confirm API.
  - BPP sends back order confirmation with details of the order with an order id using the on_confirm API.

  #### Examples payload for APIs in Order phase.

  **Select**
  - Select request generally includes provider_id, item_id and preferred fulfillment mode. 

  ```
  {
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "select",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "6d098f3a-4873-4b2e-935e-e4d6be92eb01",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:44:47.217Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider"
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "quantity": {
            "selected" :{
              "count" : 2
            }
          }
        }
      ],
      "fulfillments": [
        {
          "id": "f1"
        }
      ]
    }
  }
}
```
**A typical on_select**
- on_select typically includes provider details, item details(including any add ons if any), fulfillment details of the selected fulfillment option and a quotation.
- At the end of an select and on_select cycle, at least an estimate quote has to be finalized between buyer and seller.

```
{
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_select",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "6d098f3a-4873-4b2e-935e-e4d6be92eb01",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:44:47.229Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
        "descriptor": {
          "name": "Venky.Mahadevan@Bazaar"
        },
        "locations": [
          {
            "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
          }
        ]
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "descriptor": {
            "images": [
              {
                "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
              }
            ],
            "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
            "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
            "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that’s free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won’t spill a drop even when it’s tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
          },
          "category_ids": [
            "c1"
          ],
          "quantity": {
            "selected" :{
              "count" : 2
            }
          },
          "price": {
            "listed_value": "1200.0",
            "currency": "INR",
            "value": "1200.0"
          }
        }
      ],
      "fulfillments": [
        {
          "id": "f1"
        }
      ],
      "quote": {
        "price": {
          "currency": "INR",
          "value": "1500.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "INR",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "INR",
              "value": "300.0"
            }
          }
        ]
      }
    }
  }
}
```

**on_select with xinput**

- If a seller needs custom information before they can provide a quotation, they can send an xinput form as part of the on_select callback, requesting buyer to fill the custom info in the form. The buyer need to fill the form and call select API again to inform the seller. Seller can send the quotation as part of a following on_select.
- For example, when a user is applying for a financial support, the provider can send an xinput as part of the on_select callback to request for certain documents or details.
- Xinput can contain a simple form or a multi part form. The xinput.head.index is used to represent the current state of the form. xinput.head.headings is used to represent the title of all the parts of a form.

```
{
    "context": {
        "domain": "dsep:scholarships",
        "location": {
            "city": {
                "name": "Bangalore",
                "code": "std:080"
            },
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "action": "on_select",
        "timestamp": "2023-08-02T09:12:12.680Z",
        "ttl": "PT10M",
        "version": "0.7.0",
        "bap_id": "ps-bap-network.becknprotocol.io",
        "bap_uri": "https://ps-bap-network.becknprotocol.io/",
        "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
        "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io/",
        "transaction_id": "a9aaecca-10b7-4d19-b640-022723112309",
        "message_id": "6114a3e5-acb0-4c99-b017-0ead5e894bad"
    },
    "message": {
        "order": {
            "provider": {
                "id": "BX213573733",
                "descriptor": {
                    "name": "XYZ Education Foundation",
                    "short_desc" : "Short Description about the Foundation",
                    "images": [
                        {
                            "url" : "url of the image of the provider"
                        }
                    ]
                },
                "locations" : [
                    {
                        "id" : "L1",
                        "city" :"{
                          "name": Pune"
                        }
                        "state" : "Maharastra"
                    },
                    {
                        "id" : "L2",
                        "city" :"{
                          "name": Thane"
                        }
                        "state" : "Maharastra"
                    },
                    {
                        "id" : "L3",
                        "city" :"{
                          "name": Lucknow"
                        }
                        "state" : "Uttar Pradesh"
                    }
                ],
                "rateable": false
            },
            "items": [
                {
                    "id": "SCM_63587501",
                    "descriptor": {
                        "name": "XYZ Education Scholarship for Undergraduate Students",
                        "short_desc": "XYZ Education Scholarship for Undergraduate Students"
                    },
                    "price": {
                        "currency": "INR",
                        "value": "Upto RS.1000 per year"
                    },
                    "time" : {
                        "label" : "Start  & End date of the Application",
                        "range" : {
                            "start" : "2022-09-01T00:00:00.000Z",
                            "end" : "2022-10-31T00:00:00.000Z"
                        }
                    },
                    "rateable": false,
                    "tags": [
                        {
                            "display": true,
                            "descriptor": {
                                "code": "soc-elg",
                                "name": "Social Eligibility"
                            },
                            "list": [
                                {
                                    "value" : "SC"
                                },
                                {
                                    "value" : "ST"
                                },
                                {
                                    "value" : "OBC"
                                }]
                        },
                        {
                            "display": true,
                            "descriptor": {
                                "code": "gen-elg",
                                "name": "Gender Eligibility"
                            },
                            "list": [
                                {
                                    "value" : "Female"
                                },
                                {
                                    "value" : "Transgender"
                                }]
                        },
                        {
                            "display": true,
                            "descriptor": {
                                "code": "fin-elg",
                                "name": "Financial Eligibility"
                            },
                            "list": [
                                {
                                    "value" : "Max Family Income - Rs.500000.00"
                                }] 
                            
                        },
                        {
                            "display": true,
                            "descriptor": {
                                "code": "acad-elg",
                                "name": "Academic Eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "class",
                                        "name": "Class"
                                    },
                                    "value": "12th"
                                },
                                {
                                    "descriptor": {
                                        "code": "percentage",
                                        "name": "Percentage"
                                    },
                                    "value": ">= 50"
                                }

                            ] 
                            
                        },
                        {
                            "display": true,
                            "descriptor": {
                                "code": "docs-reqd",
                                "name": "Documents Required"
                            },
                            "list": [
                                {
                                    "value" : "10th Marksheet"
                                },
                                {
                                    "value" : "Aadhar Card of the Student"
                                },
                                {
                                    "value" : "Aadhar Card of the Parent"
                                },
                                {
                                    "value" : "Pan Card of the Parent"
                                }
                            ]
                        },
                        {
                            "display": true,
                            "descriptor": {
                                "code": "add-info",
                                "name": "Additional Information"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "faq-url",
                                        "name": "Frequently Asked Questions's URL"
                                    },
                                    "value" : "https://www.xyz-scholarship.com/faq" 
                                },
                                {
                                    "descriptor": {
                                        "code": "tnc-url",
                                        "name": "Terms and Conditions's URL"
                                    },
                                    "value": "https://www.xyz-scholarship.com/tnc"
                                }
                            ]
                        }
                    ],
                    "location_ids": ["L1","L2"],
                    "category_ids": ["DSEP_CAT_1"]
                }
            ],
            "fulfillments": [
                {
                    "id": "DSEP_FUL_63587501",
                    "type": "SCHOLARSHIP",
                    "tracking": false,
                    "agent" : {
                        "person" : {
                            "name" : "Ekstep Foundation SPoc"
                        },
                        "contact" : {
                            "email" : "ekstepsupport@ekstep.com"
                        }
                    }
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
                          "max": 3
                      },
                      "headings": [
                          "Personal Details",
                          "Educational Details",
                          "Financial Details",
                          "Review & Submit"
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
    }
}
```

**init**
- Generally used to provide the billing details of a customer to a BPP.
- It is also used to provide any additional info required from the BPP side to confirm an order.
- Below is an example of a typical init payload.

```
{
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "init",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "message_id": "06f49c01-65b7-4754-8b37-ab4e33688154",
    "timestamp": "2023-11-06T09:45:40.407Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider"
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "quantity": {
            "selected" :{
              "count" : 2
            }
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "919122343344"
            }
          }
        }
      ],
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      }
    }
  }
}
```

**on_int**
- Generally contains the full order terms including fulfillment details, payment terms and details, return terms, replacement terms, cancellation terms, and refund terms etc wherever applicable.
- Based on the use case, an on_init may only contain a subset of the above terms.
- Similar to on_select, BPP can send an xinput as part of the on_init callback if BPP requires additional details to confirm the terms of the order.

```
{
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_init",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "06f49c01-65b7-4754-8b37-ab4e33688154",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:45:40.421Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
        "descriptor": {
          "name": "Venky.Mahadevan@Bazaar"
        },
        "locations": [
          {
            "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
          }
        ]
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "descriptor": {
            "images": [
              {
                "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
              }
            ],
            "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
            "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
            "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that’s free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won’t spill a drop even when it’s tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
          },
          "category_ids": [
            "c1"
          ],
          "quantity": {
            "selected" :{
              "count" : 2
            }
          },
          "price": {
            "listed_value": "1200.0",
            "currency": "INR",
            "value": "1200.0"
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "919122343344"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "currency": "INR",
          "value": "1500.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "INR",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "INR",
              "value": "300.0"
            }
          }
        ]
      },
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      },
      "payments": [
        {
          "status": "NOT-PAID",
          "type": "PRE-FULFILLMENT",
          "collected_by": "BAP",
          "params": {
            "amount": "1500",
            "currency": "INR",
            "bank_code": "INB0004321",
            "bank_account_number": "1234002341"
          }
        }
      ],
      "cancellation_terms": [
        {
          "cancel_by": {
            "duration": "P7D"
          },
          "cancellation_fee": {
            "amount": {
              "currency": "INR",
              "value": "100"
            }
          },

        }
      ],
      "refund_terms": [
        {
          "refund_eligible": "true",
          "fulfillment_state": {
            "code" : ""
          }
        }
      ]
    }
  }
}
```

### Payment in a transaction

- The payment for a transaction can either be collected by a BAP or a BPP. It is represented by the payment.collected_by parameter in the on_init response.
- The status of the payment is represented by payment.status field.
- Payment type is represented by payment.type field. It indicates the stage in the order lifecycle when the payment has to be completed. Example values are:
  - PRE-ORDER: Payment is expected before the order is even confirmed.
  - PRE-FULFILMENT: Payment is made after order confirmation, but before fulfillment begins.
  - POST-ORDER: Payment happens immediately after placing the order.
  - POST-FULFILMENT: Payment is made after the order has been fulfilled
- The details of the payment is represented by the payment.params object. It contains amount and currency fields.
- transaction_id is unused until the payment has been completed. It is used to provide the proof of payment from a BAP to a BPP.
- There are 2 ways to represent where the payment need to be done. It can either be the bank details under payment.params object or the payment.url field.

### Payment collected by a BAP.

In cases where the payment is collected by a BAP, the payment object would have these details.
- The payment.collected_by field would be set to BAP.
- payment.param.amount and payment.param.currency would be set to appropriate values.
- payment.type(Pre-Order, Pre-Fulfillment, Post-Order, Post-Fulfillment etc), payment.status(Inititated, Completed etc) would be set to appropriate value.
- The BPP does not have to send the payment url or the bank details in the payment.params object.
- When a BAP collects payment for a transaction, BAP is responsible for settling the payment with the corresponding BPPs—either online or offline—as per the terms defined in their mutual business agreement.

The payment object inside the on_init callback would look like this.

```
"payments": [
  {
    "collected_by": "BAP",
    "type": "PRE-ORDER",
    "params": {
      "amount": "1500",
      "currency": "INR",
    }
    "status": "NOT-PAID",
  }
]
```

The BAP would process the payment from the customer, validate the payment and the send an updated payment object as part of the confirm/ request made to the BPP. 

- The payment object would have status changed to PAID.
- There is no need to send a proof of payment, i.e. transaction_id as part of the confirm request.
- The payment sent as part of the confirm request would look like below.


```
"payments": [
  {
    "collected_by": "BAP",
    "type": "PRE-ORDER",
    "params": {
      "amount": "1500",
      "currency": "INR",
    }
    "status": "PAID",
  }
]
```

### Payment collected by a BPP.

In cases where the payment is collected by a BPP, the payment object would have these details.
- The payment.collected_by field would be set to BPP.
- payment.param.amount and payment.param.currency would be set to appropriate values.
- payment.type(Pre-Order, Pre-Fulfillment, Post-Order, POst-Fulfillment etc), payment.status(Inititated, Completed etc) would be set to appropriate value.
- The BPP would need to send the payment receivers details in either url or in the payment.params object.

The payment object inside the on_init callback would look like this.

```
"payments": [
  {
    "collected_by": "BPP",
    "type": "PRE-ORDER",
    "params": {
      "amount": "1500",
      "currency": "INR",
    }
    "url": "https://payment-gateway.com/checkout?amount=1500&currency=INR&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback
    "
    "status": "NOT-PAID",
  }
]
```

The BAP would render the payment.url on the UI, the customer completes the payment and then the BAP sends an updated payment object as part of the confirm/ request made to the BPP. 

- The payment object would have status changed to PAID.
- The BAP need to send a proof of payment, i.e. transaction_id as part of the confirm request.
- The BPP validates the payment using the transaction_id and responds accordingly.
- The payment sent as part of the confirm request would look like below.

```
"payments": [
  {
    "collected_by": "BPP",
    "type": "PRE-ORDER",
    "params": {
      "amount": "1500",
      "currency": "INR",
      "transaction_id": "284724892494"
    }
    "url": "https://payment-gateway.com/checkout?amount=1500&currency=INR&order_id=123456&customer_id=78910&callback_url=https://yourwebsite.com/payment/callback
    "
    "status": "PAID",
  }
]
```

### Confirming an Order

- The BAP agrees on all the terms of the order, completes the payment if it is involved, and sends a confirmation to the BPP using confirm/ call.
- The BPP verifies the terms of the order and if everything looks okay it sends the order confirmation using on_confirm callback.
- Must contain all final values from the previous stages.
- Should include payment details, if applicable (COD, pre-paid, etc.).
- on_confirm callback contains an order id.
- A confirm payload will look like this.

confirm
```
{
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "confirm",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "message_id": "06f49c01-65b7-4754-8b37-ab4e33688154",
    "timestamp": "2023-11-06T09:45:40.407Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider"
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "quantity": {
            "selected" :{
              "count" : 2
            }
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "919122343344"
            }
          }
        }
      ],
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      },
      "payments": [
        {
          "status": "PAID",
          "type": "PRE-ORDER",
          "collected_by": "BPP",
          "params": {
            "amount": "1500",
            "currency": "INR"
            "transaction_id": "284724892494"
          }
        }
      ]
    }
  }
}
```

The on_confirm API is the callback response to confirm. It carries the final confirmation details of the order.
  - The message.order object includes:
      - A unique order ID (assigned by the BPP)
      - All previously confirmed elements (item, payment, fulfillment)
      - Status of the order (e.g., confirmed, pending, failed)

on_confirm
```
{
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_init",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "06f49c01-65b7-4754-8b37-ab4e33688154",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:45:40.421Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "12345678",
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
        "descriptor": {
          "name": "Venky.Mahadevan@Bazaar"
        },
        "locations": [
          {
            "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
          }
        ]
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "descriptor": {
            "images": [
              {
                "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
              }
            ],
            "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
            "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
            "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that’s free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won’t spill a drop even when it’s tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
          },
          "category_ids": [
            "c1"
          ],
          "quantity": {
            "selected" :{
              "count" : 2
            }
          },
          "price": {
            "listed_value": "1200.0",
            "currency": "INR",
            "value": "1200.0"
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "919122343344"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "currency": "INR",
          "value": "1500.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "INR",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "INR",
              "value": "300.0"
            }
          }
        ]
      },
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      },
      "payments": [
        {
          "status": "PAID",
          "type": "PRE-ORDER",
          "collected_by": "BPP",
          "params": {
            "amount": "1500",
            "currency": "INR",
            "transaction_id": "284724892494"
          }
        }
      ],
      "cancellation_terms": [
        {
          "cancel_by": {
            "duration": "P7D"
          },
          "cancellation_fee": {
            "amount": {
              "currency": "INR",
              "value": "100"
            }
          },

        }
      ],
      "refund_terms": [
        {
          "refund_eligible": "true",
          "fulfillment_state": {
            "code" : ""
          }
        }
      ]
    }
  }
}
```

### Checking the status of an order

- BAP can check the status of an order by calling the status api of the BPP.
- BAP needs to send the order id received during the on_confirm callback as part of the status call.
- BPP sends a response as part of an on_status callback.
- BPP can also send unsolicited on_status callback to the BAP.
- The current status of the order can be found in the fulfillment.state object of the on_status callback payload.

status
```
{
    "context": {
        "domain": "solar-rooftop:uei",
        "action": "status",
        "location": {
            "country": {
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
      "order_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

The on_status API is a callback sent by the BPP to inform the BAP about the current status of a previously confirmed order. it is used:
  - After the BAP sends a status request
  - Or unsolicited by BPP when status changes (e.g., order shipped, ride started, service in-progress)
  -  what to include in an on_status callback
      - id: Unique order ID (from on_confirm)
      - status: Status code (e.g., in-progress, completed, cancelled, pending)
      - fulfillments: Describes current fulfillment progress (optional but recommended)
      - updated_at: Timestamp of the last status change (optional but recommended)

on_status
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_status",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "b8c1e69c-fbbc-439b-a5de-2adcc74fa0da",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
        "descriptor": {
          "name": "Venky.Mahadevan@Bazaar"
        },
        "locations": [
          {
            "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
          }
        ]
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "descriptor": {
            "images": [
              {
                "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
              }
            ],
            "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
            "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
            "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that's free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won't spill a drop even when it's tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
          },
          "category_ids": [
            "c1"
          ],
          "quantity": {
            "selected" :{
              "count" : 2
            }
          },
          "price": {
            "listed_value": "1200.0",
            "currency": "INR",
            "value": "1200.0"
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "919122343344"
            }
          },
          "state": {
            "descriptor": {
              "code": "PACKING",
              "short_desc": "Order is getting packed ..."
            },
            "updated_at": "2023-05-26T05:23:04.443Z"
          }
        }
      ],
      "quote": {
        "price": {
          "currency": "INR",
          "value": "1500.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "INR",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "INR",
              "value": "300.0"
            }
          }
        ]
      },
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      },
      "payments": [
        {
          "status": "PAID",
          "type": "PRE-FULFILLMENT",
          "collected_by": "BPP",
          "params": {
            "transaction_id" : "raz816863816313",
            "amount": "1500",
            "currency": "INR",
            "bank_code": "INB0004321",
            "bank_account_number": "1234002341"
          }
        }
      ],
      "cancellation_terms": [
        {
          "cancellation_fee": {
            "amount": {
              "currency": "INR",
              "value": "100"
            }
          }
        }
      ]
    }
  }
}
```

### Tracking the order delivery

- If the order is being delivered, the users can track the order using the track api.
- BAP needs to send the order id as part of the track call.
- BAP can also send a callback url on which the BPP can send regular tracking updates.
- BPP sends a response using the on_track API.
- on_track can contain a url which can be used to track the shipment.
- on_track can also contain the exact location of the item, embedded in the on_track message.tracking.location
- Tracking URL gets active after the order is shipped.

track
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "track",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "fd585827-7607-4ce8-b257-9597b0ed0f52",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38"
  }
}
```

When the status or location of a delivery, ride, or fulfillment is changing frequently and should be relayed
Respond quickly to track requests with accurate location data
Use meaningful status codes (e.g., left-warehouse, out-for-delivery)
For high-frequency updates, send on_track periodically (e.g., every few minutes)

on_track
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_track",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "fd585827-7607-4ce8-b257-9597b0ed0f52",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "tracking": {
      "url": "https://abc/tracking/201f6fa2-a2f7-42e7-a2e5-8947398747",
      "status": "active"
    }
  }
}
```

### Updating part of an order

- Users can update parts of an existing order. It can be dependent on parts of an order the BPP allows to be updated.
- BAP can use the update/ api to update a part of an order.
- BAP needs to specify the update_target path of the part they wish to update. it is specified under message.order.update_target
- BAP also need to specify the details of the update target that need to be updated.
- BPP sends the updated order details as part of the on_update callback.

update
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "update",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "090adec2-696d-47c9-8452-797a91e7bebc",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
      "fulfillments": [
        {
          "customer": {
            "contact": {
              "phone": "+91-8056475647"
            }
          }
        }
      ]
    },
    "update_target": "order.fulfillments[0].customer.contact.phone"
  }
}
```

on_update
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_update",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "090adec2-696d-47c9-8452-797a91e7bebc",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
        "descriptor": {
          "name": "Venky.Mahadevan@Bazaar"
        },
        "locations": [
          {
            "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
          }
        ]
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "descriptor": {
            "images": [
              {
                "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
              }
            ],
            "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
            "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
            "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that's free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won't spill a drop even when it's tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
          },
          "category_ids": [
            "c1"
          ],
          "quantity": {
            "selected" :{
              "count" : 2
            }
          },
          "price": {
            "listed_value": "1200.0",
            "currency": "INR",
            "value": "1200.0"
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "+91-8056475647"
            }
          },
          "state": {
            "descriptor": {
              "code": "PACKING",
              "short_desc": "Order is getting packed ..."
            },
            "updated_at": "2023-05-26T05:23:04.443Z"
          }
        }
      ],
      "quote": {
        "price": {
          "currency": "INR",
          "value": "1500.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "INR",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "INR",
              "value": "300.0"
            }
          }
        ]
      },
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      },
      "payments": [
        {
          "status": "PAID",
          "type": "PRE-FULFILLMENT",
          "params": {
            "transaction_id" : "raz816863816313",
            "amount": "1500",
            "currency": "INR",
            "bank_code": "INB0004321",
            "bank_account_number": "1234002341"
          }
        }
      ],
      "cancellation_terms": [
        {
          "cancellation_fee": {
            "amount": {
              "currency": "INR",
              "value": "100"
            }
          }
        }
      ]
    }
  }
}
```

### Request for a support

- Users can request for a support using the support api.
- BAP needs to send the order id.
- Additionally, user's phone number can also be sent as part of the support call.
- BPP responds with a on_support callback. on_support can contain support phone, email or a url.

support
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "support",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "support": {
      "ref_id": "894789-43954",
      "phone": "+91 4444444444",
      "email": "me@gmail.com"
    }
  }
}
```
on_support
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_support",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "support": {
      "ref_id": "d4975df5-b18c-4772-80ad",
      "callback_phone": "+91 8765495826",
      "phone": "+91 9876543298",
      "email": "abcd.support@support.com"
    }
  }
}
```

### Cancelling an Order
Cancelling an order requires BAP to send a reason for cancellation. The reason can be a value from a pre defined enum or a text. Each BPP define their own list of reasons for cancelling an order. BAP needs to fetch those reasons defined by that BPP.

-There is a set of separate meta API endpoints which are supposed to be implemented by BAP and BPP to use this feature.
- BPP implements the get_cancellation_reasons/ api which can be called by the BAP.
- BPP sends the list of supported cancellation reasons by calling the cancellation_reasons/ callback api implemented by the BAP.

get_cancellation_reasons
Note: There is no message object in this API.

```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "get_cancellation_reasons",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  }
}
```

cancellation_reasons
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "cancellation_reasons",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "cancellation_reasons": [
      {
        "id": "c1",
        "descriptor": {
          "code": "CHANGE_OF_MIND",
          "short_desc": "Customer changed their mind after placing the order."
        }
      },
      {
        "id": "c",
        "descriptor": {
          "code": "FOUND_BETTER_PRICE",
          "short_desc": "Customer found a better price elsewhere."
        }
      },
      {
        "id": "c3",
        "descriptor": {
          "code": "DELIVERY_DELAY",
          "short_desc": "Expected delivery time is too long."
        }
      },
      {
        "id": "c4",
        "descriptor": {
          "code": "ORDER_BY_MISTAKE",
          "short_desc": "Customer placed the order by mistake."
        }
      }
    ]
  }
}
```

- Customer chooses one of the cancellation reasons received from the BPP.
- BAP call the cancel/ api to raise a cancel request to the order if the order still qualifies to be cancelled. BAP sends the reason selected by the customer.
- BPP processes the cancel request and if the order is elligible for cancellation, BPP updates the oprder details as per the cancellation terms agreed during the initialisation of the order.
- BPP uses the on_cancel api callback to provide the updated order details to the BAP.

cancel
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "cancel",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
    "cancellation_reason_id": "c3"
  }
}
```

on_cancel
```
{
  "context": {
    "domain": "local-retail",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_cancel",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "06f49c01-65b7-4754-8b37-ab4e33688154",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:45:40.421Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "12345678",
      "provider": {
        "id": "./retail.kirana/ind.blr/33@tourism-bpp-infra2.becknprotocol.io.provider",
        "descriptor": {
          "name": "Venky.Mahadevan@Bazaar"
        },
        "locations": [
          {
            "id": "./retail.kirana/ind.blr/1@tourism-bpp-infra2.becknprotocol.io.provider_location"
          }
        ]
      },
      "items": [
        {
          "id": "./retail.kirana/ind.blr/247@tourism-bpp-infra2.becknprotocol.io.item",
          "descriptor": {
            "images": [
              {
                "url": "https://tourism-bpp-infra2.becknprotocol.io/attachments/view/253.jpg"
              }
            ],
            "name": "Isothermal Stainless Steel Hiking Flask MH500 Yellow - Water bottle",
            "short_desc": "InstaCuppa Stainless Steel Thermos Flask Water Bottle with Sports Sipper Lid, Double Walled Vacuum Insulation",
            "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that’s free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won’t spill a drop even when it’s tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
          },
          "category_ids": [
            "c1"
          ],
          "quantity": {
            "selected" :{
              "count" : 2
            }
          },
          "price": {
            "listed_value": "1200.0",
            "currency": "INR",
            "value": "1200.0"
          }
        }
      ],
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "city": {
                  "name": "Gangamuthanahalli"
                },
                "state": {
                  "name": "Karnataka"
                },
                "country": {
                  "code": "IND"
                },
                "area_code": "75001"
              },
              "contact": {
                "phone": "919246394908",
                "email": "nc.rehman@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "Motiur Rehman"
            },
            "contact": {
              "phone": "919122343344"
            }
          },
          "state": {
            "descriptor": {
              "code": "CANCELLED"
            }
          }
        }
      ],
      "quote": {
        "price": {
          "currency": "INR",
          "value": "1500.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "INR",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "INR",
              "value": "300.0"
            }
          }
        ]
      },
      "billing": {
        "name": "Motiur Rehman",
        "phone": "9191223433",
        "email": "nc.rehman@gmail.com",
        "address": "123, Terminal 1, Kempegowda Int'l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
        "city": {
          "name": "Gangamuthanahalli"
        },
        "state": {
          "name": "Karnataka"
        }
      },
      "payments": [
        {
          "type": "PRE-ORDER",
          "status": "PAID",
          "collected_by": "BPP",
          "params": {
            "amount": "1500",
            "currency": "INR",
            "transaction_id": "284724892494"
          }
        },
        {
          "type": "REFUND",
          "status": "INITIATED",
          "params": {
            "amount": "1400",
            "currency": "INR"
          }
        },
      ],
      "cancellation_terms": [
        {
          "cancel_by": {
            "duration": "P7D"
          },
          "cancellation_fee": {
            "amount": {
              "currency": "INR",
              "value": "100"
            }
          },

        }
      ],
      "refund_terms": [
        {
          "refund_eligible": "true",
          "fulfillment_state": {
            "code" : ""
          }
        }
      ],
      "cancellation": {
        "cancelled_by": "CONSUMER",
        "time": "2025-03-06T09:45:40.421Z",
        "reason": {
          "id": "c3",
          "descriptor": {
            "code": "DELIVERY_DELAY",
            "short_desc": "Expected delivery time is too long."
          }
        }
      }
    }
  }
}
```


### Providing rating for an Order

Providing rating to an order requires BAP to send the rating category to BPP. The rating category can be a value from a pre defined enum. Each BPP define their own list of rating categories. BAP needs to fetch those categories defined by that BPP.

-There is a set of separate meta API endpoints which is supposed to be implemented by BAP and BPP to use this feature.
- BPP implements the get_rating_categories/ api which can be called by the BAP.
- BPP sends the supported rating_categories by calling the rating_categories/ callback api implemented by the BAP.

get_rating_categories
Note: There is no message object in this API.

get_rating_categories
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "get_rating_categories",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  }
}
```

rating_categories
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "rating_categories",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "1f507212-23fe-4acc-a34e-d25b459ad105",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "rating_categories": [
      "Item",
      "Fulfillment",
      "Provider",
      "Agent"
    ]
  }
}
```


- Customer chooses one of the rating category received from the BPP.
- BAP call the rating/ api to provide a rating for the order. BAP selects a category provided by the BPP.
- BPP processes the rating request.
- BPP uses the on_rating api callback to provide the BAP with a feedback_form that BAP chooses to fill.

rating
```
{
    "context": {
        "domain": "retail:open-amsterdam",
        "location": {
            "country": {
              "code": "IND"
            },
            "city": {
              "code": "std:080"
            }
          },
        "action": "rating",
        "version": "1.1.0",
        "bap_id": "farm-fresh-bap-id",
        "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
        "bpp_id": "farm-fresh-bpp-subId",
        "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
        "message_id": "b0c5b14f-a7db-4329-b7e2-42a5f6486085",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T10:14:10.295Z",
        "ttl": "PT10M"
    },
    "message": {
        "ratings": [
            {
                "id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
                "rating_category": "Order",
                "value": "8"
            }
        ]
    }
}
```

on_rating
```
{
  "context": {
    "domain": "retail:open-amsterdam",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "code": "std:080"
      }
    },
    "action": "on_rating",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "b0c5b14f-a7db-4329-b7e2-42a5f6486085",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "feedback_form": {
      "form": {
        "url": "https://inds-network-bpp.becknprotocol.io/feedback/portal"
      }
    }
  }
}
```


## Integrating with your software

This section gives general walkthrough of how you would integrate your software with the Beckn network (say the sandbox environment). Refer to the [starter kit](https://github.com/beckn/missions/blob/main/docs/starter_kit/starter_kit.md) for details on how to register with the sandbox and get credentials.

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

![Seeker platform testing sandbox ](https://github.com/beckn/missions/blob/main/docs/assets/images/seeker_deployment.png)

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

![Provider platform testing sandbox](https://github.com/beckn/missions/blob/main/docs/assets/images/provider_deployment.png)


## Link to the domain specific implementation guide

Retail:
  - https://github.com/beckn/missions/blob/main/OPEN-AMSTERDAM/implementation_guide_open_amsterdam.md
  - https://github.com/beckn/missions/blob/main/OPEN-KENYA/implementation_guide_products.md
  - https://github.com/beckn/missions/blob/main/UAI/implementation-guides/retail/implementation_guide_equipment_purchase.md

Skilling and Education:
  - https://github.com/beckn/missions/blob/main/OPEN-BELEM/implementation_guide_belem_courses.md
  - https://github.com/beckn/missions/blob/main/OPEN-GAMBIA/implementation_guide_gambia_courses.md
  - https://github.com/beckn/missions/blob/main/OPEN-KENYA/implementation_guide_learning.md

Services:
  - https://github.com/beckn/missions/blob/main/OPEN-KENYA/implementation_guide_services.md
  - https://github.com/beckn/missions/blob/main/UAI/implementation-guides/advisory/implementation_guide_disease_control.md
  - https://github.com/beckn/missions/blob/main/UAI/implementation-guides/retail/implementation_guide_equipment_rental.md
  - https://github.com/beckn/missions/blob/main/UEI/implementation-guides/EV-charging/implementation_guide_charging.md

Discovery:
  - https://github.com/beckn/missions/blob/main/UAI/implementation-guides/advisory/implementation_guide_price_listing.md
  - https://github.com/beckn/missions/blob/main/UAI/implementation-guides/advisory/implementation_guide_weather_data.md
  - https://github.com/beckn/missions/blob/main/UAI/implementation-guides/schemes/implementation_guide_scheme_discovery.md
  - https://github.com/beckn/missions/blob/main/VISTAAR/implementation_guide_knowledge_advisory.md

