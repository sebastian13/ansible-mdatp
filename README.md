# Ansible Windows Defender ATP

This role deploys Microsoft Defender for Endpoint. It is based on the official instruction [Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide)

## Requirements

1. Download the onboarding package from the Microsoft Defender portal, see [official instructions](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide#download-the-onboarding-package).
2. Put the zip file, as it is, to `.\files`.

## Example Playbook

```yaml
- hosts: all
  become: true

  tasks:
    - name: Install & Onboard
      import_role:
        name: windows-defender-atp
```

### Run the playbook

```bash
ansible-playbook windows-defender-atp.yml --limit XYZ.example01
```

### Validate the Onboarding

```bash
ansible -m shell -a 'mdatp connectivity test' all
```

```bash
ansible -m shell -a 'mdatp health' all
```
