## IoT nested edge hierarchy demo

### Files in this repository fall into two categories:

[`ansible`](./ansible/) - Ansible scripts (and device inventory files) for performing automated device tasks in parallel.

[`configs`](./configs/) - Device configs (for CLI) and sample deployment JSON files.

### Prerequisites:

- Devices
  - Online with static (or relatively static) IP
  - Linux, Python, IoT Edge agent device prerequisites
- IoT Hub
- Azure CLI (IoT Extension)
  - `az iot edge devices create` will create the device configurations, bundled scripts and certs, and configure everything to the hub.
  - [iot edge config file](./configs/edge_hierarchy_config.yaml)
- Ansible
  - Uses a root controller / node to manage groups of devices through "Playbooks". Playbooks allow us to run the same parameterized commands on multiple devices at once using SSH - which speeds up device configuration steps that are typically done manually.
  - [hosts file example](./ansible/device_inventory.yaml)
  - The following playbooks are used in this demo:
    - [User and access setup through SSH](./ansible/ssh_user_configuration.yaml)
    - [Known hosts config](./ansible/add_known_hosts.yaml)
    - [Install IoT Edge runtime](./ansible/install_edge_agent.yaml)
    - [Install device bundle produced by CLI](./ansible/install_device_bundle.yaml)

## Steps

### 1. [Setup]

Make sure all devices are online and pingable. 

Since we're installing the Edge Agent and setting up a separate user inside ansible tasks, the only configuration these devices need are a relatively stable IP address.

For this demo, I've used the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to image each device with:

- Current version of Raspbian Lite (headless)
- An `edge` user authorized with a pre-existing SSH key I've created.
- Hostnames configured to their eventual IoT Hub device identities (`pi-1`, `pi-2`, etc)

### 2. [Ansible] - [Known Hosts config](./ansible/add_known_hosts.yaml)

This step will scan all hosts for their public keys, and add them to the controlling node's `known_hosts`. This allows us to ssh between devices without being prompted.

### 3. [Ansible] - [Setup users and ssh access](./ansible//ssh_user_configuration.yaml)

This step will setup an `edge-user` user on each device, copy a key to each device's authorized_keys store, and enable the user for sudo permission (to run device bundle scripts)

### 4. [Ansible] - [Install IoT Edge Runtime](./ansible/install_edge_agent.yaml)

This step sets up package sources for the Azure IoT Edge runtime and installs the agent on the device.

### 5. [IoT CLI] - Run `az iot edge devices create`

Using the config file, we will create the Edge device identities on the hub, and produce an archived bundle that contains:

- A self-signed root CA (or loaded from properties)
- CA Signed chained device certs
- Preconfigured per-device config.toml
- Preconfigured per-device install script

### 6. [Ansible] - [Install Device Bundles](./ansible/install_device_bundle.yaml)

This step runs on all devices, copies and untars the appropriate `device_id.tgz` package produced by the CLI, and runs the install script on the target device.
This script:

- Copies all certificates to their proper locations
- Copies the device config.toml to the /etc/aziot/ directory
- Updates the device OS certificate store
- Runs `iotedge config apply -C config.toml` to restart the edge agent with the new configuration.

Once this script is run, the edge agent and hub should begin updating on all devices, and you should see them start connecting to the hub in a few moments.
