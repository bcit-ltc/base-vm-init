# Docker daemon configuration

This project runs two Ansible playbooks to configure a secure remote Docker daemon:

- `install-docker.yaml`: installs docker on a remote node and configures the daemon for secure connections
- `install-vault-agent.yaml`: installs Vault as an agent running as a daemon process to periodically retrieve certificates to secure the Docker daemon.

## Prerequisites

- Ansible
- Docker
- Vault
- Client certificates and certificate authority (see [vault configuration > approle-commands.md](https://issues.ltc.bcit.ca/ltc-infrastructure/vault-configuration/-/blob/main/approle/approle-commands.md))

See the front matter of the playbooks for additional info.

## Connecting to the remote Docker Daemon

After running the playbooks, you need to generate "client certificates" that allow you to securely connect to the Docker daemon:

```bash
vault write -format=json pki-ltc/sica/issue/pki-ltc-cert common_name="{serverName}" > certs.json
```

Then, store the certificates in a location where Docker looks for them:

```bash
mkdir ~/.docker && \
cat certs.json | jq -r .data.certificate > ~/.docker/cert.pem && \
cat certs.json | jq -r .data.private_key > ~/.docker/key.pem && \
cat certs.json | jq -r .data.issuing_ca > ~/.docker/ca.pem
```

Lastly, configure your local Docker client to connect securely:

```bash
export DOCKER_HOST={serverName}
export DOCKER_TLS_VERIFY=1
```

Now, you are ready to run commands on the remote Docker daemon:

```bash
docker info
```

## Confirming Vault Agent is periodically retrieving certificates

Vault is installed as a long-running agent daemon process that retrieves certificates when the existing certificates are 2/3 of the way through their time-to-live. The agent uses its token to retrieve certificates, write them to a location known to the Docker daemon, and then restarts Docker.

To ensure the process is running as intended, check the system status:

```bash
systemctl status vault
```

If the agent has failed to retrieve certificates, re-run the `run-agent.yaml` playbook to re-initialize the process.
