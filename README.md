# NotifyBot

A lightweight notification router for [Prometheus Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/).
NotifyBot receives alerts from Alertmanager via webhook and forwards them to
different destinations — Slack, TTS, Telegram, or any HTTP-based API — based on
the labels attached to each alert.

You define routing rules in a simple config file: match a set of alert labels,
and send matching alerts to a chosen notification URL. NotifyBot also applies
per-team rate limiting and suppression so a storm of alerts doesn't flood your
channels.

---

## Features

- Receives Alertmanager alerts over a webhook endpoint
- Routes alerts to different destinations based on their labels
- Works with any HTTP notification target (Slack, Telegram, TTS, custom APIs, …)
- Template alert fields (such as `$description`) into the outgoing URL
- Per-team rate limiting: max alerts before suppression, delay between
  notifications, and a minimum time between triggers
- Hot-reload of the config file — changes are picked up without a restart
- Configurable Alertmanager receiver name for clean integration

---

## How It Works

```
Prometheus  ──►  Alertmanager  ──(webhook)──►  NotifyBot  ──►  Slack
                                                         ├──►  Telegram
                                                         ├──►  TTS API
                                                         └──►  Custom HTTP API
```

1. Alertmanager is configured with a webhook receiver pointing at NotifyBot.
2. NotifyBot reads its routing rules from `notify.conf`.
3. For each incoming alert, NotifyBot matches the alert's labels against the
   rules and forwards it to the matching notification URL.
4. Rate limiting and suppression are applied per team to avoid flooding.

---

## Requirements

- Linux (binary distribution)
- A running Prometheus Alertmanager instance
- One or more HTTP notification endpoints (Slack webhook, Telegram bot API,
  TTS API, etc.)

---

## Installation

```bash
chmod +x notifybot
./notifybot
```

By default NotifyBot listens on port `8083` and reads `notify.conf` from the
current directory.

---

## Usage

```bash
./notifybot -configfile ./notify.conf -port 8083 -maxalerts 5 -delay 15 -alertmanager-uri http://localhost:9093
```

### Command-Line Options

| Flag | Default | Description |
|------|---------|-------------|
| `-configfile string` | `notify.conf` | Path to the notification configuration file |
| `-port string` | `8083` | Port to run the NotifyBot server on |
| `-alertmanager-uri string` | `http://localhost:9093` | Alertmanager base URI |
| `-receiver string` | `notifybot` | Alertmanager receiver name to filter alerts |
| `-maxalerts string` | `3` | Maximum number of alerts before suppression (per team) |
| `-delay string` | `2` | Delay in seconds between notifications |
| `-timelimit string` | `30` | Minimum time (in seconds) between alert triggers |
| `-configcheck string` | `10` | Interval in seconds to check the config file for changes |
| `-help` | | Show help information |

---

## Configuration (`notify.conf`)

Each line in `notify.conf` is a routing rule with the format:

```
<label matchers>:<notification_url>
```

The part before the final `:` is a comma-separated list of `key=value` label
matchers. The part after it is the URL the alert is forwarded to when **all**
matchers match.

### Valid formats

```
key=value:notification_url
key1=val1,key2=val2:notification_url
key="value with spaces":notification_url
key="a,b,c":notification_url
k1=v1,k2="v 2",k3="a,b,c":notification_url
key='value':notification_url
```

Quoting rules:

- Use double or single quotes around a value that contains **spaces**.
- Use quotes around a value that contains **commas**, so the comma isn't read as
  a separator between matchers.

### Templating

Alert fields can be injected into the notification URL using `$`-prefixed
placeholders. For example, `$description` is replaced with the alert's
description before the request is sent.

### Example `notify.conf`

```conf
# Simple — all alerts with team=Devops go to this Slack webhook
team=Devops:https://slack.com/webhook?text=$description

# Multiple labels — must match BOTH team and namespace
team=Devops,namespace=frontend:https://slack.com/webhook?text=$description

# Value with spaces — quote it
team=Devops,namespace=backend,job="my ne job":https://slack.com/webhook?text=$description

# Value with commas — quote it so commas aren't treated as separators
team=Devops,namespace=prod,job="my new job",members="a,b,c":https://slack.com/webhook?text=$description
```

> The config file is re-checked every `-configcheck` seconds (default 10), so
> edits take effect without restarting NotifyBot.

---

## Alertmanager Integration

Point an Alertmanager receiver at NotifyBot's `/notifybot` webhook endpoint. The
receiver **name** must match NotifyBot's `-receiver` flag (default `notifybot`).

If you run NotifyBot with the default receiver name:

```yaml
receivers:
  - name: 'notifybot'
    webhook_configs:
      - url: 'http://localhost:8083/notifybot'
```

If you start NotifyBot with a custom receiver name, e.g.
`-receiver webhook-notifier`, then use that name in `alertmanager.yml`:

```yaml
receivers:
  - name: 'webhook-notifier'
    webhook_configs:
      - url: 'http://localhost:8083/notifybot'
```

And route alerts to it:

```yaml
route:
  receiver: 'webhook-notifier'
```

---

## Running as a systemd Service

Create `/etc/systemd/system/notifybot.service`:

```ini
[Unit]
Description=NotifyBot - Alertmanager notification router
After=network.target

[Service]
ExecStart=/usr/local/bin/notifybot -configfile /etc/notifybot/notify.conf -port 8083 -alertmanager-uri http://localhost:9093
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now notifybot
sudo systemctl status notifybot
```

---

## Rate Limiting & Suppression

NotifyBot limits how many notifications it sends per team to prevent channel
flooding during an alert storm:

- **`-maxalerts`** — maximum number of alerts per team before further alerts are
  suppressed.
- **`-delay`** — delay (seconds) inserted between consecutive notifications.
- **`-timelimit`** — minimum time (seconds) that must elapse between alert
  triggers.

Tune these to match how noisy your alerts are and how much traffic your
notification endpoints can absorb.

---

## License

This is **proprietary freeware**. The source code is **not** provided and
remains the property of the author.

You may download, install, and run this binary free of charge for both personal
and commercial use. You may **not** reverse engineer, modify, redistribute,
resell, or sublicense it. See the [LICENSE](LICENSE) file for the full terms.

The software is provided "as is", without warranty of any kind.

## Support

This project does not provide source code or accept code contributions. For
bug reports or feature requests, please open an issue.
