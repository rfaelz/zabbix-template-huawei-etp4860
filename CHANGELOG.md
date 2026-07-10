# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.2.1] - 2026-07-10

### Fixed
- Remove `dependencies` block from trigger prototype "Rectifier: In protection or off": the
  Zabbix 7.0 YAML importer fails with "depends on a trigger that does not exist" when a trigger
  prototype references a regular template trigger, even though the trigger already exists in the
  database. Dependency works correctly when set via the UI. Added note to trigger description
  and README instructing users to configure this (and the SOC→discharging dependency) manually
  after import.

## [1.2.0] - 2026-07-10

### Changed
- **LLD macros**: all 7 discovery rules now use device-assigned name macros (`{#RECTNAME}`,
  `{#BATTNAME}`, `{#ACNAME}`, `{#DCNAME}`, `{#LVDNAME}`, `{#TEMPNAME}`, `{#DINAME}`) instead
  of the generic `{#IDX}`. The OID column used for discovery changed from `.1` (index) to `.3`
  (name). `{#SNMPINDEX}` is preserved in all `snmp_oid`, `key`, and expression fields.
- **English names**: all item names, item prototype names, trigger names, trigger prototype names,
  discovery rule names and value map entries translated from Portuguese to English. Descriptions
  were already in English and remain unchanged.
- **Template internal description**: updated to reflect SNMP v1/v2c/v3 compatibility, LLD name
  macros, sentinel handling, and Claude Code attribution.
- **README**: rewritten — removed host-configuration, import-instructions, troubleshooting and
  screenshots sections; updated About section, macro table, trigger overview and dependencies.
  Added Claude Code credit.

## [1.1.0] - 2026-07-10

### Fixed
- **Sentinela 2147483647 (INT32_MAX)**: substituído IN_RANGE/DISCARD_VALUE por step JAVASCRIPT
  (`return value==2147483647?0:value;`) em 14 itens de tensão/corrente/potência/uso-de-carga.
  Durante queda de CA, o iSitePower retorna esse sentinela; agora é convertido a 0 em vez de
  descartar o sample e gerar gap nos gráficos.
  Itens afetados: `rect.group.current`, `rect.group.dc.power`, `rect.group.ac.power`,
  `rect.output.current`, `rect.output.voltage`, `rect.dc.power`, `rect.ac.voltage`,
  `ac.volt.a/b/c`, `ac.curr.a/b/c`, `dc.load.usage`.
  Mantidos com IN_RANGE (sentinela = inválido real): `rect.temperature`, `batt.string.temperature`.
- **Trigger backup bateria**: expressão corrigida para `<=` (firmware reporta horas inteiras,
  `< 1` nunca dispararia) e gateada em `batt.charge.status=3` para só alarmar durante descarga real.
  Recovery também ajustado (`>` + `<>3`).

### Added
- **Dependência de trigger**: "Retificador: Em proteção ou desligado" agora depende de
  "Bateria em descarga" — suprime alarmes individuais de retificador durante queda de CA,
  quando todos desligam ao mesmo tempo (comportamento normal de proteção).

### Removed
- Item escalar `acs.total.current` (hwAcsTotalCurrent): sempre retorna 0 neste hardware.
- Item escalar `rect.group.load.usage` (hwRectsLoadUsage): sempre retorna 0 neste hardware.

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
