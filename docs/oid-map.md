# OID Map — Huawei ETP4860-B1A2 (hwSiteMonitorMIB)

**OID root:** `1.3.6.1.4.1.2011.6.164`  
**MIB:** `EMAP-MIB` → `hwSiteMonitorMIB` = `{ enterprises(2011) huaweiUtility(6) 164 }`  
**Objects base:** `1.3.6.1.4.1.2011.6.164.1` (`hwSiteMonitorMIBObjects`)

All OIDs are in numeric form. Scale column shows the factor applied to convert the raw SNMP
integer to the real-world unit (e.g. `÷10` means multiply by 0.1 in Zabbix preprocessing).  
`2147483647 (0x7FFFFFFF)` is the firmware sentinel for "invalid/not configured" — items with
this possible value have an `IN_RANGE` preprocessing step to discard it.

---

## Group 1 — Site Info (Scalars)

| OID (numeric) | MIB name | Type | Unit | Scale | Notes |
|---|---|---|---|---|---|
| `…1.1.1.2.0` | hwSiteName | STRING | — | — | Site name |
| `…1.1.1.10.0` | hwSiteRunningStatus | INTEGER | — | — | Enum: `1`=normal, `2`=alarm |
| `…1.1.2.3.0` | hwAlarmQuantity | Gauge32 | — | ×1 | Active alarm count |

---

## Group 2 — Monitor (Scalars, single monitor index=1)

| OID (numeric) | MIB name | Type | Unit | Scale | Notes |
|---|---|---|---|---|---|
| `…1.2.1.3.0` | hwMonsQuantity | Gauge32 | pcs | ×1 | — |
| `…1.2.2.1.1.5.1` | hwMonEquipSoftwareVersion | STRING | — | — | Firmware version |
| `…1.2.2.1.1.99.1` | hwMonitorOperStatus | INTEGER | — | — | Enum: 1=normal, 2=commRs485Fail, 3=commNetFail, 4=fault, 254=alarmResume, 255=unknown |

---

## Group 3 — Rectifier Group (Scalars)

| OID (numeric) | MIB name | Type | Unit | Scale | Walk value | Real value |
|---|---|---|---|---|---|---|
| `…1.3.1.3.0` | hwRectsTotalCurrent | Gauge32 | A | ÷10 | 209 | 20.9 A |
| `…1.3.1.4.0` | hwRectsTotalQuantity | Gauge32 | pcs | ×1 | 3 | 3 |
| `…1.3.1.5.0` | hwRectsRatedVoltage | Gauge32 | V | ÷10 | 535 | 53.5 V |
| `…1.3.1.9.0` | hwRectsTotalDCPower | Gauge32 | W | ×1 | 1091 | 1091 W |
| `…1.3.1.10.0` | hwRectsLoadUsage | Gauge32 | % | ×1 | 0 | 0 % |
| `…1.3.1.11.0` | hwRectsTotalACInputPower | Gauge32 | W | ×1 | 1152 | 1152 W |
| `…1.3.1.14.0` | hwRectsOverVoltProtThres | Gauge32 | V | ÷10 | 595 | 59.5 V |

---

## Group 3 — Rectifier Modules (LLD Table)

**Discovery OID:** `1.3.6.1.4.1.2011.6.164.1.3.2.1.1.1` (hwRectConfigIndex column)  
**Index range:** 28672–30719 (observed: 28672, 28673, 28674)

### Config table (`.1.3.2.1.1.*`)

| Column | OID suffix | MIB name | Type | Unit | Scale |
|---|---|---|---|---|---|
| 1 | `.1.{idx}` | hwRectConfigIndex | Integer32 | — | ×1 |
| 3 | `.3.{idx}` | hwRectEquipName | STRING | — | — |
| 5 | `.5.{idx}` | hwRectSoftwareVersion | STRING | — | — |
| 9 | `.9.{idx}` | hwRectRatedCurrent | Gauge32 | A | ÷10 |
| 10 | `.10.{idx}` | hwRectEfficency | Gauge32 | % | ×1 * |

> *\* MIB SYNTAX says 0–1000 (÷10), but based on observed value 95 and physical context,*  
> *this template treats the raw value as integer percent (95 = 95%). See issue discussion in repo.*

### Oper table (`.1.3.2.2.1.*`)

| Column | OID suffix | MIB name | Type | Unit | Scale | Walk example |
|---|---|---|---|---|---|---|
| 2 | `.2.{idx}` | hwRectOutputCurrent | Gauge32 | A | ÷10 | 76 → 7.6 A |
| 3 | `.3.{idx}` | hwRectOutputVoltage | Gauge32 | V | ÷10 | 520 → 52.0 V |
| 5 | `.5.{idx}` | hwRectDCOutputPower | Gauge32 | W | ×1 | 401 W |
| 6 | `.6.{idx}` | hwRectACVoltage | Gauge32 | V | ÷10 | 2288 → 228.8 V |
| 7 | `.7.{idx}` | hwRectRatedEfficiency | Gauge32 | % | ×1 | 96 % |
| 8 | `.8.{idx}` | hwRectifierTemperature | Integer32 | °C | ÷10 | 310 → 31.0 °C |
| 99 | `.99.{idx}` | hwRectOperStatus | INTEGER | — | — | Enum (see value map) |

**hwRectOperStatus enum:** `1`=normal, `2`=fault, `3`=protect, `4`=commFail, `5`=switchOff,
`6`=invalid, `7`=noConfig, `8`=acOff, `9`=rectLost, `254`=alarmResume, `255`=unknown

---

## Group 4 — Battery Group (Scalars)

| OID (numeric) | MIB name | Type | Unit | Scale | Walk value | Real value |
|---|---|---|---|---|---|---|
| `…1.4.1.3.0` | hwBattsTotalQuantity | Gauge32 | strings | ×1 | 1 | 1 |
| `…1.4.1.8.0` | hwSetBattsFloatVoltage | Gauge32 | V | ÷10 | 535 | 53.5 V |
| `…1.4.1.9.0` | hwSetBattsBoostVoltage | Gauge32 | V | ÷10 | 564 | 56.4 V |
| `…1.4.1.13.0` | hwSetBattsRatedCapacity | Gauge32 | Ah | ×1 | 180 | 180 Ah |
| `…1.4.2.1.0` | hwBattsTotalCurrent | Integer32 | A | ÷10 | 0 | 0.0 A |
| `…1.4.2.2.0` | hwBattsPreDischargeTime | Gauge32 | min | ×1 | 8 | 8 min |
| `…1.4.2.3.0` | hwBattsChargeStatus | INTEGER | — | — | 4 | hibernating |
| `…1.4.2.9.0` | hwBattsRemainCapacity | Gauge32 | Ah | ×1 | 180 | 180 Ah |
| `…1.4.2.10.0` | hwTotalRemainingCapacityPercent | Gauge32 | % | ×1 | 100 | 100 % |

**hwBattsChargeStatus enum:** `1`=floatCharge, `2`=boostCharge, `3`=disCharge, `4`=hibernating *(normal)*, `7`=offline, `255`=unknown

---

## Group 4 — Battery Strings (LLD Table)

**Discovery OID:** `1.3.6.1.4.1.2011.6.164.1.4.4.1.1.1`  
**Index:** Integer 1–100 (observed: 5; indices are non-sequential)

### Config table (`.1.4.4.1.1.*`)

| Column | OID suffix | MIB name | Unit | Scale |
|---|---|---|---|---|
| 3 | `.3.{idx}` | hwBattStringEquipName | — | — |
| 8 | `.8.{idx}` | hwSetBattStdCapacity | Ah | ×1 |

### Oper table (`.1.4.4.2.1.*`)

| Column | OID suffix | MIB name | Unit | Scale | Walk example |
|---|---|---|---|---|---|
| 2 | `.2.{idx}` | hwBattStringCurrent | A | ÷10 | 0 → 0.0 A |
| 3 | `.3.{idx}` | hwBattStringRemainCapacityPercent | % | ÷10 | 1000 → 100.0 % |
| 4 | `.4.{idx}` | hwBattStringRemainCapacity | Ah | ×1 | 180 Ah |
| 6 | `.6.{idx}` | hwBattStringTemprature | °C | ÷10 | 2147483647=invalid |
| 7 | `.7.{idx}` | hwBattStringMidpointVoltage | V | ÷10 | 0 → 0.0 V |
| 99 | `.99.{idx}` | hwBattStringOperStatus | — | — | Enum (see value map) |

**hwBattStringOperStatus enum:** `1`=normal, `2`=tempHigh, `3`=tempLow, `4`=loopBreak, `5`=overCurrent, `6`=unbalance, `7`=discharge, `8`=fuseBreak, `254`=alarmResume, `255`=unknown

---

## Group 5 — AC Distribution (Scalars + LLD)

**Group scalars:**

| OID | MIB name | Unit | Scale |
|---|---|---|---|
| `…1.5.1.3.0` | hwAcsTotalCurrent | A | ÷10 |
| `…1.5.1.4.0` | hwAcsInputQuantity | pcs | ×1 |

**Discovery OID:** `1.3.6.1.4.1.2011.6.164.1.5.2.1.1.1` (index 1 observed: "AC Distribution")

| Column | OID suffix | MIB name | Unit | Scale | Walk (idx 1) |
|---|---|---|---|---|---|
| 4 | `.4.{idx}` | hwApOrAblVoltage | V | ÷10 | 2287 → 228.7 V |
| 5 | `.5.{idx}` | hwBpOrBclVoltage | V | ÷10 | 2285 → 228.5 V |
| 6 | `.6.{idx}` | hwCpOrCalVoltage | V | ÷10 | 2288 → 228.8 V |
| 7 | `.7.{idx}` | hwAphaseCurrent | A | ÷10 | 17 → 1.7 A |
| 8 | `.8.{idx}` | hwBphaseCurrent | A | ÷10 | 17 → 1.7 A |
| 9 | `.9.{idx}` | hwCphaseCurrent | A | ÷10 | 16 → 1.6 A |
| 99 | `.99.{idx}` | hwAcInputOperStatus | — | — | `1`=normal, `2`=acOff, `3`=acAbsent, `4`=voltHigh, `5`=voltLow, `6`=freqHigh, `7`=freqLow, `254`=alarmResume, `255`=unknown |

---

## Group 6 — DC Distribution (Scalars + LLD)

**Group scalars:**

| OID | MIB name | Unit | Scale | Walk value |
|---|---|---|---|---|
| `…1.6.1.3.0` | hwDcsTotalVoltage | V | ÷10 | 520 → 52.0 V |
| `…1.6.1.4.0` | hwDcsTotalCurrent | A | ÷10 | 205 → 20.5 A |
| `…1.6.1.5.0` | hwDcsTotalPower | W | ×1 | 1064 W |
| `…1.6.1.9.0` | hwDcsPerfoLoadConsume | kWh | ×1 | 17652 kWh |

**Discovery OID:** `1.3.6.1.4.1.2011.6.164.1.6.2.1.1.1` (index 1 observed: "DC Distribution")

| Column | OID suffix | MIB name | Unit | Scale |
|---|---|---|---|---|
| 4 | `.4.{idx}` | hwDcOutputVoltage | V | ÷10 |
| 5 | `.5.{idx}` | hwDcOutputCurrent | A | ÷10 |
| 6 | `.6.{idx}` | hwDcOutputPower | W | ×1 |
| 8 | `.8.{idx}` | hwLoadUsage | % | ×1 |
| 99 | `.99.{idx}` | hwDcOutputOperStatus | — | — |

**hwDcOutputOperStatus enum:** `1`=normal, `2`=voltHigh, `3`=voltLow, `4`=dcAbsent, `5`=overload, `254`=alarmResume, `255`=unknown

---

## Group 7 — LVD Units (LLD)

**Discovery OID:** `1.3.6.1.4.1.2011.6.164.1.7.2.1.1.1` (indices 1=LLVD1, 3=BLVD observed)

| Column | OID suffix | MIB name | Unit | Scale | Walk (idx 1) |
|---|---|---|---|---|---|
| 3 | `.3.{idx}` | hwLvdUnitEquipName | — | — | "LLVD1" |
| 5 | `.5.{idx}` | hwSetLvdVoltage | V | ÷10 | 440 → 44.0 V |
| 8 | `.8.{idx}` | hwSetLvdReconnectedVoltage | V | ÷10 | 515 → 51.5 V |
| 99 | `.99.{idx}` | hwLvdBranchOperStatus | — | — | `1`=normal, `2`=lowVoltOff, `3`=backgroundOff, `4`=localManualOff, `5`=lowTempOff, `6`=highTempOff, `7`=timeOff, `8`=capacityOff, `9`=forceOff, `254`=alarmResume, `255`=unknown |

---

## Group 8 — Environment (LLD)

**Temp sensors discovery OID:** `1.3.6.1.4.1.2011.6.164.1.8.2.1.1.1` (index 1 observed; sensor not installed: value=2147483647)

| Column | OID suffix | MIB name | Unit | Scale |
|---|---|---|---|---|
| 3 | `.3.{idx}` | hwEnvTempEquipName | — | — |
| 4 | `.4.{idx}` | hwEnvTemperature | °C | ×1 |

**Digital inputs discovery OID:** `1.3.6.1.4.1.2011.6.164.1.8.2.3.1.1` (indices 1–5: DI1–DI5 observed)

| Column | OID suffix | MIB name | Unit | Notes |
|---|---|---|---|---|
| 3 | `.3.{idx}` | hwSpareDigitalEquipName | — | "DI1"…"DI5" |
| 99 | `.99.{idx}` | hwSpareDigitalOperStatus | — | 1=normal |

---

## Standard MIB-2 Items

| OID | MIB name | Unit | Scale |
|---|---|---|---|
| `1.3.6.1.2.1.1.3.0` | sysUpTime | s | ÷100 (TimeTicks) |
