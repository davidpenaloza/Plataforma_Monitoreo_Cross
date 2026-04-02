# Catálogo de funciones KQL propuestas (segunda pasada)

**Workspace objetivo de creación manual:** `ams-uat-dataplatform-laws`.

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

### 7.3 Función KQL propuesta real

```kql
.create-or-alter function with (
  folder = "monitoreo/mlp/pdmcaex",
  docstring = "Estado global PDM CAEX para dashboard cross (status, color, evidencia, última actualización)."
)
fn_mon_mlp_pdmcaex_global() {
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
        | extend fuente = tostring(payload.fuente)
        | extend fuente = iff(isempty(fuente), fuente_detectada, toupper(fuente))
        | extend ultimo_timestamp_fuente = datetime_local_to_utc(todatetime(payload.ultimo_timestamp), "America/Santiago")
        | extend alertar = tostring(payload.alertar)
        | project fuente, last_update_utc = TimeGenerated, ultimo_timestamp_fuente, alertar;

    let evaluated =
        thresholds
        | join kind=leftouter latest_by_source on fuente
        | extend Diferencia_Minutos = iff(isnull(ultimo_timestamp_fuente), 999999, datetime_diff('minute', now(), ultimo_timestamp_fuente))
        | extend source_status = iff(alertar == "Si" or abs(Diferencia_Minutos) > umbral_min, "ALERT", "OK")
        | extend source_evidence = strcat(
            fuente,
            ":status=", source_status,
            ",diff_min=", tostring(Diferencia_Minutos),
            ",alertar=", iff(isempty(alertar), "null", alertar)
        );

    evaluated
    | summarize
        has_alert = max(iif(source_status == "ALERT", 1, 0)),
        evidence = strcat_array(make_list(source_evidence), " | "),
        last_update_utc = max(last_update_utc)
    | extend status = iff(has_alert == 1, "ALERT", "OK")
    | extend color = case(
        status == "ALERT", "#E53935",
        status == "WARN",  "#FFF4CC",
        "#EAF4EA"
    )
    | project status, color, evidence, last_update_utc
}
```

### 7.4 Wrapper delgado para la variable Grafana

```kql
fn_mon_mlp_pdmcaex_global()
| project color
| take 1
```

### 7.5 Metadata de implementación solicitada

- **Fuente probable:** Azure Monitor Logs / Log Analytics.
- **Tabla probable:** `SparkLoggingEvent_CL`.
- **Tipo de validación esperada:**
  - Visual: chip cambia de color en Capa 2.
  - Funcional: `status` coincide con alertas por fuente.
  - Umbral: validar que 10080/40/70 minutos reflejan acuerdos operativos.
- **Regla detectada:** ALERT global si al menos una fuente (`HOROMETROS`, `MINECARE`, `DISPATCH`) queda en ALERT.
- **Ambigüedades / supuestos:**
  1. El campo `payload.fuente` puede no venir consistente; por eso se usa `fuente_detectada` como fallback.
  2. Se mantiene conversión de zona horaria `America/Santiago` por paridad con query actual.
  3. `WARN` no aparece en la lógica original; queda reservado para futura regla si negocio lo requiere.
  4. Si una fuente no tiene registros recientes, se fuerza `Diferencia_Minutos` alto para evitar falso verde.

