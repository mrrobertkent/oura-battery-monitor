# Oura Battery Monitor — Setup

## Import CSV Data Tables

Use n8n **Data Tables → Create from CSV**:

- [oura_battery_monitor_state.csv](../schemas/oura_battery_monitor_state.csv)
- [oura_battery_alerts.csv](../schemas/oura_battery_alerts.csv)

Confirm these exact types:

**oura_battery_monitor_state**

| Column | Type |
|---|---|
| ring_key | String |
| battery_level | Number |
| last_source_at | Date |
| last_source_seen_at | Date |
| last_checked_at | Date |
| next_poll_at | Date |
| charging | Boolean |
| in_charger | Boolean |
| low_alert_sent | Boolean |
| unresponsive_alert_sent | Boolean |
| dead_alert_sent | Boolean |
| snoozed_until | Date |

The seed row `ring_key=oura` must remain.

**oura_battery_alerts**

| Column | Type |
|---|---|
| receipt_id | String |
| ring_key | String |
| alert_type | String |
| status | String |
| sent_at | Date |
| expires_at | Date |
| acknowledged_at | Date |
| canceled_at | Date |
| resolved_at | Date |

Delete the `TEMPLATE_DELETE_AFTER_IMPORT` placeholder row after importing.

## Configure workflow

1. Import [workflow](../workflows/oura-pushover-n8n-workflow.json).
2. Select your existing Oura OAuth2 credential in **Get Oura Battery**.
3. Select your Pushover credential in **Send Pushover Emergency**, **Send Test Pushover**, **Read Pushover Receipt** and **Cancel Old Pushover Receipt**.
4. Enter your Pushover User Key in the two Pushover nodes.
5. Select the correctly named Data Table in each Data Table node.
6. Run **Manual Test**; it checks the battery immediately and sends a separate labeled emergency test message.
7. Confirm receipt lookup, cancellation and table writes before activation. Activate only one copy.

## Rules

- Oura API polling: 8h above 40%, 6h at 31–40%, 4h at 21–30%, 2h at 11–20%, 1h at 8–10%, 30min at 6–7%, 15min at 0–5%.
- A 15-minute schedule checks whether the battery API poll is due and separately reconciles active Pushover acknowledgments.
- 1–5%: `Battery: X%. Charge now.`
- No newer source reading for 150 minutes at critical battery: `Unresponsive`.
- Reported 0%: `Battery dead (0%).`
- Emergency Priority: retry every 30 minutes, stop after 3h or earlier when acknowledged/canceled.
- Acknowledgment causes a **6-hour snooze** for repeat/Unresponsive alerts; a new Dead alert still sends.
- Charging cancels the active receipt. A new escalation cancels the previous receipt before sending.
- Battery at 15%+ resets alert eligibility.

## Retention

The monitoring table keeps one permanent row. Daily cleanup deletes closed receipt records older than 30 days, and retains at most 250 additional recent closed records. Active receipts are excluded.

## Reliability

This workflow was tested using simulated data, not live n8n executions. Check imported node settings for your installed n8n version. In particular, Pushover receipt GET/cancel authentication and Data Table delete operations require live testing. A single workflow avoids intentional stacking, but parallel executions and uncertain API responses can still create duplicates. Do not enable multiple active copies.
