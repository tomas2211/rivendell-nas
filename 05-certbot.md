# Certbot

## Prepare user
1. sudo useradd -m -s /bin/bash certbot
2. sudo loginctl enable-linger certbot

**As certbot user**

3. systemctl --user enable --now podman-auto-update.timer
4. mkdir -p .config/containers/systemd

## Cloudflare

Generate Cloudflare token and save it to `cloudflare/credentials`:
```ini
dns_cloudflare_api_token = <TOKEN>
```

## First-time setup

This will issue a certificate and setup renewals:
```bash
podman run --name dns-cloudflare --rm -v /home/certbot/letsencrypt:/etc/letsencrypt -v /home/certbot/letsencrypt/log:/var/log/letsencrypt -v /home/certbot/cloudflare/credentials:/opt/cloudflare/credentials:ro "docker.io/certbot/dns-cloudflare:latest" certonly --non-interactive --dns-cloudflare --dns-cloudflare-credentials /opt/cloudflare/credentials --dns-cloudflare-credentials /opt/cloudflare/credentials --agree-tos --email <email> -d "<domain>" -d "*.<domain>"
```

## Quadlets

To renew, we just need to run:
```bash
podman run --name dns-cloudflare --rm -v /home/certbot/letsencrypt:/etc/letsencrypt -v /home/certbot/letsencrypt/log:/var/log/letsencrypt -v /home/certbot/cloudflare/credentials:/opt/cloudflare/credentials:ro "docker.io/certbot/dns-cloudflare:latest" renew
```

This translates to a service:

```ini
[Unit]
Description=Certbot renewal
Wants=network-online.target
After=network-online.target

[Container]
Image=docker.io/certbot/dns-cloudflare:latest
ContainerName=certbot
Volume=%h/cloudflare:/opt/cloudflare:Z,ro
Volume=%h/letsencrypt:/etc/letsencrypt:Z
Volume=%h/letsencrypt/log:/var/log/letsencrypt:Z
Exec=renew

[Service]
Type=oneshot

Restart=on-abnormal
RestartSec=3600
TimeoutStopSec=60
TimeoutStartSec=10800

NotifyAccess=all

[Install]
WantedBy=default.target
```

Which gets a timer under `~/.config/systemd/user/certbot.timer`

```ini
[Unit]
Description=Renews any certificates that need them renewed
Requires=certbot-renew.service

[Timer]
Unit=certbot-renew.service
OnCalendar=weekly

[Install]
WantedBy=timers.target
```

To enable the timer, run: `systemctl --user enable --now certbot.timer`

To verify that the renew unit is loaded: `systemctl --user status certbot-renew`

Place shell scripts to deploy the certificates in `~/letsencrypt/renewal-hooks/deploy`. Pro tip: shell in the podman container may struggle with shebang. It's better not to specify it.

Use the renew command from the top with `--dry-run --run-deploy-hooks --no-random-sleep-on-renew` to verify that deploy hooks work.
