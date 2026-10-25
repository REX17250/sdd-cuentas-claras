# Tareas 001 — MVP de cuentas-claras

- **Estado:** listas para implementar (2026-10-25)
- **Origen:** [spec 001, versión 2](spec.md), [plan](plan.md), [modelo de datos](data-model.md) y [contrato de la CLI](contracts/cli.md)

## Cómo usar esta lista

- Orden de dependencia: cada tarea indica de cuáles depende. Empieza por el dominio; la interfaz va al final.
- `[P]` = se puede hacer en paralelo con las otras `[P]` de la misma fase (no comparten archivos).
- Cada tarea cabe en 20-40 minutos e incluye sus tests: **se escriben primero, se ven fallar y después se escribe el código**.
- Cada test cita en su docstring los RF/CL/RNF que verifica (plan 8.3).
- **Al terminar cada tarea**, `pytest -q` debe pasar entero (principio 4) y se hace un commit en la rama de trabajo. Es parte del «Hecho cuando» de todas, aunque no se repita.
- Ninguna tarea está implementada todavía.

---

## Fase 0 — Preparación

- [ ] **T-1 · Esqueleto del proyecto** — principios 1 y 4
  - Depende de: —
  - Archivos: `pyproject.toml`, `cuentas/__init__.py`, `cuentas/__main__.py`, paquetes vacíos `cuentas/{domain,application,infrastructure,cli}/`, `tests/conftest.py`
  - Hacer: `pyproject.toml` con Python ≥ 3.12, sin dependencias, `pytest` como dependencia de desarrollo y `testpaths = ["tests"]`; `__main__.py` que llama a una `main` provisional y devuelve 0.
  - Test: un test de humo que importa los cuatro paquetes.
  - Hecho cuando: `pytest -q` pasa y `python -m cuentas` termina con código 0.

- [ ] **T-2 · Reloj fijo y fixtures** — RF-4.6, principio 3
  - Depende de: T-1
  - Archivos: `cuentas/application/ports.py` (protocolo `Clock`), `cuentas/infrastructure/clock.py` (`SystemClock`), `tests/conftest.py` (`FixedClock`, fixture `today = 2026-10-25`)
  - Test: `FixedClock` devuelve la fecha fijada y se puede mover un día; `SystemClock` devuelve un `date`.
  - Hecho cuando: el test pasa y ningún test usa `date.today()` directamente.

- [ ] **T-3 · Tests de arquitectura** — principios 1, 3, 5, 6 y 7
  - Depende de: T-1
  - Archivos: `tests/architecture/test_layers.py`
  - Hacer: con `ast`, comprobar que `domain` no importa `sqlite3`, `argparse`, `os`, `time`, ni `application`/`infrastructure`/`cli`; que solo `infrastructure` importa `sqlite3`; que todo `import` del paquete es de la biblioteca estándar o del propio proyecto; que no hay `float(` en `domain`.
  - Test: el propio archivo, más casos que pasan código de ejemplo con `import sqlite3` y con `float(` a la función de análisis y esperan que los detecte.
  - Hecho cuando: los tests pasan con el esqueleto y los casos de ejemplo prueban que cada regla detecta su violación.

## Fase 1 — Dominio

- [ ] **T-4 · Errores y modelos del dominio** `[P]` — RF-1 a RF-11 (tipos)
  - Depende de: T-1
  - Archivos: `cuentas/domain/errors.py`, `cuentas/domain/models.py`, `tests/domain/test_models.py`
  - Hacer: un tipo de error por E-1…E-14, con sus datos (por ejemplo `AmountsDoNotAdd(total, diferencia)`); dataclasses inmutables `Group`, `Member`, `Expense`, `Share`, `Payment` con importes en centavos.
  - Test: cada error guarda sus datos; los modelos no se pueden modificar.
  - Hecho cuando: el test pasa.

- [ ] **T-5 · Importes** `[P]` — RF-1.1, RF-1.2, RF-1.3, CL-13
  - Depende de: T-4
  - Archivos: `cuentas/domain/money.py`, `tests/domain/test_money.py`
  - Hacer: `parse_amount` y `format_amount` (plan 5.1).
  - Test: «12» → 1200; «1,5» y «1.5» → 150; «0,01» → 1; «1000000» → 100 000 000; rechazados «1.234,50», «+5», «5 Bs», «5,001», «0», «1000000,01», «», « 5»; formato «1234,50 Bs» y «0,05 Bs».
  - Hecho cuando: todos los casos pasan.

- [ ] **T-6 · Porcentajes** `[P]` — RF-7.1, CL-17
  - Depende de: T-4
  - Archivos: `cuentas/domain/money.py` (`parse_percent`), `tests/domain/test_percent.py`
  - Hacer: texto con hasta dos decimales (coma o punto) → centésimas de punto enteras; mayor que 0 y como máximo 10 000.
  - Test: «33,33» y «33.33» → 3333; «100» → 10 000; «0,01» → 1; rechazados «0», «100,01», «-5», «33,333», «abc».
  - Hecho cuando: todos los casos pasan, sin usar `float`.

- [ ] **T-7 · Nombres y descripciones** `[P]` — RF-1.5, RF-1.6, RF-1.7, RF-4.2, CL-11, CL-14
  - Depende de: T-4
  - Archivos: `cuentas/domain/names.py`, `tests/domain/test_names.py`
  - Hacer: `clean`, `validate_name`, `name_key`, `clean_description` (plan 5.2).
  - Test: « Ana  María » → «Ana María»; «Ana», « ána », «ANA» misma clave; «Peña» ≠ «Pena»; «pingüino» = «pinguino»; rechazados «   », «!!!», 41 caracteres y un salto de línea; descripción de 80 aceptada y 81 rechazada.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-8 · Fechas** `[P]` — RF-1.4, RF-4.6, RF-4.7, CL-12
  - Depende de: T-2, T-4
  - Archivos: `cuentas/domain/dates.py`, `tests/domain/test_dates.py`
  - Hacer: `parse_date(texto | None, hoy)`: `None` → hoy; `DD/MM/AAAA` entre 01/01/2000 y hoy.
  - Test: sin fecha → 25/10/2026; «25/10/2026» aceptada; «26/10/2026», «31/12/1999», «31/02/2026», «2026-10-25» y «1/1/2020» rechazadas con E-9.
  - Hecho cuando: todos los casos pasan con el reloj fijo.

- [ ] **T-9 · Reparto en partes iguales** — RF-4.9, RF-5.1, RF-5.2, CL-1, CL-2, CL-3, CL-16
  - Depende de: T-4
  - Archivos: `cuentas/domain/splits.py` (`split_equal`), `tests/domain/test_split_equal.py`
  - Test: 10 000 entre 3 → 3334/3333/3333; 1 entre 3 → 1/0/0; 2 entre 3 → 1/1/0; 1 entre 50 → 1 y 49 ceros; 5 000 casos al azar suman el total y ninguna parte difiere de otra en más de 1.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-10 · Reparto por montos exactos** — RF-6.1, RF-6.2, RF-6.3, CL-5
  - Depende de: T-9
  - Archivos: `cuentas/domain/splits.py` (`split_exact`), `tests/domain/test_split_exact.py`
  - Test: montos que suman el total se aceptan tal cual; suma 1 centavo menos → E-11 con diferencia 1; monto 0 → E-10; participante sin monto o monto de quien no participa → E-10.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-11 · Reparto por porcentajes** — RF-7.2, RF-7.3, RF-7.4, RF-7.5, CL-4, CL-17
  - Depende de: T-6, T-9
  - Archivos: `cuentas/domain/splits.py` (`split_percent`), `tests/domain/test_split_percent.py`
  - Test: 10 000 al 3333/3333/3334 → 3333/3333/3334; uno al 10 000 → el total; suma 9 999 → E-12; un caso donde los sobrantes van al mayor resto y otro donde se desempata por orden de alta; 5 000 casos al azar suman el total.
  - Hecho cuando: todos los casos pasan, solo con enteros.

- [ ] **T-12 · Saldos** — RF-10.1, RF-10.3, CL-6, CL-8, CL-15, CL-19
  - Depende de: T-4
  - Archivos: `cuentas/domain/balances.py`, `tests/domain/test_balances.py`
  - Test: el ejemplo de `data-model.md` (11 666 / −3 333 / −8 333); quien pagó sin participar sube el importe completo; gasto con solo el pagador no cambia nada; sin gastos todos 0; saldo mayor que 100 000 000 se calcula bien; 5 000 grupos al azar suman 0.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-13 · Plan de liquidación** — RF-11.1, RF-11.2, RF-11.3, RF-11.4, CL-9, CL-10, CL-22, CL-23
  - Depende de: T-12
  - Archivos: `cuentas/domain/settlement.py`, `tests/domain/test_settlement.py`
  - Test: el ejemplo de `data-model.md` (Caro 8 333 → Ana; Beto 3 333 → Ana); un deudor y tres acreedores → 3 transferencias; todos en 0 → lista vacía; +1/−1 → una de 1 centavo; dos deudores iguales → primero el de antes en orden de alta; 5 000 casos al azar: el plan deja todo en 0 y tiene como mucho k − 1 transferencias.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-14 · Aviso de pago** — RF-9.3, CL-7
  - Depende de: T-12
  - Archivos: `cuentas/domain/balances.py` (`payment_warning`), `tests/domain/test_payment_warning.py`
  - Test: quien debía 30 paga 30 → sin aviso; paga 50 → aviso con 20 a favor; quien no debía nada paga 10 → aviso con su nuevo saldo.
  - Hecho cuando: todos los casos pasan.

## Fase 2 — Persistencia

- [ ] **T-15 · Migraciones y apertura de la base** — RF-12.1, RF-12.4, principio 6
  - Depende de: T-3
  - Archivos: `cuentas/infrastructure/migrations.py`, `cuentas/infrastructure/sqlite_repository.py` (apertura), `tests/infrastructure/test_migrations.py`
  - Hacer: esquema versión 1 de `data-model.md`; `foreign_keys = ON`; base nueva → crea todo; `user_version` mayor → `DataUnreadable`; archivo que no es SQLite → `DataUnreadable`.
  - Test: base nueva con las 6 tablas y `user_version = 1`; versión 2 → error sin tocar el archivo (mismo contenido byte a byte); archivo de texto → error.
  - Hecho cuando: los tres casos pasan.

- [ ] **T-16 · Repositorio: grupos e integrantes** — RF-2, RF-3, CL-21
  - Depende de: T-15, T-7
  - Archivos: `cuentas/infrastructure/sqlite_repository.py`, `tests/infrastructure/test_repo_members.py`
  - Test: guardar y leer grupos en orden de creación con conteos; `name_key` repetido rechazado por la base; mismo nombre en dos grupos permitido; quitar un integrante con un gasto → bloqueado por `RESTRICT`.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-17 · Repositorio: gastos y pagos** — RF-4.1, RF-8, RF-9, RNF-4, CL-24
  - Depende de: T-16
  - Archivos: `cuentas/infrastructure/sqlite_repository.py`, `tests/infrastructure/test_repo_expenses.py`
  - Test: guardar un gasto con sus partes y releerlo igual; números `G`/`P` desde contadores, sin reutilizar tras eliminar; eliminar un gasto borra sus partes; listados en orden fecha/número; una falla simulada entre el gasto y sus partes no deja nada guardado.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-18 · Errores de datos y bloqueo** — RF-12.2, RF-12.3, CL-25
  - Depende de: T-17
  - Archivos: `cuentas/infrastructure/sqlite_repository.py`, `tests/infrastructure/test_repo_errors.py`
  - Test: con otra conexión que tiene una transacción abierta → `DataBusy`; error de escritura simulado → `SaveFailed` y datos intactos; ningún error de `sqlite3` sale del repositorio sin traducir.
  - Hecho cuando: todos los casos pasan.

## Fase 3 — Casos de uso

- [ ] **T-19 · Repositorio en memoria para tests** `[P]`
  - Depende de: T-4
  - Archivos: `tests/fakes.py`
  - Hacer: implementación del protocolo `Repository` en memoria, con las mismas reglas de unicidad y numeración.
  - Test: los tests de T-16 y T-17 que no dependen de SQLite pasan también contra el falso.
  - Hecho cuando: los mismos tests pasan contra los dos repositorios.

- [ ] **T-20 · Casos de uso de grupos e integrantes** — RF-2.1-2.4, RF-3.1-3.7, CL-11, CL-20, CL-21
  - Depende de: T-16, T-19
  - Archivos: `cuentas/application/services.py`, `tests/application/test_groups_members.py`
  - Test: crear, repetido, listar; agregar, repetido, límite de 50, quitar sin historial, quitar con historial → E-6, quitar tras eliminar su único gasto; grupo o integrante inexistente → E-4.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-21 · Caso de uso: registrar gasto** — RF-4.1-4.9, CL-15, CL-18
  - Depende de: T-20, T-8, T-9, T-10, T-11
  - Archivos: `cuentas/application/services.py`, `tests/application/test_add_expense.py`
  - Test: las tres formas de reparto; participantes escritos en otro orden → reparto por orden de alta; descripción inválida → E-7; participantes vacíos o repetidos → E-8; pagador o participante inexistente → E-4; fecha futura → E-9; límite de 10 000 → E-5; grupo sin integrantes → E-4; solo el pagador como participante → aceptado.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-22 · Casos de uso: listar y eliminar gastos** — RF-8.1, RF-8.2, RF-8.3, CL-8, CL-24
  - Depende de: T-21
  - Archivos: `cuentas/application/services.py`, `tests/application/test_expenses_list_delete.py`
  - Test: orden fecha/número; eliminar `G2` y crear otro → `G3`; número inexistente o de pago → E-13; eliminar el único gasto deja saldos en 0.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-23 · Casos de uso de pagos** — RF-9.1-9.5, CL-7
  - Depende de: T-21, T-14
  - Archivos: `cuentas/application/services.py`, `tests/application/test_payments.py`
  - Test: registrar con y sin fecha; a uno mismo → E-14; importe inválido → E-1; integrante de otro grupo → E-4; aviso M-7 cuando supera la deuda; listar y eliminar con `P{n}`.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-24 · Casos de uso de saldos y plan** — RF-10.1-10.4, RF-11.1-11.5, CL-10, CL-18
  - Depende de: T-22, T-23, T-13
  - Archivos: `cuentas/application/services.py`, `tests/application/test_balances_settlement.py`
  - Test: el ejemplo de `data-model.md` de punta a punta; grupo sin integrantes → M-10; todos a mano → plan vacío; pedir el plan no crea ningún pago.
  - Hecho cuando: todos los casos pasan.

## Fase 4 — Interfaz de línea de comandos

- [ ] **T-25 · Catálogo de mensajes** `[P]` — RNF-6, sección 5 de la spec
  - Depende de: T-4
  - Archivos: `cuentas/cli/messages.py`, `tests/cli/test_messages.py`
  - Hacer: M-1…M-12 y E-1…E-17 como funciones con sus parámetros.
  - Test: cada mensaje con datos de ejemplo coincide letra por letra con la sección 5; cada error del dominio tiene su texto.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-26 · Parser y ayuda en español** — contrato 1 y 2, D-21
  - Depende de: T-25
  - Archivos: `cuentas/cli/parser.py`, `tests/cli/test_parser.py`
  - Test: cada comando del contrato se analiza bien; `--monto "Ana=12,50"` y un nombre con `=`; dos formas de reparto a la vez, comando desconocido y argumentos faltantes → mensaje en español y código 1; `ayuda`, `-h` y sin argumentos muestran la ayuda.
  - Hecho cuando: todos los casos pasan y ningún texto en inglés llega a la salida.

- [ ] **T-27 · Formato de salida** `[P]` — RF-1.1, RF-10.2, contrato 3
  - Depende de: T-25
  - Archivos: `cuentas/cli/render.py`, `tests/cli/test_render.py`
  - Test: listados de gastos, saldos («le deben», «debe», «al día») y plan con el ejemplo del contrato, letra por letra.
  - Hecho cuando: todos los casos pasan.

- [ ] **T-28 · `main` de punta a punta** — RF-2 a RF-12, contrato 4, D-30
  - Depende de: T-24, T-26, T-27, T-18
  - Archivos: `cuentas/cli/main.py`, `cuentas/__main__.py`, `tests/cli/test_main.py`
  - Hacer: `main(argv, db_path, clock)`; una lectura del reloj por operación; error de dominio → stderr y código 1; `DataUnreadable`/`SaveFailed`/`DataBusy` → stderr y código 2; Ctrl+C → código 1 sin guardar.
  - Test: el recorrido completo del ejemplo (crear grupo, tres integrantes, dos gastos, un pago, saldos, liquidar) con salidas exactas; un caso por código 0, 1 y 2; datos ilegibles en cada comando de lectura.
  - Hecho cuando: todos los casos pasan y `python -m cuentas ayuda` funciona a mano.

## Fase 5 — Calidad y cierre

- [ ] **T-29 · Rendimiento** — RNF-3
  - Depende de: T-28
  - Archivos: `tests/test_performance.py`
  - Test: base con 50 integrantes y 10 000 gastos al azar; `gasto listar`, `saldos` y `liquidar` en menos de 1 s cada uno.
  - Hecho cuando: el test pasa en la computadora de desarrollo y queda anotado el tiempo medido.

- [ ] **T-30 · Trazabilidad** — RNF-7, criterio de finalización 1
  - Depende de: T-28
  - Archivos: `tests/test_traceability.py`
  - Hacer: extraer de `spec.md` todos los `RF-x.y`, `RNF-x` y `CL-x` y comprobar que cada uno aparece en la docstring de algún test; añadir los tests o las citas que falten.
  - Test: el propio archivo, que falla si se borra a propósito una cita (se comprueba y se deshace).
  - Hecho cuando: no queda ningún identificador sin cubrir.

- [ ] **T-31 · Cierre** — criterios de finalización 2 a 6
  - Depende de: T-29, T-30
  - Hacer: ejecutar la suite con Python 3.12 y con la versión instalada; revisar juntos los textos del catálogo; recorrer los criterios de finalización con su evidencia; actualizar el «Estado» de `AGENTS.md`.
  - Hecho cuando: cada criterio de finalización está marcado con su evidencia y `pytest -q` pasa.

## Matriz de cobertura

| RF | Tareas | RF | Tareas |
|---|---|---|---|
| RF-1.1-1.3 | T-5, T-27 | RF-7 | T-6, T-11, T-21 |
| RF-1.4 | T-8 | RF-8 | T-17, T-22 |
| RF-1.5-1.7 | T-7, T-16, T-20 | RF-9 | T-14, T-17, T-23 |
| RF-2 | T-16, T-20, T-28 | RF-10 | T-12, T-24, T-27 |
| RF-3 | T-16, T-20, T-28 | RF-11 | T-13, T-24, T-28 |
| RF-4 | T-8, T-17, T-21 | RF-12 | T-15, T-18, T-28 |
| RF-5 | T-9, T-21 | RNF-3 | T-29 |
| RF-6 | T-10, T-21 | RNF-6 | T-25, T-26 |
| | | RNF-7 | T-30 |

Total: 31 tareas en 6 fases. Las `[P]` (T-4…T-8, T-19, T-25, T-27) se pueden repartir entre dos personas.
