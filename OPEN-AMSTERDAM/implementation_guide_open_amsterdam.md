# Open Amsterdam Implementation Guide

#### Version 1.1

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 07-08-2024 | 1.0     | Initial Version                                     |
| 04-03-2025 | 1.1     | Action schema correction                            |


## Introduction

This document provides material that helps network participants build and integrate their application with the open amsterdam nework for retail purposes. This document is part of the starter kit that provides information about the network, learning resources, network participant checklist etc. This document only focuses on the implementation of the seeker/provider platform. It assumes the reader has a good overview of the Beckn network, its APIs, the overall structure of the schema etc.

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

![Open Amsterdam Outcome Visualization](images/open_amsterdam_outcome_visualization.svg)


### Use case - Discovery, order and fulfillment of Local Retail Products

This use case follows Nina, a 45-year-old bakery owner, as she shops for bulk ingredients.

**Discovery**:

- Nina opens the "Open Amsterdam" app/website on her mobile phone, creates an account and logs in.

- She browses catalogs of different stores in Haarlemmerbuurt, looking for a store that has the baking ingredients she needs.

- Nina explores the shop's online catalog, navigating through various categories and checking product details to ensure they meet her requirements.

**Order**:

- Nina adds items to her cart and proceeds to checkout.

- She selects her preferred fulfillment option:
  - Store pickup, or
  - Delivery (if offered by the retailer)

- She enters her delivery address if choosing delivery.

- She selects a payment method:
  - Cash on delivery
  - Payment link provided by the retailer 
  - Other options (to be discussed)

- Before confirming, she reviews her purchase including:
  - Price
  - Product descriptions
  - Images

- Nina completes the transaction and receives a confirmation email with purchase details, estimated delivery time, and a note thanking her for supporting their business.

**Fulfillment**:

- The seller processes Nina's order.

- Nina receives notifications in her app about the order status - when it's packed, shipped, and available for pickup/delivery.

- Based on her chosen fulfillment option, Nina either picks up the order at the store or receives it via delivery (tracked offline).

**Post Fulfillment**:

- Nina provides ratings for the seller(s) and/or products.

- If needed, she can raise customer support issues via phone, email and/or chat.

- Inspired by her positive experience, Nina actively engages with the local community, attending events and participating in discussions about supporting local businesses. She continues to support the online market of Haarlemmerbuurt, Amsterdam.

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

### Use case - Discovery, order and fulfillment of Local Retail Products

- The flow allows users to discover local retail stores, browse their catalogs, place orders and track fulfillment
- Orders can be fulfilled through store pickup or delivery based on retailer capabilities
- Payment options include cash on delivery and online payment methods

## API Calls and Schema

### search

Search request can contain one or more search criterion within it. Use the following list on how to specify the criterion:

- The location to search around is specified in the message->intent->fulfillment->stops[0]->location field
- If searching by free text, it is specified in message->intent->descriptor->name
- If searching by category, it is specified in message->intent->category->descriptor->code

```json
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
    "action": "search",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap.com",
    "bap_uri": "https://farm-fresh-bap.com",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "timestamp": "2023-11-06T09:41:09.673Z"
  },
  "message": {
    "intent": {
      "category": {
        "descriptor": {
          "code": "electronics"
        }
      },
      "item": {
        "descriptor": {
          "name": "earphone"
        }
      },
      "fulfillment": {
        "type": "DELIVERY",
        "stops": [
          {
            "location": {
              "gps": "28.4594965,77.0266383"
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
- Each item is the catalog listing for a an item to purchase

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
                "long_desc": "<div> <ul> <li>ULTRA MODERN DESIGN - Our thermos bottle is crafted with a unique and modern design. Gone are the days of old and boring flasks. Guaranteed to impress your colleagues, friends & family.</li> <li>ADVANCED TEMPERATURE CONTROL – A double-wall, vacuum-insulated design helps lock in heat for up to 12 hours and cold for up to 24!</li> <li>ELIMINATES CONDENSATION – Offering improved grip and control, these innovative dual-layer bottles offer a slip-resistant surface that's free of sweat and condensation..</li> <li>LEAK-PROOF and ECO-FRIENDLY – Remove, and clean, the large, screw on lid provides faster access to water inside and won't spill a drop even when it's tipped upside or put in your gym bag.</li> <li>The distress quilted jacket is a versatile fashion choice you can wear on any occasion. A style essential piece for Women which will reveal your strong sense of personality</li> </ul> <div> <p><b>Product Details</b></p> <ul> <li>Advanced Temperature Retention.This thermos water bottle ensures your beverages will remain hot or cold for a long time.Hot for up to 12 hours.Cold for up to 24 hours.</li> <li>Retains Original Flavors.Vacuum insulation ensures this travel thermos water bottle is airtight and retains the original flavor of your beverages.Also, this bottle is B.P.A Free.</li> <li>Premium Quality Materials.This stylish bottle is a double-walled vacuum insulated and made from premium 304-grade stainless steel - which makes this flask bottle.</li> </ul> </div>"
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

### select

**sending a select request**

- Choose the item(s) from the catalog and request quote
- The chosen items are specified in message->order->items

```json
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
            "selected": {
              "count": 2
            }
          }
        }
      ],
      "fulfillments": [
        {
          "type": "DELIVERY",
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
    "domain": "retail:open-amsterdam",
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

### init

**send init request**

- The draft order including billing details.
- Billing details specified in message->order->billing

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

### on_init

- Contains payment terms. Payment terms specified in message->order->payments
- Cancellation terms specified in message->order->cancellation_terms
- Here we show the BPP as payment collector. In case the BAP specifies that it collects the payment in the init, the url field within payments will be empty

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
          "params": {
            "amount": "1500",
            "currency": "INR",
            "bank_code": "INB0004321",
            "bank_account_number": "1234002341"
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
    "domain": "retail:open-amsterdam",
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
    "message_id": "58d0a142-68e3-48d9-92e8-fbc791e44949",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.280Z"
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
          "type": "PRE-FULFILLMENT",
          "params": {
            "transaction_id" : "raz816863816313",
            "amount": "1500",
            "currency": "INR",
            "bank_code": "INB0004321",
            "bank_account_number": "1234002341"
          }
        }
      ]
    }
  }
}
```

### on_confirm

- Order confirmed.

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
    "action": "on_confirm",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "58d0a142-68e3-48d9-92e8-fbc791e44949",
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

### status

- Request for status on order. order_id is specifiedin message->order_id

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
    "action": "status",
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
    "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38"
  }
}
```

### on_status

- Status of requested order.
- Primarily the fulfillment status is specified in message->order->fulfillments[]->state


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

### update 

- Update the state of the order

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

### on_update 

- Confirmation of succesfull updated operation
- State in message->order->fulfillments[]->state

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


### support

- Request for support information.
- If regarding a specific order, specify the order_id in message->support->ref_id

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

### on_support

- Contains support information. If integrated with a CMS, the URL could contain order specific support info
- In all other cases, contains general support info (message->support)

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

### track

- Request for a tracking URL for the order. Order Id must be specified in message->order_id

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

### on_track

- Tracking URL after the order is shipped
- The tracking URL will be in message->tracking->url

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

### cancel

- Request to cancel an order. order_id to be specified in message->order_id

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
    "message_id": "c53b2e4b-ff6d-434b-ba2c-329a3260b3e9",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T10:14:10.295Z",
    "ttl": "PT10M"
  },
  "message": {
    "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
    "cancellation_reason_id": "4",
    "descriptor": {
      "short_desc": "Order delayed"
    }
  }
}
```

### on_cancel

- Confirmation of cancelled order. The cancelled order will be in message->order


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
    "action": "on_cancel",
    "version": "1.1.0",
    "bap_id": "farm-fresh-bap-id",
    "bap_uri": "https://55a6-124-123-32-28.ngrok-free.app",
    "bpp_id": "farm-fresh-bpp-subId",
    "bpp_uri": "https://4e21-124-123-32-28.ngrok-free.app",
    "message_id": "c53b2e4b-ff6d-434b-ba2c-329a3260b3e9",
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
              "code": "CANCELLED",
              "short_desc": "Cancelled due to ..."
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

### rating

- Rate different categories including fulfillment. Specify in message->ratings
- Value should be a decimal between 0 and 5.
- Id can be the order id.

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

### on_rating

- Confirmation of rating.
- Optionally send a form for additional input (message->feedback_form). Empty message otherwise.

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

## Taxonomy and layer 2 configuration

- **Product Categories** - The values allowed for product categories are specified in the Open Amsterdam retail taxonomy. This includes categories like:
  - food-ingredients
  - fresh-produce
  - household-items
  - electronics

- **Fulfillment Types** supported are:
  - Store pickup
  - Home delivery (where available)

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

- [Postman collection for Open Amsterdam Retail](./postman/open_amsterdam_retail_postman_collection.json)
- [Layer2 config for Open Amsterdam Retail](./layer2/open_amsterdam_retail_1.1.0.yaml)
- When installing layer2 using Beckn-ONIX use this web address (https://raw.githubusercontent.com/beckn/missions/main/OPEN-AMSTERDAM/layer2/open_amsterdam_retail_1.1.0.yaml)

## Sandbox Details(TBD)

