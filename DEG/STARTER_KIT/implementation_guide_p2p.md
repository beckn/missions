# UEI Implementation Guide - Battery P2P

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 15-04-2025 | 1.0     | Initial Version                                     |

## Introduction

This document provides example JSONs for the Energy: Battery P2P use case.

## Structure of the document

This document has the following parts:

1. Outcome Visualization - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
3. API Calls and Schema - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. Taxonomy and layer 2 configuration - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
7. Sandbox Details - Sandbox links to BAP, Registry/Gateway and BPP.

## Outcome Visualization

![Battery P2P Outcome Visualization](https://github.com/beckn/missions/blob/main/UEI/images/uei_outcome_visualization.svg)

### Use case - Discovery, order and fulfillment of Battery P2P services
<>

## API Calls and Schema

### search

```
{
  "context": {
    "domain": "energy",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
          "name": "Lucknow",
          "code": "std:522"
      }
    },
    "version": "1.1.0",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {
          "code": "energy"
        },
        "quantity": {
          "selected": {
            "measure": {
              "value": "10",
              "unit": "kWH"
            }
          }
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
            "location": {
                "address": "der://uppcl.meter/98765456"
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

```
{
    "context": {
        "domain": "energy",
        "action": "on_search",
        "location": {
            "country": {
                "name": "India",
                "code": "IND"
            }
        },
        "city": "std:080",
        "version": "1.1.0",
        "bap_id": "p2pTrading-bap.com",
        "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
        "bpp_id": "p2pTrading-bpp.com",
        "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
        "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
        "timestamp": "2023-07-16T04:41:16Z"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "p1072",
                    "descriptor": {
                        "name": "Mukesh Shankar",
                        "images": [
                            {
                                "url": "https://p1072.in/images/logo.png"
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
                        },
                        {
                            "id": "2",
                            "descriptor": {
                                "code": "EV",
                                "name": "electric vehicle"
                            }
                        },
                    ],
                    "locations": [
                        {
                            "descriptor": {
                                "code": "house"
                            },
                            "address": "der://mukesh.house"
                            "id": "1",
                            "gps": "12.345345,77.389754"
                        },
                        {
                            "descriptor": {
                                "code": "ev"
                            },
                            "address": "der://mukesh.ev"
                            "id": "2",
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
                                    "location": {
                                        "address": "der://uppcl.meter/92982739" // meter id
                                    },
                                    "time": {
                                        "range": {// UPPCL permits this date range to include only the next day. Other implementations may allow a broader time range selection.
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
                            "descriptor": {
                                "code": "energy"
                            },
                            "price": {
                                "value": "5",
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
                                    "location": {
                                        "address": "der://uppcl.meter/92982739" // meter id
                                    },
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
                            "descriptor": {
                                "code": "energy"
                            },
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
                                    "location": {
                                        "address": "der://uppcl.meter/92982739" // meter id
                                    },
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
                            "descriptor": {
                                "code": "energy"
                            },
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

### init

```
{
  "context": {
    "domain": "energy",
    "action": "init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
          "name": "Lucknow",
          "code": "std:522"
      }
    },
    "version": "1.1.0",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-energy-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "p1072"
      },
      "items": [
        {
          "id": "uid_xyz",
          "descriptor": {
            "code": "energy"
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
              "location": {
                   "address": "der://uppcl.meter/98765456"
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

```
{
  "context": {
    "domain": "energy",
    "action": "on_init",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
          "name": "Lucknow",
          "code": "std:522"
      }
    },
    "version": "1.1.0",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "p1072",
        "descriptor": {
          "name": "Mukesh Shankar",
          "images": [
            {
              "url": "https://p1072.in/images/logo.png"
            }
          ]
        },
        "credentials": [
            {
                "Descriptor": {
                    "Name": "Solar Panel Ownership Certificate",
                    "code": "SPOC"
                },
                "type": "VerifiableCredential",
                "url": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
            },
            {
                "Descriptor": {
                    "Name": "P2P Trading Licence",
                    "code": "P2PTL"
                },
                "Url": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
            },
            {
                "Descriptor": {
                    "Name": "Proof of Identity",
                    "code": "POID"
                },
                "Url": "https://digilocker.com/43846384/Adhar.pdf"
            },
            {
                "Descriptor": {
                    "Name": "Proof of Address",
                    "code": "POAD"
                },
                "Url": "https://digilocker.com/43846384/Adhar.pdf"
            }
        ]
      },
      "items": [
        {
          "id": "uid_xyz",
          "descriptor": {
            "code": "energy"
          },
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
              "location": {
                   "address": "der://uppcl.meter/92982739" // meter id
              },
              "time": {
                "range": {// UPPCL permits this date range to include only the next day. Other implementations may allow a broader time range selection.
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            },
            {
              "type": "end",
              "location": {
                   "address": "der://uppcl.meter/98765456" // meter id
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

```
{
  "context": {
    "domain": "energy",
    "action": "confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
          "name": "Lucknow",
          "code": "std:522"
      }
    },
    "version": "1.1.0",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "bpp_id": "example-energy-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "provider": {
        "id": "p1072"
      },
      "items": [
        {
          "id": "uid_xyz",
          "descriptor": {
            "code": "energy"
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
              "location": {
                   "address": "der://uppcl.meter/98765456"
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

```
{
  "context": {
    "domain": "energy",
    "action": "on_confirm",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
          "name": "Lucknow",
          "code": "std:522"
      }
    },
    "version": "1.1.0",
    "bap_id": "p2pTrading-bap.com",
    "bap_uri": "https://api.p2pTrading-bap.com/pilot/bap/energy/v1",
    "bpp_id": "p2pTrading-bpp.com",
    "bpp_uri": "https://api.p2pTrading-bpp.com/pilot/bpp/",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "order": {
      "id": "6743e9e2-4fb5-487c-92b7",
      "provider": {
        "id": "p1072",
        "descriptor": {
          "name": "Mukesh Shankar",
          "images": [
            {
              "url": "https://p1072.in/images/logo.png"
            }
          ]
        },
        "credentials": [
             {
                "Descriptor": {
                    "Name": "Solar Panel Ownership Certificate",
                    "code": "SPOC"
                },
                "type": "VerifiableCredential",
                "url": "https://https://drive.google.com/drive/u/0/my-drive/43846384/SolarPanelOwnershipCer.json"
            },            
            {
                "Descriptor": {
                    "Name": "P2P Trading Licence",
                    "code": "P2PTL"
                },
                "Url": "https://https://drive.google.com/drive/u/0/my-drive/43846384/Licence.pdf"
            },
            {
                "Descriptor": {
                    "Name": "Proof of Identity",
                    "code": "POID"
                },
                "Url": "https://digilocker.com/43846384/Adhar.pdf"
            },
            {
                "Descriptor": {
                    "Name": "Proof of Address",
                    "code": "POAD"
                },
                "Url": "https://digilocker.com/43846384/Adhar.pdf"
            }
        ]
      },
      "items": [
        {
          "id": "uid_xyz",
          "descriptor": {
            "code": "energy"
          },
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
              "location": {
                   "address": "der://uppcl.meter/92982739" // meter id
              },
              "time": {
                "range": {// UPPCL permits this date range to include only the next day. Other implementations may allow a broader time range selection.
                  "start": "2024-10-04T10:00:00",
                  "end": "2024-10-04T18:00:00"
                }
              }
            },
            {
              "type": "end",
              "location": {
                   "address": "der://uppcl.meter/98765456" // meter id
              },
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
