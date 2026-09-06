# sanchez-post1-u2
"Post-contenido — Exportación de reportes académicos con patrones creacionales justificados"
## Decisiones de diseño

### Decisión 1 — Factory Method vs. Abstract Factory (Parte 1)

**Patrón elegido:** Abstract Factory

**Justificación:** El problema exige crear, para cada formato de exportación, dos productos relacionados —el cuerpo del reporte (`ReportBody`) y el encabezado/pie de página (`ReportHeaderFooter`)— que deben pertenecer obligatoriamente al mismo formato entre sí. No es un caso de un único producto que varía (lo que llevaría a Factory Method), sino de una familia completa que debe mantenerse coherente. Esto se confirma al proyectar el formato futuro (CSV): agregarlo no implicaría una sola implementación nueva, sino una familia completa (`CsvReportBody` + `CsvHeaderFooter`). El riesgo real que el problema plantea no es "instanciar la clase equivocada" de forma aislada, sino "mezclar piezas de familias distintas" (por ejemplo, un cuerpo en Excel con un encabezado en PDF), que es precisamente el riesgo que Abstract Factory está diseñado para evitar mediante una única fábrica (`ReportFormatFactory`) que produce ambas piezas coordinadas.

### Decisión 2 — Mecanismo de extensibilidad de formatos (Parte 1)

**Opción elegida:** Registro dinámico con `Map<String, Supplier<ReportFormatFactory>>`

**Justificación:** Un `switch` o cadena de `if/else` sobre el string de formato obligaría a modificar ese método cada vez que se agregue un formato nuevo (como el CSV planeado), violando el principio Abierto/Cerrado. El registro dinámico (`ReportFactoryRegistry`) permite añadir un formato nuevo simplemente llamando a `register()` con su Supplier correspondiente, sin tocar el código existente ni el método `resolve()`.

### Decisión 3 — Builder vs. constructor telescópico vs. setters (Parte 2)

**Opción elegida:** Builder

**Justificación:** Un constructor con los 9 parámetros de `ExportConfig` obligaría al cliente a recordar el orden exacto de argumentos, varios del mismo tipo (`String`, `boolean`), con alto riesgo de invertirlos sin que el compilador lo detecte. Constructores sobrecargados por cada combinación común escalarían mal, dado que 8 parámetros opcionales generan demasiadas combinaciones razonables. Setters sueltos permitirían que el objeto quedara en un estado a medio configurar, sin un punto único donde validar la consistencia (por ejemplo, pedir `compress=true` sin indicar `outputPath`). El Builder resuelve las tres limitaciones: expone métodos encadenables legibles por nombre, y centraliza la validación de estado en `build()`, antes de que exista un objeto inconsistente.

### Decisión 4 — ¿ReportFactoryRegistry necesita ser Singleton? (Parte 2)

**Conclusión:** NO conviene convertirlo en Singleton.

**Justificación:** Aplicando los criterios de la Guía Teórica, `ReportFactoryRegistry` no necesita identidad de objeto (nunca se sustituye por un mock ni se inyecta por constructor en este proyecto; se usa siempre por acceso estático) y su inicialización no es costosa (un `Map` con tres entradas en un bloque `static`, sin trabajo de carga que justifique inicialización perezosa). Además, el propio campo `static final Map` ya garantiza una única fuente de verdad compartida por toda la JVM, sin necesitar la maquinaria adicional de Singleton (constructor con guardas + `getInstance()`). No existe en este proyecto ningún escenario futuro realista (como un registro independiente por institución en una plataforma multi-tenant) que requeriría múltiples registros. Por estas razones, el diseño actual —clase final, constructor privado, solo miembros estáticos— ya es la solución correcta, y convertirlo en Singleton solo añadiría ceremonia sin resolver ningún problema real.
## Conclusiones
Este post-contenido me sirvió para entender que elegir un patrón creacional no es cuestión de memorizar cuál "se usa más", sino de analizar bien el problema antes de programar. Al principio pensé que Factory Method y Abstract Factory eran casi lo mismo, pero al tener que justificar por qué descarté uno, me di cuenta de que la diferencia está en si manejas un solo producto o una familia completa que tiene que mantenerse coherente. Con el Builder fue parecido: entendí que no se trata solo de "verse más limpio" que un constructor largo, sino de poder validar que el objeto quede en un estado correcto antes de crearlo. Y con lo de Singleton, lo más útil fue aprender a no aplicarlo por costumbre — antes hubiera dicho que un registro central "obviamente" debía ser Singleton, pero analizando los criterios (si necesito identidad de objeto, si la inicialización es costosa, etc.) me di cuenta de que muchas veces ya tienes la garantía que necesitas sin toda esa ceremonia extra.