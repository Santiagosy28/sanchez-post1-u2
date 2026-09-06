# sanchez-post1-u2
"Post-contenido — Exportación de reportes académicos con patrones creacionales justificados"
## Decisiones de diseño

### Decisión 1 — Factory Method vs. Abstract Factory (Parte 1)

**Patrón elegido:** Abstract Factory

**Justificación:** El problema exige crear, para cada formato de exportación, dos productos relacionados —el cuerpo del reporte (`ReportBody`) y el encabezado/pie de página (`ReportHeaderFooter`)— que deben pertenecer obligatoriamente al mismo formato entre sí. No es un caso de un único producto que varía (lo que llevaría a Factory Method), sino de una familia completa que debe mantenerse coherente. Esto se confirma al proyectar el formato futuro (CSV): agregarlo no implicaría una sola implementación nueva, sino una familia completa (`CsvReportBody` + `CsvHeaderFooter`). El riesgo real que el problema plantea no es "instanciar la clase equivocada" de forma aislada, sino "mezclar piezas de familias distintas" (por ejemplo, un cuerpo en Excel con un encabezado en PDF), que es precisamente el riesgo que Abstract Factory está diseñado para evitar mediante una única fábrica (`ReportFormatFactory`) que produce ambas piezas coordinadas.

### Decisión 2 — Mecanismo de extensibilidad de formatos (Parte 1)

**Opción elegida:** Registro dinámico con `Map<String, Supplier<ReportFormatFactory>>`

**Justificación:** Un `switch` o cadena de `if/else` sobre el string de formato obligaría a modificar ese método cada vez que se agregue un formato nuevo (como el CSV planeado), violando el principio Abierto/Cerrado. El registro dinámico (`ReportFactoryRegistry`) permite añadir un formato nuevo simplemente llamando a `register()` con su Supplier correspondiente, sin tocar el código existente ni el método `resolve()`.