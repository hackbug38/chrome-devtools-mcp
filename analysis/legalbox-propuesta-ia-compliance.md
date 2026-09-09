# LegalBox: líneas de producto de compliance de IA, motor de cumplimiento y capa de trazabilidad

Propuesta técnica y comercial sobre la base de [Graphify](https://github.com/Graphify-Labs/graphify).

---

## Nota de alcance y fuentes

Este documento analiza **dos proyectos externos** al repositorio que lo aloja. No describe
`chrome-devtools-mcp` ni guarda relación con él; se archiva aquí por ser el repositorio disponible
en la sesión de trabajo.

**Fecha del análisis: 9 de septiembre de 2026.** El calendario normativo que sostiene la propuesta
cambió seis semanas antes de esta fecha, así que la vigencia del documento es corta y conviene
refrescarlo antes de usarlo comercialmente.

Dos limitaciones que hay que conocer antes de leer:

1. **No se pudo leer legalbox.plus.** El proxy de red de la sesión bloquea el dominio. Todo el
   perfil de producto de LegalBox proviene de resultados de búsqueda y fichas de terceros, no de la
   web del cliente. **Su web ya menciona IA en el alcance del producto**, de modo que antes de
   enviar nada hay que verificar qué tienen construido, para posicionar esta propuesta como
   profundización y no como descubrimiento de la categoría.
2. **Graphify se analizó por lectura remota** de su rama `v8`, sin clonar el repositorio. Las
   afirmaciones sobre su código están contrastadas contra fuente, salvo dos que quedan marcadas
   como supuestos pendientes.

A lo largo del documento se distingue de forma explícita entre **lo que funciona de serie**, **lo
que hay que construir** y **lo que no se ha podido verificar**.

---

## 1. Base normativa verificada

Contrastado con EUR-Lex, el Consejo, el Parlamento Europeo y la Comisión.

**Reglamento (UE) 2026/1744, de 8 de julio de 2026** (Digital Omnibus sobre IA), que modifica el
Reglamento (UE) 2024/1689 y otros, **en vigor desde el 27 de julio de 2026**.

| Obligación                                                                                  | Fecha          | Estado a 9-sep-2026     |
| ------------------------------------------------------------------------------------------- | -------------- | ----------------------- |
| Art. 4, alfabetización en IA                                                                | feb 2025       | **Ya aplicable**        |
| Art. 50, transparencia: aviso de interacción, etiquetado de contenido sintético, deepfakes  | **2 ago 2026** | **Ya aplicable**        |
| Fin del periodo de gracia del marcado de contenido generado por IA, reducido de 6 a 3 meses | **2 dic 2026** | **A unas 12 semanas**   |
| Alto riesgo autónomo, Anexo III                                                             | **2 dic 2027** | Aplazado desde ago 2026 |
| Alto riesgo embebido en producto, Anexo I                                                   | 2 ago 2028     | Aplazado                |

### Los cuatro matices que son directamente vendibles

**El error más extendido del mercado es creer que el Omnibus aplaza el AI Act en bloque.** Solo
aplaza las obligaciones de alto riesgo de los anexos III y I. Decir la verdad del calendario es
diferenciación comercial frente a quien está vendiendo miedo al alto riesgo, que se ha ido
dieciséis meses.

**El art. 4 quedó matizado.** La obligación es _adoptar medidas_ que apoyen la alfabetización en
IA, **no garantizar un nivel** concreto en cada persona, atendiendo a conocimientos técnicos,
experiencia, educación, formación, contexto de uso y colectivos sobre los que se usará el sistema.
Se vende "medidas y evidencia", nunca "certificación de nivel".

**El art. 50 contiene dos deberes distintos** que casi nadie está tratando por separado:
información **perceptible por la persona** y **marcado técnico legible por máquina**. Son dos
entregables diferentes, con destinatarios diferentes.

**El Omnibus simplifica los deberes documentales para pymes y small mid-caps.** La obligación
documental existe, pero en versión reducida. Eso es, literalmente, un producto de plantillas.

Además, proveedores y responsables del despliegue soportan apartados distintos del art. 50, por lo
que **clasificar el rol del cliente es el primer paso de cualquier servicio** de este tipo.

---

## 2. Punto de partida: LegalBox y Graphify

### 2.1 LegalBox, según fuentes secundarias

Ocean Legaltech, S.L., constituida en 2023, Moguer (Huelva).

- **Producto**: SaaS de cumplimiento RGPD para autónomos, micropymes, pymes y asesorías, que
  automatiza la creación y actualización de textos legales de protección de datos, comercio
  electrónico e IA.
- **Alcance**: política de privacidad, política de cookies, términos y condiciones, formularios web,
  registro de actividades de tratamiento, NDAs, contratos con terceros y protocolos. Más de 50
  documentos.
- **Promesa central**: textos "automatizados y actualizados de forma automática con cada
  actualización legislativa".
- **Planes**: desde 4,99 €/mes con plan gratuito. `Legal Box Pro` para menos de 3 usuarios con web;
  `Legal Box Business` para menos de 10 usuarios con web y venta online. Suscripción anual.
- **Servicios**: soporte comercial, técnico y jurídico, y figura de Delegado de Protección de Datos.
- **Canal**: presencia en catálogos de terceros, lo que sugiere venta indirecta vía asesorías.

**El dolor está en la promesa central.** "Actualizados automáticamente con cada cambio
legislativo" hoy se sostiene casi con seguridad a mano, con un jurista revisando novedades y
editando plantillas. Ese coste crece con plantillas × normas × configuraciones, es decir **crece
con el propio éxito comercial**. Es un problema de margen bruto, no de producto.

### 2.2 Graphify, y por qué encaja aquí

Convierte un corpus en un knowledge graph consultable. Python, Apache-2.0 con MIT para
contribuciones anteriores al recambio de licencia. Pipeline con un módulo por etapa: `detect.py` →
`extract.py` → `build.py` → `cluster.py` → `analyze.py` → `report.py` → `export.py` → `serve.py`.

Modelo de datos deliberadamente fino. Nodo: `id`, `label`, `file_type`, `source_file`,
`source_location`. Arista: `source`, `target`, `relation`, `confidence`, donde `confidence` es
`EXTRACTED` (explícito en el origen), `INFERRED` (resuelto) o `AMBIGUOUS` (incierto).

En el análisis previo, aplicar Graphify a Legaltech topaba con cuatro blockers. **Al dirigir la
herramienta al corpus de LegalBox, los cuatro desaparecen:**

| Blocker en Legaltech genérico                  | Por qué no aplica aquí                                                                        |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Secreto profesional: los documentos van al LLM | El corpus son **sus** plantillas y normativa **pública**. Ningún dato privilegiado de cliente |
| Techo práctico de 1.000 a 10.000 documentos    | 50-500 plantillas más unos cientos de artículos son unos pocos miles de nodos. Holgado        |
| Sin aislamiento multi-tenant                   | Herramienta interna, un proceso, su infraestructura                                           |
| Sin OCR, con fallo silencioso en PDF escaneado | Sus plantillas son texto digital, no escaneos                                                 |

Y hay un quinto punto que es el más importante: **el tier de provenance `EXTRACTED` sí es legítimo
en este montaje**. El mapeo norma→cláusula lo escribe un jurista y se parsea de forma
determinista, así que la cadena de trazabilidad no depende de aristas inferidas por un modelo. Es
el único escenario analizado en el que eso se cumple.

### 2.3 Lo que se descarta, y conviene que quede escrito

Los casos de **data room de M&A** y de **asset tracing / investigación** están fuera del ICP de
LegalBox y no deben reactivarse. Un autónomo que paga 4,99 €/mes no tiene data rooms ni litigios de
trazado de activos. Además, en el análisis previo el caso de investigación resultó inviable por
escala: un asunto grande son millones de documentos frente a un techo práctico de miles.

---

## 3. Parte 1: líneas de producto

Cinco líneas, ordenadas por tiempo hasta la primera factura. Las tres primeras son el núcleo de
compliance de IA.

### L1 · Legal Box IA Transparencia

**Vender ya. Campaña con fecha al 2 de diciembre de 2026.**

Encaja de forma natural con su modelo: add-on de suscripción sobre la base instalada, ticket bajo y
volumen alto. Sus clientes son casi siempre **responsables del despliegue** de IA de terceros:
chatbot de atención, generadores de copy, herramientas de marketing.

Contenido del módulo:

- **Inventario de sistemas de IA** del cliente. Es la pieza que habilita todo lo demás, y hoy
  prácticamente ninguna pyme lo tiene.
- **Clasificación de rol** por sistema: proveedor frente a responsable del despliegue.
- **Textos y avisos en las dos modalidades del art. 50**: aviso perceptible de interacción con IA
  en el chatbot, y **marcado técnico** del contenido sintético.
- **Cláusula de IA** en política de privacidad y aviso legal, y aviso en formularios.
- **Expediente de alfabetización del art. 4**: plan de medidas y registro de formación, redactado
  como medidas y no como certificación de nivel.
- **Cláusulas con proveedores de IA**: encargo del tratamiento más reparto de responsabilidades
  del AI Act.

El gancho es el **2 de diciembre de 2026**: fecha cierta, a unas doce semanas, y afecta a cualquier
cliente que publique contenido generado con IA.

### L2 · Legal Box IA Evidencia

**La línea diferenciada, y la que rompe el techo de precio.**

Es la capa de trazabilidad de la Parte 5. Se factura por sistema monitorizado o por volumen, no por
plantilla, y eso les saca del techo de 4,99 €/mes.

Se vende hoy por cuatro motivos, y solo tres son obligación inmediata: prueba de cumplimiento del
art. 50; RGPD art. 22 y 15.1.h en decisiones automatizadas; **presión de compras**, porque los
clientes grandes de sus clientes ya exigen garantías de IA por contrato; y preparación para
diciembre de 2027.

### L3 · Legal Box IA Alto Riesgo

**Programa con horizonte a diciembre de 2027.**

Para el subconjunto de clientes realmente en Anexo III. El segmento accesible más común en pymes es
**selección de personal y gestión de empleo**; le siguen crédito, seguros, educación y biometría.

Contenido: evaluación de brecha contra Anexo III, evaluación de impacto en derechos fundamentales
del art. 27, documentación técnica en la versión simplificada para pymes, y preparación de
conformidad. Consultoría más suscripción, con dieciséis meses de recorrido para venderlo sin prisa.

### L4 · Canal para asesorías

**Multiplicador comercial, y el único punto donde vuelve el riesgo técnico serio.**

Su canal indirecto ya existe. Un panel multi-cliente para asesorías que gestionan muchas pymes
multiplica el alcance sin coste de adquisición.

Pero **aquí reaparece el blocker que en el resto del diseño desaparecía**. Graphify no tiene modelo
de autorización: `project_path` lo aporta quien llama, `_resolve_graph_path()` lo resuelve sin
allowlist, y `security.py` valida rutas y URLs pero no autoriza. Un panel multi-cliente mal
construido es una fuga de información entre clientes de la asesoría.

> **Regla no negociable:** un proceso y un almacén por cliente, autorización en el proxy inverso,
> y nunca un `serve` compartido entre clientes.

### L5 · Legal Box Norma

**Motor interno primero, producto después.**

Es el motor de la Parte 4. Nace como herramienta interna que sostiene la promesa de "textos
actualizados con cada cambio legislativo" y, una vez maduro, es vendible a otras legaltech y
asesorías como gestión de cambio normativo.

Es el activo que más se aprecia con el tiempo, porque la librería de mapeos norma→cláusula se
enriquece con cada cliente y con cada novedad legislativa.

---

## 4. Parte 2: el motor de la arista esperada

### 4.1 El problema, en una frase

**Graphify construye el grafo de lo que _está_ enlazado. El cumplimiento necesita el grafo de lo
que _debe_ estar enlazado.**

Graphify no tiene noción de arista esperada ni lenguaje de aserción en ningún punto de su pipeline.
El motor es la diferencia entre ambos grafos. **Es IP de LegalBox, no de Graphify**, y por eso debe
vivir en un paquete propio que importa `graphify`, nunca en un fork.

### 4.2 El DSL de expectativas

YAML escrito por jurista y versionado en git. Cada fichero declara una o varias aristas esperadas.

```yaml
# expectativas/rai-art50-chatbot.exp.yaml
id: exp.rai.art50.1.aviso_interaccion
norma:
  ref: 'RAI:art:50.1' # Reglamento (UE) 2024/1689
  vigencia_desde: 2026-08-02
titulo: 'Aviso perceptible de interacción con un sistema de IA'

aplica_si: # predicado de alcance
  todas:
    - {campo: cliente.sistemas_ia, op: no_vacio}
    - {campo: sistema.interactua_con_personas, op: eq, valor: true}
    - {
        campo: sistema.rol_cliente,
        op: en,
        valor: [responsable_despliegue, proveedor],
      }

espera: # LA ARISTA ESPERADA
  - desde: {legal_type: norma, ref: 'RAI:art:50.1'}
    relacion: implementada_por
    hasta:
      legal_type: clausula
      etiquetas_contiene: [aviso_ia_interaccion]
      en_plantilla_en: [politica_privacidad, aviso_legal, widget_chatbot]
    cardinalidad: '>=1'
    confianza_minima: EXTRACTED
    max_saltos: 3

prohibe: # expectativa negativa
  - desde: {legal_type: cliente}
    relacion: declara
    hasta: {legal_type: clausula, etiquetas_contiene: [no_usamos_ia]}
    motivo: 'Contradice el inventario de sistemas de IA del cliente'

severidad: bloqueante
remedio:
  plantilla: clausulas/aviso-ia-interaccion.md
  responsable: juridico
```

Las **expectativas negativas** son baratas de implementar y muy valiosas: detectan la contradicción
entre lo que el cliente declara y lo que su propio inventario dice. Es una clase de error que hoy
nadie detecta porque requiere cruzar dos documentos distintos.

### 4.3 Semántica de evaluación: cuatro estados, nunca un booleano

Es la decisión de diseño más importante del sistema, y viene **forzada por un hallazgo verificado**:
para contenido extraído por LLM de documentos, el tier `EXTRACTED` no se puebla nunca, porque
`EXTRACTED` significa parseo determinista con score 1.0 y en un PDF no hay parseo determinista.

Un motor booleano, por tanto, tendría solo dos comportamientos posibles: marcar todo como
incumplimiento, o **blanquear conjeturas del modelo como cumplimiento**. Ninguno es defendible ante
un cliente, y el segundo es exposición de responsabilidad profesional.

| Estado      | Cuándo                                                                                                                         | Quién actúa                                 |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------- |
| `CUMPLE`    | Existe camino con confianza ≥ umbral y firma humana vigente                                                                    | Nadie                                       |
| `BRECHA`    | No existe camino                                                                                                               | Jurídico, con remedio propuesto             |
| `REVISIÓN`  | Hay camino pero solo por aristas `INFERRED` o `AMBIGUOUS`, o por debajo del umbral, o la plantilla cambió tras la última firma | **Jurídico. Es la superficie del producto** |
| `NO_APLICA` | El predicado de alcance es falso                                                                                               | Nadie, pero se registra el motivo           |

`REVISIÓN` no es un estado de error: **es la cola de trabajo del jurista**, y ahí está el valor del
producto. Convierte "reléete las 50 plantillas" en "revisa estas 7 aristas".

### 4.4 El algoritmo, y el detalle que decide si funciona

```python
def evaluar(G, exp, contexto):
    if not cumple_alcance(exp.aplica_si, contexto):
        return NO_APLICA

    # EL PUNTO CRÍTICO: evaluar sobre una vista filtrada, no sobre el grafo completo
    H = nx.subgraph_view(
        G,
        filter_edge=lambda u, v: (
            G.edges[u, v].get('relation') in exp.relaciones_permitidas
            and rango_confianza(G.edges[u, v].get('confidence')) >= exp.confianza_minima
        ),
    )

    origen = resolver_id(exp.espera.desde)
    candidatos = [n for n in H.nodes if cumple_predicado(H.nodes[n], exp.espera.hasta)]

    caminos = []
    for c in candidatos:
        if nx.has_path(H, origen, c):
            p = nx.shortest_path(H, origen, c)
            if len(p) - 1 <= exp.espera.max_saltos:
                caminos.append(p)

    return clasificar(caminos, exp)
```

**Sin el filtrado previo de aristas el motor no sirve para nada.** Sobre el grafo completo,
cualquier par de nodos acaba conectado a través de nodos hub y entonces _todo_ cumple. Es el mismo
fenómeno que en el análisis de Graphify hacía que su función `god_nodes()` devolviera `Contrato`,
`Parte` o `Sociedad`: los hubs dominan la topología y ahogan la señal.

Dos detalles de implementación que importan:

- `nx.subgraph_view` con `filter_edge` devuelve una **vista perezosa**, así que no duplica el grafo
  en memoria.
- El **límite de saltos** es imprescindible: un camino de nueve aristas no es cumplimiento, es
  coincidencia topológica.

### 4.5 Cómo se monta sobre las restricciones reales de Graphify

Todas verificadas en la rama `v8`:

- **`file_type` es un enum cerrado**: `code`, `document`, `paper`, `image`, `rationale`, `concept`.
  No se puede añadir `norma` ni `clausula`. Los nodos jurídicos van como `document` y el tipo real
  en una clave propia `legal_type`. En cambio **`relation` es libre**, así que `implementada_por`,
  `deriva_de`, `deroga` o `declara` son válidos sin tocar nada.
- **Los atributos extra sobreviven**: `build.py` hace
  `G.add_node(node['id'], **{k: v for k, v in node.items() if k != 'id'})` y `validate.py` tolera
  claves desconocidas. Pero `security.py` limita el label a 256 caracteres, los valores a 512 y las
  listas a 50 elementos, y **escapa HTML**, lo que mancha nombres que contengan `&`.
- **`normalize_id()` es lossy**: pasa a minúsculas, normaliza NFKC y sustituye todo lo que no sea
  `\w`. Así `art. 50` y `art 50` colisionan, y `art. 50 RAI` frente a `art. 50 RGPD` se fusionan en
  silencio. **Hay que namespacear** con `make_id(norma, tipo, referencia)`. Sin esto, dos artículos
  distintos acaban siendo el mismo nodo.
- **Trampa de precedencia, y es la que anularía todo el sistema**: `.yaml` y `.yml` ya están en
  `DOC_EXTENSIONS`, así que sin un guard explícito en `detect.py` y `collect_files()` los ficheros
  `.exp.yaml` se irían a la pasada semántica con LLM, y el mapeo determinista pasaría a ser
  conjetura del modelo. Con eso se perdería exactamente la defendibilidad que justifica el
  proyecto.
- **Alta del extractor: son cuatro ediciones**, según `extractors/MIGRATION.md`. Registro en
  `extractors/__init__.py`, re-export de fachada en `extract.py`, sufijo en los dispatch de
  `extract()` y de `collect_files()`, y extensión en `detect.py`; más dependencia en
  `pyproject.toml` y test. Al implementar hay que confirmar si el test va en
  `tests/test_extractors_registry.py` o en `tests/test_languages.py`, porque `ARCHITECTURE.md` y
  `MIGRATION.md` discrepan.
- **Side-car SQLite indexado por `id` de nodo** para todo lo que sea fecha o rango: `vigencia_desde`,
  `vigencia_hasta`, plazos, umbrales, firmas e histórico de evaluaciones. Nada en Graphify indexa ni
  consulta atributos por rango, y **el calendario del AI Act es exactamente eso**.

### 4.6 Gobernanza, que es lo que lo hace defendible

Graphify no tiene registro de auditoría de grado legal. Su `querylog.py` es opcional y con forma de
consulta, no de evidencia, y `manifest.py` es solo un punto de partida. Hay que añadir:

- **Log append-only de evaluaciones**: qué versión de expectativa, qué versión de plantilla, qué
  veredicto y qué firma humana.
- **Firma con caducidad implícita.** Cada `CUMPLE` referencia una firma con autor y fecha. Si la
  plantilla cambia después, el estado **cae automáticamente a `REVISIÓN`**. Ese decaimiento
  automático es, técnicamente, la pieza que sostiene la promesa comercial de "actualizado con cada
  cambio legislativo".
- **Reproducibilidad demostrable.** El `cluster.py` de Graphify va con semilla y tiene cadena de
  respaldo determinista, así que un re-run completo da idénticos recuentos de nodos, aristas y
  comunidades. Es un argumento fuerte ante un auditor y conviene medirlo desde el primer día.

### 4.7 El papel del LLM, deliberadamente estrecho

El modelo **propone** candidatos de mapeo para que un jurista los acepte, y con ello redacta
borradores de ficheros de expectativa. **Nunca emite el veredicto.**

El veredicto es evaluación determinista de grafo más firma humana, porque tiene que ser
reproducible y atribuible. Un veredicto de modelo no es auditable, y la auditabilidad _es_ el
producto.

---

## 5. Parte 3: la capa de trazabilidad, más allá de Graphify

### 5.1 La tesis

Graphify responde **qué norma exige qué**. Una capa de observabilidad responde **qué hizo realmente
el sistema de IA**. El producto es la **unión de ambas**: la traza de una decisión de IA, enlazada
al artículo que la gobierna, a la cláusula que la comunica y a la evidencia conservada. En el
segmento MYPE eso no lo ofrece nadie.

### 5.2 Una corrección necesaria: trazabilidad, no interpretabilidad

Para una MYPE que despliega IA de terceros **no hay interpretabilidad del modelo, y el AI Act
tampoco la exige**. El art. 86 habla de explicar el papel del sistema en el procedimiento decisorio
y los elementos principales de la decisión: es **transparencia procedimental**, no mecanicista.

Lo alcanzable y lo exigible es **trazabilidad**: qué entró, qué salió, cuándo, con qué versión de
modelo y de prompt, y quién supervisó.

> Vender "explicamos el modelo" sería insostenible en cuanto un cliente lo audite.
> Vender "reconstruimos y probamos la decisión" es cierto, verificable y suficiente para la norma.

### 5.3 Stack recomendado

**Formato de cable: OpenTelemetry GenAI semantic conventions.** Estándar respaldado por la CNCF y
desarrollado por el GenAI SIG desde abril de 2024, con esquema para llamadas a LLM, pasos de agente,
consultas a bases vectoriales, uso de tokens y coste. Lo adoptan Google Cloud, AWS, Azure y Datadog.
Elegirlo evita lock-in de proveedor, que en un producto de compliance con vida de varios años es un
requisito y no una preferencia.

**Almacén de trazas: Langfuse autoalojado, no LangSmith.** Langfuse tiene **núcleo MIT** y en junio
de 2025 pasó a MIT todas las funciones de producto: tracing, prompts, evals, playground y colas de
anotación. El self-hosting está disponible en todos los niveles, incluido el gratuito, con SSO y
RBAC de organización en la distribución OSS. Ingesta **OTLP sobre HTTP**, en JSON y protobuf; gRPC
todavía no está soportado.

LangSmith es propietario y SaaS estadounidense, lo que para un proveedor español de cumplimiento
RGPD es el encaje equivocado. Autoalojar en la UE resuelve residencia del dato y coste a la vez.

> **Aviso, con ironía incorporada:** los módulos de **audit logging y políticas de retención de
> datos requieren licencia comercial** al autoalojar Langfuse. Son justo dos piezas que un producto
> de evidencia necesita. Hay que presupuestar la licencia o implementar retención y log de auditoría
> por fuera, pero no se puede planificar como si todo fuera gratuito.

**Orquestación y supervisión humana: LangGraph.** Sus primitivas de checkpointing e interrupción con
humano en el bucle encajan con el art. 14 y, sobre todo, **son la misma cola de `REVISIÓN`** del
motor de la Parte 2. El grafo de estado _es_ el rastro de auditoría, así que una sola pieza sirve al
flujo interno del jurista y a la evidencia del cliente.

**Evals periódicos** para el art. 15, exactitud y robustez, conservando resultados como evidencia
fechada.

### 5.4 El mapeo obligación → artefacto técnico

Es el contenido central de esta línea, y su valor está en **separar lo exigible hoy de lo que da
recorrido comercial**.

| Obligación                                                    | Artefacto técnico                                   | Exigible       |
| ------------------------------------------------------------- | --------------------------------------------------- | -------------- |
| Art. 50.1, aviso perceptible de interacción                   | Evento de traza que acredite que el aviso se mostró | **Ya**         |
| Art. 50.2, marcado legible por máquina de contenido sintético | Marca y hash en la traza de generación              | **2 dic 2026** |
| Art. 4, alfabetización                                        | Registro de medidas y de formación por usuario      | **Ya**         |
| RGPD art. 22 y 15.1.h, decisiones automatizadas               | Traza reconstruible más lógica aplicada             | **Ya**         |
| Art. 26.6, conservación de logs                               | Retención de trazas                                 | dic 2027       |
| Art. 14, supervisión humana                                   | Interrupciones y decisiones humanas en LangGraph    | dic 2027       |
| Art. 15, exactitud y robustez                                 | Resultados de evals periódicos                      | dic 2027       |
| Art. 86, derecho a explicación                                | Traza reconstruible por decisión individual         | dic 2027       |

Cuatro filas ya son exigibles. Las cuatro de diciembre de 2027 son el argumento de suscripción
plurianual.

### 5.5 Riesgos de esta línea, sin adornos

**Es un producto mucho más grande que el de plantillas.** Infraestructura, almacenamiento,
retención, seguridad y guardias. Es casi otra empresa. Conviene entrar por L1 y L5 y financiar L2
con esos ingresos, no arrancar por aquí.

**Guardar prompts y salidas es tratar datos personales.** LegalBox pasaría a ser **encargado del
tratamiento** de sus clientes, con contrato de encargo, política de retención y probablemente su
propia evaluación de impacto. Una herramienta de cumplimiento que genera obligaciones nuevas al
proveedor es un riesgo real; hay que diseñarla con minimización y seudonimización desde el primer
día.

**Límite de integración, y acota mucho el mercado.** Capturar trazas exige instrumentar el uso de
IA. Si la pyme usa ChatGPT en el navegador **no hay nada que instrumentar**. Solo funciona donde la
IA está embebida en una aplicación o se consume por API: chatbot en web, e-commerce con
recomendador, asesoría con IA en su flujo, o cliente que es a su vez SaaS. Ese es el segmento
accesible real de L2, y es bastante menor que su base instalada.

**Trampa del art. 25, que además es oportunidad de upsell.** Si el cliente pone su marca en un
sistema de IA o lo modifica sustancialmente, **se convierte en proveedor** y asume obligaciones
mucho más pesadas. Detectarlo en el inventario de L1 es a la vez un servicio y un disparador de
venta hacia L3.

---

## 6. Hoja de ruta y precios

| Fase | Contenido                                                               | Plazo        | Precio           |
| ---- | ----------------------------------------------------------------------- | ------------ | ---------------- |
| 0    | Verificar qué tiene ya LegalBox de IA y cerrar el punto ciego de la web | 1 semana     | Sin coste        |
| 1    | L1 Transparencia más L5 motor, con el art. 50 como primera expectativa  | 4-6 semanas  | 6-15k€           |
| 2    | Campaña del 2 de diciembre sobre la base instalada                      | oct-nov 2026 | Ingreso de ellos |
| 3    | L2 Evidencia, piloto con 3-5 clientes instrumentables                   | Q1 2027      | Por definir      |
| 4    | L3 Alto Riesgo, con horizonte a dic 2027                                | 2027         | Consultoría      |

### Piloto de la fase 1

**Alcance mínimo creíble:** el art. 50 del AI Act más el calendario del Reglamento 2026/1744,
contra las plantillas de política de privacidad, aviso legal y términos y condiciones, en los planes
Pro y Business.

**Secuencia:** normativa y plantillas a markdown → extractor de expectativas con el guard de
precedencia → grafo compuesto con `merge-graphs` → motor de evaluación → cola de `REVISIÓN` →
`watch` y hooks de git para reejecución al cambiar una plantilla.

**Criterios de aceptación:**

- Toda plantilla afectada por el art. 50 queda identificada, y un jurista de LegalBox confirma la
  lista.
- **Cero falsos negativos.** Ninguna plantilla afectada queda fuera. Es la puerta dura, porque un
  falso negativo es un cliente incumpliendo.
- Al menos una dependencia norma-plantilla no documentada hasta ahora sale a la luz.
- Reproducibilidad: un re-run completo da idénticos recuentos de nodos, aristas y comunidades.
- Tiempo de actualización incremental por debajo de un umbral acordado, apoyado en la caché
  semántica de Graphify.

El argumento de venta de la fase 1 **no es cumplimiento, es margen bruto**: automatizar el
mantenimiento normativo abarata la única partida que escala con su base de clientes.

---

## 7. Riesgos transversales y licencias

**Licencia de Graphify.** Apache-2.0, con MIT para contribuciones anteriores al recambio. La
reventa como servicio está permitida, sin copyleft y con concesión de patente. Al **distribuir**
(instalación on-premise, contenedor, CLI) hay que incluir `LICENSE`, `LICENSE-MIT` y `NOTICE` y
marcar los ficheros modificados, conforme al §4.b. El §6 no concede derechos de marca: no se puede
comercializar como "Graphify-algo".

**El upstream es competidor.** Graphify Labs está lanzando su propia plataforma comercial. El moat
de LegalBox no puede ser el grafo: tiene que ser la **librería de expectativas** y el **mapeo
norma→artefacto**. De ahí la regla de arquitectura: todo el desarrollo va como paquete propio que
importa `graphify`, y los extractores se escriben de forma que sean upstreamables. Nunca un fork,
que además genera deuda de merge.

**Dos supuestos pendientes de verificar en fuente**, ambos relevantes:

1. Si `serve.py` implementa realmente el flag `--api-key` con autenticación Bearer. No pudo
   confirmarse en la lectura remota. No prometer autenticación sin comprobarlo.
2. Si `graphify query` es totalmente local. La documentación de benchmarks apunta a embeddings
   locales, pero no se verificó. Es una cuestión relevante para confidencialidad.

**Base probatoria de la extracción.** Los benchmarks publicados de Graphify (LOCOMO,
LongMemEval-S) miden QA conversacional, no recuperación documental jurídica, y la cifra de acierto
es del 45,3%. **No hay base publicada sobre precisión de extracción de cláusulas.** Es otra razón
por la que el veredicto tiene que ser determinista y con firma humana, y no un juicio del modelo.

---

## Fuentes

**Normativa y calendario**

- [Reglamento (UE) 2026/1744 — EUR-Lex](https://eur-lex.europa.eu/eli/reg/2026/1744/oj/eng)
- [AI Omnibus enters into force — Comisión Europea](https://digital-strategy.ec.europa.eu/en/news/ai-omnibus-enters-force)
- [Council gives final green light to simplify and streamline rules — Consejo](https://www.consilium.europa.eu/en/press/press-releases/2026/06/29/artificial-intelligence-council-gives-final-green-light-to-simplify-and-streamline-rules/)
- [Council and Parliament agree to simplify and streamline rules — Consejo](https://www.consilium.europa.eu/en/press/press-releases/2026/05/07/artificial-intelligence-council-and-parliament-agree-to-simplify-and-streamline-rules/)
- [MEPs support postponement of certain rules on AI — Parlamento Europeo](https://www.europarl.europa.eu/news/en/press-room/20260316IPR38219/meps-support-postponement-of-certain-rules-on-artificial-intelligence)
- [Digital Omnibus on AI — EU Artificial Intelligence Act](https://artificialintelligenceact.eu/ai-act-explorer/digital-omnibus/)
- [Europa reescribe el AI Act: el Ómnibus de IA ya es ley — Economist & Jurist](https://www.economistjurist.es/articulos-juridicos-destacados/europa-reescribe-el-ai-act-el-omnibus-de-ia-ya-es-ley-y-la-transparencia-no-espera/)

**LegalBox**

- [Legal Box Plus — web del producto](https://legalbox.plus/en/) (no accesible desde la sesión)
- [Legal Box Plus — LinkedIn](https://es.linkedin.com/company/legalboxplus-plus)
- [Legal Box Plus — Derecho Práctico, guía Legaltech](https://derechopractico.es/guialegaltech/legal-box-plus/)
- [Ocean Legaltech S.L. — Axesor](https://www.axesor.es/Informes-Empresas/10812397/OCEAN_LEGALTECH_SL.html)

**Stack técnico**

- [Graphify — repositorio](https://github.com/Graphify-Labs/graphify)
- [Langfuse — repositorio](https://github.com/langfuse/langfuse)
- [Langfuse — licencia en self-hosting](https://langfuse.com/self-hosting/license-key)
- [Langfuse — integración OpenTelemetry](https://langfuse.com/integrations/native/opentelemetry)
- [OpenTelemetry — GenAI observability](https://opentelemetry.io/blog/2026/genai-observability/)

---

_Análisis a 9 de septiembre de 2026. El calendario del AI Act cambió el 27 de julio de 2026;
verificar vigencia antes de uso comercial._
