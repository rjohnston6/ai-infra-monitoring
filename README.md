<!-- About Project -->
# 🚧 WORK IN PROGRESS 🚧

<!-- Getting Started -->
## Getting Started
For detailed configuration for deploying each component refer to `README.md` in each directory for **important** information.

Review the documentation for each step and deploy appropriately.
1. Deploy the [tig_stack][1]
2. Deploy a Telegraf collector for NX-OS Streaming telemetry [nxos_telegraf][2]

   As an alternative to deploying two telegraf containers the telegraf configuration in the [tig_stack][1] can be edited to exclude the telegraf service.
3. Deploy a telegraf container on each accelerated computing node using the container configuration provided in [rjohnston6/nvidia-smi-telegraf][3]. The configuration is stored in a seperate repo to reduce the amount of content cloned to a compute node.

## Configure Grafana
To complete the setup and start visualizing data in Grafana the InfluxDBv2 data source must be added. 

1. Navigate to the Grafana Web UI using http://<host_ip>:3000 and login using the admin username and password defined in [grafana.env][4] default is "admin/admin". 
2. Navigate to `Connections > Data Sources` and select Influxdb.
3. Configure the connection using the following settings.
   
   | Setting | Suggested/Example Value |
   | --- | --- |
   | Name | influxdb |
   | Query Language | Flux |
   | HTTP: URL | http://influxdb:8086 |
   | Auth | All Settings Disabled |
   | InfluxDB Details: Organization | ai-visibility |
   | InfluxDB Details: Token | supersecrettoken |
   | InfluxDB Details: Default Bucket | ai-visibility |
   | InfluxDB Details: Min time Interval | 1s |

4. Click `Save & Test` to save and confirm connectivity.

## Create Grafana Dashboards

> [!IMPORTANT]
> It is critical that all devices and servers in the setup source time from an NTP server. If not dashboards will show inaccurately timed data

An example dashboard is provided to provide a starting point to visualize various data points from the data collected and stored in the InfluxDBv2 instance.
![Example Dashboard Screenshot][5]

Some modifications are required to the dashboard to fully utilize and visualize the date. Specifically the configurations for `Switch Interface Utilization` panel as the configuration needs to be configured for the appropriate switches and interfaces. Use the following to update.

1. Click the 3 Dots in the Right of the `Switch Interface Utilization` pane and select `edit`.
2. In the queries section edit the following filter to match the switch to monitor.
   ```flux
   |> filter(fn: (r) => r["source"] == "198.18.0.40")
   ```
3. Edit the filter for the interfaces, add additional interfaces using the or operator.
   ```flux
   > filter(fn: (r) => r["name"] == "eth1/1" or r["name"] == "eth1/2")
   ```
4. Repeat for both queries listed.
5. Save Dashboard.

<!-- Roadmap -->

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[1]: /tig_stack/
[2]: /nxos_telegraf/
[3]: https://github.com/rjohnston6/nvidia-smi-telegraf
[4]: /tig_stack/grafana.env
[5]: /grafana/img/example_dashboard.png