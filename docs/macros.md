# Template Macros — Huawei ETP4860 Power System SNMP

All thresholds are expressed in real engineering units (V, A, °C, %, min) after
preprocessing — never in raw SNMP integer form.

| Macro | Default | Unit | Description |
|---|---|---|---|
| `{$SNMP_USERNAME}` | *(empty)* | — | SNMPv3 security name (username). Must match the ETP4860 SNMP user configuration. |
| `{$SNMP_AUTH_PASSPHRASE}` | *(empty, secret)* | — | SNMPv3 authentication passphrase. Protocol: SHA-512. |
| `{$SNMP_PRIV_PASSPHRASE}` | *(empty, secret)* | — | SNMPv3 privacy (encryption) passphrase. Protocol: AES-256. |
| `{$SNMP_CONTEXT_NAME}` | *(empty)* | — | SNMPv3 context name. Leave empty if not required by the device. |
| `{$SNMP.TIMEOUT}` | `5m` | — | Duration without SNMP data before declaring the device unreachable (nodata trigger). |
| `{$LLD.INTERVAL}` | `1h` | — | Polling interval for all Low-Level Discovery rules. |
| `{$VDC.MIN}` | `47` | V | DC bus lower voltage alarm threshold. Trigger fires when voltage persistently drops below this value. |
| `{$VDC.MAX}` | `58` | V | DC bus upper voltage alarm threshold. Trigger fires when voltage persistently exceeds this value. |
| `{$RECT.TEMP.MAX}` | `60` | °C | Maximum rectifier module temperature before alarming. |
| `{$BATT.SOC.MIN}` | `30` | % | Battery State-of-Charge lower alarm threshold. Trigger fires when overall SOC drops below this value. |
| `{$BATT.BACKUP.MIN}` | `30` | min | Minimum estimated battery backup time. Trigger fires when hwBattsPreDischargeTime drops below this value. |
| `{$BATT.TEMP.MAX}` | `40` | °C | Maximum battery string temperature before alarming. |
| `{$AC.VOLT.MIN}` | `195` | V | Minimum per-phase AC voltage. Any phase persistently below this triggers a warning. |
| `{$AC.VOLT.MAX}` | `265` | V | Maximum per-phase AC voltage. Any phase persistently above this triggers a warning. |
| `{$ENV.TEMP.MAX}` | `45` | °C | Maximum environment temperature sensor reading before alarming. |
| `{$ENV.TEMP.MIN}` | `5` | °C | Minimum environment temperature sensor reading before alarming. |

## Overriding Macros per Host

All macros can be overridden at the host level in Zabbix:
**Host → Macros → Inherited and host macros → add override**

This allows different thresholds for outdoor sites (higher `{$ENV.TEMP.MAX}`)
or sites with smaller batteries (higher `{$BATT.SOC.MIN}`).
