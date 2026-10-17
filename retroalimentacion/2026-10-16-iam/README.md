# Contratos del modelo de acceso (paso 3)

Son las instrucciones con que se construyó el modelo de acceso entre el 15/10/2026 y el 16/10/2026, y la propuesta que llevó sus decisiones al diccionario. Se guardan como historia de la [revisión del 16/10/2026](../2026-10-16-iam.md).

**No son la regla vigente.** Cada ronda corrige a la anterior, y todas quedaron reemplazadas por el diccionario v1.18 (D-55 a D-72), que el responsable del diccionario validó el 17/10/2026 con ajustes propios. Donde estos archivos y el diccionario difieran, manda el diccionario. Las ramas que nombran (`feature/access-model`, worktrees `mm-acc-be` y `mm-acc-fe`) ya se integraron en `develop`.

| Archivo | Qué es |
|---|---|
| [access-model-spec.md](access-model-spec.md) | Ronda 1 (15/10): el contrato base, a partir de la auditoría del 14/10 |
| [access-model-spec-r2.md](access-model-spec-r2.md) | Ronda 2: Row Level Security, cancelar bloqueos programados, códigos de error |
| [access-model-spec-r3.md](access-model-spec-r3.md) | Ronda 3: correcciones de la revisión adversarial |
| [access-model-spec-r4.md](access-model-spec-r4.md) | Ronda 4: el administrador de accesos y las dos líneas de bloqueo |
| [access-model-spec-r5.md](access-model-spec-r5.md) | Ronda 5: cierre de la separación de funciones |
| [diccionario-v1.18-propuesta.md](diccionario-v1.18-propuesta.md) | La propuesta de cambios al diccionario. Lo aplicado difiere en lo que decidió el responsable sobre sus puntos abiertos: `revoked_by` entra al diseño de `user_roles`, `termination_reason` pasa a candidato, y los pendientes del paso 5 y de los reingresos tienen fecha |
