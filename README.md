# dnf_automatic

This role installs and configures `dnf-automatic` and enables the
`dnf-automatic.timer` service.

The role manages:

- the `dnf-automatic` package
- `/etc/dnf/automatic.conf`
- `/etc/systemd/system/dnf-automatic.timer.d/time.conf`
- an optional fallback reboot drop-in at
  `/etc/systemd/system/dnf-automatic.service.d/autoreboot.conf`

## Requirements

- A host with working DNF repositories
- EL8 or newer for the intended DNF automatic workflow

## Supported Platforms

The role metadata currently lists:

- EL 8
- EL 9
- EL 10
- Fedora (untested)

## Role Variables

## Scheduling

`dnf_automatic_systemd_timer_time`

- Default: `23:45`
- Sets the `OnCalendar` value for `dnf-automatic.timer`

Examples:

- `23:45`
- `Tue *-*-* 22:00:00 UTC`
- `Tue *-*-01..07 22:00:00 UTC`
- `Tue *-*-08..14 22:00:00 UTC`
- `Tue *-*-15..21 22:00:00 UTC`
- `Tue *-*-22..28 22:00:00 UTC`
- `Tue *-*~01..07 22:00:00 UTC`

`dnf_automatic_systemd_timer_random_delay`

- Optional
- If set, configures the systemd timer random delay override

## Reboot Behavior

`dnf_automatic_commands_reboot`

- Optional
- Valid values: `never`, `when-changed`, `when-needed`
- Any other value causes the role to fail during validation
- When supported, this is written to the native `reboot = ...` option in
  `/etc/dnf/automatic.conf`

Native reboot support is enabled when:

- the installed `dnf` package version-release is at least `4.14.0-6`

On older systems, the role falls back to a systemd `ExecStartPost` reboot
action.

`dnf_automatic_systemd_post_update_action`

- Optional
- Custom shell command for the fallback systemd reboot path
- Used only when native reboot support is not available
- If this variable is defined on a system with native reboot support, the role
  fails and requires `dnf_automatic_commands_reboot` instead
- If unset, the role chooses a default fallback command from
  `dnf_automatic_commands_reboot`

Built-in fallback behavior:

- `never`: no fallback reboot action is created
- `when-changed`: reboot if package changes were applied
- `when-needed`: reboot only when `dnf needs-restarting -r` indicates one is
  required

## automatic.conf Variables

These variables map to `/etc/dnf/automatic.conf`.

### `[commands]`

`dnf_automatic_commands_upgrade_type`

- Default: `default`

`dnf_automatic_commands_random_sleep`

- Default: `0`

`dnf_automatic_commands_network_online_timeout`

- Default: `60`

`dnf_automatic_commands_download_updates`

- Default: `yes`

`dnf_automatic_commands_apply_updates`

- Default: `yes`

### `[emitters]`

`dnf_automatic_emitters_system_name`

- Default: `{{ ansible_fqdn }}`

`dnf_automatic_emitters_emit_via`

- Default: `stdio`

### `[email]`

These are optional and are only written if defined.

- `dnf_automatic_email_email_from`
- `dnf_automatic_email_email_to`
- `dnf_automatic_email_email_host`

### `[command]`

These are optional and are only written if defined.

- `dnf_automatic_command_command_format`
- `dnf_automatic_command_stdin_format`

### `[command_email]`

These are optional and are only written if defined.

- `dnf_automatic_command_email_command_format`
- `dnf_automatic_command_email_stdin_format`
- `dnf_automatic_command_email_email_from`
- `dnf_automatic_command_email_email_to`

### `[base]`

`dnf_automatic_base_options`

- Optional dictionary
- Each key/value pair is written into the `[base]` section

Example:

```yaml
dnf_automatic_base_options:
  debuglevel: 1
```

## Example Playbooks

Minimal example:

```yaml
- hosts: all
  roles:
    - linux-system-roles.dnf_automatic
```

Weekly updates with reboot when required:

```yaml
- hosts: all
  vars:
    dnf_automatic_systemd_timer_time: 'Tue *-*-* 21:00:00 UTC'
    dnf_automatic_systemd_timer_random_delay: 0
    dnf_automatic_commands_reboot: when-needed

  roles:
    - linux-system-roles.dnf_automatic
```

Custom fallback reboot command for older systems:

```yaml
- hosts: all
  vars:
    dnf_automatic_commands_reboot: when-needed
    dnf_automatic_systemd_post_update_action: >-
      if ! /usr/bin/dnf needs-restarting -r ; then
      shutdown -r +5 rebooting after applying package updates; fi

  roles:
    - linux-system-roles.dnf_automatic
```

## Dependencies

None.

## License

MIT

## Author

Klaas Demter
