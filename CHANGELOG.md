# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.0.0] - 2026-06-22

### Added
- Initial release of the Zabbix 7.0 template for Huawei ETP4860-B1A2
- SNMPv3 authPriv support (SHA-512 / AES-256) with macro-based credential management
- 22 scalar items covering system, rectifier group, battery group, AC/DC bus metrics
- 7 Low-Level Discovery rules:
  - Rectifier modules (hwRectConfigTable / hwRectOperTable, indices 28672–30719)
  - Battery strings (hwBattStringConfigTable / hwBattStringOperTable)
  - AC distribution inputs (hwAcInputTable)
  - DC distribution outputs (hwDcOutputTable)
  - LVD branches (hwLvdBranchTable)
  - Environment temperature sensors (hwEnvTempSensorTable)
  - Digital inputs / Spare digitals (hwSpareDigitalTable)
- 34 item prototypes with correct scale factors (÷10 for voltages/currents/temperatures)
- 16 trigger prototypes with severity-appropriate thresholds and recovery expressions
- 9 template-level triggers
- 8 value maps covering all status enumerations (exact MIB enum values)
- All OIDs in numeric form — no MIB files required
- All thresholds via macros (no hardcoded values in triggers)
- IN_RANGE preprocessing to discard firmware sentinel value 2147483647
- Trigger dependency: SOC low suppressed when battery is already known to be discharging
- Tested against firmware: iSitePower V100R022C00SPC121
