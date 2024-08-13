Beckn FAQ will cover all the frequently asked questions which has been asked by the community

Q1. **Is cloud credentials required for development and any associated costs. – Does beckn provide the EC2 instance required for hosting the beckn artefacts such as registry, protocol server etc ?**

We have a BOC (Beckn Open Collective) provided public network. NP’s can utilise that for testing purposes. Sometimes the network facilitator also has a sandbox environment for participants related to them.

Q2. **Are there any rate limits or usage quotas for the Beckn APIs that we should be aware of?**

NO, Beckn is a specification with no run time limitation. Some of the reference implementations of the components might have it, but the protocol itself is agnostic. 

Q3. **What is the recommended approach (OAuth, API keys, or JWT??) for authenticating API requests to the Beckn network?**

Beckon Protocol has a [authentication section](https://github.com/beckn/protocol-specifications/blob/master/docs/BECKN-006-Signing-Beckn-APIs-In-HTTP-Draft-01.md)  that describes the process with examples.

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
