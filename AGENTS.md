# AGENTS.md

## Proyecto
Plataforma de Monitoreo Cross en Grafana Azure Managed Grafana.

## Objetivo del proyecto
Refactorizar y ordenar dashboards cross de monitoreo manteniendo completamente el enfoque visual en paneles `text/html`, con cards, chips de color, headers y badges, sin perder el look & feel actual.

El foco principal es:
1. mejorar mantenibilidad,
2. reducir duplicación,
3. preparar una mejora de performance,
4. mantener el dashboard visualmente atractivo y ejecutivo.

---

## Restricciones obligatorias

### 1. Mantener `text/html`
Toda la capa visual debe seguir usando paneles `text/html`.

No reemplazar la visualización por:
- stat panels,
- tables,
- rows visualmente pobres,
- layouts genéricos de Grafana,
salvo que el cambio sea explícitamente solicitado.

### 2. No romper la estética actual
Mantener:
- cards por faena / producto,
- chips de colores,
- headers con badges de entorno,
- estructura visual ejecutiva,
- layout agradable y legible.

Se permiten mejoras visuales solo si:
- ordenan el dashboard,
- corrigen inconsistencias,
- mejoran legibilidad,
- no cambian radicalmente el diseño.

### 3. No asumir infraestructura nueva
No asumir que existirán o estarán disponibles:
- Azure Functions nuevas,
- Azure Container Apps Jobs,
- tablas resumen,
- custom tables,
- DCR,
- summary rules,
- pipelines nuevos,
- procesos batch automáticos,
salvo que el usuario lo pida explícitamente.

### 4. Diseñar para el estado actual
La solución debe funcionar bajo este supuesto:
- el dashboard seguirá usando `text/html`,
- la lógica de monitoreo se puede modularizar con funciones KQL,
- las funciones se crearán manualmente en `ams-uat-dataplatform-laws`,
- no se debe depender de una tabla resumen central en esta etapa.

### 5. No inventar acceso real
No asumir:
- permisos Azure,
- permisos Grafana,
- acceso a LAW,
- acceso a despliegue,
- acceso a APIs,
si no están explícitamente disponibles.

---

## Arquitectura objetivo del dashboard

La plataforma se organiza en 3 capas conceptuales:

### Capa 1: Ingestas Cross
Vista transversal por faena o ámbito.
Muestra fuentes o ingestas relevantes.
Debe ser ejecutiva, simple y visual.

### Capa 2: Resumen Ejecutivo
Vista por producto.
Muestra el estado actual del producto con el mínimo necesario de señales.
Debe ser más liviana que la capa detallada.

### Capa 3: Detalle por producto
Vista detallada por producto.
Debe mostrar capas como:
- Ingestas,
- Procesamiento / Transformación,
- Front,
- Recomendación,
- Calidad,
- otras según el producto.

Debe seguir en `text/html`.

---

## Prioridades del proyecto

Orden de prioridad:

1. No romper el look visual
2. Mantener todo en `text/html`
3. Simplificar el dashboard
4. Mejorar mantenibilidad
5. Reducir duplicación
6. Mejorar performance dentro de las restricciones actuales
7. Preparar una futura evolución a arquitectura con snapshot central, sin implementarla ahora

---

## Decisiones técnicas esperadas

### Variables
- Mantener variables solo cuando sean necesarias para alimentar chips/cards HTML.
- Favorecer variables delgadas.
- Evitar lógica monstruosa repetida dentro de múltiples variables.
- Corregir variables inexistentes o mal referenciadas.
- Corregir placeholders mal escritos como `:raw}` u otros errores equivalentes.

### KQL
- Mover lógica repetida hacia funciones KQL externas cuando tenga sentido.
- Pensar las funciones para ser creadas manualmente en `ams-uat-dataplatform-laws`.
- Priorizar mantenibilidad sobre sofisticación excesiva.
- Documentar limitaciones de consultas cross-workspace si aparecen.

### HTML
- Mantener cards/chips/badges.
- Corregir:
  - backgrounds duplicados,
  - estilos que pisan el color dinámico,
  - errores de sintaxis HTML/CSS inline,
  - referencias incorrectas de variables.
- No degradar el dashboard a un formato visual menos ejecutivo.

### Organización
- Favorecer una estructura más limpia por faena, producto y capa.
- Si se usan rows, que ayuden a ordenar, no a empeorar la UX.
- Evitar paneles gigantes inmanejables si se pueden separar sin perder la estética.

---

## Naming conventions

### Variables de Grafana
Usar nombres consistentes, preferentemente:

- `var_<faena>_<producto>_<capa>`
- `var_<faena>_<producto>_<capa>_<componente>`

Ejemplos:
- `var_mlp_ada_global`
- `var_mlp_ada_ingestas`
- `var_mlp_ada_ingestas_pi`
- `var_cent_mt_sulf_proc`
- `var_amsa_ppfm_global`

Evitar nombres ambiguos o mezclas inconsistentes.

### Funciones KQL
Usar nombres consistentes, preferentemente:

- `fn_mon_<faena>_<producto>_<scope>`
- `fn_cross_<scope>`

Ejemplos:
- `fn_mon_mlp_ada_global`
- `fn_mon_mlp_ada_ingestas`
- `fn_mon_cent_mtsulf_global`
- `fn_cross_resumen_ejecutivo`

### Capas
Usar nombres homogéneos:
- `global`
- `ingestas`
- `procesamiento`
- `transformacion`
- `front`
- `recomendacion`
- `calidad`

No mezclar sin necesidad `procesamiento` y `transformacion` dentro del mismo producto si representan la misma idea.

---

## Lo que Codex debe hacer

Cuando trabajes sobre este proyecto:

1. Analiza el JSON model actual.
2. Detecta inconsistencias.
3. Corrige errores de variables, placeholders y estilos.
4. Simplifica la capa 1 y la capa 2 si están demasiado cargadas.
5. Mantén el detalle visualmente atractivo.
6. Reduce duplicación de HTML y KQL cuando sea posible.
7. Propón funciones KQL para externalizar la lógica repetida.
8. Documenta claramente:
   - supuestos,
   - limitaciones,
   - cambios realizados,
   - cambios no realizados.

---

## Entregables esperados de cualquier refactor

Toda propuesta o refactor debe incluir:

### A. JSON refactorizado
El dashboard actualizado.

### B. Documento de arquitectura
Archivo markdown con:
- problemas detectados,
- decisiones tomadas,
- arquitectura objetivo,
- riesgos,
- limitaciones.

### C. Funciones KQL propuestas
Archivo markdown con:
- nombre de la función,
- objetivo,
- variables o componentes que alimenta,
- KQL propuesta.

### D. Matriz de mapeo
Tabla tipo:

`producto | capa | componente | variable actual | variable nueva | función KQL`

### E. Lista de pendientes
Sección final:
- “Cambios no implementados y por qué”
- “Dependencias externas necesarias”
- “Validaciones manuales pendientes”

---

## Qué NO hacer

No hacer estas cosas salvo que se pidan explícitamente:

- reemplazar toda la visualización por paneles estándar de Grafana,
- introducir una tabla resumen central como dependencia obligatoria,
- asumir jobs o procesos automáticos,
- romper el naming existente sin justificarlo,
- inventar acceso a Azure o Grafana,
- hacer refactors visuales radicales,
- eliminar chips/cards ejecutivos,
- dejar el dashboard más feo aunque quede más simple técnicamente.

---

## Criterios de éxito

La solución será considerada buena si:

- el dashboard sigue viéndose bonito,
- sigue usando `text/html`,
- queda más ordenado,
- tiene menos duplicación,
- la capa 1 y 2 quedan más ejecutivas,
- el detalle queda mejor organizado,
- las variables quedan más consistentes,
- las funciones KQL necesarias quedan claras,
- el equipo puede continuar manualmente en Azure/Grafana sin rediseñar todo.

---

## Regla final
Si hay conflicto entre:
- estética visual
- y refactor técnico

favorecer una solución que mantenga la estética, siempre que no aumente innecesariamente la complejidad.


## Calidad esperada de documentación
- No dejar documentos en estado de borrador conceptual.
- Toda matriz debe servir como backlog técnico accionable.
- Toda propuesta KQL debe distinguir entre:
  - implementable ahora,
  - esqueleto pendiente de contexto,
  - placeholder temporal.
- Evitar documentos ambiguos o excesivamente genéricos.
- Cuando falte contexto para implementar lógica real, documentar exactamente qué falta y por qué.

- ## Regla permanente para funciones KQL en Log Analytics Workspace

Cuando propongas funciones KQL para Azure Monitor / Log Analytics Workspace:

1. NO uses ni sugieras `.create-or-alter function` como método principal de creación en LAW.
2. Asume que la creación se hará manualmente desde:
   - Logs
   - ejecutar/probar query
   - Save
   - Save as function
3. Entrega siempre la función en formato compatible con ese flujo:
   - cuerpo de query listo para pegar en Logs
   - nombre sugerido de función
   - folder sugerido
   - docstring o descripción sugerida
4. Toda función debe estar documentada explícitamente con:
   - objetivo operativo
   - capa a la que pertenece
   - variable(s) de Grafana que alimenta
   - tipo de estado que representa (actual vs histórico)
   - contrato de salida esperado
   - dependencias de tablas/campos
   - supuestos y limitaciones
5. Toda función debe diseñarse con salida estándar, salvo justificación contraria:
   - status
   - color
   - evidence
   - last_update_utc
6. `status` debe ser la señal lógica principal.
7. `color` debe ser una derivación estandarizada de `status`, no la fuente de verdad.
8. `evidence` y `last_update_utc` deben usarse como soporte de trazabilidad operativa.
9. Antes de proponer una función, aclara si está pensada para:
   - estado actual
   - estado histórico
   - capa ejecutiva
   - detalle diagnóstico
10. No introduzcas dependencia de `$__timeFrom`, `$__timeTo`, `startQuery` o `endQuery` si la función está pensada para chips/card ejecutivos de estado actual.
11. Si una función depende de campos opcionales en payloads JSON, diseña fallback explícito y documentado.
12. Si una función puede ser frágil por parametrización o por alcance cross-workspace, dilo explícitamente y propone una alternativa más robusta.
13. No dejes esqueletos vacíos sin explicación. Si falta contexto para implementar la lógica real, documenta:
   - qué falta
   - qué tabla/campo probable se necesita
   - qué regla operacional debe validarse
