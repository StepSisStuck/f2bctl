# Recidive jail guidance

Fail2Ban's recidive jail watches Fail2Ban's own log for repeat ban events and applies a longer penalty. `f2bctl` reads the active jail's policy from Fail2Ban and uses those values when displaying status and candidate counts.

The included [`../examples/recidive.local`](../examples/recidive.local) is disabled by default. Treat its values as a starting point, not a universal policy.

Before enabling recidive:

1. Confirm that `/var/log/fail2ban.log` exists and is retained long enough for your chosen `findtime`.
2. Review `maxretry`, `findtime`, and `bantime` for the services you operate.
3. Add loopback and only deliberately trusted addresses to `ignoreip`.
4. Test the full Fail2Ban configuration with `fail2ban-client -t`.
5. Monitor the jail after enabling it and verify that legitimate clients are not caught.

Do not use permanent bans as a default. Long bans can create operational risk when addresses are shared or reassigned.
