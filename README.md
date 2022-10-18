## IoT nested edge hierarchy demo

### Prerequisites:
- Devices
  - online with static (or relatively static) IP
  - linux, python, edge device prerequisites
- IoT Hub
- Azure CLI (IoT Extension)
  - `az iot edge hierarchy create` will create the device configurations, bundled scripts and certs, and configure everything to the hub.
  - [iot edge config file](./edge_hierarchy_config.yaml)
  - [Example device bundle](./example_device_bundle/)
- Ansible
  - Uses a root controller / node to manage groups of devices through "Playbooks". Playbooks allow us to run the same parameterized commands on multiple devices at once using SSH - which speeds up device configuration steps that are typically done manually.
  - [hosts file example](device_inventory.yaml)
  - The following playbooks are used in this demo:
    - [User and access setup through SSH](./ssh_user_configuration.yaml)
    - [Known hosts config](./add_known_hosts.yaml)
    - [Install IoT Edge runtime](./install_edge_agent.yaml)
    - [Install device bundle produced by CLI](./install_device_bundle.yaml)

## Steps
### 1. [Config] 
Make sure all devices are online and pingable. Since we're installing the Edge Agent and setting up a separate user inside ansible tasks, the only configuration these devices need are a relatively stable IP address.
### 2. [Ansible] - known hosts config
This step will scan all hosts for their public keys, and add them to the controlling node's `known_hosts`. This allows us to ssh between devices without being prompted.
### 3. [Ansible] - setup users and ssh access
This step will setup an `edge-user` user on each device, copy a key to each device's authorized_keys store, and enable the user for sudo permission (to run device bundle scripts)
### 4. [Ansible] - Install IoT Edge Runtime
This step sets up package sources for the Azure IoT Edge runtime and installs the agent on the device.
### 5. [CLI] - Run `az iot edge hierarchy create`
Using the config file, we will create the Edge device identities on the hub, and produce a bundle that contains:
- A self-signed root CA (or loaded from properties)
- CA Signed chained device certs
- Preconfigured per-device config.toml
- Preconfigured per-device install script
### 6. [Ansible] - Install Device Bundle
This step runs on all devices, grabs the appropriate `device_id.tgz` package produced by the CLI, and runs the install script on the target device.