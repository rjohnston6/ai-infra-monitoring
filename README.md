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

## Create Grafana Dashboards

<!-- Roadmap -->

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[1]: /tig_stack/
[2]: /nxos_telegraf/
[3]: https://github.com/rjohnston6/nvidia-smi-telegraf