# Ansible Role Syncthing
This is an Ansible role that installs Syncthing on Linux. It uses the tarball from the official website so this role is not dependent on any package manager.  

This role can also optionally configure the devices and folders on the Syncthing instance.  

Tested on Fedora 36, should work on any Linux distribution that the Syncthing tarball supports.  

## Requirements

### Base requirements
None  

## Variables
| Variable | Default | Description |
|----------|---------|-------------|
| `syncthing_version` | `v1.22.0` | Version of Syncthing to install. |
| `syncthing_arch` | `amd64` | Architecture for the Syncthing package. |
| `syncthing_url` | See [defaults/main.yml](./defaults/main.yml) | Base URL for the Syncthing tarball. |
| `syncthing_user` | `{{ ansible_user_id }}` | User to configure Syncthing for. |
| `syncthing_install_user` | `true` | Whether to configure and enable syncthing to run as this user |
| `syncthing_hosts` | `{{ ansible_play_hosts_all }}` | List of hosts (including self) to use as Syncthing peers. |
| `syncthing_configure` | `true` | Whether to configure Syncthing. |
| `syncthing_bootstrap_only` | `false` | Whether to only configure Syncthing once. |
| `syncthing_gui_enabled` | `true` | Whether to enable the Syncthing GUI. |
| `syncthing_gui_address` | `127.0.0.1:8384` | Address to bind the Syncthing GUI to. |
| `syncthing_gui_apikey` | `""` | API key to use for the Syncthing GUI. |
| `syncthing_gui_theme` | `default` | Theme to use for the Syncthing GUI. |

## Configuration

This Ansible role will overwrite your Syncthing configuration, be careful!  

This role by default will also overwrite any manual changes, if you want to just use this role to install and set up Syncthing one time, and do the rest manually, set `syncthing_bootstrap_only` to true.  
Otherwise, every time this role is run, it will overwrite configuration that is not stored in the Ansible variables.  

### Devices
The `syncthing_devices` variable is used to configure remote devices on the Syncthing instance.  *This variable is not required*, if this variable is not set, this role will generate it based on the other hosts in this play.  
If you want to configure devices manually, you can set this variable yourself.

<details> <summary> Static list of devices </summary>

```yaml
syncthing_devices: # Static list of devices
  - name: "My Laptop"
    device_id: "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
    address: dynamic
  - name: "My PC"
    device_id: "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
    address: dynamic
```

</details>

A likely scenario is that you just want to add existing hosts not managed by this Ansible role (like your mobile phone) to this list, in which case you can use the variable `syncthing_external_devices`.  
```yaml
syncthing_external_devices: # Append to auto generated devices
  - name: "My Non-Ansible Managed Phone"
    device_id: "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
    address: dynamic
```

### Folders
The `syncthing_folders` variable is a dict of dicts. The key is the folder ID, and the value is a dict of folder configuration.  

```yaml
syncthing_folders:
  sync:
    label: Sync
    path: "/home/{{ syncthing_user }}/Sync"
```

The key is used as the folder ID in Syncthing, this must be the same on every device you want using this folder. When this key is set on multiple hosts this folder will be shared across these hosts.  

If you defined hosts that are not in the Ansible play (with `syncthing_external_devices` or  `syncthing_devices`), you will need to specify that this folder should be shared with those hosts by giving the ID in the list `manual_shared_with`.  

<details> <summary> Example for external devices </summary>

```yaml
syncthing_folders:
  sync:
    label: Sync
    path: "/home/{{ syncthing_user }}/Sync"
    manual_shared_with: # Append to auto generated shared devices
      - "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
```

</details>

The folder configuration can include `label`, `path`, `type`, `rescanIntervalS`, `fsWatcherEnabled` , `fsWatcherDelayS`, `ignorePerms` and `autoNormalize`. The `label` and `path` keys are required for each folder.  

You can set the default configuration for all folders by setting the `syncthing_folders_default` variable. This will also be used in the UI as the defaults for management outside of Ansible.  

<details> <summary> Defaults </summary>

```yaml
syncthing_folder_defaults: # Will be used if missing from folder
  path: "/home/{{ syncthing_user }}/Sync"
  type: sendreceive
  rescanIntervalS: "3600"
  fsWatcherEnabled: "true"
  fsWatcherDelayS: "10"
  ignorePerms: "false"
  autoNormalize: "true"
```

</details>

### Example

The following is an example of a playbook that uses this role.  

<details> <summary> <code> inventory.ini </code> </summary>

```ini
[syncthing]
desktop
laptop
```

</details>

<details> <summary> <code> group_vars/syncthing.yml </code> </summary>

```yaml
syncthing_external_devices: # Append to auto generated devices
  - name: "My Non-Ansible Managed Phone"
    device_id: "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
    address: dynamic
```

The device entries for `desktop` and `laptop` will be generated automatically based on the hosts in the play.  

</details>

<details> <summary> <code> host_vars/desktop.yml </code> </summary>

```yaml
syncthing_folders:
  all:
    label: Shared with all
    path: /home/{{ syncthing_user }}/Sync
    manual_shared_with: # Append to auto generated shared devices
      - "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
  phone:
    label: Shared with my phone
    path: /home/{{ syncthing_user }}/Phone
    manual_shared_with: # Append to auto generated shared devices
      - "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
```

</details>

<details> <summary> <code> host_vars/laptop.yml </code> </summary>

```yaml
syncthing_folders:
  all:
    label: Shared with all
    path: /home/{{ syncthing_user }}/Sync
    manual_shared_with: # Append to auto generated shared devices
      - "AAAAAAA-BBBBBBB-CCCCCCC-DDDDDDD-EEEEEEE-FFFFFFF-GGGGGGG-HHHHHHH"
```

Since the `phone` key doesn't appear here, it won't be shared with this device.  

</details>

<details> <summary> <code> playbook.yml </code> </summary>

```yaml
- hosts: syncthing
  roles:
    - ansible_role_syncthing
```

</details>
