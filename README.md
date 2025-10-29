# labbot-deploy

This repository containers an Ansible playbook & role to deploy LabBot and related monitoring infrastructure (Grafana, Prometheus, cAdvisor, and node_exporter).

While it may work with others, this role is designed to be run against a Debian-based Linux distribution.

## Usage

1. Enter the [`host_vars`](host_vars/) directory and create a copy of [`labbot_host.example.yml`](host_vars/labbot_host.example.yml) named `labbot_host.yml`.
2. Set all relevant values in `labbot_host.yml`. See [Variables](#variables) for more info.
3. Run the following commands:

```sh
# Create a Python virtual environment (venv)
python3 -m venv .venv
# Activate the venv
source .venv/bin/activate
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

## Variables

Below is a description of the variables, both required and optional, for this deployment. Variables marked as **required** must be defined in `host_vars` as described in [Usage](#usage).

### Deployment Config

* `labbot_app_base_dir` (optional): The parent directory for container configuration paths. Default: `/opt`.
* `labbot_container_name_prefix` (optional): Prefix for all container names. Default: `labbot_`.

### Bot Config

* `labbot_discord_token` (**required**): The Discord bot token for LabBot.

### Monitoring/Web Config

* `labbot_grafana_domain` (**required**): Domain name for the Grafana web interface.
* `labbot_grafana_username` (optional): Username for Grafana access. Default: `labbot_admin`.
* `labbot_grafana_password` (**required**): Password for Grafana access.
* `labbot_grafana_container_user` (optional): User ID for Grafana container. Default: current user ID.
* `labbot_prometheus_container_user` (optional): User ID for Prometheus container. Default: current user ID.
* `labbot_prometheus_users` (**required**): List of users for Prometheus authentication. Each user requires:
  * `username` (**required**): Username for Prometheus.
  * `password` (**required**): Plaintext password for Prometheus.
  * `password_bcrypt` (**required**): Hashed version of the password. This can be generated using: `htpasswd -nBC 10 '<password>' | tr -d ':\n'`.
* `labbot_prometheus_scrape_interval` (optional): Prometheus metrics scraping interval in seconds. Default: `10`.
* `labbot_prometheus_retention_size` (optional): Prometheus [retention size](https://prometheus.io/docs/prometheus/latest/storage/#operational-aspects). Default: `0` (disabled).
* `labbot_prometheus_open_port` (optional): Whether to bind Prometheus UI port on the host for debugging. Default: `false`.
* `labbot_cadvisor_open_port` (optional): Whether to bind cAdvisor UI port on the host for debugging. Default: `false`.
* `labbot_certbot_letsencrypt_email` (**required**): Email address for LetsEncrypt certificate registration.
* `labbot_certbot_dry_run` (optional): Whether to run Certbot (for SSL certificates) in dry-run mode for testing/debugging. Default: `false`.

### Backup Config (Optional)

> [!NOTE]
> The backup variables below are only needed if backups are enabled (`labbot_enable_bot_backup: true`).

* `labbot_enable_bot_backup` (optional): Enable or disable bot backups. Default: `false`.

* `labbot_backup_report_webhook` (required if backup enabled): Discord webhook URL for backup report notifications.
* `labbot_backup_report_mention_user_id` (required if backup enabled): User ID to mention in case of a backup failure.
* `labbot_rclone_configs` (required if backup enabled): List of rclone remotes for backup uploads.  
  For more information, see the [stefangweichinger.ansible-rclone](https://github.com/stefangweichinger/ansible-rclone/blob/main/README.md#rclone_configs-) documentation.
* `labbot_rclone_backup_remotes` (optional): List of strings, where each string is the name of a defined rclone remote (see `labbot_rclone_configs`), to which backups should be uploaded.  
  This is useful if you use virtual remotes in rclone, such as the [crypt](https://rclone.org/crypt/) remote, as you would likely not want the backup to be uploaded to both your origin and `crypt` remote, for example.  
  If unset or an empty list (default), backups will be uploaded to ALL remotes.
* `labbot_rclone_config_location` (optional): Path to the rclone configuration file. Default: `/root/.config/rclone/labbot_backup_remotes.conf`.

### Misc. Config (Optional)
* `labbot_ssh_keys` (optional): List of SSH keys to install on the host.

---

## Disclaimer

Most common settings are variablised, however I only prepared this for shared use/more generic environments after extracting it from my own infrastructure definitions, so there may be some things that are still 'hardcoded', otherwise inoptimal for a shared configuration, or not fully tested.
