# Oura Battery Monitor

Personal Oura Ring battery monitoring with **n8n** and **Pushover**.

[Import the workflow](workflows/oura-pushover-n8n-workflow.json) · [Installation guide](docs/pushover-setup.md)

## Notifications

| Battery event | Pushover message |
|---|---|
| Reported 1–5% | `Battery: X%. Charge now.` |
| Unchanged critical percentage for 2½ hours | `Unresponsive` |
| Reported 0% | `Battery dead (0%).` |

Each alert is sent once per discharge cycle; rearmed after charging to at least 15%. Pushover Emergency Priority repeats unacknowledged alerts every 30 minutes for up to three hours. Other Pushover notifications are unaffected.

## Adaptive polling

The workflow checks when the next API request is due every 15 minutes. It queries Oura every 8 hours at high charge, progressively more often as battery falls, down to every 15 minutes at 5% or below. **Manual runs always query Oura immediately.** Monitoring and alert state is stored in an n8n Data Table.

## Oura application pages

[Website](https://mrrobertkent.github.io/oura-battery-monitor/) · [Privacy Policy](https://mrrobertkent.github.io/oura-battery-monitor/privacy.html) · [Terms of Service](https://mrrobertkent.github.io/oura-battery-monitor/terms.html)

Never commit credentials or Pushover User Keys to this public repository. Independent personal-use project, not affiliated with Oura.
