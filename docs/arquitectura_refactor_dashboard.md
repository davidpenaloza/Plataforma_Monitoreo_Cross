# Arquitectura objetivo y plan de ejecución – Dashboard Cross AMSA (segunda pasada)

## 1) Resumen ejecutivo

Esta segunda pasada consolida el refactor del dashboard cross con un enfoque **operable** para el equipo:

- Se mantiene el diseño ejecutivo en paneles `text/html` (cards, chips, badges y headers).
- Se elimina deuda de consistencia que aún podía generar ambigüedad (naming `cen` vs `cent`, referencias y fallback visuales).
- Se define una arquitectura objetivo por capas **accionable** para ejecutar manualmente en `ams-uat-dataplatform-laws`, sin depender de tabla resumen ni nuevos jobs.
- Se aterriza una hoja de ruta realista para mejora de performance y mantenibilidad sin rediseño radical.

Resultado: el dashboard queda estable para operación inmediata y con una guía clara para evolucionar a wrappers KQL delgados.

---

## 2) Contexto y restricciones (no negociables)

### Contexto
El dashboard integra monitoreo cross en tres capas:
1. Ingestas Cross.
2. Resumen Ejecutivo.
3. Detalle por producto.

### Restricciones de esta etapa
1. Mantener todo en `text/html`.
2. Mantener estética ejecutiva actual.
3. No introducir infraestructura nueva (jobs, tabla resumen, DCR, summary rules, pipelines auxiliares).
4. No asumir acceso real a Azure/Grafana/LAW.
5. Diseñar funciones para creación manual en `ams-uat-dataplatform-laws`.

---

## 3) Diagnóstico del estado actual

## 3.1 Hallazgos estructurales
- El JSON contenía alta mezcla entre lógica operacional y presentación HTML, con repetición de patrones de chips/cards.
- Existían referencias a variables inexistentes en bloques CENTINELA/AMSA, afectando confiabilidad de render.
- Persistía naming inconsistente (`var_cen_*` vs `var_cent_*`) que introducía ambigüedad técnica.

## 3.2 Hallazgos de visualización
- Algunos estilos tenían overrides que podían ocultar el color dinámico real.
- Varias tarjetas de Capa 1 y Capa 2 tenían fallback visual implícito no documentado.

## 3.3 Hallazgos de mantenibilidad
- Variables de gran tamaño (KQL embebido) conviven con variables de estado simple.
- Falta de separación explícita entre variables “productivas” y variables “fallback transitorio”.

---

## 4) Principios de diseño adoptados

1. **Conservación visual primero**: ningún cambio que degrade look & feel.
2. **Refactor incremental**: estabilizar referencias antes de extraer lógica.
3. **Compatibilidad retroactiva**: evitar quiebres en paneles existentes.
4. **Delgado en dashboard, pesado en KQL**: mover complejidad a funciones externas gradualmente.
5. **Trazabilidad**: toda variable debe mapear a componente/capa/producto.

---

## 5) Arquitectura objetivo por capas

## Capa 1 – Ingestas Cross
**Propósito:** visibilidad transversal de salud de fuentes por faena.

**Diseño objetivo de datos:**
- 1 variable por componente crítico (`dispatch`, `pi`, `jigsaw`, etc.).
- Cada variable retorna color o estado normalizado (`OK/WARN/ALERT` + color).
- Funciones KQL explícitas por producto/capa, evitando mega-funciones genéricas.

**Resultado esperado:** lectura rápida de degradaciones de ingesta sin abrir detalle.

## Capa 2 – Resumen Ejecutivo
**Propósito:** estado ejecutivo por producto con mínima carga cognitiva.

**Diseño objetivo de datos:**
- 1 variable global por producto.
- Reglas de agregación documentadas (por ejemplo: ALERT domina WARN, WARN domina OK).
- Reutilización de señales ya calculadas en Capa 1/3 para evitar duplicación lógica.

**Resultado esperado:** un semáforo ejecutivo confiable para toma de decisión.

## Capa 3 – Detalle por producto
**Propósito:** diagnóstico accionable por subcapa (ingestas/procesamiento/front/calidad).

**Diseño objetivo de datos:**
- Variables de detalle por componente técnico.
- Cards por subcapa con chips sólo donde haya señal real.
- Orden estable por producto/capa/componente para facilitar mantenimiento manual.

**Resultado esperado:** trazabilidad desde el resumen hasta la causa operativa.

---

## 6) Estrategia recomendada para esta etapa (sin tabla resumen ni job)

## Fase 1 (inmediata)
- Mantener dashboard estable con variables actuales + fallback controlado.
- Normalizar naming y referencias.
- Documentar cobertura real en matriz única (esta entrega).

## Fase 2 (primera ola de funciones)
- Implementar funciones para Capa 1 y Capa 2 de mayor criticidad.
- Sustituir variables `constant` transitorias por wrappers query delgados.

## Fase 3 (segunda ola)
- Extraer detalle por producto/capa.
- Homologar criterios de severidad y ventanas de tiempo.

Esta secuencia maximiza impacto con riesgo operativo bajo.

---

## 7) Qué mejora realmente en performance

1. **Menos duplicación de consultas** al migrar patrones repetidos a funciones KQL reutilizables.
2. **Menos costo de render/reintentos** al eliminar referencias rotas en HTML.
3. **Menos recomputación conceptual** al centralizar reglas de color/severidad por función.
4. **Menor esfuerzo de diagnóstico** (impacta performance operativa del equipo).

> Importante: en esta etapa no se promete optimización infra (porque no se crean tablas resumen ni jobs nuevos). La mejora es por simplificación lógica y reducción de duplicación.

---

## 8) Qué mejora realmente en mantenibilidad

1. Naming más consistente y menos ambigüedad entre variables equivalentes.
2. Separación clara entre estado actual y transición (fallback vs wrapper futuro).
3. Backlog explícito por componente, prioridad y criticidad en matriz.
4. Catálogo KQL accionable con prioridades por olas.

---

## 9) Riesgos reales

1. **Riesgo de falso verde** en variables fallback `constant` si no se migra a wrappers reales.
2. **Riesgo de divergencia** si distintas funciones implementan reglas de severidad diferentes.
3. **Riesgo de regresión visual** al tocar HTML grande sin validación incremental.
4. **Riesgo de deuda perpetua** si la ola 1 de funciones no se ejecuta en plazo.

Mitigación recomendada: ejecutar migración por lotes pequeños (producto/capa), con checklist visual y funcional.

---

## 10) Backlog técnico sugerido

1. Migrar variables fallback críticas de CENTINELA (Capa 1 y 2) a wrappers KQL reales.
2. Estandarizar salida de funciones (`status`, `severity_rank`, `color`, `last_update_utc`).
3. Definir convención única de ventanas temporales por tipo de señal (ingesta/procesamiento/front).
4. Reducir HTML repetido mediante bloques plantilla internos (sin cambiar `text/html`).
5. Limpiar deuda de sintaxis HTML no crítica en paneles de detalle (anidaciones y spans).

---

## 11) Conclusión ejecutiva final

El dashboard queda **estable, visualmente consistente y listo para operación**, con una ruta técnica clara para evolucionar sin cambios disruptivos. La siguiente ganancia real depende de implementar la primera ola de funciones en `ams-uat-dataplatform-laws` y reemplazar fallback transitorio por señales operativas reales.

En términos ejecutivos: **se redujo riesgo de quiebre visual hoy, y se habilitó mejora sostenida mañana**.
