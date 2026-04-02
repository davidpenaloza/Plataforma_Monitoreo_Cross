# fn_mon_mlp_pdmcaex_global

## Nombre de la función
`fn_mon_mlp_pdmcaex_global`

## Objetivo
Entregar el estado global ejecutivo de MLP PDM CAEX para el dashboard cross, consolidando señales de `HOROMETROS`, `MINECARE` y `DISPATCH`.

## Tipo de función
**Estado actual** (no histórica, no depende de rango de Grafana).

## Workspace objetivo
`ams-uat-dataplatform-laws`

## Variable(s) de Grafana que alimenta
- `var_mlp_pdmcaex_global`

## Contrato de salida esperado
La función debe retornar 1 fila con:
- `status`
- `color`
- `evidence`
- `last_update_utc`

## Resource que debe usarse desde Grafana
**Resource principal obligatorio:** `ams-uat-dataplatform-laws`.

Referencia típica (resourceId):
`/subscriptions/0d996eb2-802f-4ef8-8ae6-d385c74da7e6/resourceGroups/ams-uat-dataplatform-rg/providers/Microsoft.OperationalInsights/workspaces/ams-uat-dataplatform-laws`

## Query wrapper en Grafana
```kql
fn_mon_mlp_pdmcaex_global()
| project color
| take 1
```

## Query de validación
```kql
fn_mon_mlp_pdmcaex_global()
| project status, color, evidence, last_update_utc
| take 1
```

## Fuente probable
Azure Monitor Logs / Log Analytics.

## Tabla(s) probable(s)
- `SparkLoggingEvent_CL`

## Regla operacional esperada
- Evaluar última señal por fuente (`HOROMETROS`, `MINECARE`, `DISPATCH`).
- `ALERT` por fuente si:
  - `alertar` normalizado indica alerta, o
  - atraso (`delay_min`) supera `umbral_min`, o
  - falta timestamp usable.
- Estado global:
  - `ALERT` si al menos una fuente está en `ALERT`.
  - `OK` en caso contrario.
- Color:
  - `ALERT -> #E53935`
  - `OK -> #EAF4EA`
  - `WARN` reservado para futura regla.

## Supuestos
1. `payload.ultimo_timestamp` viene en horario local Chile cuando existe.
2. `TimeGenerated` es fallback válido para estado actual.
3. Si no hay timestamp usable por fuente, debe tratarse como riesgo operativo.

## Limitaciones
1. Dependencia de formato JSON consistente en `Message`.
2. Valores de `payload.alertar` pueden variar entre fuentes/versiones.
3. Umbrales (`10080`, `40`, `70`) deben validarse con SLA vigente.

## KQL actual o propuesta
> Propuesta lista para pegar en Logs y guardar como función (Save as function).

```kql
// fn_mon_mlp_pdmcaex_global
// Tipo: estado actual (no usar $__timeFrom/$__timeTo)

let thresholds = datatable(fuente:string, umbral_min:int)
[
    "HOROMETROS", 10080,
    "MINECARE",      40,
    "DISPATCH",      70
];

let latest_by_source =
    SparkLoggingEvent_CL
    | where log_logger_s == "PDM_CAEX"
    | where Message has_any ("HOROMETROS", "MINECARE", "DISPATCH")
    | extend fuente_detectada = case(
        Message has "HOROMETROS", "HOROMETROS",
        Message has "MINECARE",   "MINECARE",
        Message has "DISPATCH",   "DISPATCH",
        "OTRA"
    )
    | where fuente_detectada in ("HOROMETROS", "MINECARE", "DISPATCH")
    | summarize arg_max(TimeGenerated, *) by fuente_detectada
    | extend payload = parse_json(Message)
    | extend fuente_payload = toupper(tostring(payload.fuente))
    | extend fuente = iff(isempty(fuente_payload), fuente_detectada, fuente_payload)
    // timestamp principal
    | extend payload_ts_raw = tostring(payload.ultimo_timestamp)
    | extend payload_ts_utc = datetime_local_to_utc(todatetime(payload_ts_raw), "America/Santiago")
    // fallback
    | extend event_ts_utc = TimeGenerated
    | extend reference_ts_utc = coalesce(payload_ts_utc, event_ts_utc)
    | extend ts_origin = iff(isnotnull(payload_ts_utc), "payload.ultimo_timestamp", "TimeGenerated")
    // normalización alertar
    | extend alertar_raw = tostring(payload.alertar)
    | extend alertar_norm = tolower(trim(' "\t\r\n', alertar_raw))
    | extend alertar_norm = replace_string(alertar_norm, "í", "i")
    | extend alertar_flag = alertar_norm in ("si", "true", "1", "yes", "alert", "alerta")
    // diferencia con signo (no abs)
    | extend diff_min_signed = datetime_diff('minute', now(), reference_ts_utc)
    | extend delay_min = iff(diff_min_signed < 0, 0, diff_min_signed)
    | project fuente, event_ts_utc, reference_ts_utc, ts_origin, diff_min_signed, delay_min, alertar_norm, alertar_flag;

let evaluated =
    thresholds
    | join kind=leftouter latest_by_source on fuente
    | extend has_ts = isnotnull(reference_ts_utc)
    | extend source_status = case(
        has_ts == false, "ALERT",
        alertar_flag == true, "ALERT",
        delay_min > umbral_min, "ALERT",
        "OK"
    )
    | extend source_reason = case(
        has_ts == false, "timestamp_missing",
        alertar_flag == true, strcat("alertar=", alertar_norm),
        delay_min > umbral_min, strcat("delay_min=", tostring(delay_min), " > ", tostring(umbral_min)),
        diff_min_signed < 0, strcat("future_ts_skew_min=", tostring(-1 * diff_min_signed)),
        "ok"
    )
    | extend source_evidence = strcat(
        fuente,
        "{status=", source_status,
        ",reason=", source_reason,
        ",ts_origin=", coalesce(ts_origin, "none"),
        ",ref_ts=", iff(isnull(reference_ts_utc), "null", tostring(reference_ts_utc)),
        ",event_ts=", iff(isnull(event_ts_utc), "null", tostring(event_ts_utc)),
        "}"
    );

evaluated
| summarize
    has_alert = max(iif(source_status == "ALERT", 1, 0)),
    evidence_alert = strcat_array(make_list_if(source_evidence, source_status == "ALERT"), " | "),
    evidence_all = strcat_array(make_list(source_evidence), " | "),
    last_update_utc = max(event_ts_utc)
| extend status = iff(has_alert == 1, "ALERT", "OK")
| extend color = case(
    status == "ALERT", "#E53935",
    status == "WARN",  "#FFF4CC",
    "#EAF4EA"
)
| extend evidence = iff(status == "ALERT", evidence_alert, strcat("OK:", evidence_all))
| project status, color, evidence, last_update_utc
```
