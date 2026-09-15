# LAN Automation with Ansible

An Ansible-based network automation workflow for collecting CLI output from Cisco and HP Aruba switches without maintaining a custom Python command runner.

## Overview

The playbook uses Ansible network collections and an encrypted credential workflow to execute commands across an inventory and save timestamped reports.

## Supported Collections

- `cisco.ios`
- `arubanetworks.aos_switch`
- `ansible.netcommon`

## Project Structure

```text
inventory.yml
commands.yml
playbook.yml
requirements.yml
group_vars/
└── all/
    └── vault.yml
output/
```

## Setup

Install Ansible and the required collections:

```bash
pip install ansible-core
ansible-galaxy collection install -r requirements.yml
```

Store network credentials in the Ansible Vault file and encrypt it:

```bash
ansible-vault encrypt group_vars/all/vault.yml
```

Never commit plaintext credentials.

## Run

```bash
ansible-playbook -i inventory.yml playbook.yml -e @commands.yml --ask-vault-pass
```

Limit execution to one device:

```bash
ansible-playbook -i inventory.yml playbook.yml -e @commands.yml --ask-vault-pass --limit core-switch-01
```

## Output

The playbook produces timestamped per-device command output and a combined CSV report for analysis in tools such as Excel.

## Why Ansible

The project demonstrates how the same network automation problem can be expressed declaratively through inventories, playbooks, collections, and Ansible Vault instead of a custom Python application.

## Future Direction

- Configuration deployment
- Privileged-mode support
- Scheduled compliance checks
- Configuration diffing
- Device validation and reporting
- Integration with ISE and posture workflows

## Security

Use Ansible Vault or another approved secrets-management system. Run automation only against authorized network devices and validate configuration changes before production deployment.

## Author

**Dev Bhargav**

- GitHub: https://github.com/majordevbhargav
- LinkedIn: https://www.linkedin.com/in/devbhargav100
