# ZFS Encrypted Datasets Playbook

An Ansible playbook that automates the creation of encrypted ZFS datasets across multiple hosts, with key backup to local storage and optional password manager integration (1Password or Keeper).

## What It Does

1. Installs `zfsutils-linux` on target hosts
2. Generates a 32-byte random encryption key (if one doesn't already exist)
3. Creates encrypted ZFS datasets using native ZFS encryption
4. Fetches the encryption key back to the control node for local backup
5. Optionally uploads the key to 1Password and/or Keeper vault

## Prerequisites

- Ansible installed on the control node
- Target hosts running a Debian/Ubuntu-based OS with ZFS support
- A ZFS pool already created on each target host
- **For 1Password integration:** `op` CLI installed, `OP_SERVICE_ACCOUNT_TOKEN` environment variable set
- **For Keeper integration:** `keeper` CLI installed, config file at `~/.keeper/config.json` (or set `KEEPER_CONFIG_PATH`)

## Quick Start

1. Copy the sample inventory and edit it with your hosts:

   ```bash
   cp inventory.yml.sample inventory.yml
   ```

2. Edit `inventory.yml` with your host details, zpool names, and desired datasets:

   ```yaml
   all:
     hosts:
       myserver:
         ansible_host: 192.168.1.10
         ansible_user: admin
         zpool_name: tank
         zfs_datasets:
           - edata
           - backups
   ```

3. Review and adjust variables in `group_vars/all.yml` as needed.

4. Run the playbook:

   ```bash
   ansible-playbook -i inventory.yml zfs_encrypted_datasets.yml
   ```

## Configuration

Default variables are defined in `group_vars/all.yml`:

| Variable | Default | Description |
|---|---|---|
| `zfs_key_path` | `/etc/zfs_drivekey.bin` | Path to the encryption key on target hosts |
| `local_key_backup_dir` | `./keys` | Local directory for key backups |
| `vault_integration` | `"1password"` | Vault provider: `"none"`, `"1password"`, `"keeper"`, or `"both"` |
| `vault_key_title_template` | `"ZFS-Encryption-Key-{{ inventory_hostname }}"` | Title template for vault entries (no spaces; use hyphens) |
| `vault_fail_on_upload_error` | `false` | Whether to fail the playbook if vault upload fails |
| `vault_update_existing` | `false` | When false, never overwrite an existing vault record; only create when missing |

### Per-Host Variables (set in inventory)

| Variable | Description |
|---|---|
| `zpool_name` | Name of the existing ZFS pool on the host |
| `zfs_datasets` | List of dataset names to create under the pool |

### 1Password Settings

| Variable | Default | Description |
|---|---|---|
| `onepassword_vault` | `"Infrastructure"` | 1Password vault name |
| `onepassword_item_type` | `"document"` | Item type for the key |
| `onepassword_tags` | `["zfs", "encryption-key"]` | Tags applied to the item |

### Keeper Settings

| Variable | Default | Description |
|---|---|---|
| `keeper_folder_path` | `"ZFS Keys"` | Keeper folder for key storage |
| `keeper_config_path` | `"~/.keeper/config.json"` | Path to Keeper CLI config |

## Project Structure

```
.
├── group_vars/
│   └── all.yml                      # Default variables
├── keys/                            # Local key backups (git-ignored)
├── tasks/
│   ├── vault_upload_1password.yml   # 1Password upload logic
│   └── vault_upload_keeper.yml      # Keeper upload logic
├── inventory.yml.sample             # Sample inventory file
└── zfs_encrypted_datasets.yml       # Main playbook
```

## Security Notes

- Encryption keys are 32 bytes of random data from `/dev/urandom`
- Key files on target hosts are set to `0600` (root-only read/write)
- The `keys/` directory and `inventory.yml` are git-ignored to prevent accidental commits of sensitive data
- Back up your encryption keys securely -- losing them means losing access to your encrypted datasets
