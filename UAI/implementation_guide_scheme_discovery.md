# UAI Implementation Guide - Scheme Discovery

#### Version 1.0

## Version History

| Date       | Version | Description                                         |
| ---------- | ------- | --------------------------------------------------- |
| 06-11-2024 | 0.1     | Initial Version                                     |
| 14-11-2024 | 0.2     | Internal Review Comments Incorprated                |
| 18-11-2024 | 1.0     | Final Version                                       |
| 04-12-2024 | 1.1     | A new section created for Schema Details and added details for Scheme Discovery      |

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

![UAI Knowledge Advisory Outcome Visualization](images/uai_knowledge_advisory_outcome_visualization.svg)

### Use case - Discovery of Schemes
Farmer searching for government schemes and looking for ways to fulfill the schemes from service providers on the UKI Network

Abhiraj is a farmer in Nashik. He has been trying to find insurance products to protect himself from adverse climatic conditions. He uses a beckn enabled app which is integrated to the UKI network to know the available schemes and how to apply. In case he is not able to apply by himself, he wants to book an agent who can assist him in accessing the scheme benefits. 

#### Discovery
- Search all schemes
- Search by Name
- Search by eligibility criteria
- Search for Pradhan Mantri Fasal Bhima Yojana. (Abhiraj can use text (in any language)).
- Abhiraj receives the details of the schemes from a list of service providers (BPP)
- Eligibility criteria can include various items like (tentative list can include more variables):
    Age
    Differently abled
    Caste category
    Ownership of land - Yes/No
    Gender - Male/Female
    No. of girl children
    No. of children
    Ration card type
    Nature of Job - farmer, carpenter, dairy farmer, etc.
    Enterprise category - Agriculture, agriculture allied, others
    Type of business support required - Registration/Other
    Location
    Type of Scheme - Central, State, District
    Born and brought up in the current state of residence
    Do you have a voters id
    Are you a taxpayer
    Do you have insurance for cattle/small ruminants?
    What was the maximum level of education that you have achieved

#### Order
- Abhiraj selects the service provider (BPP) of his choice to view more details about the scheme. 
- He receives details of the scheme, required documents and steps on how to apply and an option to book an agent.
- The booking is confirmed by the service provider (BPP) and the details of the agent (Shwetha) is shared.
- The service provider shares an order ID.

#### Fulfilment
- Abhiraj receives status updates from the service provider on the day of the service delivery
- Shwetha calls Abhiraj to ensure the documents are ready for collection. 
- If Abhiraj does not have the necessary documents, she closes the request as closed with appropriate reason.
- If Abhiraj has all the necessary documents, Shwetha reaches Abhiraj’s location to collect the documents. 
- The documents are authenticated, if the documents are valid, the status is updated as documents collected.  
- The documents are authenticated, if the documents are invalid, the order ID is closed with appropriate reason. 
- Shwetha performs the necessary steps to ensure the scheme is applied on behalf of Abhiraj with the concerned government department.
- In case of any failure in disbursement of the scheme, the status is updated by the BPP, also Shwetha calls to inform Abhiraj and resolves the same
- In case of successful disbursement, the status is updated.
- Throughout the process Abhiraj should be able to track the progress of his application.

#### Post Fulfilment
- Abhiraj submits a rating for the service provided by the agent (Shwetha). The service provider (BPP) receives the rating and can request for further feedback.
- Rating: 
    Product/service quality 
    Service Provider(on-time, polite)
- Support
    Abhiraj can reach out to support services in case of any issues with the service delivery

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

![ACK NACK for messages](/docs/assets/images/ack_nack.png)

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

### Use case - Discovery and consumption of free knowledge advisory

**Search for resources on topic**

![Search for resources on infested disease](Images/Advisory/free/UAI%20advisory-Discovery.jpg)

**Seek support and provide rating**

![Support call to get contact information of Provider platofrm](./Images/Advisory/free/uai-Post%20Fulfilment.jpg)

### Use case - Discovery and consumption of paid knowledge advisory

**Search for resources on topic**

![Search for resources on infested disease](./Images/Advisory/paid/UAI%20advisory-Discovery.jpg)

**Place an order for the advisory**

![Select the advisory, initialise and confirm the order ](./Images/Advisory/paid/UAI%20advisory-Order.jpg)

**fulfillment of an active order**

![check the order status, update the order details or cancel the order ](./Images/Advisory/paid/UAI%20advisory-Fulfilment.jpg)

**Seek support and provide rating**

![Support call to get contact information of Provider platofrm](./Images/Advisory/paid/UAI%20advisory-Post%20Fulfilment.jpg)

## API Calls and Schema

### Discovery of schemes

#### search

#### search request for agricultural schemes can be structured as below.
**search by topic**

- The topic to search is specified in the message->intent->descriptor->name field.
- The desired language is specified in a tag named languages.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
    "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "descriptor": {
        "name": "agricultural scheme"
      },
      "item": {
        "tags": [
          {
            "descriptor": {
              "name": "languages"
            },
            "list": [
              {
                "value": "mr"
              },
              {
                "value": "en"
              }
            ]
          }
        ]
      },
      "fulfillment": {
        "stops": [
          {
            "type": "applicable-region",
            "location": {
              "state": {
                "name": "Maharshtra"
              },
              "country": {
                "name": "India"
              }
            }
          }
        ]
      }
    }
  }
}
```

**search by topic and applicant details**

- The topic to search is specified in the message->intent->descriptor->name field.
- The desired language is specified in a tag named languages.
- Details are specified using tags.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
    "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "descriptor": {
        "name": "agricultural scheme"
      },
      "item": {
        "tags": [
          {
            "descriptor": {
              "name": "languages"
            },
            "list": [
              {
                "value": "mr"
              },
              {
                "value": "en"
              }
            ]
          }
        ]
      },
      "fulfillment": {
        "stops": [
          {
            "type": "applicable-region",
            "location": {
              "state": {
                "name": "Maharshtra"
              },
              "country": {
                "name": "India"
              }
            }
          }
        ]
      }
      "tags": [
        {
          "descriptor": {
              "code": "Demographic-details",
              "name": "Demographic details"
          },
          "list": [
            {
              "descriptor": {
                  "code": "gender"
              },
              "value": "Male"
            },
            {
              "descriptor": {
                  "code": "caste-category"
              },
              "value": "General"
            }
          ]
        },
        {
          "descriptor": {
              "code": "personal-details",
              "name": "Personal details"
          },
          "list": [
            {
              "descriptor": {
                  "code": "differently-abled"
              },
              "value": "No"
            },
            {
              "descriptor": {
                  "code": "land-owner",
                  "name": "Do you or any of your family members own any land"
              },
              "value": "No"
            },
            {
              "descriptor": {
                  "code": "girl-children",
                  "name": "Number of girl children you have"
              },
              "value": "2"
            }
          ]
        },
        {
          "descriptor": {
              "code": "economic-details",
              "name": "Economic details"
          },
          "list": [
            {
              "descriptor": {
                  "code": "ration-card-type"
              },
              "value": "Below-Poverty"
            },
            {
              "descriptor": {
                  "code": " business-support-required",
                  "name": "What business support do you require?"
              },
              "value": "Loan"
            }
          ]
        },
        {
          "descriptor": {
              "code": "employment-details",
              "name": "Employment details"
          },
          "list": [
            {
              "descriptor": {
                  "code": "job-type"
              },
              "value": "self-employed"
            },
            {
              "descriptor": {
                  "code": " enterprise-category"
              },
              "value": "individual"
            }
          ]
        },
      ]
    }
  }
}
```

**If a BAP already have the name/id of a scheme(cached or fetched using another search request), the BAP can find the scheme using the scheme name or id**
**search by scheme name/id**

- The desired language is specified in a tag named languages.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "search",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_uri": "https://example-bap-client.becknprotocol.io",
    "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
    "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {  // we can use any of the name or code.
          "name": "Mahatma Gandhi National Rural Employment Guarantee Act",
          "code": "1234"
        },
        "tags": [
          {
            "descriptor": {
              "name": "languages"
            },
            "list": [
              {
                "value": "mr"
              },
              {
                "value": "en"
              }
            ]
          }
        ]
      }
    }
  }
}
```


#### on_search

**on_search with catalog of results**

- The catalog that comes back has a list of providers.
- Each provider has a list of items.
- Each item is the catalog listing for a resource.
- The name, short_desc and long_desc fields contain the name and description of the resource.
- Further, if the resource is a video or a pdf, its mimetype and url are specified in the media field.

```

```

#### support

**sending a support request**

- In most cases the support request is a call to get contact details of the provider platform (Phone, Web url etc).
- However in rare cases a reference id might be sent to provide a context to the request.

```
{
  "context": {
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "IND"
      }
    },
    "action": "support",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "d8b23543-24b4-48eb-ae8a-4a5db68f8d09",
    "transaction_id": "fa2c9c8b-ba24-4d2b-bd9c-3e03d7f6b193",
    "timestamp": "2024-07-02T09:18:30Z"
  },
  "message": {
    "ref_id": "9e188d26-0b1b-4920-a586-6006b0bcf768"
  }
}
```

#### on_support

**Getting an on_support callback**

- In most cases the returned details are common contact information of the provider platform.
- However it is possible to connect this to other customer management solutions.

```
{
  "context": {
    "domain": "advisory:uai",
    "location": {
      "country": {
        "code": "IND"
      }
    },
    "action": "on_support",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "d8b23543-24b4-48eb-ae8a-4a5db68f8d09",
    "transaction_id": "fa2c9c8b-ba24-4d2b-bd9c-3e03d7f6b193",
    "timestamp": "2024-07-02T09:18:30Z",
    "ttl": "PT10M"
  },
  "message": {
    "support": {
      "ref_id": "9e188d26-0b1b-4920-a586-6006b0bcf768",
      "phone": "18001801551",
      "url": "https://agritech.tnau.ac.in/agriculture/agri_faqs.html"
    }
  }
}

```

#### get_rating_categories

- BAP can fetch a list of categoried for which a BPP accepts rating
- ```message``` field is not needed in the get_rating_categories request.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  }
}
```

#### rating_categories

- The BPP responds with a list of categories in which ratings can be provided.

```
{
  "context": {
    "domain": "advisory:uai",
    "action": "get_rating_categories",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      }
    },
    "city": "std:080",
    "version": "1.1.0",
    "bap_id": "{bap_id}",
    "bap_uri": "{bap_url}",
    "bpp_id": "{bpp_id}",
    "bpp_uri": "{bpp_url}",
    "transaction_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "message_id": "6743e9e2-4fb5-487c-92b7-13ba8018f176",
    "timestamp": "2023-07-16T04:41:16Z"
  },
  "message": {
    "rating_categories": [
      Item,
      Order,
      Fulfillment,
      Provider
    ]
  }
}
```

#### rating

**Rating a resource**

- When the user wants to rate the resource, it happens in two steps.
- The user sends a numerical (0-5) rating in the request.
- The response will have a link to the form where the user can provide text feedback.

```
{
  "context": {
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "IND"
      }
    },
    "action": "rating",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "e8c50b1e-6512-42b3-b0b4-8f8a703a5c66",
    "transaction_id": "b7204c3a-9f5e-418f-80a3-ae5dd4e5b97a",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "ratings": [
      {
        "id": "19a02a67-d2f0-4ea7-b7e1-b2cf4fa57f56",
        "rating_category": "Provider",
        "value": "5"
      }
    ]
  }
}
```

#### on_rating

**Getting on_rating response**

- The on_rating response has a link to the form where the user can specify a text feedback that goes alongside the numerical rating provided earlier.

```
{
  "context": {
    "domain": "advisory:uai",
    "location": {
      "country": {
        "name": "IND"
      }
    },
    "action": "on_rating",
    "version": "1.1.0",
    "bap_id": "example-bap.becknprotocol.io",
    "bap_url": "https://example-bap-client.becknprotocol.io",
    "bpp_id": "example-bpp.becknprotocol.io",
    "bpp_uri": "https://example-bpp-network.becknprotocol.io",
    "message_id": "e8c50b1e-6512-42b3-b0b4-8f8a703a5c66",
    "transaction_id": "b7204c3a-9f5e-418f-80a3-ae5dd4e5b97a",
    "timestamp": "2024-07-02T09:15:30Z",
    "ttl": "PT10M"
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

## Taxonomy and layer 2 configuration

- Any specific tags, enumerations, and rules we add for the use cases or required by the network, will go here.

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

![Seeker platform testing sandbox ](/docs/assets/images/seeker_deployment.png)

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

![Provider platform testing sandbox](/docs/assets/images/provider_deployment.png)

## Schema Details

| SN | Use Case                  | Input Details                        | Values                           | Data Types        |
|----|---------------------------|--------------------------------------|----------------------------------|-------------------|


## Links to artefacts


- [Postman collection for UAI](./postman/Uai.postman_collection.json)

## Sandbox Details

### Registry/Gateway:
- **Gateway Sandbox:** gateway-uai.becknprotocol.io
- **Registry Sandbox:** registery-uai.becknprotocol.io

### BPP:
 - **BPP Client Sandbox:** bpp-ps-client-sandbox-uai.becknprotocol.io
 - **BPP Network Sandbox:** bpp-ps-network-sandbox-uai.becknprotocol.io
 - **BPP Sandbox:** bpp-unified-sandbox-uai.becknprotocol.io

### Domain name:
    advisory:uai
