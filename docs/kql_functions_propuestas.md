# Funciones KQL propuestas para `ams-uat-dataplatform-laws`

> Objetivo: mover lógica repetida de variables hacia funciones reutilizables, manteniendo variables del dashboard como wrappers delgados.

## Convenciones

- `fn_mon_<faena>_<producto>_<scope>` para producto/faena.
- `fn_cross_<scope>` para agregados transversales.
- Salida estándar recomendada para colores: `color` (`#EAF4EA`, `#FFF4CC`, `#FDECEC` o equivalente).

## Propuestas

### 1) fn_mon_mlp_ada_ingestas
- **Objetivo:** estado por componente de ingesta ADA (dispatch, drillit, pi, plans, blockgrade, meteodata).
- **Alimenta:** `var_mlp_ada_dispatch`, `var_mlp_ada_drillit`, `var_mlp_ada_pi`, `var_mlp_ada_plans`, `var_mlp_ada_blockgrade`, `var_mlp_ada_meteodata`.
- **KQL base (esqueleto):**
```kql
fn_mon_mlp_ada_ingestas(component:string) {
  // TODO: consolidar reglas actuales de lateness/alerta por componente
  print component=component, color="#EAF4EA"
}
```

### 2) fn_mon_mlp_pdmcaex_global
- **Objetivo:** estado global producto PDM CAEX.
- **Alimenta:** `var_mlp_pdmcaex_global`.
```kql
fn_mon_mlp_pdmcaex_global() {
  print color="#EAF4EA"
}
```

### 3) fn_mon_cent_global
- **Objetivo:** resumen ejecutivo CENTINELA por producto (mtsulf, siro_mol, siro_flot, ada, mtox, mp10).
- **Alimenta:** `var_cent_*_status`.
```kql
fn_mon_cent_global(producto:string) {
  print producto=producto, color="#EAF4EA"
}
```

### 4) fn_mon_cent_ingestas
- **Objetivo:** color por componente de ingestas CENTINELA (dispatch/jigsaw/pi/vulcan/planes/etc.).
- **Alimenta:** `var_cent_dispatch`, `var_cent_jigsaw`, `var_cent_pi`, etc.
```kql
fn_mon_cent_ingestas(componente:string) {
  print componente=componente, color="#EAF4EA"
}
```

### 5) fn_mon_cmz_rep_global
- **Objetivo:** estado ejecutivo de Reporte CMZ.
- **Alimenta:** `var_cmz_repEjecutivo_transformacion` y potenciales variables de ingesta/envío.
```kql
fn_mon_cmz_rep_global(capa:string) {
  print capa=capa, color="#EAF4EA"
}
```

### 6) fn_mon_amsa_ppfm_global
- **Objetivo:** estado global PPFM / AMSA CROSS.
- **Alimenta:** `var_amsa_rep_ppfm_global`.
```kql
fn_mon_amsa_ppfm_global() {
  print color="#EAF4EA"
}
```

## Patrón de variable wrapper delgado (dashboard)

```kql
fn_mon_cent_ingestas("dispatch")
| project color
| take 1
```

## Estrategia de adopción sugerida

1. Migrar primero Capa 1 y Capa 2 (alto valor visual, baja complejidad).
2. Luego migrar detalle por producto/capa en bloques.
3. Dejar fallback visual verde suave para evitar paneles “rotos”.
