# ADR-003 · Los libros se dan de baja, no se borran

- **Estado:** aceptada
- **Contexto:** una línea de préstamo apunta a un libro. Si el libro se borra físicamente, el historial y el reporte quedan rotos.
- **Decisión:** el libro lleva `deleted_at`. Dar de baja lo llena; el catálogo filtra por `deleted_at IS NULL` en un único lugar
  (filtro global). Un libro con préstamos abiertos **no** se da de baja (409 `BOOK_HAS_OPEN_LOANS`). La unicidad del ISBN es un
  índice **parcial** sobre libros activos, así un ISBN dado de baja puede volver a registrarse.
- **Alternativas descartadas:** borrado físico con cascada (destruye el histórico); marcar `active = false` (no dice cuándo).
- **Consecuencias:** `loan_item.book_id` lleva una FK `RESTRICT` como **barrera de última instancia**: un `DELETE` manual o un cambio de
  código futuro fallan ruidosamente en vez de corromper el histórico. El binario de la portada es el único dato que sí se borra físicamente.
- **Cómo se verifica:** consultar un libro dado de baja devuelve 404; un `DELETE` por `psql` sobre un libro con líneas falla por la FK.
- **Incongruencia detectada (se registra, no se corrige para no cambiar el alcance):**
  - Este ADR y CA-03.5 de [`spec.md`](../spec.md) hablan de «préstamos **abiertos**», pero en [`api-contract.md`](../api-contract.md)
    `open` es un estado calculado distinto de `overdue`. **Falta** precisar si un libro con líneas vencidas sin devolver
    (`overdue`) también impide la baja.
  - **Falta** definir qué pasa con la portada al dar de baja el libro: [`data-model.md`](../data-model.md) §7 solo dice que el
    binario es el único dato que se borra físicamente, no si la baja lógica lo borra.
