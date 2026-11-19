# Ansible Microsoft Defender for Endpoint

This role installs and configures Microsoft Defender for Endpoint on Linux. It is based on the official Microsoft guidance for [Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/defender-endpoint/linux-install-with-ansible).

## Requirements

1. Download the onboarding package from the Microsoft Defender portal as described in the official documentation.
2. Provide the onboarding information, either
   - **(recommended)** via inventory variable `mdatp_onboard_json` (see [Onboarding via group_vars](#onboarding-via-group_vars)), or
   - by putting the zip file unchanged into the `files/` directory of this role or your playbook.
3. Ensure the target hosts are supported Debian or Ubuntu Linux systems.

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `mdatp_onboard_json` | `~` | Content of `mdatp_onboard.json` (YAML mapping or raw JSON string). Takes precedence over the zip. |
| `mdatp_onboarding_package` | `GatewayWindowsDefenderATPOnboardingPackage.zip` | Fallback onboarding zip, used when `mdatp_onboard_json` is not set. |
| `mdatp_channel` | `prod` | Microsoft package channel (`prod`, `insiders-fast`, `insiders-slow`). The deprecated variable `channel` is still honored. |
| `mdatp_package_state` | `latest` | `latest` keeps the agent up to date on every run, `present` only installs it. |
| `mdatp_managed_config` | `{}` | Additional settings for `mdatp_managed.json`, merged on top of all generated settings. |

## Antivirus Settings

All settings are written to `/etc/opt/microsoft/mdatp/managed/mdatp_managed.json` ([preferences](https://learn.microsoft.com/en-us/defender-endpoint/linux-preferences)). Managed settings override local `mdatp config` changes. Policies from the Defender portal (security settings management) still take precedence.

| Variable | Default | Description |
| --- | --- | --- |
| `mdatp_enforcement_level` | `real_time` | `real_time` (recommended), `on_demand`, `passive`, `audit`. Without this setting, mdatp runs in `passive` mode. |
| `mdatp_behavior_monitoring` | `true` | Behavior monitoring, only effective with `real_time`. |
| `mdatp_scan_max_threads` | `2` | Threads for on-demand scans (1-64). |
| `mdatp_threat_pua` | `block` | Potentially unwanted applications: `audit`, `block`, `off`. |
| `mdatp_threat_archive_bomb` | `audit` | Archive bombs: `audit`, `block`, `off`. |
| `mdatp_cloud_enabled` | `true` | Cloud-delivered protection. |
| `mdatp_automatic_definition_update` | `true` | Automatic security intelligence updates. |
| `mdatp_sample_submission` | `none` | `none`, `safe`, `all`. |
| `mdatp_cloud_block_level` | `normal` | `normal`, `moderate`, `high`, `high_plus`, `zero_tolerance`. |
| `mdatp_proxy` | `''` | Proxy URL, e.g. `http://proxy.example.com:3128/`. |
| `mdatp_device_tag` | `''` | Device tag (`GROUP`), e.g. the customer name, for filtering and device groups in the portal. |
| `mdatp_exclusions` | `[]` | Exclusions in Microsoft's JSON format ([exclusions](https://learn.microsoft.com/en-us/defender-endpoint/linux-exclusions)). |
| `mdatp_exclusions_merge_policy` | `merge` | `merge` or `admin_only` (only managed exclusions apply). |

Example `group_vars` for a customer:

```yaml
mdatp_device_tag: customer-a
mdatp_exclusions:
  - { "$type": "excludedPath", "isDirectory": true, "path": "/var/lib/postgresql/", "scopes": ["epp"] }
```

Check the effective settings (managed values are marked with `[managed]`):

```bash
ansible -m shell -a 'mdatp health' all
```

## Scheduled Scans

Following Microsoft's recommendation ([configure scans](https://learn.microsoft.com/en-us/defender-endpoint/configure-anti-virus-scans-linux), [schedule scans](https://learn.microsoft.com/en-us/defender-endpoint/schedule-antivirus-scans-linux)), the role configures by default a **daily quick scan** (the weekly full scan is disabled by default, see `mdatp_scan_weekly_day`) using the built-in scheduler in `/etc/opt/microsoft/mdatp/managed/mdatp_managed.json`.

The built-in scheduler requires mdatp `101.26032.0000` or later. On older agents the role falls back to cron automatically and prints a warning.

| Variable | Default | Description |
| --- | --- | --- |
| `mdatp_scan_method` | `builtin` | `builtin`, `cron` or `none`. |
| `mdatp_scan_daily` | `true` | Enable the daily quick scan. |
| `mdatp_scan_daily_time_of_day` | `120` | Minutes after midnight (local time), `120` = 02:00. |
| `mdatp_scan_daily_interval` | `0` | Additional quick scan every N hours, `0` = disabled (builtin only). |
| `mdatp_scan_weekly_day` | `0` | `0` = disabled, `1`-`7` = Sunday-Saturday, `8` = every day. |
| `mdatp_scan_weekly_time_of_day` | `180` | Minutes after midnight, `180` = 03:00. |
| `mdatp_scan_weekly_type` | `full` | `quick` or `full`. |
| `mdatp_scan_randomize_hours` | `3` | Randomize the start within N hours (0-23). |
| `mdatp_scan_low_priority` | `true` | Run scans with reduced CPU priority (builtin only). |
| `mdatp_scan_when_idle` | `false` | Delay scans until the system is idle (builtin only). |
| `mdatp_scan_check_for_definitions_update` | `true` | Update definitions before scanning (builtin only). |
| `mdatp_scan_ignore_exclusions` | `false` | Ignore antivirus exclusions during scheduled scans (builtin only). |
| `mdatp_scan_custom_paths` | `[]` | Paths for an additional weekly custom scan (always cron), e.g. `['/docker']`. |
| `mdatp_scan_custom_day` | `1` | `1`-`7` = Sunday-Saturday, `8` = every day. |
| `mdatp_scan_custom_time_of_day` | `300` | Minutes after midnight, `300` = 05:00. |

Example `group_vars` for a customer with large file servers:

```yaml
mdatp_scan_weekly_type: quick
mdatp_scan_custom_paths:
  - /docker
```

Check the configuration and scan history:

```bash
ansible -m shell -a 'mdatp health --details scheduled_scan' all
ansible -m shell -a 'mdatp scan list' all
```

## Onboarding via group_vars

This allows multiple tenants/customers in one inventory. Extract `mdatp_onboard.json` from the onboarding zip and put its content into the group_vars of the respective customer, e.g. `inventory/customer-a/group_vars/all/mdatp.yml`:

```yaml
---
mdatp_onboard_json: |
  {
    "onboardingInfo": "<paste the unchanged value from mdatp_onboard.json here>"
  }
```

The simplest way is to paste the complete file content as a literal block (`|`), as shown above. The role parses the JSON string and writes it back to `/etc/opt/microsoft/mdatp/mdatp_onboard.json`.

Keep `onboardingInfo` byte-identical, it is signed. Treat it as a secret: anyone with it can onboard devices
into your tenant. Encrypt it with `ansible-vault` and never commit the onboarding zip or `mdatp_onboard.json`
(`files/.gitignore` excludes them).

The file is only written if it does not exist yet, so already onboarded devices are not touched.

## Upgrading from earlier versions

- The former weekly custom scan of `/docker` and `/var/lib/docker` is no longer configured by default and
  its cronjob is removed. To keep it:
  ```yaml
  mdatp_scan_custom_paths:
    - /docker
    - /var/lib/docker
  ```
- `mdatp_package_state` defaults to `latest`: the agent is upgraded on every run. Set `present` to keep the old behavior.
- `channel` is renamed to `mdatp_channel`; the old name still works.
- The PUA policy is no longer set via `mdatp threat policy set`, but in `mdatp_managed.json` (`mdatp_threat_pua`).

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

- Either `mdatp_onboard_json` must be set or the onboarding package must be present in a `files/` directory before the role runs.
- This role assumes you already have the required Microsoft Defender licensing and enrollment configuration in place.
