## Beckn FAQ

Beckn FAQ will cover some questions frequently asked by the community.

Q1. **Is cloud credentials required for development and any associated costs. – Does beckn provide the EC2 instance required for hosting the beckn artefacts such as registry, protocol server etc ?**

We have a BOC (Beckn Open Collective) provided public network. NP’s can utilise that for testing purposes. Sometimes the network facilitator also has a sandbox environment for participants related to them.

Q2. **Are there any rate limits or usage quotas for the Beckn APIs that we should be aware of?**

NO, Beckn is a specification with no run time limitation. Some of the reference implementations of the components might have it, but the protocol itself is agnostic.

Q3. **What is the recommended approach (OAuth, API keys, or JWT??) for authenticating API requests to the Beckn network?**

Beckon Protocol has a [authentication section](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-006-Signing-Beckn-APIs-In-HTTP-Draft-01.md) that describes the process with examples.

Q4. **What is the process for registering service providers on the Beckn network?**

a. API details for onboarding.

- Beckn deals with Provider Platform (BPP). The onboarding of Providers onto BPP is outside of Beckn specification. You can have an independent process for this.

b. Is it possible to include additional parameters (competencies, location, etc …) as part of onboarding?

- Yes. Some of these are possible through the attributes of the Provider. (Please refer to the [spec here](https://github.com/beckn/protocol-specifications/blob/master/api/transaction/build/transaction.yaml)) for details. You can search for “Provider:” within the spec for its schema). Additional attributes can be specified through [tags](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-009-Tags-the-Edge-of-Beckn.md).

c. Can we query providers list based on additional parameters?

- Yes. The search intent allows complex queries including those values specified through tags.

Q5. **What are the common error codes and messages returned by the Beckn APIs?**

Please refer to [this document](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-005-Error-Codes-Draft-01.md)

Q6. **What are the upcoming features or updates planned for the Beckn protocol that we should be aware of?**

There is a version 1.2 of the specification that is in the pipeline. It should not affect current designs and implementations.

Q7. **As per our conceived architecture, We will use Firebase as a backend-as-a-service for notifications, authentication, and the database and Node.js (protocol server) applications to connect with Beckn to handle the responsibilities of both the BAP and BPP. We would like to discuss best practices/security to be followed when communicating with the Beckn network.**

The link between the Beckn Adapter and the Beckn network is secured by the authentication specified in an earlier question above. The Beckn Adapter (if it is not embedded within the BAP/BPP) itself is usually hosted on the same VPC as the corresponding BAP or BPP. The rest are implementation details specific to your use-cases.

Q8. **Should update request be sent only from the BAP. In my app, the provider needs to inform the consumer of change. Can I use update.**

If the provider changes the terms of the order itself, then they can send an **unsolicited on_update** to the seeker. For all other status change notifications of the order, the provider platform should use an **unsolicited on_status**.

Q9. **Where do I start the Beckn Journey?**

You can start your journey by experiencing the power of Beckn illustrated in our [experience center](https://experience.becknprotocol.io/). There you find demo apps from a dozen different domains using the power of Beckn.

The next step is to join the Discord channel called [Beckn Open Collective](https://discord.com/invite/VkURq7Zx)

Once done, you can click on the **beckn-starter-toolkit** channel on the left hand side. You will see a list of resources including video resources, starter guides etc. Knock them off one by one and you will be a Beckn master in no time.

Q10. **Can Beckn be used for rental use case?**

Absolutely. Beckn can be used for rental use case. Any commercial transaction can be conducted on Beckn

Q11. **Are there any open networks based on Beckn that are in production and have significant traffic?**

Yes. Within India, ONDC is a large open network for retail that is based on Beckn. Similarly we have the ONest network, which is a skilling network that is also based on Beckn. Apart from these there are smaller and newer networks in other countries such as KuzaOne, an agrinet network from Kenya, Open Amsterdam, a local retail network in Netherlands etc.

Q12. **Is ONDC a different protocol than Beckn?**

No. ONDC is an open network based on Beckn. Some of their products currently are based on an older version of Beckn and some of their new offerings are on latest version of Beckn.

Q13. **Can all network participants (irrespective of their network) use Beckn-ONIX?**

Yes. Beckn-ONIX can be used by any network participant from any open network based on Beckn. The two primary things that would make the network participant a participant of a particular network are

1. The registry to which you connect

2. The layer 2 config file that your network provides to its participants.

Q14. **What is a BAP?**

BAP (Beckn Application Platform) is a consumer-facing infrastructure which captures consumers’ requests via its UI applications, converts them into beckn-compliant schemas and APIs at the server side, and fires them at the network. BAPs are the initiators of transactions and have the flexibility to communicate with multiple networks and integrate the responses from these networks into a bundled transaction experience. For example, a BAP can book a cab via an urban mobility network, order a coffee from a restaurant via a local-retail network, have it picked up via an order on a delivery network and get it delivered on the way to work. 

Q15. **What is a BPP?**

BPP (Beckn Provider Platform) are provider side platforms that maintain an active inventory, one or more catalogs of products and services, implement the supply logic and enable fulfillment of orders. The BPP can be a single provider with a Beckn API implementation or an aggregator of merchants.

Q16. **What is a Gateway, why is it required?**

Beckn Gateways(BG) are extremely lean and stateless routing servers. Its purpose is to optimize discovery of BPPs by the BAPs by merely matching the context of the search. The BG takes a search request from the BAP, determines to which BPPs the message needs to be sent to (by looking up the registry) and multicasts the message only to them. It is used only in the search request.

Q17. **How can someone use the gateway? Mention the current possible scenarios.**

The Gateway is only used during the search request. When consumer searches for a product or a service, the BAP might not know the providers who have this product/service. In such cases, it sends the search request to the Gateway. The gateway looks at the context of the search message and looks up the registry for the BPPs matching the context. It then forwards the search request to all the BPPs returned by the registry. Prior to Beckn Protocol core version 1.0, the BPPs would send the responses back to the Gateway and the Gateway would send it to the BAP. This has been changed and now the BPPs directly send the responses back to the BAP. 

Q18. **What is a Registry, why is it required?**

The Beckn Registry (or the registry infrastructure in case of a more complicated infrastructure), contains a list of network participants, their role in the network, the domain they transact in and their public key. During Beckn transactions, network participants lookup in the Registry for these information about the Participant. Typically the Gateway looks for all BPPs that match the context of the search request. Also all Participants when they get a message from a Sender participant, lookup the registry to see if the sender participant is a subscribed participant and his public key. They use the public key to verify that the message is not tampered. Network Facilitators can use the registry to maintain and manage network participants including their state, KYC compliance etc. The registry has other functionalities and can be organized in a hierarchy. Refer to this [document](https://developers.becknprotocol.io/docs/introduction/the-registry-infrastructure) for more details.


Q19. **Do we need to deploy our own gateway and registry?**

Usually the registry is deployed by a Network Facilitator. There is usually one registry (or in a more general sense, one registry infrastructure) for a network. Network participants (BAP, BPP) do not need to deploy their own registry. 

The Beckn Gateway again is typically hosted by the Network Facilitator, though the protocol itself does not impose any restrictions. Similarly we can have multiple Beckn Gateways in a network, though the usecase is not typical. Again, BAPs and BPPs do not need to host their own gateway.

Q20. **Can we send extra details with Beckn APIs? Does the Beckn schema support this?**

Additional details can be sent in Beckn APIs at multiple levels. The most common way to extend the Beckn API structure is using [**TagGroups and Tags**](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-009-Tags-the-Edge-of-Beckn.md). Tags are supported in most large Beckn Objects such as Item, Fulfillment, Order etc. The second way, particularly to get additional information from the consumers is using a method called [**Xinput**](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-007-The-XInput-Schema.md). 

Q21. **What happens when you move an app from one network to another?**

While moving an existing Beckn enabled application from one network to another, the application does not need to typically change. For Consumer side applications, the following needs to be done.

1. A BAP Beckn Adaptor needs to be subscribed into the new network.
2. The Consumer side application will need to now point to the new BAP Beckn Adapter to transact on the new network.

For Provider side platforms, the following needs to be done.
1. A BPP Beckn Adaptor needs to be subscribed into the new network.
2. The BPP Beckn Adaptor needs to be configured with the Provider side Platform webhook url.
3. The Provider side platform needs to be configured to send the responses (on_search, on_select etc) to the new BPP Beckn Adaptor.

Q22. **How do you track the seeker's location during fulfillment?**

Typically in most usecases the Seeker's location is not tracked. For the purposes of fulfillment, the fullfillment location is part of the order. If the seeker wants to change that, it can be done with the update request. If the usecase requires it, the order can contain link to an external system that will capture and expose the seeker's location to the fulfillment agent. 

Q23. **Given the beckn endpoints are open, how is security established when communicating with BPP / BG endpoints.**

Beckn endpoints are protected using the Subscriber/Public Key mechanism. For example, when a Sender Network Participant sends a message to the Receiver Network Participant, it has to sign the message with its private key. The receiver Network Participant looks up the Sender Network Participant ID in the registry to check if it is a subscribed network participant. It fetches the public key of the sender in the same lookup. It uses this public key to verify that the message came from the Sender and that it has not been tampered with. Refer to [this document](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-006-Signing-Beckn-APIs-In-HTTP-Draft-01.md) for more details.
