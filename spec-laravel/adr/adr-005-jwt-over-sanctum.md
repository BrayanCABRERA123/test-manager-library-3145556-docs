# ADR-005 · Autenticación con JWT propio y no con Sanctum

- **Estado:** aceptada
- **Contexto:** el contrato ([`api-contract.md`](../api-contract.md) §4) exige un **JWT firmado**, con vigencia de 8 horas y con `sub` y
  `role` dentro, y la respuesta del login incluye `expires_in: 28800`. Laravel trae Sanctum, que emite **tokens opacos** guardados en una
  tabla `personal_access_tokens`.
- **Decisión:** emitir y validar el JWT (HS256, clave en `JWT_SECRET`) con `firebase/php-jwt`, detrás del puerto `TokenIssuer`. Un
  middleware propio valida el token y el rol en cada ruta protegida.
- **Alternativas descartadas:** Sanctum (no produce un JWT y exige una séptima tabla, fuera de [`data-model.md`](../data-model.md));
  `tymon/jwt-auth` (acopla el token al modelo Eloquent de usuario y al guard de Laravel, contra el artículo II).
- **Consecuencias:** no se usa la migración de `personal_access_tokens`; el token no tiene estado en la base y no se puede revocar antes
  de que venza, lo que el spec acepta («simple y sin estado», D-11).
- **Cómo se verifica:** decodificar el token del login muestra `sub`, `role` y una expiración de 8 horas; sin token, `GET /api/v1/books`
  responde 401 con el cuerpo de error único.
