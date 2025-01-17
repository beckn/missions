# Beckn Registry User Guide

In the Beckn ecosystem, the Registry serves as the foundational component for setting up a new network. The Registry is where all network participants register their public keys, and it acts as a lookup endpoint, enabling participants to discover and validate each other within the network.

The Registry is the core component that represents and governs a Beckn network. It serves several critical functions:

1. **Network Participant Registration**: All network participants (BAPs, BPPs, and Gateways) must register and subscribe to the Registry to be officially recognized on the network.

2. **Public Key Infrastructure**: The Registry maintains a database of public keys for all registered participants. These keys are essential for:
   - Validating the authenticity of participants during transactions
   - Ensuring secure communication between participants
   - Preventing unauthorized access to the network

3. **Transaction Validation**: During each transaction on the network:
   - The Registry validates the digital signatures of participants using their stored public keys
   - This ensures that only authorized participants can engage in transactions
   - It helps maintain the security and integrity of the network

4. **Network Discovery**: The Registry enables participants to:
   - Discover other legitimate participants on the network
   - Verify the authenticity of potential transaction partners
   - Access participant metadata and capabilities

For detailed technical information about Registry implementation and integration, please refer to the [Beckn-ONIX guide](https://github.com/beckn/beckn-onix).


## Create an Account

To create an account on the Beckn Registry, use the following link:

[Register](https://registry.becknprotocol.io/register)

![Register](assets/images/registry/15-register.png)

## Login

Once your account is created, log in using the link below:

[Login](https://registry.becknprotocol.io/login)

![Login](assets/images/registry/1-%20Login.png)

## Forgot Password

If you forget your password, reset it by selecting the "Forgot Password" option on the login page. Enter your user details to proceed. If you encounter any issues, contact the Admin.

[Forgot Password](https://registry.becknprotocol.io/login)

![Forgot Password](assets/images/registry/forget-pwd.png)

## Create New Network Domain

**Note:** The network domain represents the domain that will be used for transactions within the Beckn network. For example, if the domain field in your Postman request is "retail", you need to have a network domain called "retail" added in the registry, otherwise the transaction will not go through.


1. Click on the "Beckn Menu" located on the toolbar and select "Network Domain" from the dropdown.

   ![Select Network Domain](assets/images/registry/2-network-domain.png)

2. A list of already added network domains will appear. Click on the "Add New" option.

   ![Add New Domain Option](assets/images/registry/3-add%20new%20domain%20option.png)

3. Fill in the "Name" and "Description" fields, then click "Save" and "Done."

   **Note:** The "Schema URL" field holds the URL of the protocol specification file, which outlines the technical standards and details that the system follows.

   ![Name and Description](assets/images/registry/4-name%20and%20descp.png)

4. You can view, edit, duplicate, or delete the added domain by selecting the respective options.

   ![Domain List](assets/images/registry/5-domain%20to%20list.png)

## Create/Edit/View Network Participant

1. You will see a list of already added network participants.

   ![Select Network Participant](assets/images/registry/6-selecting%20np.png)

2. Click on "Add New" to add a participant. You can also search for an existing network participant.

   ![Add New Network Participant](assets/images/registry/7-adding%20new%20np%20and%20search.png)

3. Enter the "Participant ID" and click "Save" and "Done."

   **Note:** The "KYC Complete" checkbox reflects the KYC (Know Your Customer) status. Documents can be submitted for verification in the 'Submitted Documents' tab, but the system allows participants to mark it complete by default. No further user action is needed.

   ![KYC](assets/images/registry/16.kyc.png)

4. The network participant record will be added to the list. You can now edit the record for further actions.

   ![Edit Network Participant](assets/images/registry/8-edit%20np.png)

## Create/Edit/View Network Role

**Note:** This section describes manual network role creation through the registry UI. If you are using Beckn-ONIX to install a network participant (BAP/BPP), you can skip this step as the installation process automatically sends a request to create both the network participant entry and its associated network role in the registry. Manual creation is only needed if you are developing a protocol server from scratch or have specific requirements for manual addition. Alternatively, you can also use curl requests to add network participants programmatically.


1. From the "Admin" menu bar, select "Network Participant" and choose the participant to which a role needs to be added.

2. Click "Edit," navigate to the "Network Role" tab, and click "Add New."

   ![Network Role Tab](assets/images/registry/9-network%20role%20tab.png)

3. Choose the "Network Domain" from the dropdown, fill in the required fields, and click "Save" and "Done."

   **Notes:**
   - **Core Version:** This indicates the Beckn specification version supported. Currently, no validation against the Beckn Spec version is enforced, though it may be in the future.
   - **Status:** This shows the network role's current status, typically 'Initiated' or 'Subscribed.' No user action is required for other options.

   ![Network Role Information](assets/images/registry/10-networkRole-infor.png)

4. In the "Operation Region" tab, specify the geographic region where the network participant operates. If operating globally, leave the field blank.

   ![Operation Region](assets/images/registry/21-operatingregion.png)

## Create/Edit/View Participant Key

**Note:** This section describes manual participant key creation through the registry UI. If you are using Beckn-ONIX to install a network participant (BAP/BPP), you can skip this step as the installation process automatically sends a request to create the participant key in the registry. The public keys that would be added here can be found in the default.yml file inside the installed protocol server docker container if you used Beckn-ONIX for installation. Manual creation is only needed if you are developing a protocol server from scratch or have specific requirements for manual addition. Alternatively, you can also use curl requests to add participant keys programmatically.


1. Under "Network Participant," select the participant to edit.

2. Click on the "Participant Key" tab and click "Add New" to add a key.

   ![Participant Key](assets/images/registry/11-participant%20key.png)

3. Enter the "Key ID," "Signing Public Key," "Encryption Public Key," and other details. Select the "Valid From" and "To" dates, mark the "Verified" checkbox, and click "Save" and "Done."

   ![Participant Key Details](assets/images/registry/12-participant%20key%20details.png)

4. The participant key record will now be displayed in the list.

   ![Participant Key Record List](assets/images/registry/13-participant%20key%20record%20list.png)

## Logout

To log out, select "Admin" from the menu and then choose "Logout."

![Logout](assets/images/registry/14-logout.png)
