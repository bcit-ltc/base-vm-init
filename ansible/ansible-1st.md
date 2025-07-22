# Initialization: this procedure is to add the `ansible` user to a brand new VM

Images newly provisioned by ITS do not have the `ansible` user to run scripts. The first step is to create this user so that the rest of the scripts in this repo will run!

## Requirements

* must have your personal, public SSH key added to the `known_hosts` file on new VM by ITS
* must have your personal, private SSH key stored locally in the default location (eg. `~/.ssh/id_ed25519`)

## Procedure

1. Install Ansible on your machine locally

    `brew install ansible`

2. Add required modules to your local Ansible installation

    `ansible-galaxy install weareinteractive.users mrlesmithjr.manage-lvm kwoodson.yedit`

3. Export your login name as an environment variable

    `export USERNAME={your_login_name}`

4. Create a new Ansible inventory file with the list of VMs (or retrieve the inventory file from Vault)

    `echo -e "prod-manager-01.ltc.bcit.ca\nprod-manager-02.ltc.bcit.ca" > ~/.ansible/hosts.json`

5. Retrieve the SSH keys from the list of VMs

    `ssh-keyscan -f ~/.ansible/hosts.json >> ~/.ssh/known_hosts`

6. Confirm you can ping the machines in your inventory

    `ansible-playbook 00_ping.yaml ~/.ansible/hosts.json`

7. Run the `add_user` playbook (be sure the ssh-agent is running for passwordless login). See [GitHub SSH Agent docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent).

    `ansible-playbook 00_add_ansible_user.yaml -u $USERNAME --private-key ~/.ssh/id_ed25519`

When the `ansible` user has been added the infrastructure is ready to be created.
