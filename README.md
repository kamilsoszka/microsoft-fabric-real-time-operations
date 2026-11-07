# Microsoft Fabric Real-Time Operations Monitoring

A portfolio project that demonstrates operational telemetry monitoring in Microsoft Fabric using Eventhouse, a KQL database, a Real-Time Dashboard, and an Activator alert rule.

## Solution overview

The solution stores equipment telemetry in the `operations_events` table and uses KQL to detect critical operating conditions, compare facilities and equipment, and visualize temperature trends.

### Components

- Microsoft Fabric Eventhouse
- KQL database: `eh_real_time_operations`
- Table: `operations_events`
- Real-Time Dashboard with four visuals
- Activator rule: `Critical Equipment Temperature Rule`

## Dashboard

The dashboard contains:

1. Critical Events KPI
2. Maximum Temperature by Equipment
3. Average Temperature by Facility
4. Average Temperature Over Time

## Alert rule

The alert evaluates each `equipment_id` every 5 minutes and activates when `max_temperature_c > 85`.

The test dataset produced three activations for `MOTOR-201`, `MOTOR-202`, and `COMP-303`.

## Repository structure

```text
.
├── Critical Equipment Temperature Activator.Reflex/
├── Real-Time Operations Dashboard.KQLDashboard/
├── eh_real_time_operations.Eventhouse/
├── LICENSE
└── README.md
```

## Reproduce the project

1. Create an Eventhouse and a KQL database in Microsoft Fabric.
2. Run the setup KQL script to create the table and ingest the sample events.
3. Run the validation KQL script to verify the data and critical-event logic.
4. Use the dashboard KQL script to create the four dashboard visuals.
5. Create an Activator rule grouped by `equipment_id` with `max_temperature_c > 85`, evaluated every 5 minutes.

## Validation results

- 10 ingested telemetry events
- 9 unique equipment identifiers
- 3 facilities
- 3 critical alert activations
- Active alert status confirmed as Running

## Skills demonstrated

Microsoft Fabric, Real-Time Intelligence, Eventhouse, KQL, operational monitoring, dashboard design, threshold-based alerting, and technical documentation.

## Challenges and Lessons Learned

- **Activator threshold tuning:** Configuring the temperature rule required balancing sensitivity to avoid false positives while ensuring critical events were captured.
- **KQL windowing functions:** Designing the real-time aggregations required careful handling of time windows to accurately reflect equipment states.
- **Git integration workflow:** Managing Fabric artifacts in source control required understanding how workspace item schemas map to repository folders.

## License

This project is licensed under the [MIT License](LICENSE).
