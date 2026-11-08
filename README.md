# Microsoft Fabric Real-Time Operations Monitoring

[![Microsoft Fabric](https://img.shields.io/badge/Microsoft_Fabric-Real--Time_Intelligence-742774)](#)
[![Eventhouse](https://img.shields.io/badge/Eventhouse-KQL_Database-0078D4)](#)
[![KQL](https://img.shields.io/badge/KQL-Operational_Telemetry-00A4EF)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A Microsoft Fabric portfolio project for monitoring equipment telemetry with Eventhouse, a KQL database, a Real-Time Dashboard, and an Activator rule. The solution demonstrates operational analytics, threshold-based alerting, native Fabric Git integration, and source-controlled Real-Time Intelligence artifacts.

> [!NOTE]
> This project uses a static 10-row sample telemetry dataset covering a 10-minute interval. It demonstrates Microsoft Fabric Real-Time Intelligence components and operational-monitoring logic; it is not a continuous production streaming implementation.

## Solution overview

The solution stores equipment telemetry in the `operations_events` table and uses KQL to:

- detect critical operating conditions
- compare equipment and facilities
- visualize temperature trends
- monitor operational status
- evaluate temperature thresholds with Activator

### Fabric items

| Item | Fabric type | Purpose |
| --- | --- | --- |
| `eh_real_time_operations` | Eventhouse | Hosts the Real-Time Intelligence data layer. |
| `eh_real_time_operations` | KQL Database | Stores the `operations_events` telemetry table. |
| `Real-Time Operations Dashboard` | Real-Time Dashboard | Presents four operational-monitoring visuals. |
| `Critical Equipment Temperature Activator` | Activator | Evaluates equipment-level temperature conditions and records activations. |

## Architecture

```text
Static sample telemetry
        |
        v
eh_real_time_operations Eventhouse
        |
        v
eh_real_time_operations KQL Database
        |
        +------------------------------+
        |                              |
        v                              v
Real-Time Operations Dashboard   Fabric Activator
4 KQL-based visuals              equipment-level rule
```

## Telemetry schema

The `operations_events` table contains 11 columns:

| Column | KQL type | Meaning |
| --- | --- | --- |
| `event_id` | `string` | Unique event identifier. |
| `event_time` | `datetime` | Event timestamp. |
| `facility_id` | `string` | Facility identifier. |
| `equipment_id` | `string` | Equipment identifier. |
| `event_type` | `string` | Event category. |
| `severity` | `string` | Severity classification. |
| `temperature_c` | `real` | Temperature in degrees Celsius. |
| `vibration_mm_s` | `real` | Vibration velocity in millimetres per second. |
| `pressure_bar` | `real` | Pressure in bar. |
| `energy_kw` | `real` | Power in kilowatts. |
| `status` | `string` | Operational status. |

### Verified sample profile

| Metric | Result |
| --- | ---: |
| Rows | 10 |
| Distinct event IDs | 10 |
| Duplicate event IDs | 0 |
| Facilities | 3 |
| Equipment identifiers | 9 |
| Event types | 3 |
| Severity levels | 3 |
| Status values | 4 |
| Missing required identifiers | 0 |
| Missing event timestamps | 0 |
| Rows with missing measurements | 0 |
| Rows outside broad technical validation ranges | 0 |
| Event-time range | 2026-11-06 18:00:00 to 2026-11-06 18:09:00 UTC |
| Maximum observed temperature | 96.4 °C |
| Observed vibration range | 1.8 to 10.1 mm/s |

Observed category values include:

- `event_type`: `Telemetry`, `Anomaly`, `Maintenance`
- `severity`: `Low`, `Medium`, `High`
- `status`: `Running`, `Warning`, `Critical`, `Maintenance`

## Real-Time Dashboard

The dashboard contains four visuals:

1. **Critical Events**: KPI for events classified as critical.
2. **Maximum Temperature by Equipment (°C)**: column chart comparing maximum equipment temperature.
3. **Average Temperature by Facility**: bar chart comparing facilities.
4. **Average Temperature Over Time**: time-series chart ordered by `event_time`.

The dashboard uses the `eh_real_time_operations` KQL database as its data source and applies a one-hour dashboard time range.

### Critical-event logic

The dashboard classification uses the following logic:

```kql
status == "Critical"
or temperature_c >= 85.0
or vibration_mm_s >= 8.0
```

The Activator rule is configured as an equipment-level temperature rule evaluated every five minutes. Its documented temperature condition is greater than `85 °C`.

> [!IMPORTANT]
> The dashboard uses `temperature_c >= 85.0`, while the Activator rule uses a strict `> 85` threshold. A reading of exactly `85.0 °C` is therefore critical on the dashboard but does not satisfy the Activator temperature condition. This difference is documented explicitly and should be changed only through the Microsoft Fabric user interface if a single business threshold is required.

## Activator behavior and capacity note

The Activator evaluates each `equipment_id` every five minutes. With nine equipment identifiers, this produces up to:

```text
12 evaluations per hour × 24 hours × 9 equipment identifiers = 2,592 evaluated rows per day
```

In the trial environment, Activator accounted for approximately 74% of Eventhouse query activity during observation. This is acceptable for a short portfolio demonstration but is disproportionate to a static 10-row dataset. Stop or disable the rule when continuous evaluation is no longer needed, especially on limited trial capacity.

## Native Fabric Git integration

The workspace is connected to the `main` branch of this repository. Microsoft Fabric generated and synchronized the following native artifact folders:

```text
microsoft-fabric-real-time-operations/
├── Critical Equipment Temperature Activator.Reflex/
├── Real-Time Operations Dashboard.KQLDashboard/
├── eh_real_time_operations.Eventhouse/
├── LICENSE
└── README.md
```

The KQL database is represented within the Eventhouse artifact rather than as a separate top-level repository folder.

### Source-control policy

- Fabric item definitions are generated and committed through native Microsoft Fabric Git integration.
- Fabric-generated JSON and platform metadata are not edited manually in GitHub.
- Functional changes to Eventhouse, the KQL database, Real-Time Dashboard, or Activator are made in Microsoft Fabric and then committed through the Fabric Source control panel.
- Manual repository content is limited to project documentation and licensing.

## Git restoration test

A clean-workspace restoration test was performed twice against the `main` branch.

### Verified behavior

- The source workspace synchronized all four Fabric items successfully.
- Native folders for Activator, Eventhouse, and Real-Time Dashboard were committed by Microsoft Fabric.
- The source dashboard remained functional and displayed all four visuals.
- Re-saving the dashboard in Fabric generated and committed a new native definition.

### Current limitation

Creation of the Real-Time Dashboard in a new empty workspace failed with:

```text
Error code: InvalidArtifactDefinitionFormat
Error message: Invalid payload in definition part
Affected artifact: Real-Time Operations Dashboard.KQLDashboard
```

The same error occurred after Fabric regenerated and recommitted `RealTimeDashboard.json`. The generated definition was not manually modified. Because the Git update was transactional, the target workspace remained empty after the failed operation.

**Restoration status:**

| Scope | Result |
| --- | --- |
| Source workspace functionality | PASS |
| Native Fabric-to-Git synchronization | PASS |
| Eventhouse, KQL Database, Dashboard, and Activator status in source workspace | PASS, all synchronized |
| Full restoration into a clean workspace | FAIL, blocked by Fabric KQL Dashboard definition validation |
| Manual modification of Fabric-generated definitions | Not used |

This repository must not be presented as fully recoverable through Fabric Git until Microsoft Fabric accepts the native KQL Dashboard definition in a clean workspace.

## Validation checklist

### Data layer

- [x] `operations_events` has 11 correctly typed columns.
- [x] All 10 sample events have unique `event_id` values.
- [x] Required identifiers and timestamps are populated.
- [x] Telemetry measurements are populated.
- [x] Measurement columns use the KQL `real` type.
- [x] Event timestamps use the KQL `datetime` type.

### Dashboard

- [x] All four visuals render in the source workspace.
- [x] Dashboard queries run against `eh_real_time_operations`.
- [x] Temperature units are shown in the equipment visual title.
- [x] Critical-event logic is documented.
- [ ] Dashboard creation from Git succeeds in a clean workspace. Currently blocked by a Fabric definition-validation error.

### Activator

- [x] Rule is grouped by `equipment_id`.
- [x] Rule is evaluated every five minutes.
- [x] Three test activations were observed.
- [x] Capacity impact was reviewed.
- [ ] Dashboard and Activator boundary semantics are identical. They currently differ at exactly `85.0 °C`.

## Reproduction guidance

### Preferred path

Use native Microsoft Fabric Git integration:

1. Create an empty Fabric workspace on a supported capacity.
2. Open **Workspace settings** and select **Git integration**.
3. Select **GitHub**.
4. Connect to `kamilsoszka/microsoft-fabric-real-time-operations`.
5. Select the `main` branch.
6. Leave the Git folder empty because artifacts are stored at the repository root.
7. Select **Connect and sync**.
8. Review the result in the workspace and Source control panel.

> [!WARNING]
> At the time of the documented test, the update failed while creating `Real-Time Operations Dashboard` with `InvalidArtifactDefinitionFormat`. Do not manually edit the Fabric-generated definition merely to bypass this limitation. Retest after relevant Microsoft Fabric updates.

### Manual reconstruction fallback

If the Git limitation persists, the source workspace remains the authoritative working implementation. A manual reconstruction should follow the documented architecture, schema, dashboard visuals, KQL logic, and Activator settings. Manual reconstruction is a fallback, not proof that native Git disaster recovery works.

## Security and operational considerations

- The sample data is synthetic demonstration telemetry and contains no customer or production information.
- `Pass-through identity` is used by the dashboard data-source connection.
- Do not commit secrets, tokens, credentials, or private connection information.
- Validate workspace roles and GitHub permissions before connecting a new workspace.
- Monitor Activator query usage on trial or constrained Fabric capacities.
- Disable continuous rule evaluation when the demonstration is not actively needed.

## AI-assisted development disclosure

Developed by Kamil Soszka with Microsoft Copilot used as the sole AI-assisted development partner for architecture review, code refinement, documentation, validation, and troubleshooting. All environment operations, implementation decisions, testing, and final acceptance were performed by the project owner.

## Skills demonstrated

- Microsoft Fabric Real-Time Intelligence
- Eventhouse and KQL Database
- Kusto Query Language (KQL)
- Real-Time Dashboard design
- Fabric Activator rules
- operational telemetry modeling
- data-quality validation
- native Fabric Git integration
- reproducibility testing
- technical documentation
- capacity-awareness and troubleshooting

## Known limitations

- The dataset is static and covers only ten minutes.
- The project does not include a continuous Eventstream or external telemetry broker.
- Activator produces repeated evaluations against unchanged sample data.
- Dashboard and Activator differ at the exact `85.0 °C` boundary.
- Full clean-workspace restoration is currently blocked by a native Fabric KQL Dashboard definition error.
- Service-side identities, permissions, and runtime configuration must be validated separately in each workspace.

## License

This project is licensed under the [MIT License](LICENSE).
