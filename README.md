# Oura Battery Monitor

Oura Ring battery monitoring using **n8n**, typed **Data Tables**, and **Pushover Emergency Priority**.

[Import the n8n workflow](workflows/oura-pushover-n8n-workflow.json) · [Setup guide](docs/pushover-setup.md)

## Current behavior

- Adaptive Oura polling: every 8 hours above 40%, increasing to every 15 minutes at 5% or below.
- One **Battery: X%. Charge now.** alert on reaching 1–5%.
- **Unresponsive** after 2½ hours without a newer source reading while critically low.
- **Battery dead (0%).** when Oura reports 0%, overriding an acknowledgment snooze.
- Reconciles Pushover acknowledgments every 15 minutes, snoozing repeat and Unresponsive alerts for **six hours**, but not a new Dead alert.
- Cancels the previous active Emergency receipt before sending a replacement; cancels on charging.
- Retains resolved alert history for 30 days, with a cap of 250 closed rows.
- Manual Test separately sends a labeled Pushover test alert and queries Oura immediately.

## n8n CSV table templates

Import both CSVs in **n8n Data Tables**. Confirm Boolean, Number, and Date column types in the import preview:

- [Monitoring state](schemas/oura_battery_monitor_state.csv) — one persistent row.
- [Pushover alert history](schemas/oura_battery_alerts.csv) — delete the placeholder row after import.

See the [setup guide](docs/pushover-setup.md) for the exact types and credential configuration. The public workflow deliberately contains no credential IDs or Pushover User Key.

## Oura application pages

[Website](https://mrrobertkent.github.io/oura-battery-monitor/) · [Privacy Policy](https://mrrobertkent.github.io/oura-battery-monitor/privacy.html) · [Terms of Service](https://mrrobertkent.github.io/oura-battery-monitor/terms.html)

Independent personal integration, not affiliated with Oura.
