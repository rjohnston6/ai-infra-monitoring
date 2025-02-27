<!-- About Project -->
# 🚧 WORK IN PROGRESS 🚧
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
<!-- Roadmap -->
## Roadmap
[ ] Add example for dial-out configuration 
<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
