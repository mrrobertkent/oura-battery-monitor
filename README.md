# Oura Battery Monitor

Personal Oura Ring monitoring with **n8n** and **Pushover**.

[Importable workflow](workflows/oura-pushover-n8n-workflow.json) · [Installation guide](docs/pushover-setup.md)

The workflow adaptively polls Oura (8 hours at high charge, down to 15 minutes near depletion), remembers alert history in an n8n Data Table, and delivers Pushover Emergency Priority alerts.

| Event | Notification |
|---|---|
| Battery at 1–5% | `Battery: X%. Charge now.` |
| Source reading stops updating for 2½ hours at critically low charge | `Unresponsive` |
| Battery reported at 0% | `Battery dead (0%).` |

When Oura reports the ring is charging or placed in its charger, the workflow cancels outstanding Pushover Emergency retries using the Pushover Receipts API. Manual Test independently sends a labeled test notification and checks Oura immediately.

## Oura developer application

[Website](https://mrrobertkent.github.io/oura-battery-monitor/) · [Privacy Policy](https://mrrobertkent.github.io/oura-battery-monitor/privacy.html) · [Terms of Service](https://mrrobertkent.github.io/oura-battery-monitor/terms.html)

All API credentials belong in n8n's credential store. Independent personal-use integration, not affiliated with Oura.
