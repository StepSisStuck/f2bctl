# Optional OpenVPN Access Server integration

OpenVPN Access Server support is optional and disabled by default. `f2bctl` can summarize matching authentication-log entries for an IP address, but it does not install an OpenVPN filter, create a jail, or change firewall rules.

## Enable log inspection

Confirm the correct log path for your installation and add a root-readable `/etc/f2bctl.conf`:

```bash
ENABLE_OPENVPN_AS=1
OVPN_LOG="/var/log/openvpnas.log"
```

Restrict the file because it controls which local log `f2bctl` reads:

```bash
sudo chown root:root /etc/f2bctl.conf
sudo chmod 0600 /etc/f2bctl.conf
```

If the integration is disabled or the configured log is unreadable, the dashboard reports that state and continues without OpenVPN data.

## Optional Fail2Ban jail

[`../examples/openvpnas.local`](../examples/openvpnas.local) is a disabled template, not a drop-in promise. OpenVPN AS versions and log formats vary. Before enabling any jail:

1. Identify the authentication failures your installation actually records.
2. Build or obtain a filter appropriate for that exact format.
3. Test it with `fail2ban-regex` against sanitized sample data.
4. Add trusted addresses deliberately.
5. Run `fail2ban-client -t` before reloading Fail2Ban.

Avoid committing real logs, usernames, hostnames, IP addresses, trusted networks, or email addresses to a public repository.
