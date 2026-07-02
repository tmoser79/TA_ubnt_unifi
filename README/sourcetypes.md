# UniFi sourcetype reference

## `ubnt:unifi:gw:firewall`

UniFi gateway **netfilter/iptables-style** syslog. Key=value fields after a rule bracket.

### Sample event

```
Jun 26 22:23:10 ri-fw01 ri-fw01 [LAN_WAN-A-2147483647] DESCR="[LAN_WAN]Allow All Traffic" IN=br0 OUT=eth10 MAC=9c:05:d6:65:39:06:0a:33:2a:4f:f6:54:08:00 SRC=192.168.1.118 DST=208.67.222.222 LEN=224 TOS=00 PREC=0x00 TTL=63 ID=57225 PROTO=UDP SPT=51133 DPT=443 LEN=204 MARK=1a0000
```

### Rule bracket

`[ZONE_PAIR-ACTION_CODE-RULE_ID]` — e.g. `[LAN_WAN-A-2147483647]`

| Part | Example | Meaning |
|------|---------|---------|
| Zone pair | `LAN_WAN`, `LAN_LOCAL` | Traffic zone transition |
| Action code | `A`, `D`, `R`, `B`, `RET` | Policy decision (`vendor_action_id`) |
| Rule ID | `2147483647` | Rule number (`INT_MAX` = default catch-all) |

### MAC field (L2 header)

Forwarded traffic includes a 14-octet Ethernet header in `MAC`:

```
MAC=9c:05:d6:65:39:06:0a:33:2a:4f:f6:54:08:00
     |--- src_mac ---| |--- dest_mac ---| |type|
```

| Field | Example | Meaning |
|-------|---------|---------|
| `src_mac` | `9c:05:d6:65:39:06` | Source MAC (6 octets) |
| `dest_mac` | `0a:33:2a:4f:f6:54` | Destination MAC (6 octets) |
| `ether_type` | `08:00` | EtherType (`0x0800` = IPv4) |

Some `LAN_LOCAL` events use `MAC` for raw packet bytes instead of an Ethernet header.

### EP fields (expected)

`SRC`, `DST`, `PROTO`, `SPT`, `DPT`, `IN`, `OUT`, `DESCR`, `vendor_action_id`, `zone_pair`, `rule_id`

### TA normalized fields

`src`, `dest`, `src_port`, `dest_port`, `transport`, `ingress_interface`, `egress_interface`, `rule_description`, `vendor_action_id`, `vendor_action`, `action`, `src_mac`, `dest_mac`, `ether_type`, `vendor`, `product`

### CIM

Event type `ubnt_unifi_gw_firewall` is tagged for **Network Traffic** (`network`). The `action` field uses CIM values (`allowed`, `blocked`); use `vendor_action` for UniFi-specific granularity.

---

## `ubnt:unifi:ap:wireless`

UniFi access point wireless logs. Multiple event families share this sourcetype.

### Sample events

**Client anomaly (mcad):**
```
Jun 26 22:24:51 ri-ap02 802aa813c2ad,UAP-AC-Lite-6.8.2+15592: mcad: mcad[1837]: wireless_agg_stats.log_sta_anomalies(): bssid=80:2a:a8:15:c2:ad radio=wifi1 vap=wifi1ap3 sta=d4:8d:26:8f:06:a0 satisfaction_now=73 anomalies=tcp_latency
```

**STA tracker JSON (stahtd):**
```
Jun 26 22:21:25 ri-ap01 b4fbe473c6f5,UAP-AC-Lite-6.8.2+15592: stahtd: stahtd[1758]: [STA-TRACKER].stahtd_dump_event(): {"message_type":"STA_ASSOC_TRACKER","mac":"0a:33:2a:4f:f6:54","vap":"wifi1ap3","event_type":"dns timeout",...}
```

### Event families

| Family | Process | `event_subtype` | Notes |
|--------|---------|-----------------|-------|
| Client anomalies | `mcad` | `sta_anomaly` | `satisfaction_now`, `anomalies` |
| STA tracker | `stahtd` | `sta_tracker` | JSON payload; prefer over kernel dupes |
| Kernel STA tracker | `kernel` | `sta_tracker_kernel` | Often duplicate of stahtd |
| HSM scan | `kernel` | `hsm_scan_transition` | High-volume RF noise |
| RRM scan | `syswrapper` | `rrm_scan_trigger` | Operational/debug |

### EP fields (expected)

Envelope: `ap_hostname`, `ap_mac`, `ap_model`, `ap_firmware`, `process`, `event_subtype`

Wireless: `bssid`, `radio`, `vap`, `sta`/`mac`, `satisfaction_now`, `anomalies`, `message_type`, `event_type`, `query_2`, `query_server_2`

### TA normalized fields

`client_mac`, `query`, `dns_server`, `satisfaction`, `vendor`, `product`

### CIM

Event type `ubnt_unifi_ap_wireless_dns_timeout` is tagged for **Network Resolution (DNS)** (`resolve`) and **Network Traffic** (`network`).
