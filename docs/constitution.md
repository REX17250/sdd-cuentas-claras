# Constitución — cuentas-claras

> Estado: **cerrada** (fase 1, 2026-10-25). Decisiones D-1…D-5 y cómo se verifica cada principio en [`constitution-proposal.md`](constitution-proposal.md).

1. **Stack mínimo**: Python 3.12+ y solo la biblioteca estándar; `pytest` únicamente para tests. Toda otra dependencia necesita una decisión escrita en el plan.
2. **Spec antes que código**: ningún comportamiento se implementa sin un RF en `specs/`. Si el código y la spec no coinciden, se corrige el código; si la spec está mal, se cambia primero la spec, con su versión.
3. **Dominio sin E/S**: las reglas de dinero (repartos, saldos, liquidación) viven en una capa de dominio que no importa interfaz, base de datos ni reloj; la fecha de hoy se recibe como parámetro.
4. **Tests obligatorios**: cada RF tiene al menos un test automático y cada CL su caso; los tests citan esos identificadores. `pytest -q` pasa entero antes de cada commit.
5. **Dinero exacto**: los importes se manejan como enteros de centavos, nunca como números de coma flotante. Todo reparto suma exactamente el total, y cada redondeo sigue una regla escrita en la spec.
6. **Persistencia controlada**: una única base SQLite local, a la que solo accede la capa de persistencia; cada cambio de esquema es una migración versionada, y cada operación se guarda completa o no se guarda.
7. **Idioma**: código, identificadores, nombres de archivo y comentarios en inglés; textos de la interfaz y documentación en español.
