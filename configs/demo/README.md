Demo steps:

1. Flash Devices

2. Create device bundles (and hub config)

Note: Setup Azure CLI Extension in path
`  cd ./configs/demo
    az iot edge devices create -n $hub --cfg ./demo_config.yml -c -v --out ../../device_bundles`

3.  Boot Devices, revert known_hosts
    `
    ssh-keygen -f "~/.ssh/known_hosts" -R "pi-1"
    ssh-keygen -f "~/.ssh/known_hosts" -R "pi-2"
    ssh-keygen -f "~/.ssh/known_hosts" -R "pi-3"`
    `
4.  Ansible steps for SSH connections
    `
    cd ./ansible

        # add devices hostnames to known_hosts file
        ansible-playbook -i local_device_inventory.yaml add_known_hosts.yaml

        # setup new SSH user and add to sudoers
        ansible-playbook -i local_device_inventory.yaml ssh_user_configuration.yaml

`

5.  Ansible steps for IoT Edge
    ` # Install Azure IoT Edge
    ansible-playbook -i local_device_inventory.yaml install_edge_agent.yaml

        # Install CLI configured Device Bundle
        ansible-playbook -i local_device_inventory.yaml install_device_bundle.yaml

`

6.  Verify results
    Note: May take 5+ minutes for devices to configure, authenticate, and start to send telemetry
    `az iot hub monitor-events -n $hub --mc 10`
