# Oura Battery Monitor — Installation

## Requirements

- Oura OAuth2 API credential connected in n8n.
- Pushover Application/API Token saved as a Pushover API credential, plus your Pushover User Key.
- n8n Data Table named `oura_battery_monitor_state` with two **String** columns: `key` and `state_json`. Insert the first row: `key = oura`, `state_json = {}`.

## Import

1. Import [the workflow JSON](../workflows/oura-pushover-n8n-workflow.json) into n8n.
2. In **Get Oura Battery**, choose the connected Oura OAuth2 API credential.
3. In **Load Battery State** and **Save Battery State**, select your Data Table if not automatically resolved.
4. In **Push a message**, enter your Pushover User Key and select the Pushover API credential.
5. **Manual Test** runs two independent branches: one immediately queries Oura regardless of the next scheduled check, and one always sends a labeled **Oura Ring — Test** Pushover notification. The test message uses Emergency Priority with a **60-second retry interval and 120-second expiry**; acknowledge it in Pushover to stop retries. This branch **does not modify the Data Table**. The live battery branch may also send a real alert if the condition is met. To test, use **Test workflow** rather than executing only a single node.
6. Publish the workflow. Ensure only one copy is active.

## Oura polling intervals

| Reported charge | Interval |
|---|---|
| 41–100% | 8 hours |
| 31–40% | 6 hours |
| 21–30% | 4 hours |
| 11–20% | 2 hours |
| 8–10% | 1 hour |
| 6–7% | 30 minutes |
| 0–5% | 15 minutes |

The fixed 15-minute Schedule Trigger skips API calls until the Data Table's next due time. A missing response retries after 60 minutes. The reading is Oura cloud data, not a direct Bluetooth connection.

## Alerts and persistence

- **Low:** fresh 1–5% reading → `Battery: X%. Charge now.`
- **Unresponsive:** unchanged 1–5% reading for 150 minutes following the low-battery alert → `Unresponsive`.
- **Dead:** explicit API level 0% → `Battery dead (0%).` (independent of the Unresponsive alert).
- Charging suppresses alerts. Flags reset after a fresh 15%+ reading.
- The single Data Table row holds poll timing, previous critical reading, unchanged-since time and alert flags.
- Pushover Emergency Priority (2) resends every 30 minutes for up to 3 hours unless acknowledged; this is configured per message.

The workflow logic was tested against simulated responses. Verify the actual nodes in your n8n environment before publishing. Keep API secrets in n8n credentials. The public workflow intentionally excludes credential IDs and the Pushover User Key.
