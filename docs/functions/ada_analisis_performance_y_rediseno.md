# ADA MLP – Análisis crítico de performance y duplicación de cálculo

## Alcance revisado
Funciones ADA analizadas:
- `fn_mon_mlp_ada_global`
- `fn_mon_mlp_ada_ingestas_global`
- `fn_mon_mlp_ada_ingestas_pi`
- `fn_mon_mlp_ada_ingestas_dispatch`
- `fn_mon_mlp_ada_ingestas_drillit`
- `fn_mon_mlp_ada_ingestas_plans`
- `fn_mon_mlp_ada_ingestas_blockgrade`
- `fn_mon_mlp_ada_ingestas_meteodata`
- `fn_mon_mlp_ada_procesamiento_kpi`
- `fn_mon_mlp_ada_procesamiento_alarm`
- `fn_mon_mlp_ada_front`

## Hallazgo principal: sí hay duplicación pesada real

La estrategia actual repite en cada función casi todo el bloque base de ADA (`statusFuentes`):
- mapas de jobs (`mapContainerJobs_MLP_ADA_Refactor`),
- mapas de umbrales (`mapUmbralAlerta`),
- generación de `bins_expected_logs`,
- extracción de ejecuciones reales (`ContainerAppSystemLogs_CL` + `AzureDiagnostics`),
- acumulación por umbral,
- pivot final por fuente.

### Evidencia rápida de similitud
Se comparó el bloque KQL de las 11 funciones ADA propuestas:
- tamaño por función entre ~6.8k y ~6.9k caracteres,
- estructura casi idéntica,
- cambio principal solo en la regla final (`status_global = ...`).

## Por qué esto es costoso

1. **Costo de cómputo duplicado en LAW**
   - Si un dashboard consulta varios chips ADA simultáneamente, cada variable ejecuta nuevamente el mismo bloque pesado.

2. **Costo de latencia en Grafana**
   - Múltiples queries largas paralelas pueden elevar tiempo de render y riesgo de timeout.

3. **Costo de mantenibilidad**
   - Cambiar un umbral o regla obliga a tocar 11 funciones casi iguales.
   - Alto riesgo de drift (una función queda desalineada del resto).

## Respuesta a la pregunta técnica

> ¿Debe cada función específica ADA recalcular toda la lógica base?

**No conviene** en el estado actual.
Con las restricciones del proyecto (sin jobs, sin tabla resumen, sin recursos nuevos), lo más eficiente es **separar en capas reutilizables** dentro de funciones KQL de LAW y que wrappers de Grafana llamen funciones delgadas.

## Propuesta concreta de rediseño ADA (sin romper restricciones)

## 1) Jerarquía sugerida

### Capa base (reutilizable, estado actual)
1. `fn_mon_mlp_ada_base_statusfuentes()`
   - Devuelve snapshot base con señales por fuente (PI/Dispatch/Drillit/Plans/Blockgrade/Meteo/Settings + timestamps/evidence).
   - Aquí vive la lógica pesada oficial de `statusFuentes`.

2. `fn_mon_mlp_ada_base_ops()`
   - Encapsula bloques complementarios del query oficial: `statusKpisAlarms`, `AlertarFront`, NRT y excepciones de alarmas.

### Capa intermedia (agregación por dominio)
3. `fn_mon_mlp_ada_dom_ingestas()`
4. `fn_mon_mlp_ada_dom_procesamiento()`
5. `fn_mon_mlp_ada_dom_front()`

### Capa final (funciones públicas para Grafana)
6. `fn_mon_mlp_ada_global()`
7. `fn_mon_mlp_ada_ingestas_global()`
8. `fn_mon_mlp_ada_ingestas_pi()`
9. `fn_mon_mlp_ada_ingestas_dispatch()`
10. `fn_mon_mlp_ada_ingestas_drillit()`
11. `fn_mon_mlp_ada_ingestas_plans()`
12. `fn_mon_mlp_ada_ingestas_blockgrade()`
13. `fn_mon_mlp_ada_ingestas_meteodata()`
14. `fn_mon_mlp_ada_procesamiento_kpi()`
15. `fn_mon_mlp_ada_procesamiento_alarm()`
16. `fn_mon_mlp_ada_front()`

> Estas funciones públicas se simplifican para no recalcular todo: consumen salidas de capa base/intermedia y solo aplican selección/agregación final.

## 2) Qué funciones deberían simplificarse

- Mantener nombres públicos actuales (evita romper wrappers/paneles).
- Simplificar su cuerpo para delegar en `fn_mon_mlp_ada_base_statusfuentes()` y `fn_mon_mlp_ada_base_ops()`.
- Evitar repetir mapas/joins/pivots completos en cada una.

## 3) Wrappers finales en Grafana

Se mantienen delgados y estables (sin cambio de UX):

```kql
fn_mon_mlp_ada_global() | project color | take 1
fn_mon_mlp_ada_ingestas_dispatch() | project color | take 1
fn_mon_mlp_ada_ingestas_pi() | project color | take 1
...
```

## Limitaciones / viabilidad en Azure Monitor LAW

- KQL functions en LAW **sí permiten** composición (función llamando función), por lo que la jerarquía propuesta es viable.
- Restricción práctica: evitar dependencia circular y controlar costo de expandir funciones complejas.
- No requiere nuevos recursos Azure (cumple restricción).

## Comparación explícita

| Criterio | Estrategia actual | Estrategia propuesta |
|---|---|---|
| Reuso de lógica | Bajo (duplicación alta) | Alto (base + dominio + pública) |
| Costo de ejecución total en dashboard | Alto (N veces bloque pesado) | Medio/Bajo (bloques pesados centralizados) |
| Mantenibilidad | Baja (11 funciones casi idénticas) | Alta (reglas centrales en pocas funciones) |
| Riesgo de desalineación | Alto | Bajo |
| Esfuerzo inicial de migración | Bajo (ya está) | Medio (refactor en LAW) |
| Compatibilidad con restricciones | Parcial | Alta |

## Riesgos de la propuesta

1. Refactor inicial puede introducir regresiones si no se valida componente por componente.
2. Si `base_ops` no queda bien definido, KPI/Alarm/Front podrían mantener ambigüedad temporal.
3. Requiere disciplina para que nuevas señales ADA entren por capa base y no por duplicación local.

## Orden recomendado de implementación (incremental y seguro)

1. Crear `fn_mon_mlp_ada_base_statusfuentes()` validando paridad con función actual `global`.
2. Refactorizar `fn_mon_mlp_ada_ingestas_pi/dispatch` para consumir base.
3. Refactorizar resto de ingestas.
4. Crear `fn_mon_mlp_ada_base_ops()` y mover KPI/Alarm/Front.
5. Refactorizar `fn_mon_mlp_ada_global()` para consumir ambos bloques base.

## Conclusión

Sí: hoy hay duplicación real y costosa. La mejor alternativa, dentro de las restricciones actuales, es una jerarquía de funciones ADA en capas (base → dominio → pública) manteniendo wrappers de Grafana delgados y el dashboard 100% `text/html`.
