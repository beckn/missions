| Term                             | Definition                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Beckn Adapter**                | A piece of software that helps Network Participants connect to Beckn Network. It performs functions of validation, authentication and assist communication between network participants.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Beckn API**                    | APIs that are part of the Beckn Protocol specifications and responsible for enabling two platforms consumer & provider side to discover & transact with each other in an Open Network. (Discovery to post-fulfillment)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Beckn Application Platform (BAP) | Seeker side platform that connects to Beckn Network                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Beckn Gateway                    | Network participant that multicasts search requests from the BAP to relevant BPPs based on the context of the message.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Beckn-ONIX                       | Software stack specification by FIDE to allow easy install and maintenance of Beckn Networks. FIDE also provides a reference implementation of the same. The reference implementation contains reference implementation of Beckn Registry, reference implementation of Beckn Gateway and reference implementation of Beckn Adapters called as Protocol Servers                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Beckn Protocol                   | Open Protocol for Commerce transactions. It is a set of specifications consisting of APIs, data models, reference architecture, transaction mechanisms, and global standards that when adopted by digital platforms, enable the creation of decentralised networks. Such networks allow consumers and providers to discover, identify each other and perform transactions with each other without the need for a central intermediary. It can be thought of as a common set of rules of communication mutually agreed upon by several platforms to allow their users to perform discovery, ordering, fulfilment and post-fulfillment activities between each other in a standard way. It is a sector-agnostic protocol, meaning, any industry-specific taxonomy or knowledge model can be represented using the data model of Beckn Protocol. |
| Beckn Provider Platform (BPP)    | Provider side platform that connects to Beckn Network                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Beckn Network                    | Open Network powered by the Beckn Protocol                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Beckn Registry                   | Holds a list of network participants. It also acts as the repository of public keys of these participants. These keys are used for authenticating messages between network participants                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Core Schema                      | The Core schema specifies the structure of the data in the commerce interactions. The definition of these objects is presented as a structured schema for documentation and validation purposes and complies with the OpenAPI 3.0 specification. The schema organizes that data that is passed in each of the APIs into a several hierarchical component levels                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

Discovery

A stage that involves the searching of required item. In Beckn API it includes the search call

Fulfillment

The actual fulfillment of the order. This consists of events like delivery, tracking etc. It is also a stage that includes Beckn API calls of status, cancel, update and track.

Item

Object of an order. Anything that is searched and transacted over the network.

Network Participant

Within Beckn Network, refers to software systems that interact with Beckn Network. They serve different roles in the network. Current roles include Registry, Gateway, BAP and BPP

Open Network

Network characterized by its accessibility and inclusivity, where the infrastructure, protocols, and standards are made available to a wide range of participants, typically without restrictions.

Order

The consumer constructs the order by selecting various items from a catalog. Billing, Shipping, Payment, Fulfillment and other required informations might be part of it. When used in the context of a stage, it involves the Beckn API calls of Select, Init and Confirm

Post Fulfillment

Stage that includes the Beckn API calls of ratings and support

Protocol Server

Reference implementation of the Beckn Adapter. It is part of the Beckn ONIX reference implementation

Protocol Server - Client

An internal component of Protocol Server that faces the client (Seeker/Provider Platform)

Provider

Shop or organisation providing goods/service

Provider Platform

Platforms that allow Provider to transact on Beckn network. Also called as Beckn Provider Platform (BPP)

Sandbox

A controlled and isolated environment where software applications, code, or processes can run and be tested without affecting the broader system or other applications.

Seeker

User searching for an item or wants to buy an item.

Seeker Platform

Platforms which allows Seekers transact on Beckn networks. Also called as Beckn Application Platform (BAP)

Subscriber

Within the context of Beckn Registry, this is typically a registrant (Potential Network Participant) who has been approved to operate on the network

Taxonomy (Domain Taxonomy)

The domain taxonomy defines the industry-specific objects and enumerations.

Webhook

A network endpoint that can be configured to receive messages
