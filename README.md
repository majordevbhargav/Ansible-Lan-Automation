# LAN Automation with Ansible

An Ansible-based network automation project for collecting operational information from Cisco and HP Aruba switches using inventories, playbooks, collections, and Ansible Vault.

## Why This Project

This project is the next step in my network automation learning path:

**CLI → Python → Ansible → Infrastructure as Code**

The focus is on repeatability, structured workflows, and safer network operations.

## Supported Collections

- `cisco.ios`
- `arubanetworks.aos_switch`
- `ansible.netcommon`

## Structure

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

```bash
pip install ansible-core
ansible-galaxy collection install -r requirements.yml
```

Store credentials in Ansible Vault:

```bash
ansible-vault encrypt group_vars/all/vault.yml
```

## Run

```bash
ansible-playbook -i inventory.yml playbook.yml -e @commands.yml --ask-vault-pass
```

Limit execution when testing:

```bash
ansible-playbook -i inventory.yml playbook.yml -e @commands.yml --ask-vault-pass --limit core-switch-01
```

## What I Am Learning

- Infrastructure as Code
- Network inventories
- Declarative automation
- Multi-vendor automation
- Credential management
- Repeatable reporting

## Security

Never commit plaintext credentials. Use Ansible Vault or another approved secrets-management system.

Run automation only against authorized devices.

## Future Direction

- Configuration deployment
- Compliance checks
- Configuration diffing
- Validation workflows
- Scheduled reporting
- ISE and posture integration

## Author

**Dev Bhargav**

[GitHub](https://github.com/majordevbhargav) · [LinkedIn](https://www.linkedin.com/in/devbhargav100)
