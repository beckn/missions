# 1. Implementation Guide

This document contains the REQUIRED and RECOMMENDED standard functionality that must be implemented by any Agritech Service Provider Platform a.k.a BPPs and Agritech Consumer Platform a.k.a BAPs.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

## 1.1 Discovery of Services

### 1.1.1 Recommendations for BPPs
The following recommendations need to be considered when implementing discovery functionality for an Agritech BPP

- REQUIRED. The BPP MUST implement the `search` endpoint to receive an `Intent` object sent by BAPs
- REQUIRED. The BPP MUST return a catalog of Farm services (Farm equipment renting / labours) on the `on_search` callback endpoint specified in the `context.bap_uri` field of the `search` request body.
- REQUIRED. The BPP MUST map its Farm services to the `Item` schema.
- REQUIRED. Any Agritech service provider-related information like name, logo, short description must be mapped to the `Provider.descriptor` schema
- REQUIRED. Any form that must be filled before receiving a quotation must be mapped to the `XInput` schema
- REQUIRED. If the platform wants to group its products/service under a specific category, it must map each category to the `Category` schema
- REQUIRED. Any service fulfillment-related information MUST be mapped to the `Fulfillment` schema.
- REQUIRED. If the BPP does not want to respond to a search request, it MUST return a `ack.status` value equal to `NACK`
- RECOMMENDED. Upon receiving a `search` request, the BPP SHOULD return a catalog that best matches the intent. This can be done by indexing the catalog against the various probable paths in the `Intent` schema relevant to typical Agritech use cases

### 1.1.2 Recommendations for BAPs
- REQUIRED. The BAP MUST call the `search` endpoint of the BG to discover multiple BPPs on a network
- REQUIRED. The BAP MUST implement the `on_search` endpoint to consume the `Catalog` objects containing Farm services cataloge sent by BPPs.
- REQUIRED. The BAP MUST expect multiple catalogs sent by the respective Agritech Providers on the network
- REQUIRED. The Farm services can be found in the `Catalog.providers[].items[]` array in the `on_search` request
- REQUIRED. If the `catalog.providers[].items[].xinput` object is present, then the BAP MUST redirect the user to, or natively render the form present on the link specified on the `items[].xinput.form.url` field.
- REQUIRED. If the `catalog.providers[].items[].xinput.required` field is set to `"true"` , then the BAP MUST NOT fire a `select`, `init` or `confirm` call until the form is submitted and a successful response is received
- RECOMMENDED. If the `catalog.providers[].items[].xinput.required` field is set to `"false"` , then the BAP SHOULD allow the user to skip filling the form


### Example
A search request for a Farm services may look like this
```
{
  "context": {
    "domain": "agrinet",
    "location": {
      "country": {
        "name": "kenya"
      },
      "city": {
        "name": "Kirinyaga"
      }
    },
    "action": "search",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "timestamp": "2023-11-06T09:41:09.673Z",
    "ttl": "PT10M"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {
          "name": "ploughing"
        }
      }
    }
  }
}
```

An example catalog of Farm services may look like this
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
            "name": "kenya"
          },
          "city" : {
            "name": "kirinyaga"
          }
      },
      "action": "on_search",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "catalog": {
        "descriptor": {
          "name": "Kuza-AgriNet-BPP",
          "short_desc" : "",
          "long_desc" : "",
          "images": [
            {
              "url": "https://image_url"
            }
          ]
        },
        "providers": [
          {
            "id": "provider_id_1",
            "descriptor": {
              "name": "Provider 1",
              "short_desc" : "",
              "long_desc" : "",
              "images": [
                  {
                  "url": "https://image_url"
                  }
              ]
            },
            "locations": [
              {
                "id": "location 1",
                "gps": "12.909955,77.596316"
              }
            ],
            "categories": [
              {
                "id": "c1",
                "descriptor": {
                  "code": "Agri Services",
                  "name": "Agri Services"
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
            ]
          }
        ],
        "items": [
          {
            "id": "item_id",
            "descriptor": {
              "images": [
                {
                  "url": "https://image_url"
                }
              ],
              "name": "Ploughing",
              "short_desc": "Ploughing also turns over the upper layer of soil, bringing fresh nutrients to the surface. It buries weeds and the remains of previous crops, allowing them to break down.",
              "long_desc": "Ploughing also turns over the upper layer of soil, bringing fresh nutrients to the surface. It buries weeds and the remains of previous crops, allowing them to break down."
            },
            "matched": false,
            "recommended": true,
            "price": {
              "listed_value": "1300.0",
              "currency": "KSH",
              "value": "1200.0"
            },
            "rating": "5",
            "creator" : {
              "descriptor" : {
                "name" : "Hello Tractor"
              }
            },
            "location_ids": [
              "location_id1",
              "location_id2"
            ],
            "category_ids": [
              "c1", 
              "c2"
            ],
            "fulfillment_id": [
              "f1",
              "f2"
            ],
            "tags": [
              {
                "descriptor": {
                  "code" : "stage",
                  "name" : "Stages applicable"
                },
                "list": [
                  {
                    "value": "Land Preparation"
                  },
                  {
                    "value": "Acres"
                  }
                ]
              }
            ]
          }
        ],
        "offer": {
            "descriptor": {
                "name": "hire1get1free"
              },
              "item_ids": [
                "item_id1",
                "item_id2"
            ]
        }
      }
    }
  }     
```

## 1.2 Ordering Farm services
This section provides recommendations for implementing the APIs related to selecting & ordering Farm services.

### 1.2.1 Recommendations for BPPs

#### 1.2.1.1 Selecting a Farm service from the catalog
- REQUIRED. The BPP MUST implement the `select` endpoint on the url specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. If the Agritech provider has successfully received the information submitted by the Agritech consumer, the BPP must return an acknowledgement with `ack.status` set to `ACK` in response to the `select` request
- REQUIRED. If the Agritech provider has returned a successful acknowledgement to a `select` request, it MUST send the offer encapsulated in an `Order` object

#### 1.2.1.2 Initializing the Order
- REQUIRED. The BPP MUST implement the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.1.3 Confirming the Order
- REQUIRED. The BPP MUST implement the `confirm` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.2.2 Recommendations for BAPs

#### 1.2.2.1 Selecting a Farm service from the catalog
- REQUIRED. The BAP MUST implement the `on_select` endpoint on the url specified in the `context.bap_uri` field sent during `select`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.2.2  Initializing the Order
- REQUIRED. The BAP MUST hit the `init` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_init` endpoint on the url specified in  the `context.bap_uri` field sent during `init`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.2.2.3 Confirming the Order
- REQUIRED. The BAP MUST hit the `confirm` endpoint on the url specified in  the `context.bpp_uri` field sent during `on_search`. 
- REQUIRED. The BAP MUST implement the `on_confirm` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `confirm`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.2.3 Example Workflow

### 1.2.3 Example Requests

Below is an example of a `select` request
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
            "name": "kenya"
          },
          "city" : {
            "name": "kirinyaga"
          }
      },
      "action": "select",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "order": {
        "provider": {
          "id": "provider_id"
        },
        "items": [
          {
            "id": "item_id",
            "quantity": {
              "selected": {
                "count": 3,
                "measure" : {
                  "unit" : "Acres"
                }
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

Below is an example of an `on_select` callback
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
            "name": "kenya"
          },
          "city" : {
            "name": "kirinyaga"
          }
      },
      "action": "on_select",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "order": {
        "providers": [
          {
            "id": "provider_id_1",
            "descriptor": {
              "name": "Provider 1",
              "short_desc" : "",
              "long_desc" : "",
              "images": [
                  {
                  "url": "https://image_url"
                  }
              ]
            },
            "locations": [
              {
                "id": "location 1",
                "gps": "12.909955,77.596316"
              }
            ],
            "categories": [
              {
                "id": "c1",
                "descriptor": {
                  "code": "herbicide",
                  "name": "herbicide"
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
            ]
          }
        ],
        "items": [
          {
            "id": "item_id",
            "descriptor": {
              "images": [
                {
                  "url": "https://image_url"
                }
              ],
              "name": "Potasun",
              "short_desc": "Selective post-emergence herbicide for weed control in Irish potatoes. Mixing: 200mls/20ltr. Available in 1ltr pack",
              "long_desc": "Potasun 50EC is a selective herbicide that controls annual and perennial grasses and broad leaf weeds in tomato, cassava, Irish potato, and cocoyam. It is a selective earlier post-emergence herbicide"
            },
            "matched": true,
            "quantity": {
              "selected": {
                "count": 3,
                "measure" : {
                  "unit" : "Acres"
                }
              }
            },
            "price": {
              "listed_value": "1200.0",
              "currency": "KSH",
              "value": "1200.0"
            },
            "rating": "5",
            "creator" : {
              "descriptor" : {
                "name" : "Hello Tractor"
              }
            },
            "recommended": true,
            "location_ids": [
              "location_id1",
              "location_id2"
            ],
            "category_ids": [
              "c1", 
              "c2"
            ],
            "tags": [
              {
                "descriptor": {
                  "code": "stage",
                  "name": "stages applicable"
                },
                "list": [
                  {
                    "value": "Land Preparation"
                  },
                  {
                    "value": "Sowing"
                  }
                ]
              },
              {
                "descriptor": {
                  "code" : "disease",
                  "name": "diseases covered"
                },
                "list": [
                  {
                    "value": "Baterial Wilt"
                  }
                ]
              }
            ]
          }
        ],
        "offer": {
            "descriptor": {
                "name": "buy1get1"
              },
              "item_ids": [
                "item_id1",
                "item_id2"
            ]
        },
        "quote": {
            "price": {
            "currency": "KSH",
            "value": "1500.0"
            },
            "breakup": [
            {
                "title": "base-price",
                "price": {
                "currency": "KSH",
                "value": "1200.0"
                }
            },
            {
                "title": "taxes",
                "price": {
                "currency": "KSH",
                "value": "300.0"
                }
            }
          ]
        },
        "fulfillments": [
          {
            "id": "f1"
          }
        ]
      }
    }
  }
```


Below is an example of a `init` request
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
          "name": "kenya"
        },
        "city": {
          "name": "Kirinyaga"
        }
      },
      "action": "init",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z"
    },
    "message": {
      "order": {
        "provider": {
          "id": "provider_id"
        },
        "items": [
          {
            "id": "item_id",
            "quantity": {
              "selected": {
                "count": 3,
                "measure" : {
                  "unit" : "Acres"
                }
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
                  "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
                  "city": {
                    "name": "Nduini"
                  },
                  "county": {
                    "name": "Kirinyaga"
                  },
                  "country": {
                    "name": "Kenya"
                  },
                  "area_code": "75001"
                },
                "time": {
                  "label": "preferable time slot",
                  "timestamp": "2024-01-17T10:58:43.451Z",
                  "range": {
                    "start": "2024-01-17T16:10:00.430Z",
                    "end": "2024-01-17T16:10:00.430Z"
                  }
                },
                "contact": {
                  "phone": "2547463949",
                  "email": "nc.njugunu@gmail.com"
                }
              }
            ]
          }
        ],
        "billing": {
          "name": "John Githuru",
          "phone": "2542343344",
          "email": "nc.njugunu@gmail.com",
          "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
          "city": {
            "name": "Nduini"
          },
          "county": {
            "name": "Kirinyaga"
          }
        }
      }
    }
  }
```

Below is an example of an `on_init` callback
```
{
  "context": {
    "domain": "agrinet",
    "location": {
      "country": {
        "name": "kenya"
      },
      "city": {
        "name": "kirinyaga"
      }
    },
    "action": "on_init",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:41:09.708Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "providers": [
        {
          "id": "provider_id_1",
          "descriptor": {
            "name": "Provider 1",
            "short_desc": "",
            "long_desc": "",
            "images": [
              {
                "url": "https://image_url"
              }
            ]
          },
          "locations": [
            {
              "id": "location 1",
              "gps": "12.909955,77.596316"
            }
          ],
          "categories": [
            {
              "id": "c1",
              "descriptor": {
                "code": "herbicide",
                "name": "herbicide"
              }
            }
          ]
        }
      ],
      "items": [
        {
          "id": "item_id",
          "descriptor": {
            "images": [
              {
                "url": "https://image_url"
              }
            ],
            "name": "Potasun",
            "short_desc": "Selective post-emergence herbicide for weed control in Irish potatoes. Mixing: 200mls/20ltr. Available in 1ltr pack",
            "long_desc": "Potasun 50EC is a selective herbicide that controls annual and perennial grasses and broad leaf weeds in tomato, cassava, Irish potato, and cocoyam. It is a selective earlier post-emergence herbicide"
          },
          "quantity": {
            "selected": {
              "count": 3,
              "measure": {
                "unit": "Acres"
              }
            }
          },
          "matched": true,
          "price": {
            "listed_value": "1200.0",
            "currency": "KSH",
            "value": "1200.0"
          },
          "rating": "5",
          "creator": {
            "descriptor": {
              "name": "Hello Tractor"
            }
          },
          "recommended": true,
          "location_ids": [
            "location_id1",
            "location_id2"
          ],
          "category_ids": [
            "c1",
            "c2"
          ],
          "fulfillment_id": [
            "f1",
            "f2"
          ],
          "tags": [
            {
              "descriptor": {
                "code": "stage",
                "name": "stages applicable"
              },
              "list": [
                {
                  "value": "Land Preparation"
                }
              ]
            },
            {
              "descriptor": {
                "code": "disease",
                "name": "diseases covered"
              },
              "list": [
                {
                  "value": "Baterial Wilt"
                }
              ]
            }
          ]
        }
      ],
      "offer": {
        "descriptor": {
          "name": "buy1get1"
        },
        "item_ids": [
          "item_id1",
          "item_id2"
        ]
      },
      "quote": {
        "price": {
          "currency": "KSH",
          "value": "1700.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "KSH",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "KSH",
              "value": "300.0"
            }
          },
          {
            "title": "delivery-charges",
            "price": {
              "currency": "KSH",
              "value": "200.0"
            }
          }
        ]
      },
      "fulfillments": [
        {
          "type": "Delivery",
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
                "city": {
                  "name": "Nduini"
                },
                "county": {
                  "name": "Kirinyaga"
                },
                "country": {
                  "name": "Kenya"
                },
                "area_code": "75001"
              },
              "time": {
                "label": "preferable time slot",
                "timestamp": "2024-01-17T10:58:43.451Z",
                "range": {
                  "start": "2024-01-17T16:10:00.430Z",
                  "end": "2024-01-17T16:10:00.430Z"
                }
              },
              "contact": {
                "phone": "2547463949",
                "email": "nc.njugunu@gmail.com"
              }
            }
          ],
          "agent": {
            "person": {
              "id": "string",
              "url": "string",
              "name": "string"
            },
            "contact": {
              "phone": "string",
              "email": "string"
            }
          },
          "customer": {
            "person": {
              "name": "John Githuru"
            },
            "contact": {
              "phone": "2542343344"
            }
          }
        }
      ],
      "billing": {
        "name": "John Githuru",
        "phone": "2542343344",
        "email": "nc.njugunu@gmail.com",
        "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
        "city": {
          "name": "Nduini"
        },
        "county": {
          "name": "Kirinyaga"
        }
      },
      "payments": [
        {
          "id": "string",
          "collected_by": "string",
          "status": "NOT-PAID",
          "type": "PRE-FULFILLMENT",
          "params": {
            "amount": "1700",
            "currency": "KSH",
            "virtual_payment_address": "string"
          }
        }
      ]
    }
  }
}

```

Below is an example of a `confirm` request
```
{
  "context": {
    "domain": "agrinet",
    "location": {
      "country": {
        "name": "kenya"
      },
      "city": {
        "name": "Kirinyaga"
      }
    },
    "action": "confirm",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:41:09.708Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "provider_id"
      },
      "items": [
        {
          "id": "item_id",
          "quantity": {
            "selected": {
              "count": 3,
              "measure": {
                "unit": "Acres"
              }
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
                "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
                "city": {
                  "name": "Nduini"
                },
                "county": {
                  "name": "Kirinyaga"
                },
                "country": {
                  "name": "Kenya"
                },
                "area_code": "75001"
              },
              "time": {
                "label": "preferable time slot",
                "timestamp": "2024-01-17T10:58:43.451Z",
                "range": {
                  "start": "2024-01-17T16:10:00.430Z",
                  "end": "2024-01-17T16:10:00.430Z"
                }
              },
              "contact": {
                "phone": "2547463949",
                "email": "nc.njugunu@gmail.com"
              }
            }
          ]
        }
      ],
      "billing": {
        "name": "John Githuru",
        "phone": "2542343344",
        "email": "nc.njugunu@gmail.com",
        "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
        "city": {
          "name": "Nduini"
        },
        "county": {
          "name": "Kirinyaga"
        }
      },
      "payments": [
        {
          "id": "string",
          "collected_by": "string",
          "status": "PAID",
          "type": "PRE-FULFILLMENT",
          "params": {
            "transaction_id": "string",
            "amount": "1700",
            "currency": "KSH",
            "virtual_payment_address": "string"
          }
        }
      ]
    }
  }
}

```
Below is an example of an `on_confirm` callback
```
{
  "context": {
    "domain": "agrinet",
    "location": {
      "country": {
        "name": "kenya"
      },
      "city": {
        "name": "kirinyaga"
      }
    },
    "action": "on_confirm",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:41:09.708Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "order_id",
      "status": "active",
      "providers": [
        {
          "id": "provider_id_1",
          "descriptor": {
            "name": "Provider 1",
            "short_desc": "",
            "long_desc": "",
            "images": [
              {
                "url": "https://image_url"
              }
            ]
          },
          "locations": [
            {
              "id": "location 1",
              "gps": "12.909955,77.596316"
            }
          ],
          "categories": [
            {
              "id": "c1",
              "descriptor": {
                "code": "herbicide",
                "name": "herbicide"
              }
            }
          ]
        }
      ],
      "items": [
        {
          "id": "item_id",
          "descriptor": {
            "images": [
              {
                "url": "https://image_url"
              }
            ],
            "name": "Potasun",
            "short_desc": "Selective post-emergence herbicide for weed control in Irish potatoes. Mixing: 200mls/20ltr. Available in 1ltr pack",
            "long_desc": "Potasun 50EC is a selective herbicide that controls annual and perennial grasses and broad leaf weeds in tomato, cassava, Irish potato, and cocoyam. It is a selective earlier post-emergence herbicide"
          },
          "quantity": {
            "selected": {
              "count": 3,
              "measure": {
                "unit": "Acres"
              }
            }
          },
          "matched": true,
          "price": {
            "listed_value": "1200.0",
            "currency": "KSH",
            "value": "1200.0"
          },
          "rating": "5",
          "creator": {
            "descriptor": {
              "name": "Hello Tractor"
            }
          },
          "recommended": true,
          "location_ids": [
            "location_id1",
            "location_id2"
          ],
          "category_ids": [
            "c1",
            "c2"
          ],
          "fulfillment_id": [
            "f1",
            "f2"
          ],
          "cancellation_terms": [
            {
              "fulfillment_state": {
                "descriptor": {
                  "name": "string",
                  "code": "string",
                  "short_desc": "string",
                  "long_desc": "string"
                }
              },
              "reason_required": true,
              "cancellation_fee": {
                "percentage": "10.00",
                "amount": {
                  "currency": "string",
                  "value": "100"
                }
              },
              "xinput": {
                "form": {
                  "url": "string",
                  "data": {
                    "additionalProp1": "string",
                    "additionalProp2": "string",
                    "additionalProp3": "string"
                  },
                  "mime_type": "text/html",
                  "submission_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
                },
                "required": true
              }
            }
          ],
          "return_terms": [
            {
              "fulfillment_state": {
                "name": "string",
                "code": "string"
              },
              "return_eligible": true,
              "return_time": {
                "label": "string",
                "timestamp": "2024-01-04T07:12:23.580Z",
                "duration": "string",
                "range": {
                  "start": "2024-01-04T07:12:23.580Z",
                  "end": "2024-01-04T07:12:23.580Z"
                },
                "days": "string"
              },
              "fulfillment_managed_by": "CONSUMER"
            }
          ],
          "refund_terms": [
            {
              "fulfillment_state": {
                "name": "string",
                "code": "string"
              },
              "refund_eligible": true,
              "refund_within": {
                "label": "string",
                "timestamp": "2024-01-04T07:12:23.579Z",
                "duration": "string",
                "range": {
                  "start": "2024-01-04T07:12:23.579Z",
                  "end": "2024-01-04T07:12:23.579Z"
                },
                "days": "string"
              },
              "refund_amount": {
                "currency": "string",
                "value": "100.00"
              }
            }
          ],
          "replacement_terms": [
            {
              "fulfillment_state": {
                "name": "string",
                "code": "string"
              },
              "replace_within": {
                "label": "string",
                "timestamp": "2024-01-04T07:12:23.580Z",
                "duration": "string",
                "range": {
                  "start": "2024-01-04T07:12:23.580Z",
                  "end": "2024-01-04T07:12:23.580Z"
                },
                "days": "string",
                "schedule": {
                  "frequency": "string",
                  "holidays": [
                    "2024-01-04T07:12:23.580Z"
                  ],
                  "times": [
                    "2024-01-04T07:12:23.580Z"
                  ]
                }
              }
            }
          ],
          "tags": [
            {
              "descriptor": {
                "code": "stage",
                "name": "stages applicable"
              },
              "list": [
                {
                  "value": "Land Preparation"
                }
              ]
            },
            {
              "descriptor": {
                "code": "disease",
                "name": "diseases covered"
              },
              "list": [
                {
                  "value": "Baterial Wilt"
                }
              ]
            }
          ]
        }
      ],
      "offer": {
        "descriptor": {
          "name": "buy1get1"
        },
        "item_ids": [
          "item_id1",
          "item_id2"
        ]
      },
      "quote": {
        "price": {
          "currency": "KSH",
          "value": "1700.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "KSH",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "KSH",
              "value": "300.0"
            }
          },
          {
            "title": "delivery-charges",
            "price": {
              "currency": "KSH",
              "value": "200.0"
            }
          }
        ]
      },
      "fulfillments": [
        {
          "type": "Delivery",
          "state": {
            "descriptor": {
              "code": "PRE-Order",
              "name": "Before Order"
            },
            "updated_at": "2023-02-06T09:55:41.161Z"
          },
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
                "city": {
                  "name": "Nduini"
                },
                "county": {
                  "name": "Kirinyaga"
                },
                "country": {
                  "name": "Kenya"
                },
                "area_code": "75001"
              },
              "time": {
                "label": "preferable time slot",
                "timestamp": "2024-01-17T10:58:43.451Z",
                "range": {
                  "start": "2024-01-17T16:10:00.430Z",
                  "end": "2024-01-17T16:10:00.430Z"
                }
              },
              "contact": {
                "phone": "2547463949",
                "email": "nc.njugunu@gmail.com"
              }
            }
          ],
          "agent": {
            "person": {
              "id": "string",
              "url": "string",
              "name": "string"
            },
            "contact": {
              "phone": "string",
              "email": "string"
            }
          },
          "customer": {
            "person": {
              "name": "John Githuru"
            },
            "contact": {
              "phone": "2542343344"
            }
          }
        }
      ],
      "fulfillment_state": {
        "descriptor": {
          "name": "In Transit",
          "code": "it",
          "short_desc": "string",
          "long_desc": "string"
        },
        "updated_at": "2024-01-18T08:14:33.667Z",
        "updated_by": "string"
      },
      "billing": {
        "name": "John Githuru",
        "phone": "2542343344",
        "email": "nc.njugunu@gmail.com",
        "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
        "city": {
          "name": "Nduini"
        },
        "county": {
          "name": "Kirinyaga"
        }
      },
      "payments": [
        {
          "id": "string",
          "collected_by": "string",
          "status": "PAID",
          "type": "PRE-FULFILLMENT",
          "params": {
            "transaction_id": "string",
            "amount": "1500",
            "currency": "KSH",
            "virtual_payment_address": "string"
          }
        }
      ]
    }
  }
}
```

## 1.3 Fulfillment of Farm services
This section contains recommendations for implementing the APIs related to fulfilling a Farm services.

### 1.3.1 Recommendations for BPPs

#### 1.3.1.1 Sending status updates
- REQUIRED. The BPP MUST implement the `status` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.2 Updating the Order
- REQUIRED. The BPP MUST implement the `update` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.3 Cancelling the Order
- REQUIRED. The BPP MUST implement the `cancel` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_cancellation_reasons` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.1.4 Real-time tracking
- REQUIRED. The BPP MUST implement the `track` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.3.2 Recommendations for BAPs

#### 1.3.2.1 Receiving status updates
- REQUIRED. The BAP MUST implement the `on_status` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `status`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.2 Updating the Order
- REQUIRED. The BAP MUST implement the `on_update` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `update`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.3 Cancelling the Order
- REQUIRED. The BAP MUST implement the `on_cancel` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `cancel`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `cancellation_reasons` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_cancellation_reasons`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.3.2.4 Real-time tracking
- REQUIRED. The BAP MUST implement the `on_track` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `track`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.3.3 Example Workflow

### 1.3.4 Example Requests

Below is an example of a `status` request
```
{
  "context": {
    "domain": "agrinet",
    "location": {
      "country": {
        "name": "kenya"
      },
      "city": {
        "name": "Kirinyaga"
      }
    },
    "action": "status",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:41:09.708Z"
  },
  "message": {
    "order_id": "b989c9a9-f603-4d44-b38d-26fd72286b38"
  }
}
```

Below is an example of an `on_status` callback
```
{
  "context": {
    "domain": "agrinet",
    "location": {
      "country": {
        "name": "kenya"
      },
      "city": {
        "name": "kirinyaga"
      }
    },
    "action": "on_status",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
    "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
    "timestamp": "2023-11-06T09:41:09.708Z",
    "ttl": "PT10M"
  },
  "message": {
    "order": {
      "id": "order_id",
      "providers": [
        {
          "id": "provider_id_1",
          "descriptor": {
            "name": "Provider 1",
            "short_desc": "",
            "long_desc": "",
            "images": [
              {
                "url": "https://image_url"
              }
            ]
          },
          "locations": [
            {
              "id": "location 1",
              "gps": "12.909955,77.596316"
            }
          ],
          "categories": [
            {
              "id": "c1",
              "descriptor": {
                "code": "herbicide",
                "name": "herbicide"
              }
            }
          ]
        }
      ],
      "items": [
        {
          "id": "item_id",
          "descriptor": {
            "images": [
              {
                "url": "https://image_url"
              }
            ],
            "name": "Potasun",
            "short_desc": "Selective post-emergence herbicide for weed control in Irish potatoes. Mixing: 200mls/20ltr. Available in 1ltr pack",
            "long_desc": "Potasun 50EC is a selective herbicide that controls annual and perennial grasses and broad leaf weeds in tomato, cassava, Irish potato, and cocoyam. It is a selective earlier post-emergence herbicide"
          },
          "unit_type": "Kgs",
          "unit_size": "10",
          "matched": true,
          "price": {
            "listed_value": "1200.0",
            "currency": "KSH",
            "value": "1200.0"
          },
          "rating": "5",
          "creator": "Bayer",
          "recommended": true,
          "location_id": "location_id",
          "category_id": "c1",
          "fulfillment_id": "f1",
          "cancellation_terms": "",
          "return_terms": "",
          "refund_terms": "",
          "tags": [
            {
              "descriptor": {
                "code": "stage",
                "name": "stages applicable"
              },
              "list": [
                {
                  "value": "Land Preparation"
                }
              ]
            },
            {
              "descriptor": {
                "code": "disease",
                "name": "diseases covered"
              },
              "list": [
                {
                  "value": "Baterial Wilt"
                }
              ]
            }
          ]
        }
      ],
      "offer": {
        "descriptor": {
          "name": "buy1get1"
        },
        "item_ids": [
          {
            "item_id": "item-1"
          },
          {
            "item_id": "item-2"
          }
        ]
      },
      "quote": {
        "price": {
          "currency": "KSH",
          "value": "1700.0"
        },
        "breakup": [
          {
            "title": "base-price",
            "price": {
              "currency": "KSH",
              "value": "1200.0"
            }
          },
          {
            "title": "taxes",
            "price": {
              "currency": "KSH",
              "value": "300.0"
            }
          },
          {
            "title": "delivery-charges",
            "price": {
              "currency": "KSH",
              "value": "200.0"
            }
          }
        ]
      },
      "fulfillments": [
        {
          "type": "Delivery",
          "state": {
            "descriptor": {
              "code": "PRE-Order",
              "name": "Before Order"
            },
            "updated_at": "2023-02-06T09:55:41.161Z"
          },
          "stops": [
            {
              "location": {
                "gps": "13.2008459,77.708736",
                "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
                "city": {
                  "name": "Nduini"
                },
                "county": {
                  "name": "Kirinyaga"
                },
                "country": {
                  "name": "Kenya"
                },
                "area_code": "75001"
              },
              "time": {
                "label": "preferable time slot",
                "timestamp": "2024-01-17T10:58:43.451Z",
                "range": {
                  "start": "2024-01-17T16:10:00.430Z",
                  "end": "2024-01-17T16:10:00.430Z"
                }
              },
              "contact": {
                "phone": "2547463949",
                "email": "nc.njugunu@gmail.com"
              }
            }
          ],
          "customer": {
            "person": {
              "name": "John Githuru"
            },
            "contact": {
              "phone": "2542343344"
            }
          }
        }
      ],
      "fulfillment_state": {
        "descriptor": {
          "name": "string",
          "code": "string",
          "short_desc": "string",
          "long_desc": "string"
        },
        "updated_at": "2024-01-18T08:14:33.667Z",
        "updated_by": "string"
      },
      "billing": {
        "name": "John Githuru",
        "phone": "2542343344",
        "email": "nc.njugunu@gmail.com",
        "address": "Nduini, Kirinyaga Central, Kirinyaga, Kenya",
        "city": {
          "name": "Nduini"
        },
        "county": {
          "name": "Kirinyaga"
        }
      },
      "payments": [
        {
          "id": "string",
          "collected_by": "string",
          "status": "PAID",
          "type": "PRE-FULFILLMENT",
          "params": {
            "transaction_id": "string",
            "amount": "1500",
            "currency": "KSH",
            "virtual_payment_address": "string"
          }
        }
      ],
      "cancellation_terms": [
        {
          "descriptor": {
            "name": "terms",
            "short_desc": "",
            "long_desc": ""
          }
        },
        {
          "cancellation_fee": {
            "amount": {
              "currency": "KSH",
              "value": "100"
            }
          }
        }
      ],
      "return_terms": [
        {
          "descriptor": {
            "name": "terms",
            "short_desc": "",
            "long_desc": ""
          }
        },
        {
          "return_charges": {
            "amount": {
              "currency": "KSH",
              "value": "100"
            }
          }
        }
      ],
      "replacement_terms": [
        {
          "descriptor": {
            "name": "terms",
            "short_desc": "",
            "long_desc": ""
          }
        },
        {
          "replacement_charges": {
            "amount": {
              "currency": "KSH",
              "value": "100"
            }
          }
        }
      ],
      "refund_terms": [
        {
          "descriptor": {
            "name": "terms",
            "short_desc": "",
            "long_desc": ""
          }
        },
        {
          "refund_charges": {
            "amount": {
              "currency": "KSH",
              "value": "100"
            }
          }
        }
      ]
    }
  }
}
```

## 1.4 Post-fulfillment of Farm services
This section contains recommendations for implementing the APIs after fulfilling a Farm services

### 1.4.1 Recommendations for BPPs

#### 1.4.1.1 Rating and Feedback
- REQUIRED. The BPP MUST implement the `rating` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BPP MUST implement the `get_rating_categories` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.4.1.2 Providing Support
- REQUIRED. The BPP MUST implement the `support` endpoint on the url specified in URL specified in the `context.bpp_uri` field sent during `on_search`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.4.2 Recommendations for BAPs

#### 1.4.2.1 Rating and Feedback
- REQUIRED. The BAP MUST implement the `on_rating` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `rating`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry
- REQUIRED. The BAP MUST implement the `rating_categories` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `get_rating_categories`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

#### 1.4.2.2 Providing Support
- REQUIRED. The BAP MUST implement the `on_support` endpoint on the url specified in URL specified in the `context.bap_uri` field sent during `support`. In case of permissioned networks, this URL MUST match the `Subscriber.url` present on the respective entry in the Network Registry

### 1.4.3 Example Workflow

### 1.4.4 Example Requests

Below is an example of a `get_rating_categories` request
```
{
    "context": {
        "domain": "agrinet",
        "location": {
            "country": {
            "name": "kenya"
            },
            "city": {
            "name": "Kirinyaga"
            }
        },
        "action": "get_rating_categories",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z"
    }
}
```

Below is an example of an `rating_categories` callback
```
{
    "context": {
        "domain": "agrinet",
        "location": {
            "country": {
            "name": "kenya"
            },
            "city": {
            "name": "Kirinyaga"
            }
        },
        "action": "rating_categories",
        "version": "1.1.0",
        "bap_id": "{bap_id}",
        "bap_uri": "{bap_url}",
        "bpp_id": "{bpp_id}",
        "bpp_uri": "{bpp_url}",
        "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
        "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
        "timestamp": "2023-11-06T09:41:09.708Z"
    },
    "message": {
        "rating_categories" : [
            "Item",
            "provider",
            "Order"
        ]
    }
}
```

Below is an example of a `rating` request

```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
          "name": "kenya"
        },
        "city": {
          "name": "Kirinyaga"
        }
      },
      "action": "rating",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z"
    },
    "message": {
        "ratings": [
            {
                "id": "b989c9a9-f603-4d44-b38d-26fd72286b38",
                "rating_category": "Order",
                "value": "5"
            }
        ]
    }
  }
```
Below is an example of an `on_rating` callback
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
            "name": "kenya"
          },
          "city" : {
            "name": "kirinyaga"
          }
      },
      "action": "on_rating",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "feedback_form": {
        "form": {
          "url": "https://agrinet.kuza-bpp.kuza.one/feedback/portal"
        }
      }
    }
  }

```

Below is an example of a `support` request
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
          "name": "kenya"
        },
        "city": {
          "name": "Kirinyaga"
        }
      },
      "action": "support",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z"
    },
    "message": {
      "support": {
        "ref_id": "894789-43954",
        "phone": "+254 7684937",
        "email": "john_gitiomi@gmail.com"
      }
    }
  }
```

Below is an example of an `on_support` callback
```
{
    "context": {
      "domain": "agrinet",
      "location": {
        "country": {
            "name": "kenya"
          },
          "city" : {
            "name": "kirinyaga"
          }
      },
      "action": "on_support",
      "version": "1.1.0",
      "bap_id": "{bap_id}",
      "bap_uri": "{bap_url}",
      "bpp_id": "{bpp_id}",
      "bpp_uri": "{bpp_url}",
      "message_id": "6104c0a3-d1d1-4ded-aaa4-76e4caf727ce",
      "transaction_id": "8100d125-76a7-4588-88be-81b97657cd09",
      "timestamp": "2023-11-06T09:41:09.708Z",
      "ttl": "PT10M"
    },
    "message": {
      "support": {
        "ref_id": "d4975df5-b18c-4772-80ad",
        "callback_phone": "+254 75849302",
        "phone": "+254 87960541",
        "email": "abcd.support@support.com"
      }
    }
  }
  
```
