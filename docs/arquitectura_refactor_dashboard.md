# Refactor conservador del dashboard `Plataforma_Monitoreo_Cross`

## Alcance y supuestos

- Se mantuvo el dashboard **100% en paneles `text/html`**.
- Se respetó el estilo actual (cards, chips, badges y headers ejecutivos).
- No se asumió acceso real a Azure/Grafana/LAW.
- No se introdujeron jobs, tablas resumen, DCR ni recursos nuevos.

## Problemas detectados

1. Referencias a variables inexistentes en múltiples paneles (principalmente CENTINELA y AMSA CROSS).
2. Placeholders mal escritos en HTML (`${var_name}:raw}` en vez de `${var_name:raw}`).
3. Overrides de estilo que anulaban colores dinámicos (`background:${var...}; background:#EAF4EA;`).
4. Naming inconsistente (`var_cen_*` vs `var_cent_*`, `var_amsa_rep_ppfm` vs `var_amsa_rep_ppfm_global`).
5. Capa 1 y 2 con chips parcialmente estáticos o inconsistentes, afectando legibilidad operativa.

## Decisiones tomadas

### 1) Correcciones de consistencia primero (sin cambios radicales)

- Se corrigieron placeholders malformados.
- Se homogeneizaron referencias AMSA CROSS hacia `var_amsa_rep_ppfm_global`.
- Se eliminaron estilos duplicados que pisaban color dinámico.

### 2) Estabilización de variables faltantes

- Se agregaron variables faltantes como `constant` ocultas (`hide: 2`) con valor por defecto `#EAF4EA`.
- Esto evita errores de render en Grafana y mantiene el look mientras el equipo implementa funciones KQL reales.

### 3) Preparación para performance/mantenibilidad

- Se dejó el modelo listo para migrar gradualmente variables complejas a wrappers delgados contra funciones KQL externas (ver documento dedicado).
- El refactor evita reescribir masivamente la lógica actual para no arriesgar la estética ni la operación visual.

## Arquitectura objetivo (sin romper estado actual)

### Capa 1: Ingestas Cross
- Mantener cards por faena.
- Chips con color dinámico por variable.
- Variables delgadas orientadas a `fn_mon_<faena>_<producto>_ingestas`.

### Capa 2: Resumen Ejecutivo
- Mantener una card por faena/producto con chips globales.
- Variables delgadas orientadas a `fn_mon_<faena>_<producto>_global`.

### Capa 3: Detalle por producto
- Mantener bloques por capa (ingestas/procesamiento/front/calidad).
- Variables por componente (`..._<capa>_<componente>`).
- Reutilizar funciones KQL transversales por patrón.

## Riesgos y limitaciones

- Las nuevas variables `constant` son una medida de estabilidad visual temporal; no reemplazan lógica de monitoreo.
- Algunas secciones de detalle contienen HTML extenso con deuda técnica (anidaciones `<a>` y spans con sintaxis mejorable), no alteradas para evitar impacto visual.
- El paso a funciones KQL requiere creación manual en `ams-uat-dataplatform-laws`.

## Cambios no implementados y por qué

1. **Reescritura completa del HTML de detalle:** no implementada para no romper look & feel ejecutivo.
2. **Migración total de variables grandes actuales a wrappers KQL:** no implementada en esta iteración por riesgo funcional sin ambiente de validación real.
3. **Normalización total de nombres legacy:** se evitó cambio masivo para no romper referencias externas no visibles en este repositorio.

## Dependencias externas necesarias

- Creación manual de funciones KQL en `ams-uat-dataplatform-laws`.
- Validación visual en Azure Managed Grafana con datos reales.
- Validación funcional de cada variable vinculada a chips críticos.

## Validaciones manuales pendientes

1. Confirmar color dinámico por chip en cada card de Capa 1 y Capa 2.
2. Revisar links de detalle que abren dashboards externos.
3. Confirmar naming final esperado por el equipo para variables CENTINELA (`cent` vs `cen`).
4. Priorizar qué variables `constant` pasan primero a query wrapper KQL.
