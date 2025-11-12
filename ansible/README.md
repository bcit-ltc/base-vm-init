# Initialization

This procedure is to add the `ansible` user to a brand new VM.

New VM's do not have the `ansible` user to run scripts. The first step is to create this user so that the rest of the scripts in this repo will run.

## Requirements

- must have your personal, public SSH key added to the `known_hosts` file on new VM by ITS
- must have your personal, private SSH key stored locally in the default location (eg. `~/.ssh/id_ed25519`)
- must have access to the ansible `become` password in [Vault](https://vault.ltc.bcit.ca:8200)
  - Vault CLI

## Procedure

1. Install Ansible on your machine locally

    MacOS: `brew install ansible`

    Windows: `choclaty install ansible`

    Other: `nix-shell -p ansible`

1. Add required modules to your local Ansible installation

    `ansible-galaxy install weareinteractive.users mrlesmithjr.manage-lvm kwoodson.yedit`

1. Export your login name as an environment variable

    `export USERNAME={your_login_name}`

1. Create a new Ansible inventory file with the list of VMs (or retrieve the inventory file from Vault)

    `echo -e "vm1.domain.com\nvm2.domain.com" > ~/.ansible/hosts-1st.json`

1. Retrieve the SSH keys from the list of VMs

    `ssh-keyscan -f ~/.ansible/hosts-1st.json >> ~/.ssh/known_hosts`

1. Temporarily override the default configuration for first run

    `export ANSIBLE_CONFIG=./ansible-1st.cfg`

1. Confirm you can ping the machines in your inventory

    `ansible-playbook 00_ping.yaml -i ~/.ansible/hosts-1st.json`

1. Unset the temporary config

    `unset ANSIBLE_CONFIG`

1. List the current inventory and make note of the machines you want to configure

    `ansible-inventory --graph`

1. Set Vault's endpoint

    `export VAULT_ADDR={vaultAddress}`

1. Retrieve the ansible user's `become` password from Vault

    `vault kv get -mount="ltc-infrastructure" "ansible/become-password"`

1. Run the `add_user` playbook

    - Be sure the ssh-agent is running for passwordless login. See [GitHub SSH Agent docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent).

    `ansible-playbook 01_add_ansible_user.yaml -u $USERNAME --private-key ~/.ssh/id_ed25519 -l {clusterGroup}`

When the `ansible` user has been added the infrastructure is ready to be created.
