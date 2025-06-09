# Ansible Microsoft Defender ATP

This role deploys Microsoft Defender for Endpoint. It is based on the official instruction [Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide)

## Requirements

1. Clone this repo to roles/mdatp
2. Download the onboarding package from the Microsoft Defender portal, see [official instructions](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/linux-install-with-ansible?view=o365-worldwide#download-the-onboarding-package).
3. Put the zip file, as it is, into `.\files`.

## Example Playbook

```yaml
- hosts: all
  become: true

  roles:
    -role: mdatp
```

### Run the playbook

```bash
ansible-playbook mdatp.yml --limit XYZ.example01
```

### Validate the Onboarding

```bash
ansible -m shell -a 'mdatp connectivity test' all
```

```bash
ansible -m shell -a 'mdatp health' all
```
