# Catálogo de funciones KQL propuestas (segunda pasada)

**Workspace objetivo de creación manual:** `ams-uat-dataplatform-laws`.

> Nota de organización: desde esta iteración, cada función vive en archivo propio bajo `docs/functions/` y el avance consolidado en `docs/functions/_resumen_avance_funciones.md`.

## 0) Regla operativa de creación manual (nueva política del proyecto)

Para funciones KQL en Azure Monitor / Log Analytics Workspace:

- **No usar** `.create-or-alter function` como instrucción principal de creación.
- Asumir creación manual desde **Logs > Save > Save as function**.
- Toda propuesta debe incluir (mínimo):
  1. nombre sugerido,
  2. folder sugerido,
  3. descripción/docstring sugerida,
  4. cuerpo de query listo para pegar,
  5. contrato de salida (`status`, `color`, `evidence`, `last_update_utc`),
  6. objetivo,
  7. tipo (`estado actual` vs `histórico`),
  8. variable(s) Grafana que alimenta,
  9. fuente probable,
  10. tabla probable,
  11. supuestos,
  12. limitaciones,
  13. fallback documentado.

---

## 1) Criterios de diseño

1. Mantener convención `fn_mon_<faena>_<producto>_<capa>[_<componente>]`.
2. Evitar mega-funciones genéricas; preferir funciones explícitas por dominio cuando haya reglas distintas.
3. Estandarizar salida para todas las funciones:
   - `status` (`OK|WARN|ALERT`)
   - `color` (`#EAF4EA|#FFF4CC|#FDE2E2`)
   - `evidence` (texto corto opcional)
   - `last_update_utc` (datetime)
4. Mantener wrappers del dashboard delgados (`| project color | take 1`).

### Semántica estándar de salida (obligatoria)

- `status` es la **señal lógica principal** de monitoreo.
- `color` es una **derivación estandarizada de `status`** para render visual.
- `evidence` y `last_update_utc` son campos de **trazabilidad y soporte operativo**.


---

## 2) Olas de implementación

## Primera ola (prioridad alta, impacto ejecutivo inmediato)
Enfocada en Capa 1 y Capa 2 para eliminar fallback transitorio.

1. `fn_mon_cent_cross_ingestas_dispatch`
2. `fn_mon_cent_cross_ingestas_jigsaw`
3. `fn_mon_cent_cross_ingestas_pi`
4. `fn_mon_cent_cross_ingestas_planes`
5. `fn_mon_cent_cross_ingestas_vulcan`
6. `fn_mon_cent_cross_global_mtsulf`
7. `fn_mon_cent_cross_global_siromol`
8. `fn_mon_cent_cross_global_siroflot`
9. `fn_mon_cent_cross_global_ada`
10. `fn_mon_cent_cross_global_mtox`
11. `fn_mon_cent_cross_global_mp10`
12. `fn_mon_amsa_ppfm_global`

## Segunda ola (detalle por producto y consolidación)
1. `fn_mon_mlp_ada_ingestas_dispatch`
2. `fn_mon_mlp_ada_ingestas_pi`
3. `fn_mon_mlp_ada_ingestas_drillit`
4. `fn_mon_mlp_ada_ingestas_plans`
5. `fn_mon_mlp_ada_ingestas_blockgrade`
6. `fn_mon_mlp_ada_ingestas_meteodata`
7. `fn_mon_mlp_pdmcaex_global`
8. `fn_mon_mlp_sirosag_global`
9. `fn_mon_cmz_rep_transformacion`
10. `fn_mon_ant_mt_ingestas_global`
11. `fn_mon_ant_mt_procesamiento`
12. `fn_mon_ant_siroapilamiento_front`

---

## 3) Fichas técnicas por función (priorizadas)

## 3.1 Primera ola

### fn_mon_cent_cross_ingestas_dispatch
- **Objetivo:** calcular salud de ingestión Dispatch en CENTINELA (Capa 1).
- **Variables alimentadas:** `var_cent_dispatch`, `var_cent_mtsulf_dispatch`.
- **Alcance:** CENTINELA cross (ingestas).
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** `AzureDiagnostics` y/o `ContainerAppSystemLogs_CL` (según implementación real).
- **Tipo de validación esperada:** visual + funcional + umbral.
- **Observaciones:** reemplaza fallback `constant` en chips ejecutivos.
- **Entradas necesarias para implementar:** origen de logs real, ventana esperada, regla de lateness.
- **Esqueleto KQL:**
```kql
fn_mon_cent_cross_ingestas_dispatch() {
  // Esperado: 1 fila con status, color, evidence, last_update_utc
  // Lógica esperada:
  // 1) Obtener última evidencia de ejecución/ingesta dispatch
  // 2) Calcular minutos de desfase respecto a now()
  // 3) Traducir umbrales a status
  // 4) Mapear status->color
  print status="OK", color="#EAF4EA", evidence="pendiente_regla", last_update_utc=now()
}
```

### fn_mon_cent_cross_ingestas_jigsaw
- **Objetivo:** salud de ingesta Jigsaw en CENTINELA.
- **Variables alimentadas:** `var_cent_jigsaw`, `var_cent_mtsulf_jigsaw`, `var_cent_mtox_jigsaw`, `var_cent_mp10_jigsaw`, `var_cent_ada_jigsaw`.
- **Alcance:** Capa 1 + reutilización en detalle.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** `AzureDiagnostics` o logs de integración equivalentes.
- **Tipo de validación esperada:** visual + funcional + umbral.
- **Observaciones:** componente transversal altamente reutilizable.

### fn_mon_cent_cross_ingestas_pi
- **Objetivo:** salud PI en CENTINELA.
- **Variables alimentadas:** `var_cent_pi`, `var_cent_siro_mol_pi`, `var_cent_ada_pi`.
- **Alcance:** Capa 1 + detalle.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** `AzureDiagnostics` + fuentes PI aterrizadas en LAW.
- **Tipo de validación esperada:** visual + funcional + umbral.
- **Observaciones:** separar datos interpolated vs recorded si aplica.

### fn_mon_cent_cross_global_mtsulf
- **Objetivo:** semáforo ejecutivo de MT Sulfuro.
- **Variables alimentadas:** `var_cent_mtsulf_status`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Baja-Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** tablas de ejecución de jobs/procesos del dominio MT Sulfuro.
- **Tipo de validación esperada:** visual + funcional + umbral.
- **Observaciones:** agrega señales de ingesta/procesamiento definidas por negocio.

### fn_mon_cent_cross_global_siromol
- **Objetivo:** semáforo ejecutivo SIRO Molienda.
- **Variables alimentadas:** `var_cent_siro_mol_status`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** tablas de ejecución/latencia del dominio SIRO Molienda.
- **Tipo de validación esperada:** visual + funcional + umbral.

### fn_mon_cent_cross_global_siroflot
- **Objetivo:** semáforo ejecutivo SIRO Flotación.
- **Variables alimentadas:** `var_cent_siro_flot_status`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** tablas de ejecución/latencia del dominio SIRO Flotación.
- **Tipo de validación esperada:** visual + funcional + umbral.

### fn_mon_cent_cross_global_ada
- **Objetivo:** semáforo ejecutivo ADA Centinela.
- **Variables alimentadas:** `var_cent_ada_status`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** tablas de ADA Centinela + métricas asociadas.
- **Tipo de validación esperada:** visual + funcional + umbral.

### fn_mon_cent_cross_global_mtox
- **Objetivo:** semáforo ejecutivo MT Óxidos.
- **Variables alimentadas:** `var_cent_mtox_status`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** tablas de MT Óxidos (ingesta/proceso/front).
- **Tipo de validación esperada:** visual + funcional + umbral.

### fn_mon_cent_cross_global_mp10
- **Objetivo:** semáforo ejecutivo MP10 calidad de aire.
- **Variables alimentadas:** `var_cent_mp10_status`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** tablas de calidad de aire / MP10 (según disponibilidad real).
- **Tipo de validación esperada:** visual + funcional + umbral.

### fn_mon_amsa_ppfm_global
- **Objetivo:** estado global de productos AMSA CROSS (PPFM / OPR RRHH / Portal Energía).
- **Variables alimentadas:** `var_amsa_rep_ppfm_global`.
- **Alcance:** Capa 1 + Capa 2 AMSA CROSS.
- **Complejidad estimada:** Baja-Media.
- **Prioridad:** P1.
- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** `AzureDiagnostics` + fuentes funcionales de PPFM/OPR/Portal en LAW.
- **Tipo de validación esperada:** visual + funcional + umbral.
- **Observaciones:** hoy es señal compartida; luego dividir por producto si se requiere granularidad.

## 3.2 Segunda ola

### fn_mon_mlp_ada_ingestas_dispatch
- **Objetivo:** estado dispatch ADA MLP.
- **Variables alimentadas:** `var_mlp_ada_dispatch`.
- **Alcance:** Capa 1/3.
- **Complejidad estimada:** Media.
- **Prioridad:** P2.

### fn_mon_mlp_ada_ingestas_pi
- **Objetivo:** estado PI ADA MLP.
- **Variables alimentadas:** `var_mlp_ada_pi`.
- **Alcance:** Capa 1/3.
- **Complejidad estimada:** Media.
- **Prioridad:** P2.

### fn_mon_mlp_ada_ingestas_drillit
- **Objetivo:** estado Drillit ADA MLP.
- **Variables alimentadas:** `var_mlp_ada_drillit`.
- **Alcance:** Capa 1/3.
- **Complejidad estimada:** Media.
- **Prioridad:** P2.

### fn_mon_mlp_pdmcaex_global
- **Objetivo:** estado global producto PDM CAEX.
- **Variables alimentadas:** `var_mlp_pdmcaex_global`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media.
- **Prioridad:** P2.

### fn_mon_mlp_sirosag_global
- **Objetivo:** estado global SIRO SAG.
- **Variables alimentadas:** `var_mlp_sirosag_global`.
- **Alcance:** Capa 2.
- **Complejidad estimada:** Media-Alta.
- **Prioridad:** P2.

### fn_mon_cmz_rep_transformacion
- **Objetivo:** estado de transformación del reporte CMZ.
- **Variables alimentadas:** `var_cmz_repEjecutivo_transformacion`.
- **Alcance:** detalle CMZ.
- **Complejidad estimada:** Baja-Media.
- **Prioridad:** P2.

---

## 4) Limitaciones de diseño (importantes)

1. Sin tabla snapshot, cada función debe recalcular sobre logs/eventos y puede tener costo variable.
2. Sin acceso de validación real, umbrales y fuentes exactas deben confirmarse con el equipo.
3. El dashboard actual mezcla dominios funcionales; algunas funciones requieren separación progresiva.

---

## 5) Regla práctica para no fragilizar el diseño

- **Sí** parametrizar cuando cambia solo el nombre del componente y la lógica es idéntica.
- **No** parametrizar cuando cambian umbrales/fuentes/ventanas por componente.

Por eso esta propuesta prioriza funciones explícitas para primera ola crítica.

---

## 6) Checklist para implementar cada función

1. Identificar fuente real de datos (tabla, columna de timestamp, estado).
2. Definir ventana operativa y tolerancia (minutos).
3. Definir regla `OK/WARN/ALERT` validada por negocio.
4. Publicar función con salida estándar.
5. Cambiar variable dashboard a wrapper delgado.
6. Validar visualmente chip/card afectado.

---

## 7) Caso implementado en detalle: `var_mlp_pdmcaex_global`

### 7.1 Query actual completa (extraída del JSON)

```kql
// TABLA CON ESTRUCTURA DE LOS JOB
let tabla = datatable(
    fuente: string,
    umbral: string
)
    [
    "HOROMETROS", "> 10080",
		"MINECARE", "> 40",
		"DISPATCH", "> 70"
];

let horometros = view () {
SparkLoggingEvent_CL
        | where log_logger_s == "PDM_CAEX" and Message has "HOROMETROS"
				| order by TimeGenerated desc 
        | take 1
};

let minecare = view () {
SparkLoggingEvent_CL
        | where log_logger_s == "PDM_CAEX" and Message has "MINECARE"
				| order by TimeGenerated desc 
        | take 1
};

let distpatch = view () {
SparkLoggingEvent_CL
        | where log_logger_s == "PDM_CAEX" and Message has "DISPATCH"
				| order by TimeGenerated desc 
        | take 1
};

let union_tablas = view()
{
        union horometros,minecare,distpatch
};

let format_tables = view() {
	union_tablas
	| extend json_msg=parse_json(Message)
	| extend ultimo_timestamp_fuente = datetime_local_to_utc(todatetime(json_msg.ultimo_timestamp), 'America/Santiago'),
	        cantidad_registros = json_msg.registros,
	        estado = json_msg.estado,
	        alertar = tostring(json_msg.alertar),
	        tabla = json_msg.tabla,
	        fuente = tostring(json_msg.fuente)
	        // frecuencia_extracion = json_msg.frecuencia_extracion,
	        // timestamp = todatetime(json_msg.timestamp)
	| extend Diferencia_Minutos = datetime_diff('minute', todatetime(now()), ultimo_timestamp_fuente)
	| project fuente, tabla, ultimo_timestamp_fuente, cantidad_registros,Diferencia_Minutos, estado, alertar
};
format_tables
| join kind=inner tabla on fuente
| extend umbral_numerico = toint(trim(@'[^\d]+', umbral))
| extend es_alerta = iff(alertar == "Si" or abs(Diferencia_Minutos) > umbral_numerico, "ALERT", "OK")
| project fuente, es_alerta
| evaluate pivot(fuente, any(es_alerta))
| extend 
    MINECARE   = columnifexists("MINECARE", "OK"),
    HOROMETROS = columnifexists("HOROMETROS", "OK"),
    DISPATCH   = columnifexists("DISPATCH", "OK")

| extend mapa =
  strcat(
    "MINECARE:",   MINECARE,   ";",
    "HOROMETROS:", HOROMETROS,         ";",
    "DISPATCH:",   DISPATCH
  )

| extend status = extract(@"(ALERT)", 1, mapa)
| extend color = case(
    status == "ALERT", "#E53935",
    status == "Warn" or status == "WARN", "#FFF4CC",
    "#EAF4EA"
)
// IMPORTANTÍSIMO: quita comillas/espacios/saltos
| extend color = trim(' "\t\r\n', tostring(color))
// Entrega un único valor
| project color
| take 1
```

### 7.2 Regla detectada (análisis)

- Fuente de evaluación: `SparkLoggingEvent_CL`, logger `PDM_CAEX`, mensajes por fuente (`HOROMETROS`, `MINECARE`, `DISPATCH`).
- Umbrales actuales detectados en minutos:
  - `HOROMETROS`: 10080
  - `MINECARE`: 40
  - `DISPATCH`: 70
- Una fuente queda en `ALERT` si:
  - `alertar == "Si"`, o
  - `abs(Diferencia_Minutos) > umbral`.
- Estado global actual: si cualquier fuente está en `ALERT`, el color final queda rojo (`#E53935`), en caso contrario verde (`#EAF4EA`).

### 7.3 Ficha normalizada (estándar manual) para `fn_mon_mlp_pdmcaex_global`

1. **Nombre sugerido de la función:** `fn_mon_mlp_pdmcaex_global`.
2. **Folder sugerido:** `monitoreo/mlp/pdmcaex`.
3. **Descripción/docstring sugerida:**
   `Estado global actual PDM CAEX para dashboard cross (status, color, evidence, last_update_utc).`
4. **Objetivo explícito:** evaluar el estado global de `HOROMETROS`, `MINECARE` y `DISPATCH` y devolver una señal única para chip ejecutivo.
5. **Tipo de función:** `estado actual` (no histórica).
6. **Variable(s) de Grafana que alimenta:** `var_mlp_pdmcaex_global`.
7. **Fuente probable:** Azure Monitor Logs / Log Analytics.
8. **Tabla probable:** `SparkLoggingEvent_CL`.
9. **Contrato de salida esperado:** `status`, `color`, `evidence`, `last_update_utc`.
10. **Supuestos:**
   - `payload.ultimo_timestamp` está en horario local Chile cuando existe.
   - `TimeGenerated` es fallback válido para estado actual.
   - Si no hay timestamp usable por fuente, debe tratarse como riesgo operativo.
11. **Limitaciones:**
   - Dependencia de formato JSON en `Message`.
   - Posible variabilidad en valores de `payload.alertar`.
   - Umbrales actuales podrían requerir recalibración con datos reales.
12. **Fallback documentado:**
   - timestamp principal: `payload.ultimo_timestamp` parseado y convertido a UTC.
   - fallback: `TimeGenerated`.
   - si ambos fallan: `source_status = ALERT` con `reason=timestamp_missing`.
13. **Cuerpo de query listo para pegar en Logs (Save as function):**

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

### 7.4 Wrapper delgado para la variable Grafana

```kql
fn_mon_mlp_pdmcaex_global()
| project color
| take 1
```

### 7.5 Metadata de implementación solicitada

- **Regla detectada:** ALERT global si al menos una fuente (`HOROMETROS`, `MINECARE`, `DISPATCH`) queda en ALERT.
- **Tipo de validación esperada:**
  - Visual: chip cambia de color en Capa 2.
  - Funcional: `status` coincide con alertas por fuente.
  - Umbral: validar 10080/40/70 con acuerdo operativo.
- **Ambigüedades / supuestos:**
  1. `payload.fuente` puede llegar vacío/inconsistente.
  2. `payload.ultimo_timestamp` puede faltar o no parsear.
  3. `WARN` queda reservado para futura regla de negocio.

### 7.6 Nota técnica (cambios, supuestos y riesgos)

- **Cambios realizados:**
  1. Manejo robusto de timestamp con `payload.ultimo_timestamp` como referencia principal y `TimeGenerated` como fallback.
  2. Normalización robusta de `alertar` (`si/true/1/alerta/...`).
  3. Reemplazo de `abs(Diferencia_Minutos)` por diferencia con signo + `delay_min` para no ocultar skew de reloj/timezone.
  4. Evidencia mejorada por fuente para explicar gatillo de estado.

- **Supuestos:**
  1. `payload.ultimo_timestamp` representa hora local Chile cuando viene informado.
  2. `TimeGenerated` es el mejor fallback disponible para “estado actual”.
  3. Falta de timestamp usable debe tratarse como condición de riesgo (`ALERT`) en un resumen ejecutivo.

- **Riesgos pendientes:**
  1. Algunos mensajes `Message` podrían no ser JSON válido o tener estructura inconsistente entre fuentes.
  2. Umbrales actuales (10080/40/70) pueden no estar alineados a SLA vigentes.
  3. Diferencias negativas de tiempo pueden reflejar relojes desalineados y no necesariamente falla de proceso.

- **Qué validar con datos reales:**
  1. Distribución real de `alertar` (valores textuales concretos) para ajustar normalización.
  2. Calidad de `payload.ultimo_timestamp` por fuente.
  3. Tolerancia aceptable ante timestamps futuros (skew) y tratamiento final (WARN/ALERT).
  4. Si `last_update_utc` debe provenir de `event_ts_utc` (actual) o de `reference_ts_utc` (dato de fuente).


## 8) Normalización aplicada al resto del catálogo

- Las funciones de primera y segunda ola quedan sujetas al mismo estándar de 13 puntos definido en la sección 0.
- Para próximas iteraciones, cada función se debe publicar en este formato antes de implementación manual en `ams-uat-dataplatform-laws`.
- Este ajuste es de forma (documentación implementable), sin rediseñar lógica de negocio ya acordada.
