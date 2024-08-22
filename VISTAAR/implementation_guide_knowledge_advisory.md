# VISTAAR Implementation Guide - Knowledge Advisory

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

### Use case - Discovery and consumption of knowledge advisory

![Outcome visualization - Knowledge Advisory](./images/outcome_visualization_ka.png)

1. Moti is a farmer in Odisha. She is a wheat farmer and is struggling with Aphid infestation. She struggles to control it and is looking for effective solutions to address the disease and increase her crop yield.
2. She is introduced to Vistaar through Krishi Vigyan Kendra. A farm extension worker Ramesh then uses a Vistaar-enabled bot which is plugged into the Vistaar network.
3. He uses the bot interface to search for remedies and prevention measures for the Aphid infestation. He receives multiple search results from different experts on best practices and fertilizers to address her concerns.
4. He selects a 2-minute video from AgroExpert Solutions and clicks on the "watch it now" option. Ramesh watches the video and understands the right fertilizers Moti needs.
5. After watching the video, Ramesh selects "Get Support" and connects with a farmer associate to address further questions.
6. After the conversation, Ramesh gives a rating for the video which he watched.

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

### Use case - Discovery and consumption of knowledge advisory

**Search for resources on topic**

![Search for resources on infested disease](./images/search_ka.png)

**Support information from the provider platform**

![Support call to get contact information of Provider platofrm](./images/support_ka.png)

**Rate the viewed resource**

![Rate the viewed resource](./images/rating_ka.png)

## API Calls and Schema

### search

**search by topic**

- The topic to search is specified in the message->item->descriptor->name field.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_uri": "https://ps-bap-client.becknprotocol.io",
    "transaction_id": "7b3d0c62-7c1b-4c6b-b768-14f81b6c3c90",
    "message_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {
          "name": "cotton aphids"
        }
      }
    }
  }
}
```

**search by topic and language**

- The topic to search is specified in the message->intent->item->descriptor->name field.
- The desired language is specified in a tag named languages.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "action": "search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_uri": "https://ps-bap-client.becknprotocol.io",
    "transaction_id": "d28ec57e-8c8f-4db0-a5aa-73d6563942e1",
    "message_id": "6c8b36e8-7886-4cc8-b3a6-8a3d464fcd6c",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "intent": {
      "item": {
        "descriptor": {
          "name": "cotton aphid"
        },
        "tags": [
          {
            "descriptor": {
              "name": "languages"
            },
            "list": [
              {
                "value": "Odia"
              }
            ]
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
- Each item is the catalog listing for a resource.
- The name, short_desc and long_desc fields contain the name and description of the resource.
- Further, if the resource is a video or a pdf, its mimetype and url are specified in the media field.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "action": "on_search",
    "location": {
      "country": {
        "name": "India",
        "code": "IND"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
    "transaction_id": "7b3d0c62-7c1b-4c6b-b768-14f81b6c3c90",
    "message_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "timestamp": "2024-07-02T09:15:30Z"
  },
  "message": {
    "catalog": {
      "descriptor": {
        "name": "Cotton Aphid"
      },
      "providers": [
        {
          "id": "19a02a67-d2f0-4ea7-b7e1-b2cf4fa57f56",
          "descriptor": {
            "name": "Agri Acad",
            "short_desc": "Agri Academic aggregator",
            "images": [
              {
                "url": "https://agri_acad.example.org/logo.png"
              }
            ]
          },
          "items": [
            {
              "id": "9875",
              "descriptor": {
                "name": "Cotton aphid: Aphis gossypii (Aphididae: Hemiptera)",
                "short_desc": "TNAU Agritech portal. Aphid management",
                "long_desc": "Avoid late sowing. Treat seeds with Beauveria bassiana @ 10 g/kg. Apply nitrogenous fertilizers judiciously. Grow cowpea as intercrop or on the bunds to increase the natural enemy build up. Spray any one of the following insecticides when pest population reaches ETL: Imidacloprid 17.8% SL 40 – 50 ml/acre or Azadirachtin 0.03% EC 1000 ml/acre or Buprofezin 25%SC 400 ml/acre or Diafenthiuron 50%WP 240 g/acre or Thiacloprid 21.7%SC 40-50 ml/acre or Flonicamid 50% WG 60 g/acre or Thiamethoxam 25% WG 40 g/acre"
              }
            },
            {
              "id": "4321",
              "descriptor": {
                "name": "Cotton aphid: Aphis gossypii",
                "short_desc": "Cotton aphid and management",
                "long_desc": "",
                "media": [
                  {
                    "mimetype": "application/pdf",
                    "url": "https://agritech.tnau.ac.in/itk/pdf/itk_cotton.pdf"
                  }
                ]
              },
              "rating": "4.0"
            },
            {
              "id": "8977",
              "descriptor": {
                "name": "Aphids Pest Management in Cotton Farming - Hindi Video",
                "short_desc": "Control the Aphids in cotton farming by following the information provided in the video. ",
                "long_desc": "",
                "media": [
                  {
                    "mimetype": "video/mp4",
                    "url": "https://www.youtube.com/watch?v=bupCtQNfecg"
                  }
                ]
              },
              "rating": "3.5"
            }
          ]
        },
        {
          "id": "5e899a32-72f3-48e1-9156-5b1302c1f32a",
          "descriptor": {
            "name": "Farming videos",
            "short_desc": "Farming video repository",
            "images": [
              {
                "url": "https:/farm_video.example.org/logo.png"
              }
            ]
          },
          "items": [
            {
              "id": "53453",
              "descriptor": {
                "name": "Aphid, Thrips control in cotton",
                "short_desc": "Aphid control in cotton",
                "long_desc": "",
                "media": [
                  {
                    "mimetype": "video/mp4",
                    "url": "https://www.youtube.com/watch?v=SBs68VHPXUs"
                  }
                ]
              },
              "rating": "4.5"
            }
          ]
        }
      ]
    }
  }
}
```

### support

**sending a support request**

- In most cases the support request is a call to get contact details of the provider platform (Phone, Web url etc).
- However in rare cases a reference id might be sent to provide a context to the request.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "location": {
      "country": {
        "name": "India"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "action": "support",
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
    "message_id": "d8b23543-24b4-48eb-ae8a-4a5db68f8d09",
    "transaction_id": "fa2c9c8b-ba24-4d2b-bd9c-3e03d7f6b193",
    "timestamp": "2024-07-02T09:18:30Z"
  },
  "message": {
    "ref_id": "9e188d26-0b1b-4920-a586-6006b0bcf768"
  }
}
```

### on_support

**Getting an on_support callback**

- In most cases the returned details are common contact information of the provider platform.
- However it is possible to connect this to other customer management solutions.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "location": {
      "country": {
        "code": "IND"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "action": "on_support",
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
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

### rating

**Rating a resource**

- When the user wants to rate the resource, it happens in two steps.
- The user sends a numerical (0-5) rating in the request.
- The response will have a link to the form where the user can provide text feedback.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "location": {
      "country": {
        "name": "India"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "action": "rating",
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
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

### on_rating

**Getting on_rating response**

- The on_rating response has a link to the form where the user can specify a text feedback that goes alongside the numerical rating provided earlier.

```
{
  "context": {
    "domain": "advisory:agrinet:vistaar",
    "location": {
      "country": {
        "name": "India"
      },
      "city": {
        "name": "Balangir"
      }
    },
    "action": "on_rating",
    "version": "1.1.0",
    "bap_id": "ps-bap-network.becknprotocol.io",
    "bap_url": "https://ps-bap-client.becknprotocol.io",
    "bpp_id": "beckn-sandbox-bpp.becknprotocol.io",
    "bpp_uri": "https://sandbox-bpp-network.becknprotocol.io",
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

## Links to artefacts

- [Postman collection for VISTAAR Knowledge Advisory](./postman/Vistaar-Advisory.postman_collection.json)
- [Layer2 config for VISTAAR Knowledge Advisory](./layer2/knowledge-advisory_agrinet_vistaar_1.1.0.yaml)
- When installing layer2 using Beckn-ONIX use this web address (https://raw.githubusercontent.com/beckn/missions/main/VISTAAR/layer2/knowledge-advisory_agrinet_vistaar_1.1.0.yaml)
