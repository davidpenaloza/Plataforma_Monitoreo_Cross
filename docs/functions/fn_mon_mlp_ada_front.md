# fn_mon_mlp_ada_front

## Nombre de la función
`fn_mon_mlp_ada_front`

## Objetivo
Alias histórico de front ADA (mantener compatibilidad).

## Tipo de función
**estado actual**.

## Workspace objetivo
`ams-uat-dataplatform-laws`

## Variable(s) de Grafana que alimenta
- `var_mlp_ada_front`

## Contrato de salida esperado
- `status`
- `color`
- `evidence`
- `last_update_utc`

## Resource a usar desde Grafana
- `ams-uat-dataplatform-laws`
- `/subscriptions/0d996eb2-802f-4ef8-8ae6-d385c74da7e6/resourceGroups/ams-uat-dataplatform-rg/providers/Microsoft.OperationalInsights/workspaces/ams-uat-dataplatform-laws`

## Query wrapper en Grafana
```kql
fn_mon_mlp_ada_front()
| project color
| take 1
```

## Query de validación
```kql
fn_mon_mlp_ada_front()
| project status, color, evidence, last_update_utc
| take 1
```

## Fuente probable
Azure Monitor Logs / Log Analytics.

## Tabla probable
- `ContainerAppSystemLogs_CL`
- `AzureDiagnostics`
- `ContainerAppConsoleLogs_CL`
- `AppServiceConsoleLogs` (solo si aplica front)

## Regla operacional esperada
No recalcular bloque ADA completo en función pública; consumir funciones base y derivar señal puntual.

## Supuestos
1. Existen funciones base `fn_mon_mlp_ada_base_statusfuentes()` y `fn_mon_mlp_ada_base_ops()` en LAW.
2. Las funciones base se mantienen como única fuente de reglas compartidas.

## Limitaciones
1. Si las funciones base cambian columnas, las públicas deben ajustarse.
2. `KPIs/Alarmas/Front` dependen de calidad de logs de dominio.

## KQL propuesta (función pública simplificada)
```kql
let base = fn_mon_mlp_ada_base_statusfuentes();
let ops = fn_mon_mlp_ada_base_ops();
base
| join kind=leftouter (ops) on 1==1
| extend status = Front
| extend color = case(status == "ALERT", "#E53935", "#EAF4EA")
| extend evidence = strcat("fn=fn_mon_mlp_ada_front; rule=Front")
| extend last_update_utc = now()
| project status, color, evidence, last_update_utc
```
