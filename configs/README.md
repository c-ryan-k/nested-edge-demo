This directory contains edge hierarchy configurations and some JSON edge deployment files.

### Regarding Config Files

Edge/Local device configs are only different to remember different IP addresses as I move devices between networks.

All config files reference deployments in their local directory, so if you want to run `az iot edge devices create` with configurations in this directory, `cd` to this directory before running the command to avoid pathing issues.

You may also want to specify an output path in a sibling directory, like: `--out ../device_bundles/`

### Regarding deployments

Anything labeled "top" is for root-level parent devices. These typically run:

- An IoT Edge Proxy for passing data from leaf devices up to the hub
- A local container registry for downstream devices to pull agent images and modules from

Anything labeled "mid / middle" is for intermediate devices, they do act as a proxy for downstream devices but may also send telemetry to an upstream gateway device.

Anything labeled "low / lower" is for leaf devices that need to get agent images / configurations from upstream proxy devices. These devices do not have any proxies or container registries, and simply (at least in this scenario) run sample data telemetry to send upstream.
