<!-- About Project -->
# Configuring Nexus 9000 and Telegraf for Telemetry Collection

The configurations included provide an example configuration and steps to setup a telegraf collector to recieve streaming telemetry from remote Cisco Nexus 9000 switches. 

The configuration illustrates a single telemetry dial-in subscription to recieve interface statistics from remote switches used in the demonstration of an AI workloads impact on the network. Additional sensor paths can be configured to recieve information about switch health and performance for other use cases.

<!-- Getting Started -->
## Getting Started
To get started it is assumed that the root project has been cloned or downloaded to a local development server or workstation. The following instructions only include the details required to configure the Telegraf collector to recieve Nexus telemetry.

### Requirements
- InfluxDB v2 Instance

Before deploying this Telegraf collector a Influx DB v2 instance is required, refer to the `base_tig_stack` directory available in the root of the project for a simplified installation of a Influx DB container for storage of time series data and Grafana for visualization of data.

### Nexus 9000 Configurations
To support the connectivity from a Telegraf collector and to permit the streaming of telemetry data from a Nexus 9000 switch the following configurations are requried. The following are summarized steps for detailed instructions refer to the published Cisco documentation. 

[Configure and Verify gRPC/gNMI on Nexus 9000](https://www.cisco.com/c/en/us/support/docs/switches/nexus-9000-series-switches/220640-configure-and-verify-grpc-gnmi-on-nexus.html)

#### Configure a Self-Signed Certificate
The demonstration uses a self-signed certificate generated on each Nexus Switch for securing the gRPC communication.

1. Enable the bash-shell feature
    ```
    feature bash-shell 
    ```
2. Transition to the bash shell on the local Nexus switch executing the following command from priviledge exec prompt `switchname#` once executed the prompt will change to `bash-4.4#` 
    ```
    run bash sudo su
    ```
3. Using openssl to generate a self-signed certificate signing request.
    ```
    openssl req -x509 -newkey rsa:2048 -keyout grpc_selfsigned2048.key -out grpc_selfsigned2048.pem -days 365 -nodes
    ```

> [!NOTE]
> Once ran the openssl wizard will run to generate the certificate request, complete the answers that meet your requirements. The following is an ***example*** of the form.
```
    ......................................+++++
    ....+++++
    writing new private key to 'grpc_selfsigned2048.key'
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:
    State or Province Name (full name) [Some-State]:
    Locality Name (eg, city) []:
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:
    Email Address []:
```

4. Generate the certificate with the created request.
   ```
   openssl pkcs12 -export -out grpc_selfsign2048.pfx -inkey grpc_selfsigned2048.key -in grpc_selfsigned2048.pem -certfile grpc_selfsigned2048.pem -password pass:C1sco123!
   ```
5. Move the generated certificate to the root of the bootflash for easier referencing and locating for the next steps.
   ```
   mv grpc_selfsign2048.pfx /bootflash/grpc_selfsign2048.pfx
   ```
6. Exit the bash shell and return to the NX-OS privileged exec command line.
   ```
   exit
   ```
#### Configure the gRPC
1. Enable the grpc feature
   ```
   feature grpc
   ```
2. Configure the gRPC to use the approprate VRF for communication with telegraf collector. This is not required if using the out-of-band management interface `mgmt0`.
   ```
   grpc use-vrf default
   ```
3. Configure the trustpoint using previously created certificate
   ```
   crypto ca trustpoint grpc_trustpoint
    crypto ca import grpc_trustpoint pkcs12 grpc_selfsign2048.pfx C1sco123!
   ```
4. Associate the trustpoint with the gRPC
   ```
   grpc certificate grpc_trustpoint
   ```

#### Enable Open Config support on Nexus 9000
To use open-config sensor paths the following feature is required.
```
feature openconfig
```

#### Save Configurations
Once above configurations are complete save the configurations on each Nexus device with `copy run start`. 

Additionally, if using self-signed certificates as outlined above the certificate will need to be exported and saved locally to allow the Telegraf instance to securely connect to the remote switches. A empty `trusted_certs.pem` file is included in the repo that each certificate can be saved in. For multiple devices save all certificates in the file creating a certificate chain.

### Configure Telegraf
Telegraf is an open-source server agent used for collecting metrics. In this scenario the official Telegraf container is used to simplify the deployment of the collector. A single container is used to collect the telemetry from the switchs and export the data to a InfluxDB instance.

> [!NOTE]
> The below instructions uses a compose file to deploy the container, the information can be added to the larger compose file used in the base_tig_stack directory in the root of the project if wanting to run on the same host with the InfluxDB and Grafana container. Instructions for consolidation are not included.

#### Configure Telegraf Configuration File
The following steps are needed to adjust the Telegraf configuration file `telegraf.conf` included in the directory.

##### Step 1: Update Environment Variables
To simplify some of the configurations edit the `telegraf.env` file included. During deployment of the `compose.yaml` this will ease the updating of some of the configurations used.

The following are the variables defined the defaults.

| Variable | Default | Usage | Update Required | 
|----------| :---: |-------| :---: |
| DOCKER_TELEGRAF_HOSTNAME | nxos-collector | Hostname used for the Telegraf Container |  |
| DOCKER_INFLUXDB_URL | | IP/FQDN of Remote INFLUXDB Server Instance | :white_check_mark: |
| DOCKER_INFLUXDB_INIT_ORG | myaiorg | InfluxDB Orginization to send data to |  |
| DOCKER_INFLUXDB_INIT_BUCKET | influx | InfluxDB Bucket used to store data |  |
| DOCKER_INFLUXDB_INIT_ADMIN_TOKEN | supersecuretoken | Token to access InfluxDB | :white_check_mark: |

#### Additional Telegraf Configurations
To complete the configurations of the Telegraf configuration file the following tasks are completed directly in the `telegraf.conf` file.

1. Locate the `[[inputs.gnmi]]` section with in the configuration file.
2. Update the `addresses = [""]` string. This is a list of the ip addresses for the Nexus switches that will be streaming the telemetry date, ensure to include the port used to connect to the gNMI service on the switches. An example configuration for 2 switches would be as follows:
    ```
    addresses = ["10.1.1.10:50051", "10.1.1.11:50051"]
    ```
3. Configure the `username` and `password` to authenticate to the remote switches.
4. Locate the `[[inputs.gnmi.subscription]]` section in the configuration file. This is the configuration to configure the specific sensor path for telemetry subscription on the remote switches. A single sensor path is included if additional sensor paths are needed add additional `[[inputs.gnmi.subscriptions]]` sections.

### Deploy Telegraf Container
Once the `telegraf.env` and `telegraf.conf` files are updated and the Nexus switches are provisioned for dial-in subscription and an InfluxDB instance is up and awaiting data the contianer can be deployed.

Use the compose file to deploy the container simple execute the appropriate command depending on the use of Docker or Podman. 

**Docker**
``` bash
docker compose up -d compose.yaml
```

**Podman**
``` bash
podman compose up -d compose.yaml
```

## Conclusion
Once the container is up streamed telemetry from the Nexus switches will be sent to the InfluxDB Organization and Bucket identified in the configuration files. Using the Explorer tools in the InfluxDB the data can be viewed and filters created to assist in dashboarding.

<!-- Roadmap -->
## Roadmap
- [ ] Add example for dial-out configuration 
- [ ] Add Verification images
<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
