# Resumen migración funciones MLP (dashboard cross)

## Variables MLP analizadas
- var_mlp_pdmcaex_global
- var_mlp_sirosag_global
- var_mlp_ada_global
- var_mlp_ada_ingestas_global
- var_mlp_ada_pi
- var_mlp_ada_dispatch
- var_mlp_ada_drillit
- var_mlp_ada_plans
- var_mlp_ada_blockgrade
- var_mlp_ada_meteodata
- var_mlp_ada_kpi
- var_mlp_ada_alarm
- var_mlp_ada_front
- var_mlp_pdmcaex_dispatch
- var_mlp_pdmcaex_horometros
- var_mlp_pdmcaex_minecare
- var_mlp_pdmcaex_front
- var_mlp_sirosag_ingesta
- var_mlp_sirosag_transformacion
- var_mlp_siropotencia_ingesta
- var_mlp_siroPotencia_transformacion
- var_mlp_siropotencia_front
- var_mlp_pi_nash
- var_mlp_transf_nash
- var_mlp_prediccion_nash

## Funciones MLP documentadas y listas para implementación manual
- fn_mon_mlp_pdmcaex_global
- fn_mon_mlp_sirosag_global
- fn_mon_mlp_ada_global
- fn_mon_mlp_ada_ingestas_global
- fn_mon_mlp_ada_ingestas_pi
- fn_mon_mlp_ada_ingestas_dispatch
- fn_mon_mlp_ada_ingestas_drillit
- fn_mon_mlp_ada_ingestas_plans
- fn_mon_mlp_ada_ingestas_blockgrade
- fn_mon_mlp_ada_ingestas_meteodata
- fn_mon_mlp_ada_procesamiento_kpi
- fn_mon_mlp_ada_procesamiento_alarm
- fn_mon_mlp_ada_front
- fn_mon_mlp_pdmcaex_ingestas_dispatch
- fn_mon_mlp_pdmcaex_ingestas_horometros
- fn_mon_mlp_pdmcaex_ingestas_minecare
- fn_mon_mlp_pdmcaex_front
- fn_mon_mlp_sirosag_ingesta
- fn_mon_mlp_sirosag_transformacion
- fn_mon_mlp_siropotencia_ingesta
- fn_mon_mlp_siropotencia_transformacion
- fn_mon_mlp_siropotencia_front
- fn_mon_mlp_nash_ingesta_pi
- fn_mon_mlp_nash_transformacion
- fn_mon_mlp_nash_prediccion

## Funciones listas vs parciales
- **Listas (estructura, wrapper, contrato, recurso, KQL base):** todas las funciones listadas arriba.
- **Parciales (lógica operativa fina por validar con datos reales):** todas requieren calibración de umbrales/filtros por componente antes de pasar a producción.

## Clasificación por tipo
### Estado actual
- fn_mon_mlp_pdmcaex_global
- fn_mon_mlp_sirosag_global
- fn_mon_mlp_ada_global
- fn_mon_mlp_ada_ingestas_global
- fn_mon_mlp_ada_ingestas_pi
- fn_mon_mlp_ada_ingestas_dispatch
- fn_mon_mlp_ada_ingestas_drillit
- fn_mon_mlp_ada_ingestas_plans
- fn_mon_mlp_ada_ingestas_blockgrade
- fn_mon_mlp_ada_ingestas_meteodata
- fn_mon_mlp_ada_procesamiento_kpi
- fn_mon_mlp_ada_procesamiento_alarm
- fn_mon_mlp_ada_front
- fn_mon_mlp_pdmcaex_ingestas_dispatch
- fn_mon_mlp_pdmcaex_ingestas_horometros
- fn_mon_mlp_pdmcaex_ingestas_minecare
- fn_mon_mlp_pdmcaex_front
- fn_mon_mlp_sirosag_ingesta
- fn_mon_mlp_sirosag_transformacion
- fn_mon_mlp_siropotencia_ingesta
- fn_mon_mlp_siropotencia_transformacion
- fn_mon_mlp_siropotencia_front
- fn_mon_mlp_nash_ingesta_pi
- fn_mon_mlp_nash_transformacion
- fn_mon_mlp_nash_prediccion

### Históricas
- Ninguna en esta fase (por decisión de priorizar capa ejecutiva/chips actuales).

## Riesgos principales
1. Umbrales iniciales en KQL base deben validarse con comportamiento real por dominio.
2. No todas las variables exponen claramente la tabla/fuente final; algunas dependen de parseo de payload y reglas implícitas.
3. Existe una diferencia de naming en variable de SIRO Potencia transformacion (`var_mlp_siroPotencia_transformacion` en JSON).

## Orden recomendado de implementación real
### Prioridad 1
- fn_mon_mlp_pdmcaex_global
- fn_mon_mlp_sirosag_global
- fn_mon_mlp_ada_global

### Prioridad 2
- fn_mon_mlp_ada_ingestas_global
- fn_mon_mlp_ada_ingestas_pi
- fn_mon_mlp_ada_ingestas_dispatch

### Prioridad 3
- Resto de ADA ingestas
- PDM detalle
- SIRO Potencia
- NaSH
- SIRO SAG detalle

## Nota de alineamiento de dashboard
- No se realizó reescritura masiva de `Plataforma_Monitoreo_Cross.json`.
- Los wrappers quedan definidos en cada archivo de función para adopción progresiva en variables Grafana.


## Actualización específica ADA (query oficial)
- Las funciones ADA MLP fueron ajustadas para basarse en el bloque oficial `statusFuentes` y sus mapas/umbrales actualizados.
- Funciones impactadas: `fn_mon_mlp_ada_global`, `fn_mon_mlp_ada_ingestas_global`, `fn_mon_mlp_ada_ingestas_pi`, `fn_mon_mlp_ada_ingestas_dispatch`, `fn_mon_mlp_ada_ingestas_drillit`, `fn_mon_mlp_ada_ingestas_plans`, `fn_mon_mlp_ada_ingestas_blockgrade`, `fn_mon_mlp_ada_ingestas_meteodata`, `fn_mon_mlp_ada_procesamiento_kpi`, `fn_mon_mlp_ada_procesamiento_alarm`, `fn_mon_mlp_ada_front`.
- Nota: KPI/Alarmas/Front quedaron con integración base y requieren completar bloques `statusKpisAlarms` / `AlertarFront` del query oficial completo al momento de implementación manual.
