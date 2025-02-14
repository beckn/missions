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

The Beckn Gateway again is typically hosted by the Network Facilitator, though the protocol itself does not impose any restrictions. Similarly we can have multiple Beckn Gateways in a network, though the use case is not typical. Again, BAPs and BPPs do not need to host their own gateway.

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

Typically in most use cases the Seeker's location is not tracked. For the purposes of fulfillment, the fulfillment location is part of the order. If the seeker wants to change that, it can be done with the update request. If the use case requires it, the order can contain link to an external system that will capture and expose the seeker's location to the fulfillment agent. 

Q23. **Given the beckn endpoints are open, how is security established when communicating with BPP / BG endpoints.**

Beckn endpoints are protected using the Subscriber/Public Key mechanism. For example, when a Sender Network Participant sends a message to the Receiver Network Participant, it has to sign the message with its private key. The receiver Network Participant looks up the Sender Network Participant ID in the registry to check if it is a subscribed network participant. It fetches the public key of the sender in the same lookup. It uses this public key to verify that the message came from the Sender and that it has not been tampered with. Refer to [this document](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-006-Signing-Beckn-APIs-In-HTTP-Draft-01.md) for more details.

Q24. **I would like to participate as BAP in an open network, How do i get started. Is there any open source assets which are available which I can use.**

Currently Beckn-ONIX provides a BAP Adaptor. There is an effort to release a sample BAP application. But till then, BAP adaptor is all we have as BAP side resource. However in the development process of BAP, you can use the BOC cloud environment for BAP PS and sample BPP (sandbox)

Q25. **I would like to participate as BPP in an open network, How do i get started. Is there any open source assets which are available which I use.**

Currently Beckn-ONIX provides a BPP Adaptor. There is no sample BPP that is independently released. However in the development of BPP, you can use other resources such as Postman collection for different domains available in the beckn-sandbox repo, artefacts folder. You can use these postman collection to send requests to your BPP while you develop it.

Q26. **Is beckn build on blockchain technology**

Beckn is a open protocol used to build open networks that enable Peer-to-Peer commerce transactions. However it is not related directly to BlockChain. There are some community efforts such as [here](https://becknprotocol.io/enabling-trusted-commerce-using-beckn-protocol-and-blockchain/) which try to combine the two technologies to build applications.

Q27. **I have any existing platform both seeker and provider side, Why should i becknify my application. What advantage do i get.**

Unbundling your application and putting it on Beckn network has advantages on both the sides. Your seeker side platforms can now discover other Providers. Similarly your provider side application can now get orders from other Seeker side applications than yours. 

Q28. **What is the min infra required for setting up a Gateway**

A machine with 2 vCPU and 4 GB RAM should be sufficient

Q29. **What is the min infra required for setting up a Registry**

A machine with 2 vCPU and 4 GB RAM should be sufficient

Q30. **What is the min infra required for setting up Protocol Server**

When you install the protocol server, you also install the support services of Mongo, Redis and RabbitMQ. So while 2vCPU with 4 GB RAM is sufficient for a protocol server to function, it helps if you can add more CPU and RAM as the traffic increases.

Q31. **What does ACK indicate, when should i be sending an ACK**

In Beckn all Transaction API messages are asynchronous. That is if you send a select message to the BPP, the BPP will send on_select later. However in response to your select, the BPP will send a ACK/NACK message. An ACK means that the message you sent seems on the first look ok (syntactically correct). You can wait for a response from the participant you sent the request. Alternatively if a NACK is sent back on your select, it means that the BPP found some error in your message (the details will be in the body) and will not send you back an on_select for this. 

Q32. **What does NACK indicate? When should i be sending a NACK as a BPP and While processing a beckn BAP request, I see that some mandatory fields are missing in the request, how do i communicate back to BPP, should i be sending an NACK with an error code or an empty response with error code**

Please read the answer to the previous question also. But yes you can send a NACK if you feel you cannot proceed with the processing, like in the example you mention. You can use the body of the error that is sent back to give additional details to the sender.

Q33. **Should update request be sent only from the BAP. In my app, the provider needs to inform the consumer of change. Can I use update.**

Yes any change that affects the terms of the contract(order) can trigger an on_update. So if you are a provider and the terms of the order have changed, sending an on_update back to the BAP is a good option. For all other updates that do not involve the terms of the order (such as fulfillment status updates etc), you can send an on_status message. Since both of these messages originate on the BPP, they will not have a corresponding request message.

Q34. **What parts of beckn protocol in ondc or open mobility can be self hosted ?**

Answering generally, in any existing Beckn Network, the Consumer side application (BAP) and the Provider Platform (BPP) are hosted by network participants.

Q35. **How is grievance management done among network participants ?**

Currently the grievance management terms and process depends on the network and defined by the network facilitator. 

Q36. **How to understand beckn from scratch ?**

The best source to learn Beckn from scratch is the video series by Ravi Prakash (co-author of the Beckn Protocol). You can find it [here](https://www.youtube.com/watch?v=7Otfcy37-NE&list=PLBC6c8MLy9uVUIb1BOgdOa8tP4rX6c4aK). You can also go to [becknprotocol.io](https://becknprotocol.io) and find resources there. There you will also find link to join the discord channel. The discord channel has a starter kit for beginners.

Q37. **How does Digital signature work in beckn and any resource to understand that ?**

You can use this [Signing Beckn APIs in HTTP](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-006-Signing-Beckn-APIs-In-HTTP-Draft-01.md) to understand the process.

Q38. **How can we test a BPP without setting up a BAP ?**

You can use the Postman collection corresponding to your domain from the beckn-sandbox repository. Its under a folder called [artefacts](https://github.com/beckn/beckn-sandbox/tree/main/artefacts). These connect to cloud hosted BAP that is connected to the https://registry.becknprotocol.io. If you register your BPP with the same registry, then the request you send from Postman should reach your BPP and you can test the BPP this way.

Q39. **Any specific way of using beckn api and its fields ?**

The [Beckn Core Transaction API Spec](https://github.com/beckn/protocol-specifications/blob/master/api/transaction/build/transaction.yaml) describes the Beckn Transaction API and its fields.

Q40. **How can non-techies contribute to beckn ?**

1. Goto https://becknprotocol.io. There you have a link to join discord in the top right.
2. Use that to join the Beckn Open Collective discord server.
3. Use the general discussions lounge to talk to the community and see where you can contribute to and benefit from Beckn.

Q41. **Is Beckn a blockchain ? How does cryptography comes into picture ?**

Beckn is an open protocol that enables development of open commerce networks. However Beckn does not depend on or use Blockchain. Beckn with the help of the registry uses a signature based private-public key authentication to ensure data trust. 

Q42. **What is the difference between Beckn and ONDC ?**

Beckn is an open protocol that enables development of open commerce networks. ONDC is one implementation of Beckn that allows transactions in many domains (retail and others). 

Q43. **What makes Beckn “Decentralized” if it’s not a traditional blockchain architecture ?**

Transactions on Beckn is Peer-to-Peer. There is no one single authority who has knowledge of all transactions that happen on Beckn. In that sense Beckn is decentralised. Beckn uses a participant called **Registry** that aids in the trust aspect between the participants.

Q44. **How to test if registry and gateway are setup properly ?**

You can do it in a few ways.
1. You can send a search request to the gateway and see if it is multicasting it.
2. You can see the logs of the gateway (through https://your-gateway-address/bg/log/0) after restart and if you do not see any exceptions, its fine
3. Gateway contacts the registry on boot up and in many cases if it cannot communicate with registry, it keeps restarting. So if your gateway container is restarting, then probably the setup has issues.

Q45. **What does a Network Facilitator mean ?**

Network facilitator is someone who sets up the network and manages the governance aspects. Typically they own the registry. They may control the network participants who can transact on the network. 

Q46. **How can authentication happen on a customer facing app ? is it supported by Beckn ?**

The authentication on the BAP (Consumer facing app) towards the customer side is not covered by Beckn. BAP developers can choose the authentication they like. 

Q47. **How can we handle payments in a production grade customer app ?**

This document [Payments on Beckn networks](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-002-Payments-On-Beckn-Enabled-Networks.md) explains the concepts involved in payments on Beckn. However the actual implementation is left to the application developers. 
