# Oura Battery Monitor — n8n Installation

## Setup

1. Import [the workflow JSON](../workflows/oura-pushover-n8n-workflow.json) into n8n.
2. In **Get Oura Battery**, select your existing Oura OAuth2 API credential.
3. Create/select the n8n Data Table `oura_battery_monitor_state` with two String columns: `key` and `state_json`. Insert one row: `key = oura`, `state_json = {}`.
4. Select that table in **Load Battery State** and **Save Battery State**.
5. Configure **Push a message** and **Push Test Notification** with your Pushover User Key and Pushover API credential.
6. Select the **same Pushover API credential** in **Cancel Pushover Receipts** (HTTP Request → Predefined Credential Type → Pushover API). The Pushover credential injects its API token into the form body.
7. Execute **Manual Test** to query Oura immediately and independently send a short Pushover emergency test alert. Acknowledge the test alert to stop repeats.
8. Activate one copy of the workflow.

## Behavior

- **Battery 1–5%:** `Battery: X%. Charge now.`
- **Unresponsive:** `Unresponsive` after 150 minutes without a newer *source reading* while last known level remains between 1% and 5%.
- **Dead:** `Battery dead (0%).` after a fresh API report of 0%.
- **Charging:** a fresh observation with `charging: true` or `in_charger: true` calls Pushover's receipt cancellation endpoint to stop retries of prior emergency notifications.
- **Rearmed:** when a fresh reading is 15% or higher.
- **Emergency Priority:** the production Pushover node repeats every 30 minutes for up to three hours, stopping earlier when acknowledged or canceled.
- **Manual Test:** always runs a separate 2-minute emergency test and independently queries Oura, bypassing the next-due gate.

## Adaptive polling

The workflow starts on a 15-minute schedule and skips Oura API calls until its next due time stored in the Data Table:

| Level | Oura API interval |
|---|---|
| 41–100% | 8 hours |
| 31–40% | 6 hours |
| 21–30% | 4 hours |
| 11–20% | 2 hours |
| 8–10% | 1 hour |
| 6–7% | 30 minutes |
| 0–5% | 15 minutes |

## Oura timestamps

The workflow prefers the numerical millisecond `producer_timestamp` when it is plausible and agrees approximately with the ISO `timestamp`; otherwise it falls back to `timestamp`. The interpretation of `producer_timestamp` as the ring/source observation time is a reasonable inference, **not a verified guarantee from Oura's published documentation**. Unresponsive means the source timestamp has not advanced for 2½ hours; it is not a live Bluetooth connection test.

`next_token` is an API pagination cursor; `null` requires no action for the `latest=true` query.

## Pushover receipts

The production Pushover node returns a receipt for every Emergency Priority message. The workflow records receipt IDs and their three-hour expiry in the Data Table, and cancels still-active receipts via Pushover's API once Oura reports charging/charger or a sufficient recharge. Canceling prevents further retries; it does not retract a notification already displayed. Successful API responses must be confirmed before receipts are removed from stored state.

## Operations and security

Keep all OAuth/Pushover secrets inside n8n credentials. The public GitHub workflow intentionally has no credential IDs or Pushover User Key. The separate personal import preserves your existing credential references. Avoid overlapping active copies; the Data Table record is not transaction-locked across simultaneous executions.

**Important:** The receipt-cancellation HTTP node needs verification in your own n8n version because credential injection and form encoding vary by node version. First run a low-impact test and confirm it returns Pushover `status: 1` before relying on automatic cancellation.

The workflow's logic was exercised against simulated Oura responses; live API and receipt cancellation still require an end-to-end test.
