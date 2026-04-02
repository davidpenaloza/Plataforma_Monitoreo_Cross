# fn_mon_mlp_ada_ingestas_pi

## Nombre de la función
`fn_mon_mlp_ada_ingestas_pi`

## Objetivo
Obtener estado de **ingestas** para ADA MLP basado en la query oficial actualizada compartida por el equipo.

## Tipo de función
**estado actual** (ejecutiva / chips cross).

## Workspace objetivo
`ams-uat-dataplatform-laws`

## Variable(s) de Grafana que alimenta
- `var_mlp_ada_pi`

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
fn_mon_mlp_ada_ingestas_pi()
| project color
| take 1
```

## Query de validación
```kql
fn_mon_mlp_ada_ingestas_pi()
| project status, color, evidence, last_update_utc
| take 1
```

## Fuente probable
Azure Monitor Logs / Log Analytics.

## Tabla probable
- `ContainerAppSystemLogs_CL`
- `AzureDiagnostics`
- `ContainerAppConsoleLogs_CL`

## Regla operacional esperada
Componente PI usando job01/job02 pisystem + timestamp PI.

## Supuestos
1. Se adopta como base oficial el bloque `statusFuentes` + mapas/umbrales entregados por el equipo.
2. Para funciones ejecutivas se usa estado actual (no ventana de Grafana).
3. Los bloques `KPIs`, `Alarmas`, `Front` requieren completar integración con `statusKpisAlarms` y `AlertarFront` del query oficial completo.

## Limitaciones
1. Algunas señales (KPIs/Alarmas/Front) dependen de secciones adicionales del query oficial y necesitan validación con datos reales.
2. Debe validarse el set exacto de columnas pivot en `statusFuentes` para evitar nulos por cambios de job.

## KQL propuesta (basada en query oficial ADA)
```kql
let endQuery = bin(now(), 1m);
let startQuery = endQuery - 2d;
let mapContainerJobs_MLP_ADA_Refactor = dynamic(
{
    "mlp-prd-caj-meteo-job01": "job01_meteo",
    "mlp-prd-caj-meteo-job02": "job02_meteo",
    "mlp-prd-caj-pisystem-job01": "job01_pisystem",
    "mlp-prd-caj-pisystem-job02": "job02_pisystem",
    "mlp-prd-caj-plans-job01": "job01_plans",
    "ams-prd-caj-genshare-job01": "job01_optimizador_mezcla",
    "DISPATCH": "ADF_dispatch_ingestion",
    "DRILLIT": "ADF_drillit_ingestion",
    "BLOCKGRADE": "ADF_blockgrade_ingestion",
    "mlp-prd-caj-prfci-job01": "job01_settings",
    "mlp-prd-caj-prfci-job02": "job02_settings"
});
let mapUmbralAlerta = dynamic(
{
    "mlp-prd-caj-meteo-job01": 35,
    "mlp-prd-caj-meteo-job02": 35,
    "mlp-prd-caj-pisystem-job01": 25,
    "mlp-prd-caj-pisystem-job02": 25,
    "mlp-prd-caj-plans-job01": 25,
    "ams-prd-caj-genshare-job01": 1440,
    "DISPATCH": 35,
    "DRILLIT": 35,
    "BLOCKGRADE": 60,
    "mlp-prd-caj-prfci-job01": 60,
    "mlp-prd-caj-prfci-job02": 120
});
let maxUmbral = toscalar(
    range dummy from 1 to 1 step 1
    | extend values = mapUmbralAlerta
    | mv-expand values
    | extend value = todouble(replace_string(tostring(split(values, ":")[1]), "}", ""))
    | summarize max(value)
);
let bins_expected_logs =
    range TimeGeneratedBin_Chile from startQuery - totimespan(maxUmbral * 60s) to endQuery step 1m
    | extend TimeGeneratedBin_Chile = bin(TimeGeneratedBin_Chile, 1m)
    | extend ["expected_mlp-prd-caj-meteo-job01"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) % 15 == 0, 1, 0),
             ["expected_mlp-prd-caj-meteo-job02"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) in (1,16,31,46), 1, 0),
             ["expected_mlp-prd-caj-pisystem-job01"] = 1,
             ["expected_mlp-prd-caj-pisystem-job02"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) in (1,11,21,31,41,51), 1, 0),
             ["expected_mlp-prd-caj-plans-job01"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) == 46 and datetime_part("hour", TimeGeneratedBin_Chile) in (0,1,6,13,17,18,19), 1, 0),
             ["expected_ams-prd-caj-genshare-job01"] = iff(datetime_part("hour", TimeGeneratedBin_Chile) == 23 and datetime_part("minute", TimeGeneratedBin_Chile) == 5, 1, 0),
             ["expected_DISPATCH"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) in (6,16,26,36,46,56), 1, 0),
             ["expected_DRILLIT"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) in (1,11,21,31,41,51), 1, 0),
             ["expected_BLOCKGRADE"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) in (2,32), 1, 0),
             ["expected_mlp-prd-caj-prfci-job01"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) in (0), 1, 0),
             ["expected_mlp-prd-caj-prfci-job02"] = iff(datetime_part("minute", TimeGeneratedBin_Chile) == 0 and datetime_part("hour", TimeGeneratedBin_Chile) in (12,18), 1, 0)
    | project TimeGeneratedBin_Chile, allValues=pack_all()
    | mv-expand allValues
    | extend nombreEsperado = tostring(bag_keys(allValues)[0])
    | extend valorEsperado = toint(allValues[nombreEsperado])
    | where nombreEsperado startswith "expected_"
    | project TimeGeneratedBin_Chile, nombreEsperado, valorEsperado;
let bins_real_data =
    union
        (ContainerAppSystemLogs_CL
            | where TimeGenerated >= startQuery - totimespan(maxUmbral * 60s) and TimeGenerated < endQuery
            | where (JobName_s startswith "mlp-prd-" or JobName_s startswith "ams-prd-")
            | where Log_s contains "has successfully completed"
            | project TimeGenerated, nombreReal = JobName_s, valorReal = 1),
        (AzureDiagnostics
            | where TimeGenerated >= startQuery - totimespan(maxUmbral * 60s) and TimeGenerated < endQuery
            | where Category == "PipelineRuns"
            | where OperationName in ("PL_FULL - Succeeded", "PL_BLOCKGRADE - Succeeded")
            | extend nombreReal = iff(ResourceGroup == "MLP-PRD-RG-BLKGRDE", "BLOCKGRADE", replace_string(ResourceGroup, "MLP-PRD-RG-", ""))
            | project TimeGenerated, nombreReal, valorReal = 1)
    | summarize valorReal = sum(valorReal) by nombreReal, TimeGeneratedBin_Chile = bin(TimeGenerated, 1m)
    | extend nombreEsperado = strcat("expected_", nombreReal);
let statusFuentes =
    bins_expected_logs
    | join kind=leftouter bins_real_data on TimeGeneratedBin_Chile, nombreEsperado
    | extend nombreReal = replace_string(nombreEsperado, "expected_", "")
    | extend valorReal = coalesce(valorReal, 0)
    | extend umbralAlerta = todouble(mapUmbralAlerta[nombreReal])
    | extend range = range(TimeGeneratedBin_Chile, TimeGeneratedBin_Chile + totimespan(umbralAlerta * 60s), 1m)
    | mv-expand range to typeof(datetime)
    | summarize valorAccReal=sum(valorReal), valorAccEsperado=sum(valorEsperado) by TimeGeneratedBin_Chile=range, nombreReal
    | where TimeGeneratedBin_Chile < endQuery
    | summarize status_base = iff(countif(valorAccReal == 0 and valorAccEsperado != 0) > 0, "a", "-") by TimeGeneratedBin_Chile=bin(TimeGeneratedBin_Chile, 15m), nombreReal
    | extend status = iff(status_base == "a", "ALERT", "OK")
    | summarize status = iff(countif(status == "ALERT") > 0, "ALERT", "OK") by TimeGeneratedBin_Chile, nombreReal
    | extend nombreReal = tostring(mapContainerJobs_MLP_ADA_Refactor[nombreReal])
    | evaluate pivot(nombreReal, take_any(status), TimeGeneratedBin_Chile)
    | order by TimeGeneratedBin_Chile desc
    | take 1;

// Derivación explícita por componente basada en query oficial ADA.
let signals =
    statusFuentes
    | extend Dispatch = iff(tostring(ADF_dispatch_ingestion) == "ALERT", "ALERT", "OK")
    | extend PI = iff(tostring(job01_pisystem) == "ALERT" or tostring(job02_pisystem) == "ALERT", "ALERT", "OK")
    | extend Drillit = iff(tostring(ADF_drillit_ingestion) == "ALERT", "ALERT", "OK")
    | extend Blockgrade = iff(tostring(ADF_blockgrade_ingestion) == "ALERT", "ALERT", "OK")
    | extend Plans = iff(tostring(job01_plans) == "ALERT", "ALERT", "OK")
    | extend Meteo = iff(tostring(job01_meteo) == "ALERT" or tostring(job02_meteo) == "ALERT", "ALERT", "OK")
    | extend Settings = iff(tostring(job01_settings) == "ALERT" or tostring(job02_settings) == "ALERT", "ALERT", "OK")
    // KPIs/Alarmas/Front requieren bloques adicionales del query oficial (statusKpisAlarms/AlertarFront)
    | extend KPIs = "OK", Alarmas = "OK", Front = "OK";

signals
| extend status_global = PI
| extend status = iff(status_global == "ALERT", "ALERT", "OK")
| extend color = case(status == "ALERT", "#E53935", "#EAF4EA")
| extend evidence = strcat("component=fn_mon_mlp_ada_ingestas_pi; source=statusFuentes_oficial_ada")
| extend last_update_utc = now()
| project status, color, evidence, last_update_utc
```
