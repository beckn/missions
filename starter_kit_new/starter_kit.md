# Beckn Starter Kit

## Table of Contents

1. [Glossary](#glossary)

2. [Overview of Steps for Network Participants](#overview-of-steps-for-network-participants)





## Glossary

- **Beckn APIs:** APIs that are part of the Beckn protocol specifications and responsible for enabling two platforms (consumer & provider side) to discover & transact with each other in an Open Network. (Discovery-Order-Fulfillment-Post-fulfillment)

- **Beckn Application Platforms (BAP):** A consumer-facing infrastructure that captures consumers’ requests via its UI applications, converts them into Beckn-compliant schemas and APIs at the server side, and fires them at the network.

- **Beckn Gateway (BG):** Extremely lean and stateless routing servers. The purpose of this infrastructure is to optimize the discovery of BPPs by the BAPs by merely matching the context of the search.It multicasts search requests from the BAP to relevant BPPs.

- **Network Participants (NPs):** In the context of an "open network," a network participant typically refers to any individual, entity, or device that interacts with or is connected to the network.

- **Protocol Server:** Reference implementation of a Beckn Adapter. Beckn Adaptor helps Network Participants connect to Beckn Network. It performs functions of validation, authentication and assist communication between network participants.

- **Sandbox:** A controlled and isolated environment where software applications, code, or processes can run and be tested without affecting the broader system or other applications. Think of this environment as a playground. 

- **Open Network:** A type of network characterized by its accessibility and inclusivity, where the infrastructure, protocols, and standards are available to a wide range of participants without restrictions. 


## Overview of Steps for Network Participants

1. **Step 1:** Build understanding on Beckn protocol - Refer to the ["Introduction"](https://github.com/beckn/missions/blob/main/starter_kit/starter_kit.md#introduction) section.<TODO>

2. **Step 2:** Identify the sector or domain you want to solve the Discovery-Order-Fulfillment-Post Fulfillment (DOFP) for (e.g., Health, Mobility, Energy, Skilling, Agriculture).

3. **Step 3:** Define the role your platform will play as a Network Participant on an Open Network (BAP or BPP or a TSP).

4. **Step 4:** Define and visualize the outcomes you want to achieve with the selected Network Participant role using the "Outcome Visualization template".

5. **Step 5:** Register your platform on the Beckn sandbox by registering on the Beckn Registry for testing against other Network participants. The network facilitator will provide with 

6. **Step 6:** Deploy ONIX as a prerequisite pre-API integration step and set up the protocol server for API integration and testing.

7. **Step 7:** Map your platform schema with the Beckn API schema relevant to the transaction.

8. **Step 8:** Refer to the domain-specific Implementation guide to integrate the Beckn API as per the selected Network Participant role.

9. **Step 9:** Test with Postman collection and against the protocol server.

10. **Step 10:** Test with other NPs on the network for one complete DOFP flow in the Beckn sandbox. The Sandbox details are mentioned in the Sandbox for Network Participant section below [here](https://github.com/beckn/missions/blob/main/starter_kit/starter_kit.md#Sandbox-for-Network-Participant). <TODO>

11. **Step 11:** Prepare for Go Live access if a live network instance is set up or ready.


