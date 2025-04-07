<!-- About Project -->
# 🚧 WORK IN PROGRESS 🚧
# Deploy a Basic TIG Stack for AI Monitoring

The configurations included provide an example configuration and steps to setup a basic Telegraf, Grafana and Influx (TIG) stack. 

This TIG stack can be used to collect statistics for a number of use cases, the scenario described below is with a single Influx bucket and used to showcase statistics gathered from a pair of compute hosts used with Nvidia GPU's highlighting generative AI use cases. 

The telegraf container deployed as part of the task is configured to gather container statistics for verification, it can be removed from the compose file if desired. The `telegraf.conf` could be altered to also collect stats from other sources if preferred. In the example scenario refer to the directory [nxos_telegraf](../nxos_telegraf/) in the root repo for a telegraf collector for recieving streaming telemetry from a Nexus N9300.

<!-- Getting Started -->
## Getting Started
To get started it is assumed that the root project has been cloned or downloaded to a local development server or workstation. The following instructions only include the details required to configure the TIG stack.

### Requirements

No specific prerequisites are needed to deploy the base TIG stack.

#### Step 1: Update Environment Variables
Prior to deploying the TIG stack with the included `compose.yaml` review and configure the `influxv2.env` file. The file includes environment variables that will be used to configure the inital setup of the InfluxDB container instance and will also be leveraged on to configure the Telegraf container to be able to export data to an initial InfluxDB bucket.

| Variable | Default Value | Usage | Update Required |
|----------| :---: |-------| :---: |
| DOCKER_INFLUXDB_INIT_MODE | setup | No change required. Used to Setup InfluxDB on deploy |  |
| DOCKER_INFLUXDB_INIT_USERNAME | admin | The inital administrator user to be created |  |
| DOCKER_INFLUXDB_INIT_PASSWORD | supersecurepassword | The inital adminstrator password | :white_check_mark: |
| DOCKER_INFLUXDB_INIT_ORG | ai-visibility | InfluxDBv2 Inital Organization | |
| DOCKER_INFLUXDB_INIT_BUCKET | ai-visibility | Initial InfluxDBv2 Bucket to be created | |
| DOCKER_INFLUXDB_INIT_ADMIN_TOKEN | supersecuretoken | Token for admin user, used by Telegraf to authenticate and export data | :white_check_mark: |

#### Step 2: Configure Grafana Variables
Grafana is configured using the `grafana.env` file to set the environment variable for initalizing the Grafana container. The following should be reviewed updated appropriately.

| Variable | Default Value | Usage | Update Required |
|----------| :---: |-------| :---: |
| GF_SECURITY_ADMIN_USER | admin | The inital administrator account to be created | |
| GF_SECURITY_ADMIN_PASSWORD | admin | The initial administrator password | :white_check_mark: |
| GF_INSTALL_PLUGINS| | Plugins to be installed during deployment | |

### Step 3: Deploy TIG Stack
Once the `influxv2.env` is updated the TIG stack can be deployed using the `compose.yaml` file.

Use the compose file to deploy the container simple execute the appropriate command depending on the use of Docker or Podman. 

**Docker**
``` bash
docker compose -f compose.yaml up -d
```

**Podman**
``` bash
podman compose -f compose.yaml up -d
```

#### Step 5: TIG Stack Container Verification
Once the compose file is deployed verify the containers are up and running using `docker ps`. The following is an example output showing the 3 deployed containers and the ports they are listening on.

```bash
CONTAINER ID   IMAGE                    COMMAND                  CREATED              STATUS              PORTS                                                                       NAMES
26f235e1503c   telegraf                 "/entrypoint.sh tele…"   About a minute ago   Up About a minute   8092/udp, 8125/udp, 8094/tcp, 0.0.0.0:8125->8125/tcp, [::]:8125->8125/tcp   telegraf
e6e0ddf03136   grafana/grafana          "/run.sh"                About a minute ago   Up About a minute   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp                                 grafana-server
1ac98ed98682   influxdb:2.7.10-alpine   "/entrypoint.sh infl…"   About a minute ago   Up About a minute   0.0.0.0:8086->8086/tcp, [::]:8086->8086/tcp                                 influxdb
```

From the output InfluxDB is accessible using http://<host_ip>:8086 and Grafana is accessible using http://<host_ip>:3000.

## Additional InfluxDB Configurations
> [!NOTE]
> It is strongly recommended to complete the following configurations. They are optional during basic testing but if collecting stats for more then 24 hours the buckets may fill disk space causing services to become unavailable.

The inital bucket created during the inital `docker compose up` sets up the retention policy for indefinite, which will collect and store data with out any deletion. As this is a demonstration configuration this retention policies are inefficient and should be changed. Complete the following to update the retention policy for the inital bucket created.

1. Connect to a bash session on the InfluxDB container
   ```bash
   docker exec -it influxdb /bin/bash
   ```

2. To list the currently configured buckets use the `influx bucket list` command.
    ```bash
    influx bucket list
    ```
    Example output:
    ```bash
    aa428a7505fb:/# influx bucket list
   ID                      Name            Retention       Shard group duration    Organization ID         Schema Type
   f1db572dcc180aaa        _monitoring     168h0m0s        24h0m0s                 b1ef1db42216d126        implicit
   ae3197d074c0cc52        _tasks          72h0m0s         24h0m0s                 b1ef1db42216d126        implicit
   0a4ae682218faf76        ai-visibility   infinite        168h0m0s                b1ef1db42216d126        implicit
   ```

3. Update the retention period and shard-group-duration for the initial bucket, using the example configurations the bucket is "ai-visibility". To make the change use the bucket id provided in the previous step, with the following command.

   ```bash
   influx bucket update -i 0a4ae682218faf76 -r 48h --shard-group-duration 1h
   ```

For details on InfluxDBv2 Retention and Shard Group Durations refer to the InfluxDBv2 documentation for [Data retention in InfluxDB](https://docs.influxdata.com/influxdb/v2/reference/internals/data-retention/#:~:text=The%20InfluxDB%20retention%20enforcement%20service,%2Dcheck%2Dinterval%20configuration%20option.)

## Conclusion
At this point a basic TIG stack is configured and ready to accept data, all data will be written to a single bucket in InfluxDB. As an Alternative different sources can be written to different buckets to manage data more efficiently.

<!-- Roadmap -->
## Roadmap
- [ ] 
- [ ] 
<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
