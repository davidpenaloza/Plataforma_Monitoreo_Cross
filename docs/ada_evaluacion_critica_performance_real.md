# Evaluación crítica de performance ADA en dashboard cross (estado real)

## Conclusión directa

**Conclusión:** **mejora poco** la performance del dashboard **en su estado actual**.

- El rediseño `base -> dominio/capa -> pública` **sí mejora mantenibilidad** y reduce duplicación lógica.
- Pero la mejora de performance del dashboard será **limitada** mientras Grafana siga ejecutando muchas variables ocultas pesadas al cargar.
- En este momento, el dashboard todavía tiene queries KQL largas embebidas en variables ADA (no llamadas a `fn_mon_mlp_ada_*`), por lo que el costo de cómputo de apertura prácticamente no cambia.

---

## Hallazgos aterrizados al comportamiento real del dashboard

## 1) Carga actual de variables en apertura

- El dashboard tiene **80 variables**, y las 80 están ocultas (`hide = 2`).
- Existen **36 variables de tipo query**, y las 36 tienen `refresh = 2` (se recalculan al cargar dashboard / cambio de rango).
- Solo ADA aporta **11 variables query** con `refresh = 2`.

**Implicación real:** aunque los paneles sean `text/html`, el costo de apertura viene fuertemente de variables KQL que se evalúan de forma temprana.

## 2) ADA hoy recalcula bloques muy similares

- Las 11 variables ADA (`var_mlp_ada_*`) tienen queries muy largas (promedio ~36k caracteres por query).
- La duplicación es alta: por ejemplo, `var_mlp_ada_global` y `var_mlp_ada_dispatch` comparten prefijo de ~40k caracteres.
- Cada variable ADA consulta **7-8 resources/workspaces** de Log Analytics.

**Implicación real:** hay alto costo repetido por query al abrir, especialmente con selección multi-resource.

## 3) Rediseño propuesto existe como diseño/documentación, pero no como consumo real en dashboard

- La documentación propone funciones base reutilizables (`fn_mon_mlp_ada_base_statusfuentes`, `fn_mon_mlp_ada_base_ops`) y funciones públicas delgadas.
- Sin embargo, en el JSON actual las variables ADA todavía no invocan `fn_mon_mlp_ada_*`; mantienen KQL embebido extenso.

**Implicación real:** la mejora de performance del dashboard **no ocurre automáticamente** por tener diseño documentado; ocurre cuando las variables del JSON se migran de verdad a esas funciones y se racionaliza el patrón de refresh/carga.

---

## Respuesta a la pregunta central

> Si pasamos ADA desde muchas funciones públicas pesadas hacia jerarquía `base -> dominio/capa -> pública`, ¿mejora performance o solo mantenibilidad?

**Respuesta honesta:**

- **Mantenibilidad:** mejora claramente (sí).
- **Costo lógico por consulta individual:** mejora moderada (sí, menos duplicación interna y mejor factor común).
- **Performance perceptible del dashboard cross al abrir:** mejora **acotada** por sí sola, y en el estado actual **casi nula** porque Grafana sigue ejecutando muchas variables ocultas con `refresh = 2`.

En otras palabras: el patrón ayuda, pero **no mueve tanto la aguja** si no se cambia además la estrategia de variables/carga.

---

## Comparación explícita

| Escenario | Efecto en performance dashboard | Efecto en mantenibilidad | Riesgo |
|---|---|---|---|
| ADA actual (queries pesadas embebidas por variable) | Bajo: apertura costosa por múltiples queries duplicadas y multi-resource | Bajo/Medio: cambios caros y propensos a error | Alto riesgo de deriva y regressions en KQL duplicado |
| ADA rediseñada solo en funciones (sin migrar variables JSON) | Muy bajo: casi sin cambio perceptible | Alto: mejor orden, ownership y trazabilidad | Riesgo de “falsa sensación” de optimización |
| ADA rediseñada + variables migradas a funciones + reducción de queries de capa ejecutiva | Medio: reduce costo repetido y tiempo de apertura | Alto | Riesgo moderado de ajuste funcional por equivalencia |
| ADA rediseñada + estrategia de carga (priorizar capa ejecutiva, diferir detalle) | Medio/Alto: mayor impacto real sin romper `text/html` | Alto | Requiere disciplina en diseño de variables por capa |

---

## Qué conviene atacar primero si el foco real es performance

Orden recomendado (dentro de restricciones actuales: sin tabla resumen, sin jobs nuevos, sin infra nueva):

1. **Globales ejecutivos (capa 1 y capa 2)**
   - Consolidar en pocas señales críticas por producto/capa.
   - Evitar que cada chip del detalle dispare query propia al abrir.

2. **Resúmenes por capa (ingestas/procesamiento/front)**
   - Un query por capa con salida compacta, reutilizable en varios chips/cards.

3. **Detalle por componente (capa 3)**
   - Mantenerlo visual, pero no precargar todo en apertura; cargar bajo demanda (por navegación/row/producto cuando aplique en el modelo de variables).

---

## Recomendación concreta de siguiente paso (máximo impacto sin romper enfoque actual)

## Paso recomendado

**Hacer una migración funcional mínima de ADA en el JSON (no solo docs):**

1. Reemplazar las 11 queries ADA embebidas por llamadas a `fn_mon_mlp_ada_*`.
2. Reducir de 11 variables ADA a un set mínimo para capa ejecutiva (global + 2/3 capas), dejando detalle para vista específica de producto.
3. Mantener `text/html` y chips, pero alimentarlos desde menos variables, más compactas.
4. Revisar `refresh = 2` solo en variables imprescindibles para apertura; evitar precálculo masivo de capa 3.

**Resultado esperado realista:** mejora **algo a moderada** en apertura del dashboard cross y mejora **fuerte** en mantenibilidad.

---

## Síntesis final (sin optimismo artificial)

- El rediseño ADA propuesto es **correcto arquitectónicamente** para ordenar lógica.
- **Por sí solo**, su impacto en performance del dashboard cross es **poco**.
- La aguja de performance se mueve cuando, además del rediseño lógico, se reduce el número de variables query ejecutadas al cargar y se prioriza la capa ejecutiva sobre el detalle precalculado.
