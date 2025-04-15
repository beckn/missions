# UEI Implementation Guide - Energy Finance

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 15-04-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides example JSONs for the Energy: Energy Finance use case.

## Structure of the document

This document has the following parts:

1. Outcome Visualization - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
3. API Calls and Schema - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. Taxonomy and layer 2 configuration - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
7. Sandbox Details - Sandbox links to BAP, Registry/Gateway and BPP.

## Outcome Visualization

![Energy Finance Outcome Visualization](https://github.com/beckn/missions/blob/main/UEI/images/uei_outcome_visualization.svg)

### Use case - Discovery, order and fulfillment of Energy Finance services
<>

## API Calls and Schema

### search

```
{
    "context": {
        "domain": "deg:finance",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "search",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "c07c000b-aab6-410e-9fa2-fd11cafb6dda",
        "message_id": "10377d28-1bba-41e8-b83d-bbe7161da659",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:12:41.861Z"
    },
    "message": {
        "intent": {
            "category": {
                "descriptor": {
                    "code": "loan_type",
                    "name": "Battery"
                }
            },
            "location": {
                "circle": {
                    "radius": {
                        "type": "CONSTANT",
                        "value": "5",
                        "unit": "km"
                    }
                }
            }
        }
    }
}
```

### on_search


```
{
    "context": {
        "domain": "deg:finance",
        "action": "on_search",
        "version": "1.1.0",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "transaction_id": "c07c000b-aab6-410e-9fa2-fd11cafb6dda",
        "message_id": "10377d28-1bba-41e8-b83d-bbe7161da659",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:12:42.351Z"
    },
    "message": {
        "catalog": {
            "descriptor": {
                "name": "UrjaKart",
                "code": "UrjaKart",
                "short_desc": "A seller platform for energy solutions, enabling seamless transactions of batteries, solar panels, and other renewable energy products."
            },
            "providers": [
                {
                    "id": "103",
                    "descriptor": {
                        "name": "PNC",
                        "short_desc": "300",
                        "long_desc": "Provides capital for businesses, asset purchases, or investments",
                        "additional_desc": {
                            "url": "www.becknprotocol.io"
                        },
                        "images": [
                            {
                                "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rating": "4.2",
                    "rateable": true,
                    "items": [
                        {
                            "id": "281",
                            "descriptor": {
                                "name": "6",
                                "code": "80",
                                "short_desc": "Asset Financier",
                                "long_desc": "Battery Loan by PNC Equipment Financing 6 Months EMI Plan",
                                "images": [
                                    {
                                        "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "null",
                            "price": {
                                "value": "12"
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 10000
                                }
                            },
                            "category_ids": [
                                "36"
                            ]
                        },
                        {
                            "id": "340",
                            "descriptor": {
                                "name": "9",
                                "code": "80",
                                "short_desc": "Asset Financier",
                                "long_desc": "Battery Loan by PNC Equipment Financing 9 Months EMI Plan",
                                "images": [
                                    {
                                        "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "null",
                            "price": {
                                "value": "12",
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 10000
                                }
                            },
                            "category_ids": [
                                "36"
                            ]
                        }
                    ]
                },
                {
                    "id": "102",
                    "descriptor": {
                        "name": "Bank of America",
                        "short_desc": "250",
                        "long_desc": "Provides capital for businesses, asset purchases, or investments",
                        "additional_desc": {
                            "url": "www.becknprotocol.io"
                        },
                        "images": [
                            {
                                "url": "https://business.bofa.com/etc.clientlibs/flagship/clientlibs/clientlib-site/resources/images/BofA_logo.svg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rating": "4.5",
                    "short_desc": "250",
                    "rateable": true,
                    "items": [
                        {
                            "id": "280",
                            "descriptor": {
                                "name": "9",
                                "code": "80",
                                "short_desc": "Asset Financier",
                                "long_desc": "Battery Loan by Bank of America 9 Months EMI Plan",
                                "images": [
                                    {
                                        "url": "https://business.bofa.com/etc.clientlibs/flagship/clientlibs/clientlib-site/resources/images/BofA_logo.svg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "null",
                            "price": {
                                "value": "14",
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 10000
                                }
                            },
                            "category_ids": [
                                "36"
                            ]
                        },
                        {
                            "id": "339",
                            "descriptor": {
                                "name": "6",
                                "code": "80",
                                "short_desc": "Asset Financier",
                                "long_desc": "Battery Loan by Bank of America 6 Months EMI Plan",
                                "images": [
                                    {
                                        "url": "https://business.bofa.com/etc.clientlibs/flagship/clientlibs/clientlib-site/resources/images/BofA_logo.svg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "null",
                            "price": {
                                "value": "14",
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 10000
                                }
                            },
                            "category_ids": [
                                "36"
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

```
{
    "context": {
        "domain": "deg:finance",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "select",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "9a14790b-e44c-46b2-91ba-d770d07fd7ac",
        "message_id": "5f9725c2-0e18-4279-be08-2a86b1dfc741",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:15:43.333Z"
    },
    "message": {
        "order": {
            "items": [
                {
                    "id": "281",
                    "selected": {
                        "quantity": {
                            "count": 1
                        }
                    }
                }
            ],
            "provider": {
                "id": "103"
            }
        }
    }
}

```

### on_select

```
{
    "context": {
        "domain": "deg:finance",
        "action": "on_select",
        "version": "1.1.0",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "transaction_id": "9a14790b-e44c-46b2-91ba-d770d07fd7ac",
        "message_id": "5f9725c2-0e18-4279-be08-2a86b1dfc741",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:15:43.805Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "103",
                "descriptor": {
                    "name": "PNC",
                    "short_desc": "300",
                    "long_desc": "Provides capital for businesses, asset purchases, or investments",
                    "images": [
                        {
                            "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg",
                            "size_type": "sm"
                        }
                    ]
                },
                "rating": "4.2",
                "rateable": true
            },
            "items": [
                {
                    "id": "281",
                    "descriptor": {
                        "name": "6",
                        "code": "80",
                        "short_desc": "Asset Financier",
                        "long_desc": "Battery Loan by PNC Equipment Financing 6 Months EMI Plan",
                        "images": [
                            {
                                "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rating": "null",
                    "rateable": true,
                    "price": {
                        "value": "12",
                        "currency": "USD"
                    },
                    "quantity": {
                        "available": {
                            "count": 10000
                        }
                    },
                    "category_ids": [
                        "36"
                    ],
                    "xinput": {
                        "form": {
                            "mime_type": "text/html",
                            "url": "https://link-to-the-form"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "value": "12",
                    "currency": "USD"
                }
            }
        }
    }
}

```

### confirm

```
{
    "context": {
        "domain": "deg:finance",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "confirm",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "9a14790b-e44c-46b2-91ba-d770d07fd7ac",
        "message_id": "806aef5e-2e9e-43ba-acd5-b598e35ae61e",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:21:26.026Z"
    },
    "message": {
        "order": {
            "items": [
                {
                    "id": "281",
                    "code": "80",
                    "name": "6",
                    "short_desc": "Asset Financier",
                    "long_desc": "Battery Loan by PNC Equipment Financing 6 Months EMI Plan",
                    "images": [
                        {
                            "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg",
                            "size_type": "sm"
                        }
                    ],
                    "price": {
                        "value": "12",
                        "currency": "USD"
                    },
                    "rateable": true,
                    "quantity": {
                        "available": {
                            "count": 10000
                        }
                    }
                }
            ],
            "provider": {
                "id": "103"
            },
            "fulfillments": [
                {
                    "type": "Delivery",
                    "customer": {
                        "person": {
                            "name": "Detroit Public School "
                        },
                        "contact": {
                            "phone": "9944110000"
                        }
                    },
                    "stops": [
                        {
                            "location": {
                                "gps": "13.2008459,77.708736",
                                "address": "123, Terminal 1, Kempegowda Int'''l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
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
                                "email": "dps1289@gmail.com"
                            }
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Detroit Public School ",
                "address": "123, Terminal 1, Kempegowda Int'''l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "state": {
                    "name": "Gangamuthanahalli"
                },
                "city": {
                    "name": "Karnataka"
                },
                "email": "dps1289@gmail.com",
                "phone": "9944110000"
            },
            "payments": [
                {
                    "params": {
                        "amount": "28000",
                        "currency": "USD",
                        "bank_account_number": "1234002341",
                        "bank_code": "INB0004321"
                    },
                    "status": "PAID",
                    "type": "PRE-FULFILLMENT"
                }
            ]
        }
    }
}

```

### on_confirm

```
{
    "context": {
        "domain": "deg:finance",
        "action": "on_confirm",
        "version": "1.1.0",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "transaction_id": "9a14790b-e44c-46b2-91ba-d770d07fd7ac",
        "message_id": "806aef5e-2e9e-43ba-acd5-b598e35ae61e",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:21:27.778Z"
    },
    "message": {
        "order": {
            "id": "3708",
            "provider": {
                "id": "103",
                "descriptor": {
                    "name": "PNC",
                    "short_desc": "300",
                    "long_desc": "Provides capital for businesses, asset purchases, or investments",
                    "images": [
                        {
                            "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg",
                            "size_type": "sm"
                        }
                    ]
                },
                "rating": "4.2",
                "short_desc": "300",
                "fulfillments": [
                    {
                        "state": {
                            "descriptor": {
                                "name": "LOAN DISBURSED"
                            }
                        }
                    }
                ],
                "rateable": true
            },
            "items": [
                {
                    "id": "281",
                    "descriptor": {
                        "name": "6",
                        "code": "80",
                        "short_desc": "Asset Financier",
                        "long_desc": "Battery Loan by PNC Equipment Financing 6 Months EMI Plan",
                        "images": [
                            {
                                "url": "https://www.lawnandgardendirectory.org/Shared/images/PNC-Equipment-Finance-th1.jpg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rateable": true,
                    "price": {
                        "value": "12",
                        "currency": "USD"
                    },
                    "quantity": {
                        "selected": {
                            "count": 10000
                        }
                    },
                    "category_ids": [
                        "36"
                    ]
                }
            ],
            "quote": {
                "price": {
                    "value": "28000",
                    "currency": "USD"
                }
            },
            "billing": {
                "name": "Detroit Public School ",
                "address": "123, Terminal 1, Kempegowda Int'''l Airport Rd, A - Block, Gangamuthanahalli, Karnataka 560300, India",
                "state": {
                    "name": "Gangamuthanahalli"
                },
                "city": {
                    "name": "Karnataka"
                },
                "email": "dps1289@gmail.com",
                "phone": "9944110000"
            },
            "fulfillments": [
                {
                    "state": {
                        "descriptor": {
                            "code": "LOAN_DISBURSED",
                            "short_desc": "LOAN DISBURSED"
                        },
                        "updated_at": "2025-04-15T15:21:27.703Z"
                    }
                }
            ]
        }
    }
}

```


## Taxonomy and layer 2 configuration

## Links to artifacts

- [Postman collection ]()
- [Layer2 config ]()
- When installing layer2 using Beckn-ONIX use this web address ()

## Sandbox Details

![UEI Alliance]()

### BAP:

- **BAP Client Sandbox:** []()
- **BAP Network Sandbox:** []()

### Registry/Gateway:

- **Gateway Sandbox:** []()
- **Registry Sandbox:** []()

### BPP:

- **BPP Client Sandbox:** []()
- **BPP Network Sandbox:** []()
- **BPP Playground Sandbox:** []()
