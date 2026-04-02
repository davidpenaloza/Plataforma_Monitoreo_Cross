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
