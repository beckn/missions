# UEI Implementation Guide - Energy Retail

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 15-04-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides example JSONs for the Energy: Energy Retail use case.

## Structure of the document

This document has the following parts:

1. Outcome Visualization - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
3. API Calls and Schema - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. Taxonomy and layer 2 configuration - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
7. Sandbox Details - Sandbox links to BAP, Registry/Gateway and BPP.

## Outcome Visualization

![Energy Retail Outcome Visualization](https://github.com/beckn/missions/blob/main/UEI/images/uei_outcome_visualization.svg)

### Use case - Discovery, order and fulfillment of Energy Retail services
<>

## API Calls and Schema

### search

```
{
    "context": {
        "domain": "deg:retail",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "search",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "db0708ee-a2ae-4b13-aee2-150ce3738b40",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T14:59:57.314Z"
    },
    "message": {
        "intent": {
            "item": {
                "descriptor": {
                    "name": "battery"
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
        "domain": "deg:retail",
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
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "db0708ee-a2ae-4b13-aee2-150ce3738b40",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T14:59:57.691Z"
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
                    "id": "104",
                    "descriptor": {
                        "name": "STARMAX",
                        "short_desc": "Battery",
                        "long_desc": "Battery",
                        "additional_desc": {
                            "url": "www.becknprotocol.io"
                        },
                        "images": [
                            {
                                "url": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS59reL7gFrAuwvUJfVteNazOPfcdrlTM7WtQ&s",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "17",
                            "descriptor": {
                                "name": "DEG_RETAIL_1",
                                "code": "DEG_RETAIL_1"
                            }
                        }
                    ],
                    "rating": "4",
                    "fulfillments": [
                        {
                            "id": "3",
                            "type": "Delivery in 5 Days",
                            "rating": "4.3",
                            "rateable": true
                        }
                    ],
                    "rateable": true,
                    "items": [
                        {
                            "id": "283",
                            "descriptor": {
                                "name": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery",
                                "code": "Battery",
                                "short_desc": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery, 305Ah X 8",
                                "long_desc": "Deep Cycle Flooded Battery\nBattery cell composition: lead_acid\nCompatible with vehicle type: Golf Cart\nItem weight: 121.03 pounds\nVoltage: 6.0",
                                "images": [
                                    {
                                        "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "4.2",
                            "price": {
                                "value": "35000",
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 100
                                }
                            },
                            "fulfillment_ids": [
                                "3"
                            ]
                        }
                    ]
                },
                {
                    "id": "105",
                    "descriptor": {
                        "name": "Odyssey",
                        "short_desc": "Battery",
                        "long_desc": "Battery",
                        "additional_desc": {
                            "url": "www.becknprotocol.io"
                        },
                        "images": [
                            {
                                "url": "https://www.odysseybattery.com/wp-content/uploads/sites/2/ODYSSEY_logo.png",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "17",
                            "descriptor": {
                                "name": "DEG_RETAIL_1",
                                "code": "DEG_RETAIL_1"
                            }
                        }
                    ],
                    "rating": "5",
                    "fulfillments": [
                        {
                            "id": "3",
                            "type": "Delivery in 5 Days",
                            "rating": "4.3",
                            "rateable": true
                        }
                    ],
                    "rateable": true,
                    "items": [
                        {
                            "id": "282",
                            "descriptor": {
                                "name": "Odyssey Extreme Dual Purpose AGM 1225CCA BCI Group 6TL Battery",
                                "code": "Battery",
                                "short_desc": "HEPPC2250ST Odyssey Extreme Dual Purpose AGM 1225CCA BCI Group 6TL Heavy Duty Battery",
                                "long_desc": "The Odyssey Extreme Series Advanced Glass Mat (AGM) battery has 12 volts and 1250 Cold Cranking Amps (CCA) for reliable starting power for your vehicle in extreme conditions.\nOdyssey pure lead plates provide twice the power than conventional flooded batteries and has enough energy to power all of your vehicle's equipment.\nThis Odyssey Extreme battery is used to replaced BCI Group 6TL batteries commonly seen in H1 Military Humvees.\nEngine having trouble starting? Stop by your local Batteries Plus to have your battery tested to see if it's time for a new one.\nComes with a 4-year free replacement limited warranty.",
                                "images": [
                                    {
                                        "url": "https://bpb001produe1cdne.azureedge.net/images/Product/large/1264255.jpg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "4.4",
                            "price": {
                                "value": "30000",
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 100
                                }
                            },
                            "fulfillment_ids": [
                                "3"
                            ]
                        }
                    ]
                },
                {
                    "id": "106",
                    "descriptor": {
                        "name": "Voltronix",
                        "short_desc": "Battery",
                        "long_desc": "Battery",
                        "additional_desc": {
                            "url": "https://voltronix.com/"
                        },
                        "images": [
                            {
                                "url": "https://voltronix.com/wp-content/uploads/2021/08/New-Voltonix_logo_byline_Blue25AAE1_White-bkg.png",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "categories": [
                        {
                            "id": "17",
                            "descriptor": {
                                "name": "DEG_RETAIL_1",
                                "code": "DEG_RETAIL_1"
                            }
                        }
                    ],
                    "rating": "4",
                    "fulfillments": [
                        {
                            "id": "3",
                            "type": "Delivery in 5 Days",
                            "rating": "4.3",
                            "rateable": true
                        }
                    ],
                    "rateable": true,
                    "items": [
                        {
                            "id": "284",
                            "descriptor": {
                                "name": "Voltronix Lithium Ion Battery Kit for Energy Storage, Solar and Battery up",
                                "code": "Battery",
                                "short_desc": "Voltronix Lithium Ion Battery Kit for Energy Storage, Solar and Battery up",
                                "long_desc": "3,000 - 5,000 cycles versus 300 - 500 with lead acid which means a much longer run life and less battery replacements during time of ownership.\nUp to 60% less weight (lead acid 688 lbs vs. 200-320 for lithium batteries depending on size configuration selected).\nEliminate adding water to batteries, minimal corrosion of terminals and frame.\nLess wear and tear on electrical components equates to a more favorable discharge curve and more consistent run time.",
                                "images": [
                                    {
                                        "url": "https://m.media-amazon.com/images/I/81wzNbAfgzL._AC_SX679_.jpg"
                                    }
                                ]
                            },
                            "rateable": true,
                            "rating": "5",
                            "price": {
                                "value": "25000",
                                "currency": "USD"
                            },
                            "quantity": {
                                "available": {
                                    "count": 100
                                }
                            },
                            "fulfillment_ids": [
                                "3"
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
        "domain": "deg:retail",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "select",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "5ab57622-f297-4934-a6d9-b25ecb3ff61a",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:06:40.758Z"
    },
    "message": {
        "order": {
            "items": [
                {
                    "id": "283",
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    }
                }
            ],
            "provider": {
                "id": "104",
                "name": "STARMAX"
            },
            "fulfillments": [
                {
                    "id": "3"
                }
            ]
        }
    }
}

```

### on_select

```
{
"context": {
    "domain": "deg:retail",
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
    "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
    "message_id": "5ab57622-f297-4934-a6d9-b25ecb3ff61a",
    "ttl": "PT10M",
    "timestamp": "2025-04-15T15:06:41.120Z"
},
"message": {
    "order": {
        "provider": {
            "id": "104",
            "descriptor": {
                "name": "STARMAX",
                "short_desc": "Battery",
                "long_desc": "Battery",
                "images": [
                    {
                        "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                        "size_type": "sm"
                    }
                ]
            },
            "categories": [
                {
                    "id": "17",
                    "descriptor": {
                        "name": "DEG_RETAIL_1"
                    }
                }
            ],
            "rating": "4",
            "rateable": true
        },
        "items": [
            {
                "id": "283",
                "descriptor": {
                    "name": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery",
                    "code": "Battery",
                    "short_desc": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery, 305Ah X 8",
                    "long_desc": "Deep Cycle Flooded Battery\nBattery cell composition: lead_acid\nCompatible with vehicle type: Golf Cart\nItem weight: 121.03 pounds\nVoltage: 6.0",
                    "images": [
                        {
                            "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                            "size_type": "sm"
                        }
                    ]
                },
                "rating": "4.2",
                "rateable": true,
                "price": {
                    "value": "35000",
                    "currency": "USD"
                },
                "quantity": {
                    "available": {
                        "count": 100
                    }
                },
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
                "value": "35000",
                "currency": "USD"
            },
            "breakup": [
                {
                    "title": "Item Price",
                    "price": {
                        "currency": "USD",
                        "value": "33018.87"
                    },
                    "item": {
                        "id": "283"
                    }
                },
                {
                    "title": "Sales Tax @ 6%",
                    "price": {
                        "currency": "USD",
                        "value": "1981.13"
                    },
                    "item": {
                        "id": "283"
                    }
                },
                {
                    "title": "Delivery Charge",
                    "price": {
                        "currency": "USD",
                        "value": "20"
                    },
                    "item": {
                        "id": "283"
                    }
                }
            ]
        },
        "fulfillments": [
            {
                "id": "3",
                "type": "Delivery in 5 Days",
                "rating": "4.3",
                "rateable": true
            }
        ]
    }
}
}
```

### init

```
{
    "context": {
        "domain": "deg:retail",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "init",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "697954cd-9889-4de3-9aa7-7b7c83a07de0",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:09:34.127Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "104"
            },
            "items": [
                {
                    "id": "283",
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    }
                }
            ],
            "fulfillments": [
                {
                    "id": "3",
                    "type": "Delivery in 5 Days",
                    "customer": {
                        "person": {
                            "name": "Detroit Public School"
                        },
                        "contact": {
                            "phone": "9944110000"
                        }
                    },
                    "stops": [
                        {
                            "location": {
                                "gps": "42.30043209999999,-83.1166438",
                                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                                "city": {
                                    "name": "Detroit"
                                },
                                "state": {
                                    "name": "Michigan"
                                },
                                "country": {
                                    "code": "IND"
                                },
                                "area_code": "48209"
                            },
                            "contact": {
                                "phone": "9944110000",
                                "email": "dps1289@gmail.com",
                                "name": "Detroit Public School"
                            }
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Detroit Public School",
                "phone": "9944110000",
                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                "email": "dps1289@gmail.com",
                "city": {
                    "name": "Detroit"
                },
                "state": {
                    "name": "Michigan"
                }
            }
        }
    }
}
```

### on_init

```
{
    "context": {
        "domain": "deg:retail",
        "action": "on_init",
        "version": "1.1.0",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "country": "IND",
        "city": "std:080",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            },
            "city": {
                "name": "Bangalore",
                "code": "std:080"
            }
        },
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "697954cd-9889-4de3-9aa7-7b7c83a07de0",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:09:34.504Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "104",
                "descriptor": {
                    "name": "STARMAX",
                    "short_desc": "Battery",
                    "long_desc": "Battery",
                    "images": [
                        {
                            "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                            "size_type": "sm"
                        }
                    ]
                },
                "rating": "4",
            },
            "items": [
                {
                    "id": "283",
                    "descriptor": {
                        "name": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery",
                        "code": "Battery",
                        "short_desc": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery, 305Ah X 8",
                        "long_desc": "Deep Cycle Flooded Battery\nBattery cell composition: lead_acid\nCompatible with vehicle type: Golf Cart\nItem weight: 121.03 pounds\nVoltage: 6.0",
                        "images": [
                            {
                                "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rating": "4.2",
                    "rateable": true,
                    "price": {
                        "value": "35000",
                        "currency": "USD"
                    },
                    "quantity": {
                        "available": {
                            "count": 100
                        }
                    },
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
                    "value": "35000",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Item Price",
                        "price": {
                            "currency": "USD",
                            "value": "33018.87"
                        },
                        "item": {
                            "id": "283"
                        }
                    },
                    {
                        "title": "Sales Tax @ 6%",
                        "price": {
                            "currency": "USD",
                            "value": "1981.13"
                        },
                        "item": {
                            "id": "283"
                        }
                    },
                    {
                        "title": "Delivery Charge",
                        "price": {
                            "currency": "USD",
                            "value": "20"
                        },
                        "item": {
                            "id": "283"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Detroit Public School",
                "phone": "9944110000",
                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                "email": "dps1289@gmail.com",
                "city": {
                    "name": "Detroit"
                },
                "state": {
                    "name": "Michigan"
                }
            },
            "fulfillments": [
                {
                    "id": "3",
                    "type": "Delivery in 5 Days",
                    "stops": [
                        {
                            "location": {
                                "gps": "42.30043209999999,-83.1166438",
                                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                                "city": {
                                    "name": "Detroit"
                                },
                                "state": {
                                    "name": "Michigan"
                                },
                                "country": {
                                    "code": "IND"
                                },
                                "area_code": "48209"
                            },
                            "contact": {
                                "phone": "9944110000",
                                "email": "dps1289@gmail.com",
                                "name": "Detroit Public School"
                            }
                        }
                    ]
                },
                {
                    "id": "3",
                    "type": "Delivery in 5 Days",
                    "rating": "4.3",
                    "rateable": true
                }
            ],
            "payments": [
                {
                    "collected_by": "BPP",
                    "params": {
                        "price": "35000",
                        "currency": "USD"
                    },
                    "status": "PAID",
                    "type": "PRE-ORDER"
                }
            ]
        }
    }
}
```

### confirm

```
{
    "context": {
        "domain": "deg:retail",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "confirm",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "0d8466c4-0b76-4171-8896-aef52e43649f",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:31:08.345Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "104"
            },
            "items": [
                {
                    "id": "283",
                    "code": "Battery",
                    "name": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery",
                    "short_desc": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery, 305Ah X 8",
                    "long_desc": "Deep Cycle Flooded Battery\\nBattery cell composition: lead_acid\\nCompatible with vehicle type: Golf Cart\\nItem weight: 121.03 pounds\\nVoltage: 6.0",
                    "images": [
                        {
                            "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                            "size_type": "sm"
                        }
                    ],
                    "price": {
                        "value": "35000",
                        "currency": "USD"
                    },
                    "rating": "4.2",
                    "rateable": true,
                    "quantity": {
                        "available": {
                            "count": 100
                        },
                        "selected": {
                            "count": 1
                        }
                    }
                }
            ],
            "fulfillments": [
                {
                    "id": "3",
                    "type": "Delivery in 5 Days",
                    "stops": [
                        {
                            "location": {
                                "gps": "42.30043209999999,-83.1166438",
                                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                                "city": {
                                    "name": "Detroit"
                                },
                                "state": {
                                    "name": "Michigan"
                                },
                                "country": {
                                    "code": "IND"
                                },
                                "area_code": "48209"
                            },
                            "contact": {
                                "phone": "9944110000",
                                "email": "dps1289@gmail.com",
                                "name": "Detroit Public School"
                            }
                        }
                    ]
                }
            ],
            "billing": {
                "name": "Detroit Public School",
                "phone": "9944110000",
                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                "email": "dps1289@gmail.com",
                "city": {
                    "name": "Detroit"
                },
                "state": {
                    "name": "Michigan"
                }
            },
            "payments": [
                {
                    "params": {
                        "amount": "35320",
                        "currency": "USD"
                    },
                    "status": "PAID",
                    "type": "ON-FULFILLMENT"
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
        "domain": "deg:retail",
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
        "transaction_id": "4ff62a86-2241-41cc-9527-f0c00ecaba76",
        "message_id": "0d8466c4-0b76-4171-8896-aef52e43649f",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:31:10.095Z"
    },
    "message": {
        "order": {
            "id": "3710",
            "provider": {
                "id": "104",
                "descriptor": {
                    "name": "STARMAX",
                    "short_desc": "Battery",
                    "long_desc": "Battery",
                    "images": [
                        {
                            "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                            "size_type": "sm"
                        }
                    ]
                },
                "rating": "4",
                "rateable": true
            },
            "items": [
                {
                    "id": "283",
                    "descriptor": {
                        "name": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery",
                        "code": "Battery",
                        "short_desc": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery, 305Ah X 8",
                        "long_desc": "Deep Cycle Flooded Battery\nBattery cell composition: lead_acid\nCompatible with vehicle type: Golf Cart\nItem weight: 121.03 pounds\nVoltage: 6.0",
                        "images": [
                            {
                                "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rating": "4.2",
                    "rateable": true,
                    "price": {
                        "value": "35000",
                        "currency": "USD"
                    },
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "value": "35320",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Item Price",
                        "price": {
                            "currency": "USD",
                            "value": "33018.87"
                        },
                        "item": {
                            "id": "283"
                        }
                    },
                    {
                        "title": "Sales Tax @ 6%",
                        "price": {
                            "currency": "USD",
                            "value": "1981.13"
                        },
                        "item": {
                            "id": "283"
                        }
                    },
                    {
                        "title": "Delivery Charge",
                        "price": {
                            "currency": "USD",
                            "value": "20"
                        },
                        "item": {
                            "id": "283"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Detroit Public School",
                "phone": "9944110000",
                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                "email": "dps1289@gmail.com",
                "city": {
                    "name": "Detroit"
                },
                "state": {
                    "name": "Michigan"
                }
            },
            "fulfillments": [
                {
                    "id": "3",
                    "type": "Delivery in 5 Days",
                    "state": {
                        "descriptor": {
                            "code": "ORDER_DELIVERED",
                            "short_desc": "ORDER DELIVERED"
                        },
                        "updated_at": "2025-04-15T15:31:09.978Z"
                    }
                }
            ],
            "payments": [
                {
                    "collected_by": "BPP",
                    "params": {
                        "transaction_id": "tid:824y3rih3ru"
                        "price": "35000",
                        "currency": "USD"
                    },
                    "status": "PAID",
                    "type": "PRE-ORDER"
                }
            ]
        }
    }
}
```

### status

```
{
    "context": {
        "domain": "deg:retail",
        "bpp_id": "bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bpp_uri": "http://bpp-ps-network-strapi2-dev.becknprotocol.io",
        "bap_id": "bap-ps-network-dev.becknprotocol.io",
        "action": "status",
        "bap_uri": "https://bap-ps-network-dev.becknprotocol.io",
        "version": "1.1.0",
        "transaction_id": "af10ae14-ca94-43b3-bf79-8f114f5bd8d0",
        "message_id": "1bf7ab56-df1f-4e2d-b5c1-615517c785b0",
        "location": {
            "country": {
                "name": "United States",
                "code": "USA"
            }
        },
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:32:47.261Z"
    },
    "message": {
        "order_id": "3710"
    }
}

```

### on_status

```
{
    "context": {
        "domain": "deg:retail",
        "action": "on_status",
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
        "transaction_id": "af10ae14-ca94-43b3-bf79-8f114f5bd8d0",
        "message_id": "1bf7ab56-df1f-4e2d-b5c1-615517c785b0",
        "ttl": "PT10M",
        "timestamp": "2025-04-15T15:32:47.387Z"
    },
    "message": {
        "order": {
            "id": "3710",
            "created_at": "2025-04-15T15:31:09.948Z",
            "provider": {
                "id": "104",
                "descriptor": {
                    "name": "STARMAX",
                    "short_desc": "Battery",
                    "long_desc": "Battery",
                    "images": [
                        {
                            "url": "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS59reL7gFrAuwvUJfVteNazOPfcdrlTM7WtQ&s",
                            "size_type": "sm"
                        }
                    ]
                },
                "rating": "4",
                "rateable": true
            },
            "items": [
                {
                    "id": "283",
                    "descriptor": {
                        "name": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery",
                        "code": "Battery",
                        "short_desc": "STARMAX GC STCR-305 – 6-Volt Deep Cycle Flooded Battery, 305Ah X 8",
                        "long_desc": "Deep Cycle Flooded Battery\nBattery cell composition: lead_acid\nCompatible with vehicle type: Golf Cart\nItem weight: 121.03 pounds\nVoltage: 6.0",
                        "images": [
                            {
                                "url": "https://m.media-amazon.com/images/I/51D9ZUF9W-L._AC_SX679_.jpg",
                                "size_type": "sm"
                            }
                        ]
                    },
                    "rating": "4.2",
                    "rateable": true,
                    "price": {
                        "value": "35000",
                        "currency": "USD"
                    },
                    "quantity": {
                        "selected": {
                            "count": 1
                        }
                    },
                    "fulfillment_ids": [
                        "3"
                    ]
                }
            ],
            "price": {
                "value": "35000",
                "currency": "USD"
            },
            "fulfillments": [
                {
                    "id": "3",
                    "state": {
                        "descriptor": {
                            "code": "ORDER_DELIVERED",
                            "short_desc": "ORDER DELIVERED"
                        },
                        "updated_at": "2025-04-15T15:31:09.978Z"
                    },
                    "customer": {
                        "contact": {
                            "email": "dps1289@gmail.com",
                            "phone": "9944110000"
                        },
                        "person": {
                            "name": "Detroit"
                        }
                    },
                    "stops": [
                        {
                            "type": "start",
                            "location": {
                                "gps": "42.30043209999999,-83.1166438",
                                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                                "city": {
                                    "name": "Detroit"
                                },
                                "country": {
                                    "code": "IND"
                                },
                                "area_code": "48209",
                                "state": {
                                    "name": "Michigan"
                                }
                            },
                            "contact": {
                                "phone": "9944110000"
                            }
                        }
                    ],
                    "agent": {
                        "person": {
                            "name": "Betty Crost"
                        }
                    },
                    "rating": "4.3",
                    "rateable": true
                }
            ],
            "billing": {
                "name": "Detroit Public School",
                "address": "5890 W Vernor Hwy, Detroit, Michigan",
                "state": {
                    "name": "Michigan"
                },
                "city": {
                    "name": "Detroit"
                },
                "email": "dps1289@gmail.com",
                "phone": "9944110000"
            },
            "quote": {
                "price": {
                    "value": "35320",
                    "currency": "USD"
                },
                "breakup": [
                    {
                        "title": "Item Price",
                        "price": {
                            "currency": "USD",
                            "value": "33018.87"
                        },
                        "item": {
                            "id": "283"
                        }
                    },
                    {
                        "title": "Sales Tax @ 6%",
                        "price": {
                            "currency": "USD",
                            "value": "1981.13"
                        },
                        "item": {
                            "id": "283"
                        }
                    },
                    {
                        "title": "Delivery Charge",
                        "price": {
                            "currency": "USD",
                            "value": "20"
                        },
                        "item": {
                            "id": "283"
                        }
                    }
                ]
            },
            "status": "ACTIVE"
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
