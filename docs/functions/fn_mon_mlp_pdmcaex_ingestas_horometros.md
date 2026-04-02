# fn_mon_mlp_pdmcaex_ingestas_horometros

## Nombre de la función
`fn_mon_mlp_pdmcaex_ingestas_horometros`

## Objetivo
Entregar señal ingestas para MLP PDM CAEX consumible por el dashboard cross vía var_mlp_pdmcaex_horometros.

## Tipo de función
**estado actual**.

## Workspace objetivo
`ams-uat-dataplatform-laws`

## Variable(s) de Grafana que alimenta
- `var_mlp_pdmcaex_horometros`

## Contrato de salida esperado
La función debe retornar 1 fila con:
- `status`
- `color`
- `evidence`
- `last_update_utc`

## Resource a usar desde Grafana
**Resource principal:** `ams-uat-dataplatform-laws`.

Referencia esperada:
`/subscriptions/0d996eb2-802f-4ef8-8ae6-d385c74da7e6/resourceGroups/ams-uat-dataplatform-rg/providers/Microsoft.OperationalInsights/workspaces/ams-uat-dataplatform-laws`

## Query wrapper en Grafana
```kql
fn_mon_mlp_pdmcaex_ingestas_horometros()
| project color
| take 1
```

## Query de validación
```kql
fn_mon_mlp_pdmcaex_ingestas_horometros()
| project status, color, evidence, last_update_utc
| take 1
```

## Fuente probable
Azure Monitor Logs / Log Analytics

## Tabla(s) probable(s)
SparkLoggingEvent_CL

## Regla operacional esperada
- Evaluar la señal más reciente por fuente/tabla relevante de MLP PDM CAEX.
- `status` es la señal principal para chip/card.
- `color` deriva de `status` (`ALERT` rojo, `OK` verde, `WARN` reservado).
- Si no hay timestamp usable, marcar condición de riesgo en `evidence` y elevar a `ALERT`.

## Supuestos
1. Las señales de MLP PDM CAEX están disponibles en el workspace `ams-uat-dataplatform-laws`.
2. Cuando existe `payload.ultimo_timestamp`, representa tiempo de dato y puede convertirse a UTC.
3. `TimeGenerated` funciona como fallback de estado actual.

## Limitaciones
1. Requiere validar con datos reales los filtros exactos por componente y los umbrales operativos.
2. Algunos dominios pueden requerir tablas adicionales no visibles en la variable actual.
3. Esta propuesta evita función histórica; si se necesita tendencia, se debe crear función separada.

## KQL propuesta (lista para pegar en Logs y guardar manualmente como función)
```kql
// fn_mon_mlp_pdmcaex_ingestas_horometros
// Tipo: estado actual. Diseñada para ejecución en estado actual (sin $__timeFrom/$__timeTo).

let source_data =
    union isfuzzy=true
        (SparkLoggingEvent_CL | extend _table_name="SparkLoggingEvent_CL"),
        (ContainerAppSystemLogs_CL | extend _table_name="ContainerAppSystemLogs_CL"),
        (ContainerAppConsoleLogs_CL | extend _table_name="ContainerAppConsoleLogs_CL"),
        (AzureDiagnostics | extend _table_name="AzureDiagnostics")
    | where TimeGenerated >= ago(48h)
    // Ajustar filtro de dominio/componente para MLP PDM CAEX - ingestas
    | where tostring(*) has_any ("MLP", "PDM_CAEX", "SIRO", "NASH", "ADA")
    | summarize arg_max(TimeGenerated, *) by _table_name;

let evaluated =
    source_data
    | extend event_ts_utc = TimeGenerated
    | extend payload = parse_json(tostring(column_ifexists("Message", "")))
    | extend payload_ts_raw = tostring(payload.ultimo_timestamp)
    | extend payload_ts_utc = datetime_local_to_utc(todatetime(payload_ts_raw), "America/Santiago")
    | extend reference_ts_utc = coalesce(payload_ts_utc, event_ts_utc)
    | extend ts_origin = iff(isnotnull(payload_ts_utc), "payload.ultimo_timestamp", "TimeGenerated")
    | extend alertar_raw = tostring(payload.alertar)
    | extend alertar_norm = replace_string(tolower(trim(' "\t\r\n', alertar_raw)), "í", "i")
    | extend alertar_flag = alertar_norm in ("si", "true", "1", "yes", "alert", "alerta")
    | extend diff_min_signed = datetime_diff('minute', now(), reference_ts_utc)
    | extend delay_min = iff(diff_min_signed < 0, 0, diff_min_signed)
    | extend status_source = case(
        isnull(reference_ts_utc), "ALERT",
        alertar_flag, "ALERT",
        delay_min > 120, "ALERT",   // umbral inicial conservador; ajustar con regla de negocio
        "OK"
    )
    | extend evidence_source = strcat(_table_name, "{status=", status_source, ",ts_origin=", ts_origin, ",delay_min=", tostring(delay_min), "}");

evaluated
| summarize
    has_alert = max(iif(status_source == "ALERT", 1, 0)),
    evidence_alert = strcat_array(make_list_if(evidence_source, status_source == "ALERT"), " | "),
    evidence_all = strcat_array(make_list(evidence_source), " | "),
    last_update_utc = max(event_ts_utc)
| extend status = iff(has_alert == 1, "ALERT", "OK")
| extend color = case(status == "ALERT", "#E53935", status == "WARN", "#FFF4CC", "#EAF4EA")
| extend evidence = iff(status == "ALERT", evidence_alert, strcat("OK:", evidence_all))
| project status, color, evidence, last_update_utc
```
