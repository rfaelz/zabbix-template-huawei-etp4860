# Zabbix Template — Huawei ETP4860-B1A2 Power System

Production-quality Zabbix 7.0 LTS template for the **Huawei ETP4860-B1A2** DC power system /
rectifier, using SNMP and full Low-Level Discovery.

| | |
|---|---|
| **Supported model** | ETP4860-B1A2 |
| **Controller** | iSitePower (hwSiteMonitorMIB) |
| **Tested firmware** | iSitePower V100R022C00SPC121 |
| **Zabbix version** | 7.0 LTS (import format 7.0) |
| **OID root** | `1.3.6.1.4.1.2011.6.164` |

---

## About

Monitors the entire iSitePower stack via SNMP:

- **20 scalar items** — system availability, rectifier group totals, battery bank state, AC and DC bus metrics.
- **7 LLD rules** — auto-discovers every rectifier module, battery string, AC input, DC output, LVD branch, environment temperature sensor and digital input. Each instance is labelled with the **device-assigned name** (e.g. `{#RECTNAME}`, `{#BATTNAME}`), so items and triggers show meaningful names in the Zabbix UI without manual configuration.
- **Sentinel handling** — the firmware returns `2147483647` (INT32_MAX) on measurement OIDs when rectifiers shut down during an AC outage. All voltage/current/power/load items convert this sentinel to `0` via a JavaScript preprocessing step, keeping graphs continuous.
- **AC-outage alarm suppression** — the *Rectifier: In protection or off* trigger depends on *Battery discharging*, so individual module alerts are silenced while the entire AC feed is lost.
- **Backup time in hours** — `hwBattsPreDischargeTime` is reported in hours by the firmware (MIB says minutes); the template uses `h` units accordingly. The backup-time alert only fires while the battery is actually discharging.
- All OIDs are **numeric** — no MIB files required on the Zabbix server.

---

## Metrics collected

### Scalar items (always collected)

| Group | Item | Unit |
|---|---|---|
| System | Uptime, site alarm status | — |
| Monitor | Controller status, firmware version | — |
| Rectifiers | Total DC current, DC power, AC input power | A, W |
| Battery | Bank current, SOC %, remaining capacity, backup time, charge status | A, Ah, %, h |
| AC bus | Number of inputs | pcs |
| DC bus | Bus voltage, total current, total power, accumulated energy | V, A, W, kWh |

### Discovered via LLD

| LLD Rule | Macro | Items per instance |
|---|---|---|
| Rectifier modules | `{#RECTNAME}` | Status, DC output V/I/P, AC input V, efficiency, temperature |
| Battery strings | `{#BATTNAME}` | Status, current, SOC %, remaining capacity, temperature, midpoint voltage |
| AC inputs | `{#ACNAME}` | Status, Va/Vb/Vc, Ia/Ib/Ic |
| DC outputs | `{#DCNAME}` | Status, Vdc, Idc, Pdc, load % |
| LVD branches | `{#LVDNAME}` | Status, disconnect voltage |
| Env. temp. sensors | `{#TEMPNAME}` | Temperature |
| Digital inputs | `{#DINAME}` | Status |

---

## Template macros

| Macro | Default | Meaning |
|---|---|---|
| `{$VDC.MIN}` / `{$VDC.MAX}` | 47 / 58 V | DC bus voltage range |
| `{$BATT.SOC.MIN}` | 30 % | Low SOC alarm |
| `{$BATT.BACKUP.MIN}` | 1 h | Minimum backup time alarm |
| `{$AC.VOLT.MIN}` / `{$AC.VOLT.MAX}` | 195 / 265 V | AC voltage range |
| `{$RECT.TEMP.MAX}` | 60 °C | Rectifier overheat |
| `{$BATT.TEMP.MAX}` | 40 °C | Battery string overheat |
| `{$ENV.TEMP.MAX}` / `{$ENV.TEMP.MIN}` | 45 / 5 °C | Environment sensor range |
| `{$SNMP.TIMEOUT}` | 5m | SNMP nodata timeout |
| `{$LLD.INTERVAL}` | 1h | LLD polling interval |

---

## Trigger overview

| Trigger | Severity | Condition |
|---|---|---|
| No SNMP response | High | nodata for `{$SNMP.TIMEOUT}` |
| Site in alarm state | High | hwSiteRunningStatus = alarm |
| Monitor module fault | High | hwMonitorOperStatus ≠ normal |
| Device rebooted | Info | sysUpTime < 10 min |
| AC: Power failure | Disaster | hwAcInputOperStatus = acOff or acAbsent |
| Rectifier: Critical fault | Disaster | hwRectOperStatus = fault/commFail/rectLost |
| LVD branch disconnected | High | hwLvdBranchOperStatus ≠ normal |
| DC bus: Voltage low | High | DC bus < `{$VDC.MIN}` (3 consecutive samples) |
| Battery discharging | Average | hwBattsChargeStatus = disCharge |
| Battery: SOC low | High | SOC < `{$BATT.SOC.MIN}` |
| Battery: Backup time low | High | Backup time ≤ `{$BATT.BACKUP.MIN}` h **and** discharging |
| Rectifier: High temperature | Warning | Module temp > `{$RECT.TEMP.MAX}` (3 samples) |
| Battery string: High temperature | Warning | String temp > `{$BATT.TEMP.MAX}` (3 samples) |
| Environment: Temperature out of range | Warning | Env. sensor outside `[{$ENV.TEMP.MIN}, {$ENV.TEMP.MAX}]` |

### Trigger dependencies

- **Rectifier: In protection or off** → depends on **Battery discharging** (suppresses individual
  module alerts during a site-wide AC outage, when all rectifiers shut down simultaneously).
- **Battery: SOC below threshold** → depends on **Battery discharging** (suppresses the high-priority
  SOC alert when the discharge cause is already known).
- **Battery: Backup time below threshold** is gated directly in its expression on
  `batt.charge.status = disCharge(3)` — no separate dependency needed.

---

## Credits

- MIB source: Huawei `EMAP-MIB` (`emap_snmp.mib`) + `HUAWEI-MIB`
- Walk reference: ETP4860-B1A2 with iSitePower V100R022C00SPC121
- Author: Rafael Bastos
- Created with [Claude Code](https://claude.ai/code) (Anthropic)

## License

[MIT](LICENSE)
