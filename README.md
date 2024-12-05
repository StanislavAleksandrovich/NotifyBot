```
How to use

./notifybot -configfile ./notify.conf -port 8083 -maxalerts 5 -delay 3 -alertmanager-uri http://localhost:9093

-help
    Show help information

  -alertmanager-uri string
    Alertmanager base URI (default "http://localhost:9093")

  -configfile string
    Path to notification configuration file (default "notify.conf")

  -delay int
    Delay in seconds between notifications (default 2)

  -maxalerts int
    Maximum number of alerts before suppression (default 3)

  -port int
    Port to run the webhook server (default 8083)

  -timelimit int
    Minimum time (in seconds) between alert triggers (default 30)


notify.conf
key:value:notification_url

example
service:web-app:https://slack.com/webhook?text=$description
team:sales:https://pagerduty.com/alert?message=$description


in alertmanager.yml
receivers:
- name: 'webhook-notifier'
  webhook_configs:
  - url: 'http://localhost:8083/notifybot'
```
