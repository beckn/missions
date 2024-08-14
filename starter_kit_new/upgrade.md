## Upgrading

This section describes the process to follow to upgrade from non Beckn-ONIX installation of Protocol Server to a Beckn-ONIX based installation. It also describes the process of upgrading from one version of the software to another.

### Upgrade from non Beckn-ONIX version to Beckn-ONIX

If you had installed the components of a Beckn network using legacy installations, use the following steps to migrate to Beckn-ONIX

1. Write down all the subscription information relevant to the component (e.g. subscriber_id, subscriber_url, registry_url, webhook_url-for BPP PS). If the installed Beckn Adapter is the Protocol Server, most of these can be found in the default.yaml file within the config folder.
2. Stop the running instances of the Beckn Adapter (including support services such as Mongo, Rabbit and Redis). Previously Protocol Server used to be run using the pm2 service. So see pm2 list to see the instances running, in order to stop them.
3. Backup all the log and config data.
4. Ensure that the reverse proxies are all properly configured. The default ports used are

- BAP PS Client - 5001
- BAP PS Network - 5002
- BPP PS Client - 6001
- BPP PS Network - 6002

5. Clone the Beckn-ONIX repo and start installation. Refer to the GUI/CLI manual at the [repo](https://github.com/beckn/beckn-onix) for more details.

```
$ git clone https://github.com/beckn/beckn-onix.git
$ cd beckn-onix/install
$ ./beckn-onix.sh
```

6. Choose "Join an existing network" and the right option (BAP/BPP on what you are replacing).
7. Use the subscriber information that you copied from the older network in step 1 above.
8. Once the installation is done, run the script to download and install the layer 2 configuration for the domains. The download URL to use will be provided in the implementation guide for your network.

```
$ cd ../layer2
$ ./download_layer_2_config_bap.sh    or    download_layer_2_config_bpp.sh
```

9. Type the public key from the new installation and ensure that it is the same as that in the registry (either login to the registry or take the help of your network facilitator). In the code below it is shown for BAP. To make it work for BPP, replace bap-network with bpp-network.

```
$ docker exec -it bap-network cat config/default.yml | grep publicKey
```

### Upgrading existing Beckn-ONIX installation

If you already have a version of any of the components (BAP Protocol Server, BPP Protocol Server, Registry, Gateway) previously installed by Beckn-ONIX, use the following process to upgrade to the latest available. The example below shows the process to update the BAP Protocol Server. The process is identical for BPP Protocol Server and similar for Registry and Gateway.

1. Get the subscriber_id, subscriber_url, registry_url and webhook_url (for BPP Protocol Server) from existing installation

```
$ docker exec -it bap-network cat config/default.yml
```

2. Run the installation again to install the new components and give the noted down values when asked.

```
$ cd beckn-onix/install
$ docker compose -f docker-compose-bap.yml down
$ git pull origin main
$ ./beckn-onix.sh
```

3. The installation will complete.
4. Type the public key from the new installation and ensure that it is the same as that in the registry (either login to the registry or take the help of your network facilitator). In the code below it is shown for BAP. To make it work for BPP, replace bap-network with bpp-network. (There is a issue ticket in Beckn-ONIX repo to make this step optional.)

```
$ docker exec -it bap-network cat config/default.yml | grep publicKey
```

5. Using postman or any other method used by your business, ensure the setup is working as before.
