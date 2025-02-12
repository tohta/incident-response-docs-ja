---
cover: assets/img/covers/alerting_principles.png
description: We manage how we get alerted based on a simple principle, an alert is something which requires a human to perform an action. Anything else is a notification, which is something that we cannot control, and for which we cannot make any action to affect it. Notifications are useful, but they shouldn't be waking people up under any circumstance.
---
私たちは、シンプルな原則に基づいてアラートを管理しています。**アラートとは、人間がなんらかの行動を起こす必要があるものを指します。** それ以外のものは通知であり、それを制御することも、それに影響を与えるような行動を取ることもできません。通知は有用ですが、どのような状況であっても人を起こしてはいけません。

## Alert Priority

!!! warning "優先度の高いアラート"
    真夜中に人を呼び起こすようなことは、**即座に人間が対応できるもの** でなければなりません。そうでない場合は、その時間帯にはアラートが鳴らないように調整する必要があります。

| 優先度 | アラート | 対応 |
| -------- | ------ | -------- |
| 高 | 24時間365日体制の高優先度アラート | **直ちに人間が対応する** 必要がある。 |
| 中 |  通常業務時間帯の高優先度アラート | 24時間以内に人間が対応する必要がある。 |
| 低 | 24時間365日体制の低優先度アラート | いずれは人間が対応する必要がある。 |
| 通知 | 抑制されたイベント | 情報提供のみ、対応は不要。 |

新しいアラート/通知を設定する際は、上記の表を参考に、どのようにして人々にアラートを通知したいかを検討してください。例えば、即時の対応が必要でない場合は、優先度の高いアラートを新たに作成しないように注意してください。

## Priority Examples

#### "Production service is failing for 75% of requests, automation is unable to resolve."_
This would be a **High** priority page, requiring immediate human action to resolve.

![High Urgency](../assets/img/screenshots/high_urgency.png)

#### "Production server disk space is filling, expected to be full in 48 hours. Log rotation is insufficient to resolve."
This would be a **Medium** priority page, requiring human action soon, but not immediately.

![Medium Urgency](../assets/img/screenshots/high_business_hours.png)

#### "An SSL certificate is due to expire in one week."
This would be a **Low** priority page, requiring human action some time soon.

![Low Urgency](../assets/img/screenshots/low_urgency.png)

#### "A deployment was successful."
This would be a **Notification**, and should be sent as a suppressed event. It provides useful context should an incident occur, but does not require notifying a human.

![Notification](../assets/img/screenshots/suppressed.png)


## Alert Content

We should ensure that alerts contain enough useful context to quickly identify the issue and any potential remediation steps. Alerts with generic titles or descriptions are not useful and can cause confusion. We have a set of guidelines for the content of alerts, which all our alerts should follow,

#### Make the title/summary descriptive and concise.
  * <span class="icon bad"></span>  ALERT: Something went wrong.
  * <span class="icon good"></span> Disk is 80% full on `prod-web-loadbalancer-af5462ce`.

#### Make sure to include the metric which triggered the alert somewhere in the body.
  * <span class="icon bad"></span>  Diskspace on a disk is filling.
  * <span class="icon good"></span> `avg(last_1h):max:system.disk.in_use{env:prod-web-loadbalancer} by {host} > 0.8`

#### The body should also include a description of what the actual problem is, and why it's an issue.
  * <span class="icon bad"></span>  Disk is full.
  * <span class="icon good"></span> The disk on this host is at 80% capacity. If it becomes too full it could cause system instability as new files will not be able to be created and current files will not be written to.

#### Provide clear steps to resolve the problem, or link to a run book. Alerts with neither of these things are useless.
  * <span class="icon bad"></span>  Fix it by deleting stuff.
  * <span class="icon good"></span> Follow the run book here for identifying and resolving disk space issues: https://example.com/runbook/disk. Additionally, you should investigate whether log rotation thresholds are sufficient to prevent this happening again, the following run book has the necessary steps: https://example.com/runbook/log-rotate


## Testing Your Alerts

!!! info "Testing is Critical"
    An untested alert is equivalent to not having an alert at all. You cannot be sure it will alert you when the time comes. Testing that your alerting actually works is critical to proper service health and should be included in any release planning / deployment efforts.

Make sure to test all new and modified alerts. This is usually covered as part of [Failure Friday](https://www.pagerduty.com/blog/failure-friday-at-pagerduty/) for any new service; however, you should manually test them if you need it more quickly. Some things to test:

* Test that the threshold is set appropriately. We don't want noisy alerts.
* Test that you get alerted for the "No Data" condition if applicable. Generally, receiving no data is the same as breaking your threshold.
* Test that the alert resolves automatically when the metric returns to normal.
