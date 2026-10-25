# Modelo de datos — spec 001

Parte del [plan](plan.md). Base SQLite local (D-3). Todos los importes en **centavos enteros** (principio 5) y los porcentajes en **centésimas de punto** (RF-7.4).

## 1. Ubicación y versión

- Archivo: `~/.cuentas-claras/cuentas.db` (carpeta personal del usuario). La variable de entorno `CUENTAS_DB` permite indicar otra ruta; la usan los tests (D-24).
- Versión del esquema en `PRAGMA user_version`. La versión de este documento es **1**. Si la base tiene una versión mayor que la que conoce la aplicación → E-15 sin tocar nada (RF-12.1).
- `PRAGMA foreign_keys = ON` en cada conexión.

## 2. Tablas

### `groups`
| Columna | Tipo | Reglas |
|---|---|---|
| `id` | INTEGER | clave primaria; su orden es el orden de creación (RF-2.3) |
| `name` | TEXT | nombre limpio, tal como se escribió (RF-1.6) |
| `name_key` | TEXT | clave de equivalencia (RF-1.5); **única** |
| `last_expense_no` | INTEGER | último número `G` usado; empieza en 0 (RF-4.1, CL-24) |
| `last_payment_no` | INTEGER | último número `P` usado; empieza en 0 (RF-9.1) |

### `members`
| Columna | Tipo | Reglas |
|---|---|---|
| `id` | INTEGER | clave primaria; dentro de un grupo, su orden es el **orden de alta** |
| `group_id` | INTEGER | → `groups.id` |
| `name` | TEXT | nombre limpio |
| `name_key` | TEXT | clave de equivalencia; **única junto con `group_id`** (RF-3.2, CL-21) |

### `expenses`
| Columna | Tipo | Reglas |
|---|---|---|
| `id` | INTEGER | clave primaria |
| `group_id` | INTEGER | → `groups.id` |
| `number` | INTEGER | número `G`; **único junto con `group_id`** |
| `description` | TEXT | descripción limpia, 1-80 caracteres |
| `amount_cents` | INTEGER | 1 a 100 000 000 |
| `spent_on` | TEXT | fecha ISO `AAAA-MM-DD` |
| `payer_id` | INTEGER | → `members.id` (mismo grupo) |
| `split_mode` | TEXT | `equal`, `exact` o `percent` |

### `expense_shares`
| Columna | Tipo | Reglas |
|---|---|---|
| `expense_id` | INTEGER | → `expenses.id`, **se borra con el gasto** |
| `member_id` | INTEGER | → `members.id`; clave primaria junto con `expense_id` (sin repetidos, RF-4.4) |
| `share_cents` | INTEGER | parte calculada, ≥ 0 |
| `input_value` | INTEGER | monto indicado en centavos (`exact`), centésimas de punto (`percent`) o nulo (`equal`) |

Invariante: la suma de `share_cents` de un gasto es igual a su `amount_cents` (RF-4.9). Se comprueba en el dominio antes de guardar y en un test de la capa de persistencia.

### `payments`
| Columna | Tipo | Reglas |
|---|---|---|
| `id` | INTEGER | clave primaria |
| `group_id` | INTEGER | → `groups.id` |
| `number` | INTEGER | número `P`; **único junto con `group_id`** |
| `from_member_id` | INTEGER | → `members.id` |
| `to_member_id` | INTEGER | → `members.id`; distinto de `from_member_id` (RF-9.2) |
| `amount_cents` | INTEGER | 1 a 100 000 000 |
| `paid_on` | TEXT | fecha ISO |

Las claves foráneas de `members` usan `ON DELETE RESTRICT`: la base impide quitar a un integrante con gastos o pagos, además de la comprobación del dominio (RF-3.5).

Índices: `expenses(group_id, spent_on, number)` y `payments(group_id, paid_on, number)` para los listados ordenados (RF-8.1, RF-9.5, RNF-3).

## 3. Ejemplo

Grupo «Viaje a Sucre» con Ana, Beto y Caro (en ese orden de alta).

- **G1:** «Hostal», 300,00 Bs, pagó Ana, en partes iguales entre los tres → 100,00 cada uno.
- **G2:** «Cena», 100,00 Bs, pagó Beto, en partes iguales entre los tres → Ana 33,34, Beto 33,33, Caro 33,33 (CL-1).
- **P1:** Caro le pagó 50,00 Bs a Ana.

```text
groups:          (1, "Viaje a Sucre", "viaje a sucre", last_expense_no=2, last_payment_no=1)
members:         (1, 1, "Ana", "ana") (2, 1, "Beto", "beto") (3, 1, "Caro", "caro")
expenses:        (1, 1, 1, "Hostal", 30000, "2026-10-20", payer=1, "equal")
                 (2, 1, 2, "Cena",   10000, "2026-10-21", payer=2, "equal")
expense_shares:  (1,1,10000,null) (1,2,10000,null) (1,3,10000,null)
                 (2,1, 3334,null) (2,2, 3333,null) (2,3, 3333,null)
payments:        (1, 1, 1, from=3, to=1, 5000, "2026-10-22")
```

Saldos (RF-10.1): Ana = 30000 − 10000 − 3334 − 5000 = **11666** (le deben 116,66); Beto = 10000 − 10000 − 3333 = **−3333** (debe 33,33); Caro = 5000 − 10000 − 3333 = **−8333** (debe 83,33). Suma: 0 (RF-10.3).

Plan (RF-11.2): Caro le paga 83,33 a Ana; Beto le paga 33,33 a Ana. Dos transferencias para tres integrantes con saldo (RF-11.3).
