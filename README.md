# Ansible Microsoft Defender for Endpoint

This role installs and configures Microsoft Defender for Endpoint on Linux. It is based on the official Microsoft guidance for [Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/defender-endpoint/linux-install-with-ansible).

## Requirements

1. Download the onboarding package from the Microsoft Defender portal as described in the official documentation.
2. Put the zip file unchanged into the `files/` directory of this role.
3. Ensure the target hosts are supported Debian or Ubuntu Linux systems.

## Installation

Install directly from Ansible Galaxy:

```bash
ansible-galaxy role install sebastian13.mdatp -p roles
```

Or clone it locally:

```bash
git clone https://github.com/sebastian13/ansible-mdatp.git roles/mdatp
```

## Example Playbook

```yaml
---
- name: Install Microsoft Defender for Endpoint
  hosts: all
  become: true

  roles:
    - role: mdatp
```

### Run the playbook

```bash
ansible-playbook mdatp.yml --limit XYZ.example01
```

### Validate the onboarding

```bash
ansible -m shell -a 'mdatp connectivity test' all
```

```bash
ansible -m shell -a 'mdatp health' all
```

## Notes

- The onboarding package must be present in the role’s `files/` directory before the role runs.
- This role assumes you already have the required Microsoft Defender licensing and enrollment configuration in place.
