# Zabbix Template — Huawei ETP4860-B1A2 Power System (SNMPv3)

Production-quality Zabbix 7.0 LTS template for the **Huawei ETP4860-B1A2** DC power system /
rectifier, using SNMPv3 and full Low-Level Discovery.

| | |
|---|---|
| **Supported model** | ETP4860-B1A2 |
| **Controller** | iSitePower (hwSiteMonitorMIB) |
| **Tested firmware** | iSitePower V100R022C00SPC121 |
| **Zabbix version** | 7.0 LTS (import format 7.0) |
| **SNMP version** | SNMPv3 authPriv (SHA-512 / AES-256) |
| **OID root** | `1.3.6.1.4.1.2011.6.164` |

---

## Metrics collected

### Scalar items (always collected)

| Group | Item | Unit |
|---|---|---|
| System | Uptime, site alarm status, active alarm count | — |
| Monitor | Controller status, firmware version | — |
| Rectifiers | Total DC current, total DC power, AC input power, load usage | A, W, % |
| Battery | Bank current, SOC %, remaining capacity, backup time, charge status | A, Ah, %, min |
| AC | Total AC current | A |
| DC bus | Bus voltage, total current, total power, accumulated energy | V, A, W, kWh |

### Discovered via LLD

| LLD Rule | Macro | Items per instance |
|---|---|---|
| Rectifier modules | `{#RECTNAME}` | Status, Vout, Iout, Pdc, Vac, efficiency, temperature |
| Battery strings | `{#BATTNAME}` | Status, current, SOC%, remaining capacity, temperature, midpoint voltage |
| AC inputs | `{#ACNAME}` | Status, Va/Vb/Vc, Ia/Ib/Ic |
| DC outputs | `{#DCNAME}` | Status, Vdc, Idc, Pdc, load % |
| LVD branches | `{#LVDNAME}` | Status, disconnect voltage |
| Env. temp. sensors | `{#TEMPNAME}` | Temperature |
| Digital inputs | `{#DINAME}` | Status |

---

## Template macros

See [docs/macros.md](docs/macros.md) for the full table. Key defaults:

| Macro | Default | Meaning |
|---|---|---|
| `{$VDC.MIN}` / `{$VDC.MAX}` | 47 / 58 V | DC bus voltage range |
| `{$BATT.SOC.MIN}` | 30 % | Low SOC alarm |
| `{$BATT.BACKUP.MIN}` | 30 min | Minimum backup time alarm |
| `{$AC.VOLT.MIN}` / `{$AC.VOLT.MAX}` | 195 / 265 V | AC voltage range |
| `{$RECT.TEMP.MAX}` | 60 °C | Rectifier overheat |
| `{$SNMP.TIMEOUT}` | 5m | SNMP nodata timeout |

---

## Import instructions

1. In Zabbix: **Data collection → Templates → Import**
2. Select `templates/huawei_etp4860_snmp.yaml`
3. Check **Update existing** and **Delete missing** if re-importing
4. Click **Import**

---

## Host configuration

### 1. Create the host

**Configuration → Hosts → Create host**
- Hostname: e.g. `ETP4860-Site-A`
- Groups: add to an appropriate group
- Templates: link **Huawei ETP4860 Power System SNMP**

### 2. Add the SNMP interface

**Host → Interfaces → Add → SNMP**

| Field | Value |
|---|---|
| IP address | `<device IP>` |
| Port | `161` |
| SNMP version | `SNMPv3` |
| Security name | `{$SNMP_USERNAME}` |
| Security level | `authPriv` |
| Auth protocol | `SHA512` |
| Auth passphrase | `{$SNMP_AUTH_PASSPHRASE}` |
| Priv protocol | `AES256` |
| Priv passphrase | `{$SNMP_PRIV_PASSPHRASE}` |
| Context name | `{$SNMP_CONTEXT_NAME}` (leave empty if unused) |

### 3. Set the host-level macros

**Host → Macros → Inherited and host macros**

Override the SNMPv3 credentials (these are secret — they will not appear in the UI after saving):

```
{$SNMP_USERNAME}        = <your SNMPv3 username>
{$SNMP_AUTH_PASSPHRASE} = <your auth passphrase>   (type: Secret text)
{$SNMP_PRIV_PASSPHRASE} = <your priv passphrase>   (type: Secret text)
```

Override any threshold macros that differ from the defaults (e.g. `{$BATT.SOC.MIN}` for sites
with partial-capacity battery banks).

---

## Trigger overview

| Trigger | Severity | Condition |
|---|---|---|
| SNMP sem resposta | High | nodata for `{$SNMP.TIMEOUT}` |
| Site em alarme | High | hwSiteRunningStatus = alarm |
| Monitor com falha | High | hwMonitorOperStatus ≠ normal |
| Dispositivo reinicializado | Info | sysUpTime < 10 min |
| CA: Falha de alimentação | Disaster | hwAcInputOperStatus = acOff or acAbsent |
| Retificador: Falha crítica | Disaster | hwRectOperStatus = fault/commFail/rectLost |
| LVD desconectado | High | hwLvdBranchOperStatus ≠ normal |
| CC: Tensão baixa | High | DC bus < `{$VDC.MIN}` (3 consecutive samples) |
| Bateria em descarga | Average | hwBattsChargeStatus = disCharge |
| SOC baixo | High | SOC < `{$BATT.SOC.MIN}` |
| Tempo de backup insuficiente | High | Backup time < `{$BATT.BACKUP.MIN}` min |
| Temperatura retificador alta | Warning | Module temp > `{$RECT.TEMP.MAX}` (3 samples) |
| Temperatura bateria alta | Warning | String temp > `{$BATT.TEMP.MAX}` (3 samples) |
| Temperatura ambiente | Warning | Env. sensor out of `[{$ENV.TEMP.MIN}, {$ENV.TEMP.MAX}]` |

---

## Trigger dependency

The **Battery SOC low** trigger has a Zabbix dependency on **Battery em descarga**,
so low-SOC alerts are suppressed when the discharge cause is already known.

For AC-outage suppression of battery triggers: add manual dependencies in
**Configuration → Triggers** after import, linking battery discharge/SOC triggers
to depend on the specific AC input failure trigger discovered for your site.

---

## Troubleshooting

### "cannot parse OID" / symbolic OID errors

This template uses **only numeric OIDs**. If you see this error in the Zabbix log:
```
snmp_parse_oid(): cannot parse OID "SNMPv2-SMI::enterprises..."
```
it means another template or manual item uses symbolic OIDs.  
On Debian/Ubuntu, `snmp-mibs-downloader` is disabled by default — symbolic OIDs fail
without MIB files. This template never uses symbolic OIDs, so it is portable.

### Items stuck in "Not supported"

- Verify the SNMPv3 credentials match the ETP4860 configuration.
- Check that the Zabbix server can reach the device on UDP/161.
- Run a manual test: `snmpwalk -v3 -l authPriv -u USER -a SHA-512 -A AUTHPASS -x AES -X PRIVPASS <IP> 1.3.6.1.4.1.2011.6.164.1.3.1`

### Battery string temperature always "no data"

The walk value `2147483647` (0x7FFFFFFF) is the firmware sentinel for "sensor not installed".
The template discards this value via an `IN_RANGE` preprocessing step, so the item will have
no data until a physical temperature sensor is installed on the battery string.

### LVD discovery finds no items

The walk for `{$LLD.INTERVAL}` (default 1h) may not have run yet.
Force a manual discovery: **Host → Discovery → Run now**.

---

## Screenshots

*(Add screenshots of dashboards, graphs, and problem views here)*

---

## Credits

- MIB source: Huawei `EMAP-MIB` (`emap_snmp.mib`) + `HUAWEI-MIB`
- Walk reference: ETP4860-B1A2 with iSitePower V100R022C00SPC121
- Author: Rafael Bastos

## License

[MIT](LICENSE)
