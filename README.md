# labbot-deploy

This repository containers an Ansible playbook & role to deploy LabBot and related monitoring infrastructure (Grafana, Prometheus, cAdvisor, and node_exporter).

While it may work with others, this role is designed to be run against a Debian-based Linux distribution.

## Usage

1. Enter the [`host_vars`](host_vars/) directory
2. Create a copy of [`labbot_host.example.yml`](host_vars/labbot_host.example.yml) named `labbot_host.yml`.
3. Set all relevant values in `labbot_host.yml`.
4. Run the following commands:

```sh
# Create a Python virtual environment (venv)
python3 -m venv venv
# Activate the venv
source venv/bin/activate
# Install Ansible (in the venv)
pip install ansible
# Install required Ansible roles/collections
ansible-galaxy install -r requirements.yml
# Run the playbook to initiate the deployment
ansible-playbook playbook.yml
```

> [!TIP]
> The `labbot` Ansible role contains several optional components that are not included in the `labbot_host.yml` example file for the sake of simplicity; see [`roles/labbot/defaults/main.yml`](roles/labbot/defaults/main.yml) for advanced values such as backup options and more.

## Bot monitoring setup

If this is a fresh deployment, LabBot will not automatically serve Prometheus metrics. This is because the `prometheus_exporter` cog will not yet be installed.

If this is the case, you will see a task named _"Set LabBot prometheus_exporter cog scrape interval"_ fail during the Ansible deployment.

Follow the steps below and then re-run the playbook. It should no longer fail.

Enter a chat with the bot and run the following (where `[p]` is the configured prefix):

```
[p]repo add homelab https://github.com/rHomelab/LabBot-Cogs
[p]cog install homelab prometheus_exporter
[p]load prometheus_exporter
```

---

## Disclaimer

Most common settings are variablised, however I only prepared this for shared use/more generic environments after extracting it from my own infrastructure definitions, so there may be some things that are still 'hardcoded', otherwise inoptimal for a shared configuration, or not fully tested.
