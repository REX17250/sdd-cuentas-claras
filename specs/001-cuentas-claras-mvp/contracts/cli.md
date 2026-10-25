# Contrato de la CLI — spec 001

Parte del [plan](plan.md). Define comandos, salidas y códigos de salida. Los textos son los de la sección 5 de la [spec](../spec.md); aquí solo se fija cómo se piden y se muestran.

## 1. Invocación

`python -m cuentas <comando> <subcomando> [argumentos] [opciones]`

- Los nombres con espacios van entre comillas: `"Viaje a Sucre"`.
- Las opciones que se repiten (`--entre`, `--monto`, `--porcentaje`) admiten varios valores.
- En `--monto` y `--porcentaje`, el valor es `NOMBRE=VALOR`; se separa por el **último** `=`, así un nombre puede contener `=`.
- Toda la ayuda y los errores de uso están en español (D-21).

## 2. Comandos

| Comando | Argumentos y opciones | Éxito (stdout) | RF |
|---|---|---|---|
| `grupo crear` | `NOMBRE` | M-1 | RF-2.1, RF-2.2 |
| `grupo listar` | — | Una línea por grupo: `{grupo} — {n} integrantes, {m} gastos`; o M-9 | RF-2.3 |
| `miembro agregar` | `GRUPO NOMBRE` | M-2 | RF-3.1-3.3 |
| `miembro quitar` | `GRUPO NOMBRE` | M-3 | RF-3.4, RF-3.5 |
| `miembro listar` | `GRUPO` | Un nombre por línea, en orden de alta; o M-10 | RF-3.6 |
| `gasto agregar` | `GRUPO --desc TEXTO --importe I --pago NOMBRE --entre NOMBRE…` y **una** de `--igual` (por defecto), `--monto NOMBRE=I…`, `--porcentaje NOMBRE=P…`; opcional `--fecha DD/MM/AAAA` | M-4 con el reparto | RF-4 a RF-7 |
| `gasto listar` | `GRUPO` | Ver 3.1; o M-11 | RF-8.1 |
| `gasto eliminar` | `GRUPO G{n}` | M-5 | RF-8.2, RF-8.3 |
| `pago registrar` | `GRUPO --de NOMBRE --a NOMBRE --importe I [--fecha DD/MM/AAAA]` | M-6 (y M-7 si corresponde) | RF-9.1-9.3 |
| `pago listar` | `GRUPO` | `{P} {fecha} {A} → {B} {importe}` por línea; o M-12 | RF-9.5 |
| `pago eliminar` | `GRUPO P{n}` | M-5 | RF-9.4 |
| `saldos` | `GRUPO` | Ver 3.2; o M-10 | RF-10 |
| `liquidar` | `GRUPO` | Ver 3.3; M-8 o M-10 | RF-11 |
| `ayuda` (o sin argumentos, `-h`, `--help`) | — | Lista de comandos con un ejemplo de cada uno | — |

Reglas de `gasto agregar`:
- Con `--monto` o `--porcentaje`, cada participante de `--entre` debe tener exactamente un valor, y no puede haber valores de quien no está en `--entre` (RF-6.2, RF-7.2 → E-10).
- Usar más de una de `--igual`, `--monto`, `--porcentaje` es un error de uso.

## 3. Formatos de salida

### 3.1 `gasto listar`
```text
G1  20/10/2026  Hostal                 300,00 Bs  pagó Ana
      Ana 100,00 Bs · Beto 100,00 Bs · Caro 100,00 Bs
G2  21/10/2026  Cena                   100,00 Bs  pagó Beto
      Ana 33,34 Bs · Beto 33,33 Bs · Caro 33,33 Bs
```

### 3.2 `saldos`
```text
Ana    le deben 116,66 Bs
Beto   debe 33,33 Bs
Caro   debe 83,33 Bs
```

### 3.3 `liquidar`
```text
Caro le paga 83,33 Bs a Ana
Beto le paga 33,33 Bs a Ana
```

## 4. Códigos de salida y canales

| Código | Cuándo | Canal |
|---|---|---|
| `0` | Éxito, incluidas listas vacías, M-8 y el aviso M-7 | stdout |
| `1` | Validación o uso: E-1 a E-14 y errores de argumentos | stderr |
| `2` | Datos: E-15, E-16, E-17 | stderr |

Con Ctrl+C la aplicación termina con código `1` sin guardar la operación en curso (RNF-4).
