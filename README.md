<!-- SPDX-License-Identifier: MPL-2.0 -->

# Base VM Initialization

Scripts, configs, and Ansible modules for newly-provisioned virtual machines.

Follow these steps to prepare virtual machines to be Kubernetes cluster nodes.

## Requirements

* Ansible
* Vault

## Getting Started

1. Install Ansible and [Vault](https://www.vaultproject.io/downloads).

    ```bash
    brew install ansible vault
    ```

1. Install required Ansible Galaxy modules

    ```bash
    ansible-galaxy install weareinteractive.users kwoodson.yedit mrlesmithjr.manage-lvm
    ```

1. Login to Vault and set the VAULT_TOKEN environment variable

    ```bash
    vault login -method=oidc username={yourBcitEmail}

    export VAULT_TOKEN={YourToken}
    ```

1. Retrieve and store a local copy of the current inventory

    ```bash
    mkdir ~/.ansible && \
    curl \
      --header "X-Vault-Token: $VAULT_TOKEN" \
      $VAULT_ADDR/v1/ltc-infrastructure/data/inventory \
      > ~/.ansible/hosts.json
    ```

1. Retrieve and store a local copy of the `ansible` user credentials

    ```bash
    curl --header "X-Vault-Token: $VAULT_TOKEN" \
        $VAULT_ADDR/v1/ltc-infrastructure/data/ansible | jq -r '.data.data' > out.json && \
        cat out.json | jq -r '.id_rsa' > ~/.ansible/id_rsa && \
        cat out.json | jq -r '.["id_rsa.pub"]' > ~/.ansible/id_rsa.pub && \
        ANSIBLE_BECOME_PASSWORD=$(cat out.json | jq -r '.ansible_become_password') && \
        rm out.json && \
        echo -e "\n\n--- ANSIBLE_BECOME_PASSWORD: $ANSIBLE_BECOME_PASSWORD\n\n"
    ```

1. Confirm that you can list an inventory group

    ```bash
    ansible-inventory -i ~/.ansible/hosts.json --graph production
    ```

1. Confirm you can ping the inventory

    ```bash
    ansible-playbook -i ~/.ansible/hosts.json -l prod_01 00_ping.yaml
    ```

See the `ansible` and `kubernetes` folders for additional procedures.

## License

This Source Code Form is subject to the terms of the Mozilla Public License, v2.0. If a copy of the MPL was not distributed with this file, You can obtain one at <https://mozilla.org/MPL/2.0/>.

## About

Developed in 🇨🇦 Canada by the [Learning and Teaching Centre](https://www.bcit.ca/learning-teaching-centre/) at [BCIT](https://www.bcit.ca/).
