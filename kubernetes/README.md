# Kubernetes Nodes: configure storage and RKE2

1. Add iscsi, nfs-utils

    ```bash
    ansible-playbook storage/install_storage_packages.yaml -l phase5\:\&staging
    ```

1. Configure LVM device

    ```bash
    ansible-playbook storage/install_storage_packages.yaml -l phase5\:\&staging

    ansible-playbook storage/add_lvm_device.yaml -l phase5\:\&staging
    ```

1. Install RKE2 using "ansible-rke2" module

    > See `rke2-ansible/README.md` for instructions

1. Retrieve KUBECONFIG file and merge it with your existing `~/.kube/config` to be able to run `kubectl` commands

    1. Login to [Rancher](https://rancher2.ltc.bcit.ca).
    1. Retrieve KUBECONFIG

        ![copy kubeconfig](./kubeconfig.png)

    1. Merge into your existing `~/.kube/config` file

        ```bash
        cp ~/.kube/config ~/.kube/config.bak && KUBECONFIG=~/.kube/config:/path/to/new/config kubectl config view --flatten > /tmp/config && mv /tmp/config ~/.kube/config
        ```
