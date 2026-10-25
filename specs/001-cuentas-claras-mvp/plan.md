# Plan 001 — MVP de cuentas-claras

- **Origen:** [spec 001, versión 2](spec.md) y [constitución](../../docs/constitution.md)
- **Documentos del plan:** este archivo (cómo se construye), [`data-model.md`](data-model.md) (tablas y ejemplo) y [`contracts/cli.md`](contracts/cli.md) (comandos, salidas y códigos).
- **Fecha:** 2026-10-25

## 1. Decisiones de la fase 5 (D-21…D-32)

Se registran con la alternativa descartada; numeración continua con las [aclaraciones](clarifications.md).

| # | Decisión | Por qué | Alternativa descartada |
|---|---|---|---|
| D-21 | `argparse` con una subclase que reemplaza `error()` y la ayuda para usar el catálogo en español y el código de salida 1 | Hay 13 subcomandos con opciones repetidas; escribir el análisis a mano es mucho código propenso a errores | Análisis manual de argumentos: viable con 3 comandos, no con 13 |
| D-22 | Migraciones como una lista ordenada de sentencias SQL en el código, aplicadas según `PRAGMA user_version` | Sin dependencias; una sola fuente de la versión del esquema | Archivos `.sql` sueltos o una herramienta de migraciones (dependencia externa) |
| D-23 | Cada operación de escritura en una transacción `BEGIN IMMEDIATE` con `busy_timeout = 0`; «base bloqueada» → E-17 | Garantiza «todo o nada» (RNF-4) y detecta otra ejecución sin esperar (RF-12.3) | Esperar a que se libere: la persona no sabría por qué se cuelga |
| D-24 | Ruta `~/.cuentas-claras/cuentas.db`, sustituible con `CUENTAS_DB` | Una sola ruta en todos los sistemas; los tests usan una base temporal | Rutas por sistema operativo: más ramas sin beneficio para el MVP |
| D-25 | Cuatro capas: `domain` (reglas puras), `application` (casos de uso), `infrastructure` (SQLite y reloj), `cli` (argumentos, mensajes y salida) | Cumple el principio 3 y deja el dominio probado sin base ni reloj | Dos capas (dominio + CLI con SQL dentro): mezcla persistencia e interfaz |
| D-26 | El análisis de importes, fechas y nombres está en el dominio | Son reglas de la spec (RF-1.2, RF-1.4, RF-1.5), no detalles de la interfaz | Analizarlos en la CLI: las reglas quedarían sin probar en el dominio |
| D-27 | Saldos y plan se calculan en el dominio a partir de todos los gastos y pagos del grupo leídos de la base | 10 000 gastos caben en memoria y el cálculo es lineal (RNF-3); una sola implementación de la regla | Calcular saldos con SQL: duplica la regla de RF-10.1 en dos lenguajes |
| D-28 | Columna `name_key` con restricción de unicidad | La base también impide repetidos aunque falle el dominio | Comparar solo en el dominio: una carrera o un error dejaría duplicados |
| D-29 | Contadores `last_expense_no` y `last_payment_no` en el grupo | Números `G`/`P` que nunca se reutilizan (CL-24) | `MAX(number)+1`: reutilizaría el número si se elimina el último |
| D-30 | Códigos de salida 0 (éxito), 1 (validación o uso), 2 (datos) | Un script puede distinguir un error del usuario de un problema de datos | Un solo código de error |
| D-31 | Fechas como texto ISO en la base; `DD/MM/AAAA` solo en la interfaz | El texto ISO se ordena bien con `ORDER BY` | Guardar `DD/MM/AAAA`: no se ordena correctamente |
| D-32 | Porcentajes guardados como centésimas de punto (entero) | Cumple el principio 5 y RF-7.4 | Guardar el texto escrito o un número decimal |

## 2. Stack y dependencias

| Pieza | Uso | Por qué entra |
|---|---|---|
| Python 3.12+ | Lenguaje | Principio 1 |
| `sqlite3`, `argparse`, `dataclasses`, `unicodedata`, `datetime`, `enum`, `pathlib`, `os` | Biblioteca estándar | Principio 1 |
| `pytest` | Solo tests | Principio 1 |

No hay otras dependencias. Las pruebas «de propiedades» (miles de casos al azar) se hacen con `random.Random(semilla fija)`, sin bibliotecas externas.

## 3. Arquitectura por capas

```text
cli/            argparse, catálogo de mensajes, formato de salida, códigos de salida
  │  llama a
application/    casos de uso: abre la transacción, lee, aplica el dominio, guarda
  │  usa                          │ usa
domain/  (sin E/S)          infrastructure/   SQLite (repositorio, migraciones) y reloj
```

Reglas de dependencia: `domain` no importa nada del proyecto ni `sqlite3`, `argparse`, `os`, `datetime.date.today` o `time` (principio 3). `application` importa `domain` y recibe el repositorio y la fecha de hoy como parámetros. Solo `infrastructure` importa `sqlite3`. Solo `cli` imprime.

## 4. Estructura de carpetas

```text
cuentas/
  __main__.py              llama a cli.main y devuelve su código
  domain/
    errors.py              errores del dominio (un tipo por E-1…E-14, con sus datos)
    money.py               parse_amount, format_amount, parse_percent           RF-1.1-1.3, RF-7.1
    names.py               clean_name, name_key, validate_name, clean_description RF-1.5-1.7, RF-4.2
    dates.py               parse_date (DD/MM/AAAA, 2000-01-01..hoy)             RF-1.4, RF-4.6, RF-4.7
    splits.py              split_equal, split_exact, split_percent               RF-4.9, RF-5, RF-6, RF-7
    balances.py            compute_balances                                      RF-10
    settlement.py          settle                                               RF-11
    models.py              dataclasses inmutables: Group, Member, Expense, Share, Payment
  application/
    ports.py               protocolos Repository y Clock
    services.py            un caso de uso por operación de la CLI                RF-2 a RF-12
  infrastructure/
    sqlite_repository.py   implementación del repositorio, transacciones          RF-12, RNF-4
    migrations.py          esquema versión 1 y control de user_version           RF-12.1
    clock.py               SystemClock (fecha local de hoy)
  cli/
    messages.py            catálogo M-1…M-12 y E-1…E-17                           RNF-6
    parser.py              subclase de argparse y subcomandos                    contracts/cli.md
    render.py              formato de listados, saldos y plan
    main.py                main(argv, repo_factory, clock) → código de salida    D-30
tests/
  domain/ application/ infrastructure/ cli/ architecture/ conftest.py
```

## 5. Dominio: algoritmos

### 5.1 Importes (RF-1.2, RF-1.3)
```text
parse_amount(texto):
  si texto no coincide con ^\d+([.,]\d{1,2})?$      -> InvalidAmount          (E-1)
  separar parte entera y decimal; completar decimales a 2 dígitos
  centavos = entera × 100 + decimales
  si centavos < 1 o centavos > 100 000 000          -> InvalidAmount          (E-1)
  devolver centavos
format_amount(c): f"{c // 100},{c % 100:02d} Bs"   (c ≥ 0; los negativos los traduce render a palabras)
```

### 5.2 Nombres y descripciones (RF-1.5-1.7, RF-4.2)
```text
clean(texto): NFC; si contiene un carácter de categoría Cc/Zl/Zp -> inválido; unir con un espacio las partes separadas por espacios
validate_name(texto): limpio; 1..40 puntos de código; al menos un carácter letra o número  (E-2)
name_key(limpio): casefold; NFD; quitar U+0301 tras a/e/i/o/u y U+0308 tras u; NFC     (CL-11)
clean_description(texto): limpio; 1..80 puntos de código                                (E-7)
```

### 5.3 Repartos (RF-4.9, RF-5, RF-6, RF-7)

Entrada común: importe en centavos y la lista de participantes **ordenada por orden de alta** (el caso de uso la ordena, aunque se escriban en otro orden).

```text
split_equal(importe, participantes):                                      RF-5
  n = len(participantes); base = importe // n; sobrantes = importe % n
  parte[i] = base + (1 si i < sobrantes si no 0)

split_exact(importe, participantes, montos):                             RF-6
  exigir un monto ≥ 1 por participante y ninguno de más                  (E-10)
  si suma(montos) ≠ importe -> AmountsDoNotAdd(suma, importe − suma)      (E-11)
  parte = montos

split_percent(importe, participantes, centesimas):                       RF-7
  exigir 1 ≤ centésimas ≤ 10 000 por participante y ninguno de más       (E-10)
  si suma(centesimas) ≠ 10 000 -> PercentsDoNotAdd(suma)                  (E-12)
  producto[i] = importe × centesimas[i]
  parte[i] = producto[i] // 10 000 ; resto[i] = producto[i] % 10 000
  sobrantes = importe − suma(parte)
  ordenar índices por (−resto, orden de alta); dar 1 centavo a los primeros «sobrantes»
```
Postcondición de los tres: `suma(parte) == importe` (se comprueba con `assert` en el dominio y con tests de propiedades).

### 5.4 Saldos (RF-10)
```text
compute_balances(miembros, gastos, pagos):
  saldo[m] = 0 para todo m
  por gasto: saldo[pagador] += importe ; por cada parte: saldo[participante] −= parte
  por pago:  saldo[de] += importe ; saldo[a] −= importe
  verificar suma(saldo) == 0                                              RF-10.3
```

### 5.5 Plan de liquidación (RF-11)
```text
settle(saldos en orden de alta):
  deudores  = [(m, −s) para s < 0]; acreedores = [(m, s) para s > 0]
  mientras haya deudores:
     d = el de mayor deuda (empate: primero en orden de alta)
     a = el de mayor crédito (empate: primero en orden de alta)
     x = min(deuda de d, crédito de a); anotar «d le paga x a a»
     restar x a ambos; quitar al que llegue a 0 (si los dos llegan a 0, quitar ambos)
  devolver transferencias
```
Cada vuelta deja en cero al menos a uno; la última deja en cero a los dos, de ahí «a lo sumo k − 1 transferencias» con k integrantes con saldo (RF-11.3). Costo O(k²) con k ≤ 50.

### 5.6 Aviso de pago (RF-9.3)
```text
deuda_antes = max(0, −saldo_antes[de]); si importe > deuda_antes: aviso M-7 con saldo_despues[de]
```

## 6. Casos de uso (capa de aplicación)

Cada uno recibe el repositorio y, cuando corresponde, `hoy` (leído una vez por la CLI con el reloj). Las escrituras ocurren dentro de una transacción del repositorio; ante cualquier error, se deshace (RNF-4).

| Caso de uso | Pasos | RF |
|---|---|---|
| `create_group(nombre)` | validar, comprobar repetido, guardar | RF-2.1, RF-2.2 |
| `list_groups()` | leer con conteos | RF-2.3 |
| `add_member(grupo, nombre)` | grupo existe, validar, repetido, límite 50, guardar | RF-3.1-3.3 |
| `remove_member(grupo, nombre)` | existe; si figura en gastos/pagos → E-6; quitar | RF-3.4, RF-3.5 |
| `list_members(grupo)` | leer en orden de alta | RF-3.6 |
| `add_expense(grupo, datos, hoy)` | validar descripción, importe, fecha, pagador, participantes; ordenar por alta; límite 10 000; repartir; incrementar `last_expense_no`; guardar gasto y partes | RF-4 a RF-7 |
| `list_expenses(grupo)` | leer ordenado | RF-8.1 |
| `delete_expense(grupo, número)` | buscar `G{n}`; borrar con sus partes | RF-8.2, RF-8.3 |
| `add_payment(grupo, de, a, importe, fecha, hoy)` | validar; calcular saldos antes; guardar; calcular aviso | RF-9.1-9.3 |
| `list_payments`, `delete_payment` | como los gastos | RF-9.4, RF-9.5 |
| `get_balances(grupo)` | leer todo; `compute_balances` | RF-10 |
| `get_settlement(grupo)` | saldos; `settle`; sin escribir nada | RF-11 |

## 7. Persistencia

- Tablas, índices, restricciones y ejemplo en [`data-model.md`](data-model.md).
- Al abrir: si el archivo no existe, se crea y se aplican las migraciones (RF-12.4). Si `user_version` es mayor que la conocida → E-15. Si SQLite informa archivo dañado → E-15.
- Errores de escritura (`sqlite3.OperationalError` por disco o permisos) → E-16 tras el `ROLLBACK`. «Database is locked» → E-17.
- El repositorio traduce las excepciones de SQLite a errores propios (`DataUnreadable`, `SaveFailed`, `DataBusy`); el resto del sistema no ve `sqlite3`.

## 8. Estrategia de tests

### 8.1 Tipos de test
| Capa | Qué | Herramientas |
|---|---|---|
| Dominio | Cada regla con casos concretos de la spec (todos los CL de cálculo) y propiedades: 5 000 repartos al azar suman el total; saldos suman 0; el plan los deja en 0 y cumple RF-11.3 | `pytest`, `random.Random(20261025)` |
| Aplicación | Casos de uso con un repositorio en memoria (falso) y fecha fija | `pytest` |
| Infraestructura | Repositorio real sobre una base temporal: restricciones, transacción que falla a mitad (no queda nada), versión desconocida, archivo dañado, base bloqueada | `pytest`, `tmp_path` |
| CLI | `main(argv)` de punta a punta con base temporal y reloj fijo: salidas exactas del contrato, stdout/stderr y códigos | `pytest`, `capsys` |
| Arquitectura | Análisis de `import` con `ast` (principios 1, 3, 6), ausencia de `float(` en el dominio (principio 5), textos solo desde `messages.py` (principio 7) | `ast`, `pytest` |
| Rendimiento | 50 integrantes y 10 000 gastos: `gasto listar`, `saldos` y `liquidar` en menos de 1 s cada uno | `time.perf_counter` |

### 8.2 Datos y reloj
Fecha fija `2026-10-25` mediante un `FixedClock`; ningún test usa el reloj real. Datos ficticios.

### 8.3 Trazabilidad
Cada test lleva en su docstring los identificadores que verifica (`"""RF-5.2, CL-1"""`). Un test de trazabilidad extrae todos los `RF-x.y`, `RNF-x` y `CL-x` de `spec.md` y falla si alguno no aparece en ningún test (RNF-7, criterio de finalización 1).

## 9. Trazabilidad RF → módulo → tipo de test

| RF | Módulo principal | Tests |
|---|---|---|
| RF-1.1-1.3 | `domain/money.py`, `cli/render.py` | dominio, CLI |
| RF-1.4 | `domain/dates.py` | dominio |
| RF-1.5-1.7 | `domain/names.py` | dominio, infraestructura (unicidad) |
| RF-2 | `application/services.py` | aplicación, CLI |
| RF-3 | `application/services.py`, `infrastructure` (RESTRICT) | aplicación, infraestructura, CLI |
| RF-4 | `application/services.py`, `domain/names.py`, `domain/dates.py` | aplicación, CLI |
| RF-5, RF-6, RF-7 | `domain/splits.py` | dominio (casos y propiedades), CLI |
| RF-8, RF-9 | `application/services.py`, `domain/balances.py` (aviso) | aplicación, CLI |
| RF-10 | `domain/balances.py`, `cli/render.py` | dominio, CLI |
| RF-11 | `domain/settlement.py` | dominio (casos y propiedades), CLI |
| RF-12 | `infrastructure/*`, `cli/main.py` | infraestructura, CLI |

## 10. Comprobación contra la constitución

| Principio | Cómo lo cumple el plan |
|---|---|
| 1. Stack mínimo | Solo biblioteca estándar (sección 2); test de `import` |
| 2. Spec antes que código | Cada módulo cita sus RF; test de trazabilidad |
| 3. Dominio sin E/S | Capa `domain` pura; `hoy` llega como parámetro (sección 3); test de `import` |
| 4. Tests obligatorios | Sección 8; `pytest -q` antes de cada commit |
| 5. Dinero exacto | Centavos y centésimas enteros (D-32, 5.1, 5.3); test que prohíbe `float(` en el dominio |
| 6. Persistencia controlada | SQLite solo en `infrastructure`; migraciones versionadas (D-22); transacciones (D-23) |
| 7. Idioma | Catálogo único `cli/messages.py`; identificadores en inglés |

Ninguna puerta de la constitución se incumple; no hace falta justificar complejidad extra.

## 11. Riesgos técnicos

| # | Riesgo | Mitigación |
|---|---|---|
| R-1 | Que alguien use `float` al convertir porcentajes | `parse_percent` analiza el texto con expresión regular a centésimas enteras; test de arquitectura que prohíbe `float(` |
| R-2 | Que `argparse` muestre textos en inglés en algún error no previsto | Subclase con `error()` y ayuda propias (D-21); test de cada error de uso del contrato |
| R-3 | Orden de alta mal aplicado si se respeta el orden de escritura | El caso de uso ordena siempre por `members.id`; CL-1 y CL-23 lo prueban con nombres escritos al revés |
| R-4 | Rendimiento de `gasto listar` con 10 000 gastos y sus partes | Una sola consulta con `JOIN` e índice; test de rendimiento |
| R-5 | Comportamiento de `busy_timeout = 0` distinto entre versiones de SQLite | Test de infraestructura que abre una segunda conexión con una transacción abierta |

## 12. Autorrevisión del plan

- Todos los RF-1 a RF-12 tienen módulo y tipo de test (sección 9).
- Los CL de cálculo (CL-1-4, 15-17, 22-23) se prueban en el dominio; los de datos (CL-24, CL-25) en infraestructura; el resto en aplicación y CLI.
- No hay comportamiento en el plan que no esté en la spec, salvo detalles de interfaz definidos en el contrato (sintaxis de comandos, Ctrl+C con código 1).

## 13. Avisos para el usuario

- **A-1:** el plan añade el comportamiento de Ctrl+C (código 1, sin guardar). La spec solo exige integridad (RNF-4); si se prefiere otro código, cambiar el contrato.
- **A-2:** la variable `CUENTAS_DB` (D-24) es un detalle técnico para tests y usuarios avanzados; no figura en la spec.
