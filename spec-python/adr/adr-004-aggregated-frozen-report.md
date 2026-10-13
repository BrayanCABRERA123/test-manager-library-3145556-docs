# ADR-004 · El reporte se calcula en el motor y agrupa por valores congelados

- **Estado:** aceptada
- **Contexto:** el reporte de más prestados (HU-10) cruza `loan` y `loan_item` sobre un rango. Si leyera el título o la categoría del catálogo
  vivo, renombrar un libro o una categoría cambiaría un reporte de un periodo ya cerrado.
- **Decisión:** (1) el reporte **no se persiste**: se calcula con una sola consulta agregada por un puerto de lectura. (2) Agrupa por
  `book_id, book_title, category_name` **tal como quedaron congelados en la línea**. (3) `loan_item.category_name` **no tiene FK**
  a propósito. (4) Si un libro cambió de categoría dentro del rango, el reporte muestra **dos filas**: no elige un ganador.
- **Alternativas descartadas:** unir con el catálogo vivo (el reporte deja de ser estable); tomar «el valor más reciente del rango»
  (un préstamo nuevo cambiaría lo ya leído); materializar una tabla de reporte (segunda fuente de verdad).
- **Consecuencias:** CA-10.1 dice «una fila por libro y nombre de categoría congelado», no «una fila por libro». El reporte no
  incluye socios ni bibliotecarios (privacidad).
- **Cómo se verifica:** prestar un libro, renombrarlo, pedir el reporte del mismo rango: el título del reporte es el viejo.
- **Incongruencia detectada (se registra, no se corrige para no cambiar el alcance):**
  - La decisión agrupa por `book_id, book_title, category_name` (igual que [`api-contract.md`](../api-contract.md) §3.7), pero
    CA-10.1 de [`spec.md`](../spec.md) y las consecuencias de este ADR dicen «una fila por libro y nombre de categoría congelado»,
    sin el título. **Falta** decir que renombrar un libro dentro del rango también produce dos filas, no solo cambiarlo de categoría.
  - El reporte recibe `from` y `to` como **fechas** «ambos inclusive», pero `loaned_at` es un **instante** UTC. **Falta** definir el
    límite exacto del rango (por ejemplo, si `to` incluye todo ese día hasta las 23:59:59 UTC).
