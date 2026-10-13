# ADR-002 · Concurrencia optimista sobre los ejemplares

- **Estado:** aceptada
- **Contexto:** dos bibliotecarios pueden prestar el **último ejemplar** del mismo libro al mismo tiempo (RNF-2). Si ambos leen
  `available_copies = 1` y ambos escriben, el sistema presta un libro que no tiene.
- **Decisión:** control de concurrencia **optimista**. El libro lleva un testigo de versión; al guardar, si la versión cambió, la
  operación falla y el servidor responde **409 `BOOK_NOT_AVAILABLE`** (o reintenta una vez si el ejemplar sigue disponible).
  La **última barrera** es el `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Alternativas descartadas:** bloqueo pesimista de la fila (`SELECT … FOR UPDATE`) en cada préstamo: serializa toda la biblioteca por
  un caso raro; contador en memoria: no sobrevive a más de un proceso.
- **Consecuencias:** si el `CHECK` salta, **algo escribió fuera del adaptador**: es un defecto, no un caso de negocio. Devolver dos veces
  la misma línea no suma dos veces porque `returned_at` ya está lleno (CA-07.2).
- **Cómo se verifica:** prueba concurrente con dos peticiones simultáneas sobre un libro con un ejemplar: exactamente una recibe 201.
- **Incongruencia detectada (se registra, no se corrige para no cambiar el alcance):**
  - La decisión exige un **testigo de versión** por libro (también [`plan.md`](../plan.md) D-04 y T-13 de [`tasks.md`](../tasks.md)),
    pero la tabla `book` de [`data-model.md`](../data-model.md) §3 no tiene ninguna columna de versión, y ese documento declara que
    gana sobre los demás. **Falta** la columna del testigo (nombre, tipo y regla) en el modelo de datos.
  - «Responde 409 (o reintenta una vez si el ejemplar sigue disponible)» es ambiguo: **falta** definir cuándo se reintenta y
    cuándo se responde 409 directamente.
