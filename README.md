# Synology Backup Status - CheckMK Extension Package

A CheckMK MKP plugin for monitoring Synology Hyper Backup and Active Backup for Business via the Synology DSM API.

## Installation

### Via GUI (Recommended)

1. In CheckMK: **Setup → Maintenance → Extension packages**
2. Click **Upload package**
3. Select the file `synology_backup-1.0.7.mkp`
4. Activate the package using the plug icon
5. **Activate changes**

### Via Command Line

```bash
# As site user
OMD[mysite]:~$ mkp add /tmp/synology_backup-1.0.7.mkp
OMD[mysite]:~$ mkp enable synology_backup

# Restart CheckMK
OMD[mysite]:~$ omd restart
```

## Configuration

### 1. Create Synology DSM User

Create a user in DSM with the following permissions:

1. **DSM → Control Panel → User & Group → Create**
2. Username: e.g., `checkmk_monitor`
3. Normal user group (users)
4. **Delegated Permissions:**
   - Hyper Backup: "Manage Hyper Backup"
   - Active Backup: "Manage Active Backup for Business" (optional)

**Note:** If you encounter Error 402 (Permission denied), the user may need to be added to the administrators group, or you need to ensure DSM application access is set to "Allow" for this user.

### 2. Configure Special Agent in WATO

1. **Setup → Agents → Other integrations → Synology Backup Status**
2. Create a new rule:
   - **Username**: DSM username
   - **Password**: DSM password
   - **Port**: 5001 (HTTPS) or 5000 (HTTP)
   - **Use HTTPS**: Enable (recommended) or disable for HTTP
   - **Verify SSL**: Disable for self-signed certificates
   - **Monitor Hyper Backup**: Enable
   - **Monitor Active Backup**: Enable
   - **Backup Interval**: Default interval in hours (used for threshold calculation)

3. Set host conditions (e.g., folder or specific hosts)
4. **Activate changes**

### 3. Discover Services

1. Navigate to the host
2. Run **Service Discovery**
3. Activate new services "Hyper Backup ..." and "Active Backup ..."
4. **Activate changes**

## Adjusting Thresholds

The default thresholds for backup age can be customized:

1. **Setup → Services → Service monitoring rules**
2. Search for "Synology Hyper Backup" or "Synology Active Backup"
3. Create a new rule with custom thresholds

### Default Thresholds

| Parameter | Default | Description |
|-----------|---------|-------------|
| Warning Age | 24h | Time since last backup for WARNING |
| Critical Age | 36h | Time since last backup for CRITICAL |

## Monitored Values

### Hyper Backup

| Service | Description |
|---------|-------------|
| Status | Current task status (none, backup, waiting, etc.) |
| Last Result | Result of the last backup (done, error, etc.) |
| Backup Age | Time since the last successful backup |
| Progress | Progress of running backup |

### Active Backup

| Service | Description |
|---------|-------------|
| Status | Device/task status |
| Last Backup | Status of the last backup |
| Backup Age | Time since the last backup |
| Transfer Size | Amount of transferred data |

## Status Mapping

### Hyper Backup Status

| API Status | CheckMK Status |
|------------|----------------|
| none, backup, detect, waiting, backingup | OK |
| version_deleting, preparing_version_delete, suspended | WARN |
| error, failed | CRIT |

### Backup Result

| API Result | CheckMK Status |
|------------|----------------|
| done, backingup, resuming | OK |
| none | WARN |
| error, failed, cancelled | CRIT |

## Troubleshooting

### Test Agent Manually

```bash
OMD[mysite]:~$ ./local/share/check_mk/agents/special/agent_synology_backup \
    --host 192.168.1.100 \
    --username monitoring \
    --password secret \
    --port 5000 \
    --no-https \
    --debug
```

### Common Issues

**Login failed (Error 400):**
- Check username/password
- Verify user permissions
- Is 2FA disabled?

**Permission denied (Error 402):**
- User needs admin rights or delegated permissions
- Enable DSM application access for the user
- Try adding user to administrators group

**SSL errors:**
- Enable `--no-verify-ssl` option
- Check port (5001 for HTTPS, 5000 for HTTP)

**No tasks found:**
- Is Hyper Backup / Active Backup installed?
- Does the user have access rights?

**API not found:**
- Check DSM version (6.x/7.x)
- Check firewall rules

**Password Store issues:**
- The plugin supports CheckMK's password store
- Passwords are automatically resolved by CheckMK

## Requirements

- CheckMK 2.3.0 or newer
- Synology DSM 6.x or 7.x
- Hyper Backup and/or Active Backup for Business installed
- Network access from CheckMK server to NAS (port 5000/5001)

## Package Contents

```
synology_backup-1.0.7.mkp
├── info                                    # Manifest (Python Dict)
├── info.json                               # Manifest (JSON)
├── agents.tar
│   └── special/agent_synology_backup       # Special Agent Script
└── cmk_addons_plugins.tar
    └── synology_backup/
        ├── agent_based/
        │   ├── synology_hyper_backup.py    # Check Plugin Hyper Backup
        │   └── synology_active_backup.py   # Check Plugin Active Backup
        ├── rulesets/
        │   ├── check_parameters.py         # WATO Threshold Rules
        │   └── special_agent.py            # WATO Datasource Rule
        ├── server_side_calls/
        │   └── special_agent.py            # Agent Command Builder
        └── checkman/
            ├── synology_hyper_backup       # Manual Page
            └── synology_active_backup      # Manual Page
```

## License

GNU General Public License v2

## Changelog

### Version 1.0.7
- Initial public release
- WATO Integration
- Configurable Thresholds
