# UAI Implementation Guide - Secured Loans

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 06-11-2024 | 0.1     | Initial Version                                     |
| 14-11-2024 | 0.2     | Internal Review Comments Incorporated                |
| 18-11-2024 | 1.0     | Final Version                                       |

## Introduction

This document provides material that helps network participants build and integrate their application with the Beckn Network. This document is part of the starter kit that provides information about the network, learning resources, network participant checklist etc. This document only focuses on the implementation of the seeker/provider platform. It assumes the reader has a good overview of the Beckn network, its APIs, the overall structure of the schema etc.

## Structure of the document

This document has the following parts:

1. Outcome Visualization - This is a pictorial or descriptive representation of the different use cases that are supported by the network.
2. Flow diagrams - This section provides a pictorial representation of the message flows that happen during the use case.
3. API Calls and Schema - This section provides details on the API calls and the schema of the message that is sent in the form of sample schemas.
4. Taxonomy and layer 2 configuration - This section provides details on the taxonomy, enumerations and any rules defined for either the use case or by the network.
5. Notes on writing/integrating with your own software - This section describes ways in which you can integrate (Becknify) your new or existing software
6. Links to downloadable resources - This section contains the downloadable files referenced in this document.

## Outcome Visualisation

### Use case - Discovery and application for secured loans

1. Sanjeev, secretary of Jeevan FPO (Farmer Producer Organization) in Nashik, seeks a secured loan for cold storage and bulk input procurement.
2. Instead of manual loan outreach, Sanjeev uses a Beckn-enabled BAP on the UKI Network to explore structured credit options.
3. He searches for "Agri Infrastructure Loans" and filters by entity type (Farmer Collective), loan type (Infrastructure), amount, and special tags.
4. The app displays loan solutions from various lenders with details like interest rates, collateral requirements, and processing time.
5. Sanjeev selects "ColdChain Capital Loan" by Samunnati and receives eligibility checklist and documentation requirements.
6. He completes the application form with FPO details, UDYAM certificate, shareholder list, and collateral information.
7. After submission, Sanjeev receives a Loan Application ID and can track the status through the app.
8. The lender validates CIBIL, GST profile, and asset ownership, then sends a final loan quote and sanction letter.
9. Sanjeev accepts terms digitally via Aadhaar eSign or OTP, and the loan amount becomes available for disbursement.
10. The FPO can initiate disbursement requests via the app, and the loan is credited to their verified bank account.
11. Sanjeev submits ratings for the loan product and can contact support services if needed.

## Flow diagrams

### General Beckn message flow and error handling

This section is relevant to all the messages flows illustrated below and discussed further in the document.

Beckn is an asynchronous protocol at its core.

- When a network participant(NP1) sends a message to another participant(NP2), the other participant(NP2) immediately returns back an ACK/NACK(Acknowledgement or Negative Acknowledgement in case of error - usually with wrongly formed messages).
- An ACK is an indicator that the receiving participant(NP2) will process this message and dispatch an on_xxxxxx message to original NP (NP1)
- Subsequently after processing the message NP2 sends back the real response in the corresponding on_xxxxxx message, to which again the first participant(NP1).
- This message can contain a message field (for success) or error field (for failure)
- NP1 when it receives the on_xxxxxx message, sends back an ACK/NACK (Here in both the cases NP1 will not send any subsequent message).
- In the Use case diagrams, this ACK/NACK is not illustrated explicitly to keep the diagrams crisp.
- However when writing software we should be prepared to receive these NACK messages as well as error field in the on_xxxxxx messages
- While this discussion is from a Beckn perspective, Adapters can provide synchronous modes. For example, the Protocol Server which is the reference implementation of the Beckn Adapter provides a synchronous mode by default. So if your software calls the support endpoint on the BAP Protocol Server, the Protocol Server waits till it gets the on_support and returns back that as the response.

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

## API Calls and Schema

### Secured Loans

#### search

**search for secured loan products**

- The search includes entity type, loan type, and financial terms.

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "search",
        "version": "1.1.0",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "89b7fa3e-5df6-40dc-b457-0a8a986b7603",
        "ttl": "PT10M",
        "timestamp": "2025-06-02T10:00:00Z",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        }
    },
    "message": {
        "intent": {
            "descriptor": {
                "name": "Agri Infrastructure Loan"
            },
            "tags": [
                {
                    "descriptor": {
                        "name": "eligibility"
                    },
                    "list": [
                        {
                            "descriptor": {
                                "code": "entity-type"
                            },
                            "value": "farmer-collective"
                        }
                    ]
                },
                {
                    "descriptor": {
                        "name": "financial-terms"
                    },
                    "list": [
                        {
                            "descriptor": {
                                "code": "loan-type"
                            },
                            "value": "infrastructure"
                        }
                    ]
                }
            ]
        }
    }
}
```

#### on_search

**on_search with catalog of loan products**

- The catalog contains loan providers with their products, interest rates, and eligibility criteria.

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_search",
        "version": "1.1.0",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati-credit.uki.in",
        "bpp_uri": "https://samunnati-credit.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "89b7fa3e-5df6-40dc-b457-0a8a986b7603",
        "timestamp": "2025-06-02T10:00:03Z",
        "ttl": "PT10M"
    },
    "message": {
        "catalog": {
            "providers": [
                {
                    "id": "samunnati-001",
                    "descriptor": {
                        "name": "Samunnati",
                        "short_desc": "Agri credit solutions"
                    },
                    "items": [
                        {
                            "id": "coldchain-loan-001",
                            "descriptor": {
                                "name": "ColdChain Capital Loan",
                                "short_desc": "Infrastructure financing for FPOs"
                            },
                            "tags": [
                                {
                                    "descriptor": {
                                        "code": "loan_details"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "loan_amount"
                                            },
                                            "value": "1500000 INR"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "interest_rate"
                                            },
                                            "value": "8.5%"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "tenure"
                                            },
                                            "value": "36 months"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "processing_time"
                                            },
                                            "value": "7 days"
                                        }
                                    ]
                                },
                                {
                                    "descriptor": {
                                        "code": "eligibility"
                                    },
                                    "list": [
                                        {
                                            "descriptor": {
                                                "code": "min_shareholders"
                                            },
                                            "value": "200"
                                        },
                                        {
                                            "descriptor": {
                                                "code": "collateral_type"
                                            },
                                            "value": "UDYAM & Land Collateral"
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

#### select

**select a specific loan product and request a detailed quote**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "select",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "prov01"
            },
            "items": [
                {
                    "id": "coldchain-loan-15l"
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

#### on_select

**on_select returns an estimated quotation for the selected loan item**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_select",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital"
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            }
        }
    }
}
```

#### init

**init shares FPO's documents, billing, and fulfillment details to begin onboarding and verification**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "init",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "prov01"
            },
            "items": [
                {
                    "id": "coldchain-loan-15l"
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    }
                }
            ],
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            }
        }
    }
}
```

#### on_init

**on_init acknowledges receipt and returns fulfillment, payment, cancellation terms, and a form for collecting details**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_init",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ],
                    "xinput": {
                        "required": false,
                        "head": {
                            "descriptor": {
                                "name": "Requirements Form"
                            },
                            "index": {
                                "min": 0,
                                "cur": 0,
                                "max": 4
                            },
                            "headings": [
                                "Eligibility checklist",
                                "Required certifications",
                                "Collateral details",
                                "Bank details",
                                "Authorized signatory KYC"
                            ]
                        },
                        "form": {
                            "mime_type": "text/html",
                            "url": "https://6vs8xnx5i7.loan-finder.co.in/schems/xinput/formid/a23f2fdfbbb8ac402bfd54f",
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
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### confirm

**confirm the order and formally accept the terms of the order**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "confirm",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "order": {
            "provider": {
                "id": "prov01"
            },
            "items": [
                {
                    "id": "coldchain-loan-15l"
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    }
                }
            ],
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ]
        }
    }
}
```

#### on_confirm

**on_confirm acknowledges and confirms terms of the order, registers loan order, and issues an order id**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_confirm",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "APPLICATION_SUCCESSFUL"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### status

**status polls the current status of the loan application**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "order_id": "ord01"
    }
}
```

#### on_status

**on_status responds with latest status details such as processing, sanctioned, disbursed, or rejected**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "CIBIL_VERIFICATION_PENDING"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```


#### on_status-2

**on_status-2 with additional documentation form**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ],
                    "xinput": {
                        "required": false,
                        "head": {
                            "descriptor": {
                                "name": "Additional Documentation Form"
                            },
                            "index": {
                                "min": 0,
                                "cur": 0,
                                "max": 2
                            },
                            "headings": [
                                "Balance sheets",
                                "Collateral verification docs",
                                "Board declarations"
                            ]
                        },
                        "form": {
                            "mime_type": "text/html",
                            "url": "https://uhr487hr.loan-finder.co.in/docs/xinput/formid/48623ycd98ey8eie",
                            "resubmit": false,
                            "auth": {
                                "descriptor": {
                                    "code": "jwt"
                                },
                                "value": "eudhwq89deynwxedoweu.cuhw9e7xne9.SflKxwRJSjioeud90xunq9o"
                            }
                        }
                    }
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "SUBMIT_ADDITIONAL_DOCS"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### on_status-3

**on_status-3 with loan approval**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.7%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                },
                                {
                                    "descriptor": {
                                        "code": "approved-loan-amount"
                                    },
                                    "value": "1500000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "LOAN_APPROVED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### on_status-4

**on_status-4 with sanction letter issued**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.7%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                },
                                {
                                    "descriptor": {
                                        "code": "approved-loan-amount"
                                    },
                                    "value": "1500000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "SANCTION_LETTER_ISSUED"
                        }
                    }
                }
            ],
            "docs": [
                {
                    "descriptor":{
                        "name" :"Sanction Letter",
                        "short_desc" : "To open this document, enter the password sent to your email abc****@***.com"
                    },
                    "url" : "https://link-to-the-document.com",
                    "mime_type" : "application/pdf"
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### on_status-5

**on_status-5 with eKYC pending**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.7%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                },
                                {
                                    "descriptor": {
                                        "code": "approved-loan-amount"
                                    },
                                    "value": "1500000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "EKYC_PENDING"
                        }
                    }
                }
            ],
            "docs": [
                {
                    "descriptor":{
                        "name" :"Sanction Letter",
                        "short_desc" : "To open this document, enter the password sent to your email abc****@***.com"
                    },
                    "url" : "https://link-to-the-document.com",
                    "mime_type" : "application/pdf"
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### on_status-6

**on_status-6 with loan credited**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_status",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.7%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                },
                                {
                                    "descriptor": {
                                        "code": "approved-loan-amount"
                                    },
                                    "value": "1500000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "LOAN_CREDITED"
                        }
                    }
                }
            ],
            "docs": [
                {
                    "descriptor":{
                        "name" :"Sanction Letter",
                        "short_desc" : "To open this document, enter the password sent to your email abc****@***.com"
                    },
                    "url" : "https://link-to-the-document.com",
                    "mime_type" : "application/pdf"
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### update-1

**update for Aadhaar authorization using OTP**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "update",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "update_target": "order.fulfillments",
        "order": {
            "fulfillments": [
                {
                    "id": "f1",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "EKYC_PENDING"
                        }
                    },
                    "stops": [
                        {
                            "type": "AADHAR_EKYC",
                            "authorization": {
                                "type": "OTP",
                                "token": "1234"
                            }
                        }
                    ]
                }
            ]
        }
    }
}
```

#### update-2
```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "update",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "update_target": "order.fulfillments",
        "order": {
            "fulfillments": [
                {
                    "id": "f1",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "INITIATE_DISBURSEMENT"
                        }
                    }
                }
            ]
        }
    }
}
```

#### on_update

**on_update acknowledges the update request**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_update",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.7%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                },
                                {
                                    "descriptor": {
                                        "code": "approved-loan-amount"
                                    },
                                    "value": "1500000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "EKYC_PENDING"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### rating

**rating for post-fulfillment feedback**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "rating",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "ratings": [
            {
                "id": "prov01",
                "rating_category": "Provider",
                "value": "5"
            }
        ]
    }
}
```

#### on_rating

**on_rating with feedback form request**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_rating",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "feedback_form": {
            "form": {
                "url": "https://agri_acad.example.org/feedback"
            }
        }
    }
}
```

#### support

**support request for post-disbursal terms**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "support",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "ref_id": "ord01"
    }
}
```

#### on_support

**on_support with contact information**

```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_support",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "support": {
            "ref_id": "ord01",
            "phone": "+14155550123",
            "email": "support@credit-service-samunnati.com"
          }
    }
}
```
   
#### cancel
```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "cancel",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in",
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "ttl": "PT20S",
        "timestamp": "2025-06-02T10:01:00Z"
    },
    "message": {
        "order_id": "ord01",
        "cancellation_reason_id": "cancel/id/1234"
    }
}
```

#### on_cancel-1
```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_cancel",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "APPLICATION_CANCELLED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "299"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                },
                {
                    "type": "REFUND",
                    "status": "PAID",
                    "params": {
                        "currency": "INR",
                        "value": "0.0"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```

#### on_cancel-2
```
{
    "context": {
        "domain": "credit-services:uki",
        "action": "on_cancel",
        "version": "1.1.0",
        "location": {
            "city": {
                "code": "std:95253"
            },
            "country": {
                "code": "IND"
            }
        },
        "transaction_id": "3a5b0b23-501e-42e8-b43b-03b60e822882",
        "message_id": "1fa1e8ce-a0f6-46a6-8c49-d2c3e06be021",
        "bap_id": "credit-fpo.bap.uki.in",
        "bap_uri": "https://credit-fpo.bap.uki.in",
        "bpp_id": "samunnati.bpp.uki.in",
        "bpp_uri": "https://samunnati.bpp.uki.in"
    },
    "message": {
        "order": {
            "id": "ord01",
            "provider": {
                "id": "prov01",
                "descriptor": {
                    "name": "Samunnati",
                    "short_desc": "Agri-focused NBFC",
                    "images": [
                        {
                            "url": "https://samunnati.in/logo.png"
                        }
                    ]
                }
            },
            "items": [
                {
                    "id": "coldchain-loan-15l",
                    "descriptor": {
                        "name": "ColdChain Capital Loan",
                        "long_desc": "Secured Infra Loan for FPOs with min 200 shareholders"
                    },
                    "tags": [
                        {
                            "descriptor": {
                                "code": "financial-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "interest-rate"
                                    },
                                    "value": "8.5%"
                                },
                                {
                                    "descriptor": {
                                        "code": "loan-type"
                                    },
                                    "value": "infrastructure"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "processing-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "tat"
                                    },
                                    "value": "P7D"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "security-terms"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "collateral-required"
                                    },
                                    "value": "yes"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "limits"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "min-disbursement-amount"
                                    },
                                    "value": "1500000INR"
                                },
                                {
                                    "descriptor": {
                                        "code": "max-disbursement-amount"
                                    },
                                    "value": "2000000INR"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "eligibility"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "entity-type"
                                    },
                                    "value": "farmer-collective"
                                }
                            ]
                        },
                        {
                            "descriptor": {
                                "code": "documents-required"
                            },
                            "list": [
                                {
                                    "descriptor": {
                                        "code": "clent-identification"
                                    },
                                    "value": "UDYAM"
                                },
                                {
                                    "descriptor": {
                                        "code": "client-certificate-1"
                                    },
                                    "value": "SFAC"
                                }
                            ]
                        }
                    ]
                }
            ],
            "fulfillments": [
                {
                    "id": "f1",
                    "type": "digital",
                    "customer": {
                        "person": {
                            "name": "Sanjeev Bhosale"
                        },
                        "contact": {
                            "email": "sanjeev@jeevanfpo.in",
                            "phone": "+919812345678"
                        }
                    },
                    "state": {
                        "descriptor": {
                            "code": "APPLICATION_CANCELLED"
                        }
                    }
                }
            ],
            "quote": {
                "price": {
                    "currency": "INR",
                    "value": "1799"
                },
                "breakup": [
                    {
                        "title": "Processing Fee",
                        "price": {
                            "currency": "INR",
                            "value": "299"
                        }
                    },
                    {
                        "title": "Penalty",
                        "price": {
                            "currency": "INR",
                            "value": "1500"
                        }
                    }
                ]
            },
            "billing": {
                "name": "Jeevan Agro FPO",
                "organization": {
                    "descriptor": {
                        "name": "Jeevan Agro FPO"
                    },
                    "address": "Block 3, Agri Chambers, Nashik"
                }
            },
            "payments": [
                {
                    "collected_by": "BPP",
                    "type": "POST_FULFILLMENT",
                    "status": "NOT_PAID",
                    "url": "https://payment.quick-freights.uki.network/",
                    "params": {
                        "currency": "INR",
                        "value": "299.00"
                    }
                },
                {
                    "type": "PENALTY",
                    "status": "PROCESSING",
                    "params": {
                        "currency": "INR",
                        "value": "1500.0",
                        "bank_account_number": "1212233242"
                    }
                }
            ],
            "cancellation_terms": [
                {
                    "cancel_by": {
                        "label": "cancellation before sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0%"
                    }
                },
                {
                    "cancel_by": {
                        "label": "cancellation after sanction"
                    },
                    "cancellation_fee": {
                        "percentage": "0.1%"
                    }
                }
            ]
        }
    }
}
```


## Taxonomy and layer 2 configuration

- Any specific tags, enumerations, and rules we add for the use cases or required by the network, will go here.

## Integrating with your software

This section gives general walkthrough of how you would integrate your software with the Beckn network (say the sandbox environment). Refer to the starter kit for details on how to register with the sandbox and get credentials.

Beckn-ONIX is an initiative to promote easy install and maintenance of a Beckn Network. Apart from the Registry and Gateway components that are required for a network facilitator, Beckn-ONIX provides a Beckn Adapter. A reference implementation of the Beckn-ONIX specification is available at [Beckn-ONIX repository](https://github.com/beckn/beckn-onix). The reference implementation of the Beckn Adapter is called the Protocol Server. Based on whether we are writing the seeker platform or the provider platform, we will be installing the BAP Protocol Server or the BPP Protocol Server respectively.

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
   - the remaining components are provided by the sandbox environment
8. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

### Integrating the provider platform

If you are writing the provider platform software, the following are the steps you can follow to build and integrate your application.

1. Identify the use cases from above section that are close to the functionality you plan for your application.
2. Design and develop the component that accepts the Beckn requests and interacts with your software to do transactions. It has to be a endpoint(it is called as webhook_url in the description below) which receives all the Beckn requests (search, select etc). This endpoint can either exist outside of your marketplace/shop software or within it. That is a design decision that will have to be taken by you based on the design of your existing marketplace/shop software. This component is also responsible for sending back the responses to a the Beckn Adapter.
3. Install the BPP Protocol Server using the reference implementation of Beckn-ONIX. During the installation, you will need the address of the registry of the environment, a URL where the Beckn responses will arrive (called Subscriber URL), a subscriber_id (typically the same as subscriber URL without the "https://" prefix) and the webhook_url that you configured in the step above. Also the address of the BPP Protocol Server Client will have to be configured in your component above. This address hosts all the response endpoints (on_search,on_select etc)
4. Install the layer 2 file for the domain (Link is in the last section of this document)
5. Check with your network tech support to enable your BPP Protocol Server in the registry.
6. Once enabled, you can transact on the Beckn Network. Typically the sandbox environment will have the rest of the components you need to test your software. In the diagram below,
   - you write the Provider Platform(dark blue) - Here the component you wrote above in point 2 as well as your marketplace/shop software is together shown as Provider Platform
   - install the BPP Protocol Server (light blue)
   - the remaining components are provided by the sandbox environment
   - Use the postman collection to test your Provider Platform
7. Once the application is working on the Sandbox, refer to the Starter kit for instructions to take it to pre-production and production.

## Schema Details

| SN | Use Case                  | Input Details                        | Values                           | Data Types        |
|----|---------------------------|--------------------------------------|----------------------------------|-------------------|
| 1  | Secured Loans             | Entity Type                          | farmer-collective                | varchar           |
| 2  | Secured Loans             | Loan Type                            | infrastructure                   | varchar           |
| 3  | Secured Loans             | Loan Amount                          | 1500000 INR                     | varchar           |
| 4  | Secured Loans             | Interest Rate                        | 8.5%                             | varchar           |
| 5  | Secured Loans             | Tenure                               | 36 months                        | varchar           |
| 6  | Secured Loans             | Processing Time                      | 7 days                           | varchar           |
| 7  | Secured Loans             | Minimum Shareholders                 | 200                              | int               |
| 8  | Secured Loans             | Collateral Type                      | UDYAM & Land Collateral          | varchar           |

## Links to artefacts

- [Postman collection for UAI](../../postman/Uai.postman_collection.json)

## Sandbox Details

### Registry/Gateway:
- **Gateway Sandbox:** gateway-uai.becknprotocol.io
- **Registry Sandbox:** registery-uai.becknprotocol.io

### BPP:
 - **BPP Client Sandbox:** bpp-ps-client-sandbox-uai.becknprotocol.io
 - **BPP Network Sandbox:** bpp-ps-network-sandbox-uai.becknprotocol.io
 - **BPP Sandbox:** bpp-unified-sandbox-uai.becknprotocol.io

### Domain name:
    credit-services:uki 