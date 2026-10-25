# Spec 001 — MVP de cuentas-claras

### Historial

| Versión | Fecha | Cambio |
|---|---|---|
| 1 | 2026-10-25 | Primera versión, a partir de las [aclaraciones](clarifications.md) D-6…D-20. |

## 1. Contexto y objetivo

Cuando varias personas comparten gastos (un piso, un viaje, una cena), alguien paga y los demás quedan debiendo partes distintas. Llevarlo en una hoja de cálculo o de memoria termina en discusiones: no está claro quién debe cuánto, los centavos no cuadran y al final se hacen más transferencias de las necesarias.

**Objetivo:** una aplicación en la que la persona que lleva las cuentas registra los gastos y pagos de un grupo y obtiene, en cualquier momento, el saldo de cada integrante y un plan corto de transferencias para quedar a mano.

**Por qué importa:** si el reparto no suma exactamente el total, o si la regla de redondeo cambia de un día a otro, el grupo deja de confiar en las cuentas. El MVP prioriza la exactitud y la claridad del resultado sobre cualquier otra cosa.

## 2. Usuarios y roles

- **Persona que lleva las cuentas:** usa la aplicación en su computadora; registra grupos, integrantes, gastos y pagos, y consulta saldos. Es el único rol (D-7).
- **Integrantes del grupo:** no usan la aplicación; aparecen solo como nombres. Reciben del usuario el plan de liquidación.

## 3. Historias de usuario

- **HU-1.** Como usuario, quiero crear un grupo para separar las cuentas del piso de las de un viaje.
- **HU-2.** Como usuario, quiero agregar y quitar integrantes de un grupo.
- **HU-3.** Como usuario, quiero registrar un gasto, quién lo pagó y entre quiénes se reparte, en partes iguales, por montos exactos o por porcentajes.
- **HU-4.** Como usuario, quiero ver y eliminar los gastos registrados para corregir errores.
- **HU-5.** Como usuario, quiero registrar que un integrante le pagó a otro para que los saldos se actualicen.
- **HU-6.** Como usuario, quiero ver cuánto debe o le deben a cada integrante.
- **HU-7.** Como usuario, quiero un plan con pocas transferencias para que el grupo quede a mano.

## 4. Requisitos funcionales

Notación EARS: *El sistema deberá* (siempre), *Cuando* (evento), *Mientras* (estado), *Si… entonces* (situación no deseada).

### RF-1 — Reglas generales

- **RF-1.1** El sistema deberá manejar una sola moneda, el boliviano (Bs), con importes de dos decimales.
- **RF-1.2** El sistema deberá aceptar importes entre 0,01 y 1 000 000,00 Bs, escritos con coma o con punto como separador decimal y con hasta dos decimales.
- **RF-1.3** Si un importe no es un número, es cero o negativo, tiene más de dos decimales o supera el máximo, entonces el sistema deberá rechazarlo e indicar el motivo.
- **RF-1.4** El sistema deberá mostrar los importes con dos decimales, coma decimal y la unidad «Bs» (por ejemplo «1 234,50 Bs»).
- **RF-1.5** El sistema deberá considerar iguales dos nombres (de grupo o de integrante) que solo difieren en mayúsculas, tildes o espacios al inicio y al final.
- **RF-1.6** Si un nombre está vacío o supera 40 caracteres, entonces el sistema deberá rechazarlo.

### RF-2 — Grupos (HU-1)

- **RF-2.1** Cuando el usuario crea un grupo con un nombre válido, el sistema deberá guardarlo sin integrantes y confirmarlo.
- **RF-2.2** Si ya existe un grupo con un nombre igual, entonces el sistema deberá rechazar la creación.
- **RF-2.3** Cuando el usuario pide la lista de grupos, el sistema deberá mostrar cada grupo con su cantidad de integrantes y de gastos, en orden de creación.

### RF-3 — Integrantes (HU-2)

- **RF-3.1** Cuando el usuario agrega un integrante con un nombre válido a un grupo existente, el sistema deberá agregarlo al final del orden de alta del grupo, con saldo 0.
- **RF-3.2** Si el grupo ya tiene un integrante con un nombre igual, entonces el sistema deberá rechazar el alta.
- **RF-3.3** Si el grupo ya tiene 50 integrantes, entonces el sistema deberá rechazar el alta.
- **RF-3.4** Cuando el usuario quita un integrante cuyo saldo es 0 y que no figura en ningún gasto ni pago, el sistema deberá quitarlo.
- **RF-3.5** Si el integrante tiene saldo distinto de 0 o figura en algún gasto o pago, entonces el sistema deberá rechazar la baja e indicar el motivo.
- **RF-3.6** Cuando el usuario pide los integrantes de un grupo, el sistema deberá listarlos en orden de alta.

### RF-4 — Registro de gastos (HU-3)

- **RF-4.1** Cuando el usuario registra un gasto con descripción, importe, quién pagó, participantes y forma de reparto válidos, el sistema deberá guardar el gasto y la parte de cada participante, y confirmar con el reparto resultante.
- **RF-4.2** El sistema deberá exigir una descripción de 1 a 80 caracteres.
- **RF-4.3** El sistema deberá exigir que quien pagó y cada participante sean integrantes del grupo.
- **RF-4.4** El sistema deberá exigir al menos un participante y no aceptar participantes repetidos.
- **RF-4.5** El sistema deberá permitir que quien pagó no esté entre los participantes.
- **RF-4.6** Cuando el usuario no indica la fecha, el sistema deberá usar la fecha de hoy.
- **RF-4.7** Si la fecha indicada es posterior a hoy o no es una fecha válida, entonces el sistema deberá rechazar el gasto.
- **RF-4.8** Si el grupo ya tiene 10 000 gastos, entonces el sistema deberá rechazar el gasto.
- **RF-4.9** El sistema deberá garantizar que la suma de las partes de un gasto sea exactamente su importe.

### RF-5 — Reparto en partes iguales (HU-3)

- **RF-5.1** Cuando el reparto es en partes iguales, el sistema deberá dar a cada participante el importe dividido entre la cantidad de participantes, truncado a centavos.
- **RF-5.2** El sistema deberá asignar los centavos sobrantes de a uno, a los participantes en orden de alta en el grupo, empezando por el primero.

### RF-6 — Reparto por montos exactos (HU-3)

- **RF-6.1** Cuando el reparto es por montos exactos, el sistema deberá exigir un monto válido para cada participante.
- **RF-6.2** Si la suma de los montos no es igual al importe del gasto, entonces el sistema deberá rechazar el gasto e indicar la diferencia.

### RF-7 — Reparto por porcentajes (HU-3)

- **RF-7.1** Cuando el reparto es por porcentajes, el sistema deberá exigir un porcentaje para cada participante, con hasta dos decimales.
- **RF-7.2** Si los porcentajes no suman 100, entonces el sistema deberá rechazar el gasto e indicar cuánto suman.
- **RF-7.3** El sistema deberá calcular cada parte como el importe por su porcentaje, truncado a centavos, y asignar los centavos sobrantes de a uno a los participantes con mayor resto, desempatando por orden de alta.

### RF-8 — Consulta y eliminación de gastos (HU-4)

- **RF-8.1** Cuando el usuario pide los gastos de un grupo, el sistema deberá listarlos por fecha, con su número, descripción, importe, quién pagó y la parte de cada participante.
- **RF-8.2** Cuando el usuario elimina un gasto por su número, el sistema deberá eliminarlo junto con sus partes y recalcular los saldos.
- **RF-8.3** Si el número de gasto no existe en el grupo, entonces el sistema deberá informarlo.

### RF-9 — Pagos entre integrantes (HU-5)

- **RF-9.1** Cuando el usuario registra que un integrante pagó un importe a otro integrante distinto del mismo grupo, el sistema deberá guardar el pago y confirmarlo.
- **RF-9.2** Si quien paga y quien recibe son la misma persona, entonces el sistema deberá rechazar el pago.
- **RF-9.3** Si el pago supera lo que quien paga le debía al grupo, entonces el sistema deberá guardarlo igualmente y avisar que su saldo pasa a ser a favor.
- **RF-9.4** El sistema deberá permitir listar y eliminar pagos igual que los gastos.

### RF-10 — Saldos (HU-6)

- **RF-10.1** El sistema deberá calcular el saldo de cada integrante como: lo que pagó en gastos, más lo que pagó a otros, menos su parte en los gastos, menos lo que recibió de otros.
- **RF-10.2** Cuando el usuario pide los saldos, el sistema deberá mostrar a cada integrante con su saldo, indicando si le deben (positivo), debe (negativo) o está al día (cero).
- **RF-10.3** El sistema deberá garantizar que la suma de todos los saldos de un grupo sea cero.

### RF-11 — Plan de liquidación (HU-7)

- **RF-11.1** Cuando el usuario pide el plan de liquidación, el sistema deberá mostrar una lista de transferencias «A le paga X a B» que, si se hicieran, dejarían todos los saldos en cero.
- **RF-11.2** El sistema deberá calcular el plan haciendo que el integrante que más debe le pague al que más le deben, por el menor de los dos importes, y repetir hasta que no queden saldos.
- **RF-11.3** El sistema deberá producir como máximo una transferencia menos que la cantidad de integrantes con saldo distinto de cero.
- **RF-11.4** Si todos los saldos son cero, entonces el sistema deberá informar que el grupo está a mano.

## 5. Mensajes

| Id | Situación | Texto |
|---|---|---|
| M-1 | Grupo creado | «Grupo «{nombre}» creado.» |
| M-2 | Integrante agregado | «{nombre} se agregó a «{grupo}».» |
| M-3 | Gasto registrado | «Gasto #{n} registrado: {descripción}, {importe}, pagó {nombre}.» seguido del reparto |
| M-4 | Pago registrado | «{A} le pagó {importe} a {B}.» |
| M-5 | Pago que supera la deuda | «Aviso: después de este pago, a {A} le deben {importe}.» |
| M-6 | Grupo a mano | «Todos están a mano.» |
| E-1 | Importe inválido | «El importe debe estar entre 0,01 y 1 000 000,00 Bs, con hasta dos decimales.» |
| E-2 | Nombre repetido | «Ya existe «{nombre}».» |
| E-3 | No existe | «No existe «{nombre}».» |
| E-4 | Montos que no suman | «Los montos suman {suma}; faltan {diferencia} para {total}.» |
| E-5 | Porcentajes que no suman | «Los porcentajes suman {suma} %; deben sumar 100 %.» |
| E-6 | Baja con historial | «No se puede quitar a {nombre}: figura en gastos o pagos.» |

## 6. Datos que se guardan

- **Grupo:** nombre y orden de creación.
- **Integrante:** grupo, nombre y orden de alta.
- **Gasto:** grupo, número, descripción, importe, fecha, quién pagó, forma de reparto y la parte de cada participante.
- **Pago:** grupo, número, quién paga, quién recibe, importe y fecha.

Los saldos y el plan de liquidación no se guardan: se calculan cada vez.

## 7. Requisitos no funcionales

- **RNF-1 Exactitud:** ningún cálculo pierde ni crea centavos.
- **RNF-2 Determinismo:** los mismos datos producen siempre los mismos repartos, saldos y plan.
- **RNF-3 Rapidez:** cualquier consulta responde en menos de 1 segundo con un grupo de 50 integrantes y 10 000 gastos.
- **RNF-4 Integridad:** un gasto o pago se guarda completo o no se guarda.
- **RNF-5 Funcionamiento local:** no requiere internet ni cuenta.
- **RNF-6 Idioma:** todos los textos de la interfaz están en español.

## 8. Casos límite

| # | Situación | Comportamiento esperado |
|---|---|---|
| CL-1 | 100,00 Bs entre 3 en partes iguales | 33,34 + 33,33 + 33,33 (el centavo va al primero en orden de alta) |
| CL-2 | 0,01 Bs entre 3 en partes iguales | 0,01 + 0,00 + 0,00 |
| CL-3 | 0,02 Bs entre 3 | 0,01 + 0,01 + 0,00 |
| CL-4 | 100,00 Bs al 33,33 % / 33,33 % / 33,34 % | 33,33 + 33,33 + 33,34 |
| CL-5 | Montos exactos que suman 0,01 menos que el total | Rechazado, indica la diferencia |
| CL-6 | Quien pagó no participa | Su saldo sube por el importe completo |
| CL-7 | Pago mayor que la deuda | Se guarda y avisa (RF-9.3) |
| CL-8 | Eliminar el único gasto de un grupo | Todos los saldos vuelven a 0 |
| CL-9 | Plan con un deudor y tres acreedores | Tres transferencias del deudor |
| CL-10 | Plan con todos a mano | «Todos están a mano.» |
| CL-11 | «Ana» y « ána » en el mismo grupo | Rechazado como repetido |
| CL-12 | Gasto con fecha de mañana | Rechazado |

## 9. Fuera de alcance

- Varias monedas y tipos de cambio.
- Gastos recurrentes, categorías, fotos de recibos.
- Varias personas que pagan un mismo gasto.
- Reparto «por partes».
- Editar gastos o pagos (solo eliminar).
- Renombrar integrantes; borrar o archivar grupos.
- Cuentas de usuario, sincronización, notificaciones, pagos en línea.
- Exportar o importar datos.

## 10. Criterios de finalización

1. Cada RF y cada CL tiene al menos un test automático que lo cita, y todos pasan.
2. Los repartos de los CL-1 a CL-4 dan exactamente los importes de la tabla.
3. Un test con miles de importes al azar comprueba que todo reparto suma el total y que los saldos suman cero.
4. Se cumplen los 7 principios de la [constitución](../../docs/constitution.md).

## 11. Supuestos

- La persona que usa la aplicación es de confianza y registra los datos de buena fe.
- La fecha de hoy es la del calendario local de la computadora.

## 12. Dudas abiertas

- **[NECESITA ACLARACIÓN]** ¿Cómo se identifica un gasto para eliminarlo si hay muchos con la misma descripción? *Provisional:* por su número dentro del grupo.
