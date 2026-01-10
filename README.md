```
How to use

./notifybot -configfile ./notify.conf -port 8083 -maxalerts 5 -delay 15 -alertmanager-uri http://localhost:9093

  -help
    Show help information
  -alertmanager-uri string
    Alertmanager base URI (default http://localhost:9093)
  -configcheck string
    Interval in seconds to check for config file changes (default 10)
  -configfile string
    Path to notification configuration file (default notify.conf)
  -delay string
    Delay in seconds between notifications (default 2)
  -maxalerts string
    Maximum number of alerts before suppression (per team) (default 3)
  -port string
    Port to run the notifybot server (default 8083)
  -receiver string
    Alertmanager receiver name to filter alerts (default notifybot)
  -timelimit string
    Minimum time (in seconds) between alert triggers (default 30)


notify.conf
key=value:notification_url

example
service=web-app:https://slack.com/webhook?text=$description
team=sales:https://pagerduty.com/alert?message=$description


in alertmanager.yml use value of this flag -receiver , default notifybot

if -receiver webhook-notifier then use below

receivers:
- name: 'webhook-notifier'
  webhook_configs:
  - url: 'http://localhost:8083/notifybot'
```
