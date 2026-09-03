# f2bctl

`f2bctl` is an interactive command-line dashboard for Fail2Ban. It collects common monitoring and administration tasks into one menu so you do not have to remember a long list of `fail2ban-client`, log-search, DNS, and WHOIS commands.

It can display active jails, current bans, estimated remaining ban time, recent events, repeat offenders, recidive candidates, trusted addresses, and configuration health. It also provides confirmed manual ban/unban actions and can export an incident report for a selected IP address.

> [!IMPORTANT]
> Run the installation commands in a Linux terminal on the server where Fail2Ban is installed. Do not run them in Windows PowerShell or GitHub Desktop.

OpenVPN Access Server support is optional and disabled by default. `f2bctl` does not automatically install packages, create jails, edit Fail2Ban configuration, or change your existing ban policy.

## Screenshots

All screenshots below use simulated data and IP ranges reserved for documentation.

| Dashboard | Active bans |
| --- | --- |
| ![f2bctl dashboard](screenshots/dashboard.png) | ![Active bans and remaining time](screenshots/active-bans.png) |

| Recidive status | IP investigation |
| --- | --- |
| ![Recidive jail status](screenshots/recidive-status.png) | ![IP investigation](screenshots/ip-investigation.png) |

## Features

- Dashboard for every active Fail2Ban jail
- Current bans with estimated expiry and remaining time
- Recent ban and unban events
- Top repeat offenders in the current Fail2Ban log
- Recidive status and candidate counts based on the active jail settings
- IPv4 and IPv6 input validation
- Reverse DNS and WHOIS enrichment when optional tools are installed
- Manual ban and unban actions with a confirmation prompt
- Trusted/ignored-address display
- Fail2Ban configuration health check
- Text incident-report export
- Optional OpenVPN Access Server log inspection
- No external API keys required

## Requirements

`f2bctl` is intended for a Linux server with:

- Bash 4 or newer
- A configured and running Fail2Ban installation
- `fail2ban-client`
- Python 3
- Standard tools including `grep`, `sed`, `awk`, `sort`, `uniq`, `tail`, and `date`
- `sudo`, if the current user is not root

Optional enrichment features use:

- `dig` or `nslookup` for reverse DNS
- `whois` for network and ASN information

The dashboard will continue to work when optional enrichment tools are missing.

## Before installation

Connect to the server that actually runs Fail2Ban, for example through SSH, and confirm that Fail2Ban is working:

```bash
sudo fail2ban-client status
```

A healthy installation should return the number of active jails and their names. If `fail2ban-client` is not found or cannot connect to the Fail2Ban socket, fix the Fail2Ban installation before installing `f2bctl`.

You can also check the main requirements:

```bash
bash --version
python3 --version
command -v fail2ban-client
```

## Installation

### 1. Download the repository

Choose a working directory on the Linux server and clone the project:

```bash
git clone https://github.com/StepSisStuck/f2bctl.git
cd f2bctl
```

The `cd f2bctl` command is important: the remaining installation commands expect the local `f2b` file to be in the current directory.

If the repository is private, GitHub will require an authenticated account with access to it. Once the repository is public, anyone can clone it without repository access.

If Git is not installed, download the repository archive from GitHub, extract it on the Linux server, and enter the extracted directory before continuing.

### 2. Review and test the downloaded script

Because `f2bctl` is an administrative tool that runs with elevated permissions, review it before installation:

```bash
less f2b
```

Press `q` to leave `less`.

Check the Bash syntax:

```bash
bash -n f2b
```

No output means the syntax check passed.

Display the version and test dependencies without installing the script:

```bash
bash f2b --version
bash f2b --check
```

The dependency check distinguishes required commands from optional DNS and WHOIS tools. It does not install anything.

### 3. Install the command

From inside the cloned `f2bctl` directory, run:

```bash
sudo install -m 0755 f2b /usr/local/bin/f2b
```

This single command does three things:

- `sudo` runs the installation with permission to write to `/usr/local/bin`.
- `install -m 0755` copies the file and makes it executable. The owner can read, write, and execute it; everyone else can read and execute it.
- `f2b /usr/local/bin/f2b` copies the repository's `f2b` script to the standard system-wide command location.

It does **not** start a background service, modify Fail2Ban, create firewall rules, or enable any jail.

### 4. Verify the installation

Confirm that the command is available:

```bash
command -v f2b
f2b --version
f2b --check
```

Expected location:

```text
/usr/local/bin/f2b
```

### 5. Start the dashboard

Launch it with:

```bash
sudo f2b
```

The Fail2Ban control socket and logs are normally restricted to root. If you run `f2b` without `sudo`, the script attempts to restart itself through `sudo`.

Use the number keys to select a menu item. Select `0` to exit. The live-log view uses `Ctrl+C` to stop following the log.

## Configuration

The default settings are:

```bash
F2B_CLIENT="fail2ban-client"
F2B_LOG="/var/log/fail2ban.log"
ENABLE_OPENVPN_AS=0
OVPN_LOG="/var/log/openvpnas.log"
```

You do not need a configuration file when these defaults match your system.

To override them, create `/etc/f2bctl.conf` on the Linux server:

```bash
sudo nano /etc/f2bctl.conf
```

Add only the settings you need. For example:

```bash
F2B_LOG="/var/log/fail2ban.log"
ENABLE_OPENVPN_AS=0
```

Save the file in Nano with `Ctrl+O`, press `Enter`, and exit with `Ctrl+X`.

Protect the configuration file:

```bash
sudo chown root:root /etc/f2bctl.conf
sudo chmod 0600 /etc/f2bctl.conf
```

You can use a different configuration file temporarily by setting `F2BCTL_CONFIG`:

```bash
sudo F2BCTL_CONFIG=/path/to/f2bctl.conf f2b
```

## Optional OpenVPN Access Server integration

OpenVPN Access Server inspection is off by default. To enable it, first confirm the correct log path for your installation. Then add the following to `/etc/f2bctl.conf`:

```bash
ENABLE_OPENVPN_AS=1
OVPN_LOG="/var/log/openvpnas.log"
```

This only allows `f2bctl` to inspect matching entries in the configured log. It does not create or enable an OpenVPN Fail2Ban jail.

See [the OpenVPN Access Server guide](docs/openvpn-access-server.md) before enabling the integration. The file [`examples/openvpnas.local`](examples/openvpnas.local) is a disabled template that must be reviewed and adapted to the exact log format used by your OpenVPN AS version.

## Recidive support

When a jail named `recidive` is active, `f2bctl` reads its `maxretry`, `findtime`, and `bantime` values directly from Fail2Ban. The dashboard does not replace or silently override the configured policy.

The included [`examples/recidive.local`](examples/recidive.local) file is disabled by default. Review [the recidive guide](docs/recidive.md) before adapting or enabling it.

## Updating

Return to the directory where you cloned the repository:

```bash
cd f2bctl
git pull
```

Review and syntax-check the updated script:

```bash
bash -n f2b
bash f2b --version
```

Then replace the installed copy:

```bash
sudo install -m 0755 f2b /usr/local/bin/f2b
```

Your `/etc/f2bctl.conf` file is separate and is not overwritten by this process.

## Uninstalling

Remove the installed command:

```bash
sudo rm /usr/local/bin/f2b
```

If you created a configuration file exclusively for this tool and no longer need it, remove it separately:

```bash
sudo rm /etc/f2bctl.conf
```

Uninstalling `f2bctl` does not remove Fail2Ban and does not undo changes previously made through manual ban/unban actions.

## Troubleshooting

### `f2b: command not found`

Confirm that the script was installed in `/usr/local/bin`:

```bash
ls -l /usr/local/bin/f2b
```

If it is missing, return to the cloned repository directory and repeat the installation command.

### `fail2ban-client` is missing

Fail2Ban must be installed and configured before using this dashboard. Installation steps differ between Linux distributions, so use your distribution's official package documentation.

### Cannot connect to the Fail2Ban socket

Check whether Fail2Ban is running:

```bash
sudo systemctl status fail2ban
sudo fail2ban-client status
```

### Ban history or remaining time is unavailable

The current user must be able to read the configured `F2B_LOG`. Rotated or journal-only logging can also mean that the original ban event is no longer present. Current jail status can still work while historical details are unavailable.

### OpenVPN data is not displayed

Confirm that `ENABLE_OPENVPN_AS=1` is set, that `OVPN_LOG` points to the correct file, and that the log is readable by root. OpenVPN AS integration is intentionally disabled by default.

### `/usr/bin/env: 'bash\r': No such file or directory`

The script has Windows-style line endings. From the repository directory, convert it and reinstall:

```bash
sed -i 's/\r$//' f2b
sudo install -m 0755 f2b /usr/local/bin/f2b
```

## Security and privacy

- Run `f2bctl` only as a trusted administrator.
- Manual ban and unban actions change active firewall state and require confirmation.
- Incident reports can contain IP addresses and raw log excerpts. Review every report before sharing it.
- Do not publish screenshots containing real IP addresses, hostnames, usernames, trusted networks, email addresses, or authentication logs.
- The supplied jail examples are disabled by default and should never be enabled without review and testing.
- No external API keys are required.

## License

This project is available under the [MIT License](LICENSE).
