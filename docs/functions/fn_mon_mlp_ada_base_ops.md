# fn_mon_mlp_ada_base_ops

## Nombre de la función
`fn_mon_mlp_ada_base_ops`

## Objetivo
Centralizar señales complementarias ADA para KPI, Alarmas y Front.

## Tipo de función
**estado actual**.

## Workspace objetivo
`ams-uat-dataplatform-laws`

## Contrato de salida esperado
- `KPIs`
- `Alarmas`
- `Front`

## Resource a usar desde Grafana
- `/subscriptions/0d996eb2-802f-4ef8-8ae6-d385c74da7e6/resourceGroups/ams-uat-dataplatform-rg/providers/Microsoft.OperationalInsights/workspaces/ams-uat-dataplatform-laws`

## Query de validación
```kql
fn_mon_mlp_ada_base_ops()
| take 1
```

## KQL propuesta
```kql
// fn_mon_mlp_ada_base_ops
// Base reutilizable ADA para dominios KPI/Alarmas/Front.
let endQuery = bin(now(), 1m);
let startQuery = endQuery - 2d;
let statusKpisAlarms =
    ContainerAppSystemLogs_CL
    | where TimeGenerated >= startQuery and TimeGenerated < endQuery
    | where JobName_s startswith "mlp-prd-caj-ada-job"
    | where Log_s contains "has successfully completed"
    | summarize last_exec=max(TimeGenerated) by JobName_s
    | extend KPIs = iff(datetime_diff('minute', now(), last_exec) > 35, "ALERT", "OK"),
             Alarmas = iff(datetime_diff('minute', now(), last_exec) > 35, "ALERT", "OK");
let frontErrors =
    AppServiceConsoleLogs
    | where TimeGenerated >= endQuery - 10m and TimeGenerated < endQuery
    | where Level == "Error"
    | summarize err_count = count()
    | extend Front = iff(err_count > 2, "ALERT", "OK");
statusKpisAlarms
| summarize KPIs = iff(countif(KPIs == "ALERT") > 0, "ALERT", "OK"),
            Alarmas = iff(countif(Alarmas == "ALERT") > 0, "ALERT", "OK")
| join kind=leftouter (frontErrors) on 1==1
| extend Front = coalesce(Front, "OK")
| project KPIs, Alarmas, Front
```
