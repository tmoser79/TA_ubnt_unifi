# TA_ubnt_unifi

Splunk Technology Add-on for **Ubiquiti UniFi** logs.

Parsing, sourcetype assignment, and field extraction are expected to be performed by **Splunk Edge Processor (EP)**. This add-on provides search-time knowledge objects: field aliases, evals, event types, CIM-oriented tags, and macros.

## Supported sourcetypes

| Sourcetype | Description |
|------------|-------------|
| `ubnt:unifi:gw:firewall` | UniFi gateway netfilter traffic (USG/UDM/UXG) |
| `ubnt:unifi:ap:wireless` | UniFi access point wireless subsystem logs |

See [README/sourcetypes.md](README/sourcetypes.md) for field references and example searches.

## Prerequisites

- Splunk Enterprise or Splunk Cloud
- Splunk Edge Processor pipeline that assigns the sourcetypes above and extracts core fields
- Optional: Splunk Common Information Model (CIM) add-on for datamodel alignment

## Installation

1. Package or copy this app into `$SPLUNK_HOME/etc/apps/TA_ubnt_unifi` (or install via Splunk UI).
2. Restart Splunk or reload the app.
3. Confirm event types appear under **Settings → Knowledge → Event types**.

## Firewall action mapping

Bracket action codes in `[ZONE_PAIR-ACTION-RULEID]` are normalized via `lookups/unifi_firewall_action_map.csv`:

| `vendor_action_id` | `vendor_action` | `action` (CIM) |
|--------------------|-----------------|----------------|
| `A` | `allowed` | `allowed` |
| `D` | `dropped` | `blocked` |
| `R` | `rejected` | `blocked` |
| `B` | `blocked` | `blocked` |
| `RET` | `allowed` | `allowed` |

EP may set `vendor_action_id`, `vendor_action`, or `action` at ingest. The TA derives `vendor_action_id` from `_raw` when missing, then applies the lookup with `OUTPUTNEW` (preserving existing `vendor_action` and `action`). Unmatched codes receive `unknown` via `default_match`.

## Example searches

```spl
# Gateway firewall denies
`unifi_fw_denied`

# WAN egress excluding default catch-all rule
`unifi_fw_wan_egress` NOT rule_id=2147483647

# Wireless client issues (exclude RF scan noise)
`unifi_ap_wireless_clients`

# DNS timeouts from STA tracker JSON (preferred over kernel duplicates)
`unifi_ap_wireless_dns_timeouts`
```

## Version

1.0.0 — initial release with gateway firewall and AP wireless knowledge objects.

## Author

Tomas Moser
