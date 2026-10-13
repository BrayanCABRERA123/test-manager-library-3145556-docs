# ADR-006 · La portada se sube con POST y `_method=PUT`

- **Estado:** aceptada
- **Contexto:** el contrato ([`api-contract.md`](../api-contract.md) §3.4) define `PUT /books/{id}/cover` con `multipart/form-data`. PHP
  solo procesa archivos `multipart` en peticiones `POST`: en un `PUT` el archivo no llega a `$request->file('file')`.
- **Decisión:** la ruta sigue siendo `PUT /books/{id}/cover`, y el cliente la invoca con `POST` y el campo `_method=PUT` (suplantación de
  método de Laravel). El contrato no cambia: misma ruta, mismo campo `file`, mismas validaciones (JPG/PNG por contenido, máx. 2 MB).
- **Alternativas descartadas:** cambiar la ruta a `POST` (rompe el contrato); leer y parsear el cuerpo `multipart` a mano en un `PUT`
  (código frágil que no aporta valor).
- **Consecuencias:** el frontend envía la portada con `POST` + `_method=PUT`; cualquier otro cliente que envíe un `PUT` puro recibe 422
  porque el archivo no llega.
- **Cómo se verifica:** subir un PNG de 2 MB con `POST` + `_method=PUT` guarda `cover_key`; uno de 3 MB responde 422 (T-15).
