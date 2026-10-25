# Spec 001 — MVP de cuentas-claras

### Historial

| Versión | Fecha | Cambio |
|---|---|---|
| 1 | 2026-10-25 | Primera versión, a partir de las [aclaraciones](clarifications.md) D-6…D-20. |
| 2 | 2026-10-25 | Resuelve los hallazgos H-1…H-35 de la [revisión de QA](qa-review.md): definiciones, formato de importes y fechas, numeración G/P, cálculo entero de porcentajes, aviso de pagos, RF-12 (errores de datos), catálogo completo de mensajes y casos límite CL-13…CL-25. |

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
- **HU-8.** Como usuario, quiero que la aplicación me diga claramente qué pasó si mis datos no se pueden leer o guardar, sin perder lo que ya tenía.

## 4. Requisitos funcionales

Notación EARS: *El sistema deberá* (siempre), *Cuando* (evento), *Mientras* (estado), *Si… entonces* (situación no deseada).

### Definiciones

- **Centavo:** la unidad de cálculo. Todo importe se maneja como un número entero de centavos (100 centavos = 1 Bs).
- **Importe válido:** de 1 a 100 000 000 centavos (0,01 a 1 000 000,00 Bs).
- **Hoy:** la fecha del calendario local de la computadora, leída una sola vez al comenzar cada operación.
- **Nombre limpio:** el nombre sin espacios al inicio ni al final y con los espacios internos repetidos reducidos a uno.
- **Nombres equivalentes:** dos nombres limpios que son iguales sin distinguir mayúsculas e ignorando el acento agudo de las vocales (á, é, í, ó, ú) y la diéresis de la «u» (ü). La «ñ» es distinta de la «n».
- **Orden de alta:** el orden en que los integrantes se agregaron al grupo. Es el criterio de desempate de toda la spec, independiente del orden en que se escriban al registrar un gasto.
- **Número de gasto / de pago:** identificador dentro de un grupo, con secuencias separadas: gastos `G1, G2, …` y pagos `P1, P2, …`. Un número eliminado no se reutiliza.
- **Saldo de un integrante:** lo que pagó en gastos + lo que pagó a otros − su parte en los gastos − lo que recibió de otros. Positivo: le deben; negativo: debe; cero: al día.

### RF-1 — Reglas generales

- **RF-1.1** El sistema deberá mostrar todos los importes en bolivianos, con dos decimales, coma decimal, sin separador de miles y seguidos de « Bs» (por ejemplo «1234,50 Bs»).
- **RF-1.2** El sistema deberá aceptar como importe escrito solo dígitos, con un único separador decimal opcional (coma o punto) seguido de uno o dos dígitos; «1,5» y «1.5» equivalen a 1,50 Bs.
- **RF-1.3** Si un importe escrito no cumple RF-1.2 (letras, signos, espacios, unidad, separador de miles, más de dos decimales) o no es un importe válido, entonces el sistema deberá rechazar la operación con E-1.
- **RF-1.4** El sistema deberá aceptar fechas escritas como `DD/MM/AAAA`, desde el 01/01/2000 hasta hoy.
- **RF-1.5** El sistema deberá tratar como el mismo nombre dos nombres equivalentes.
- **RF-1.6** El sistema deberá guardar el nombre limpio, conservando mayúsculas y tildes tal como se escribieron.
- **RF-1.7** Si un nombre limpio está vacío, supera 40 caracteres, contiene caracteres de control (salto de línea, tabulador…) o no tiene ninguna letra ni número, entonces el sistema deberá rechazarlo con E-2.

### RF-2 — Grupos (HU-1)

- **RF-2.1** Cuando el usuario crea un grupo con un nombre válido, el sistema deberá guardarlo sin integrantes y responder con M-1.
- **RF-2.2** Si ya existe un grupo con un nombre equivalente, entonces el sistema deberá rechazar la creación con E-3.
- **RF-2.3** Cuando el usuario pide la lista de grupos, el sistema deberá mostrar cada grupo con su cantidad de integrantes y de gastos, en orden de creación; sin grupos, M-9.
- **RF-2.4** Si el grupo indicado en cualquier operación no existe, entonces el sistema deberá rechazarla con E-4.

### RF-3 — Integrantes (HU-2)

- **RF-3.1** Cuando el usuario agrega un integrante con un nombre válido a un grupo existente, el sistema deberá agregarlo al final del orden de alta y responder con M-2.
- **RF-3.2** Si el grupo ya tiene un integrante con un nombre equivalente, entonces el sistema deberá rechazar el alta con E-3. Un mismo nombre puede existir en grupos distintos.
- **RF-3.3** Si el grupo ya tiene 50 integrantes, entonces el sistema deberá rechazar el alta con E-5.
- **RF-3.4** Cuando el usuario quita un integrante que no figura en ningún gasto (como pagador o participante) ni en ningún pago, el sistema deberá quitarlo y responder con M-3.
- **RF-3.5** Si el integrante figura en algún gasto o pago, entonces el sistema deberá rechazar la baja con E-6.
- **RF-3.6** Cuando el usuario pide los integrantes de un grupo, el sistema deberá listarlos en orden de alta; sin integrantes, M-10.
- **RF-3.7** Si un integrante indicado en cualquier operación no existe en el grupo, entonces el sistema deberá rechazarla con E-4.

### RF-4 — Registro de gastos (HU-3)

- **RF-4.1** Cuando el usuario registra un gasto válido (descripción, importe, quién pagó, participantes, forma de reparto y fecha), el sistema deberá guardarlo con el siguiente número `G` del grupo y la parte de cada participante, y responder con M-4 y el reparto.
- **RF-4.2** Si la descripción limpia está vacía, supera 80 caracteres o contiene caracteres de control, entonces el sistema deberá rechazar el gasto con E-7.
- **RF-4.3** El sistema deberá exigir que quien pagó y cada participante sean integrantes del grupo (RF-3.7).
- **RF-4.4** Si no hay participantes o un participante está repetido (por nombre equivalente), entonces el sistema deberá rechazar el gasto con E-8.
- **RF-4.5** El sistema deberá permitir que quien pagó no esté entre los participantes, y también que sea el único participante.
- **RF-4.6** Cuando el usuario no indica la fecha, el sistema deberá usar hoy.
- **RF-4.7** Si la fecha indicada no cumple RF-1.4, entonces el sistema deberá rechazar el gasto con E-9.
- **RF-4.8** Si el grupo ya tiene 10 000 gastos guardados, entonces el sistema deberá rechazar el gasto con E-5.
- **RF-4.9** El sistema deberá garantizar que la suma de las partes de un gasto sea exactamente su importe.

### RF-5 — Reparto en partes iguales (HU-3)

- **RF-5.1** Cuando el reparto es en partes iguales entre *n* participantes, el sistema deberá dar a cada uno ⌊importe ÷ *n*⌋ centavos.
- **RF-5.2** El sistema deberá asignar los centavos sobrantes (importe mod *n*) de a uno, a los participantes en orden de alta en el grupo, empezando por el primero.

### RF-6 — Reparto por montos exactos (HU-3)

- **RF-6.1** Cuando el reparto es por montos exactos, el sistema deberá exigir para cada participante un monto escrito según RF-1.2, de 0,01 Bs como mínimo.
- **RF-6.2** Si falta el monto de algún participante, sobra uno o alguno es 0,00, entonces el sistema deberá rechazar el gasto con E-10.
- **RF-6.3** Si la suma de los montos no es igual al importe del gasto, entonces el sistema deberá rechazar el gasto con E-11, indicando la suma y la diferencia.

### RF-7 — Reparto por porcentajes (HU-3)

- **RF-7.1** Cuando el reparto es por porcentajes, el sistema deberá exigir para cada participante un porcentaje mayor que 0 y como máximo 100, escrito con hasta dos decimales (coma o punto).
- **RF-7.2** Si falta el porcentaje de algún participante, sobra uno o alguno no cumple RF-7.1, entonces el sistema deberá rechazar el gasto con E-10.
- **RF-7.3** Si los porcentajes no suman exactamente 100, entonces el sistema deberá rechazar el gasto con E-12, indicando cuánto suman.
- **RF-7.4** El sistema deberá expresar cada porcentaje en centésimas de punto (100 % = 10 000) y calcular, solo con enteros, la parte de cada participante como ⌊importe × centésimas ÷ 10 000⌋ y su resto como (importe × centésimas) mod 10 000.
- **RF-7.5** El sistema deberá asignar los centavos sobrantes de a uno a los participantes de mayor resto y, a igual resto, por orden de alta en el grupo.

### RF-8 — Consulta y eliminación de gastos (HU-4)

- **RF-8.1** Cuando el usuario pide los gastos de un grupo, el sistema deberá listarlos todos por fecha ascendente y, en el mismo día, por número ascendente, con su número, fecha, descripción, importe, quién pagó y la parte de cada participante; sin gastos, M-11.
- **RF-8.2** Cuando el usuario elimina un gasto por su número, el sistema deberá eliminarlo junto con sus partes y responder con M-5.
- **RF-8.3** Si el número no corresponde a un gasto del grupo, entonces el sistema deberá rechazar la eliminación con E-13.

### RF-9 — Pagos entre integrantes (HU-5)

- **RF-9.1** Cuando el usuario registra que un integrante pagó un importe válido a otro integrante del mismo grupo, con fecha opcional (RF-4.6, RF-4.7), el sistema deberá guardarlo con el siguiente número `P` y responder con M-6.
- **RF-9.2** Si quien paga y quien recibe son el mismo integrante, entonces el sistema deberá rechazar el pago con E-14.
- **RF-9.3** Si el importe del pago es mayor que la deuda que tenía quien paga antes del pago (su saldo negativo, en positivo, o 0 si no debía nada), entonces el sistema deberá guardar el pago igualmente y añadir el aviso M-7 con su nuevo saldo a favor.
- **RF-9.4** Cuando el usuario elimina un pago por su número, el sistema deberá eliminarlo y responder con M-5; si el número no corresponde a un pago del grupo, E-13.
- **RF-9.5** Cuando el usuario pide los pagos de un grupo, el sistema deberá listarlos por fecha ascendente y número ascendente; sin pagos, M-12.

### RF-10 — Saldos (HU-6)

- **RF-10.1** El sistema deberá calcular el saldo de cada integrante según la definición.
- **RF-10.2** Cuando el usuario pide los saldos, el sistema deberá mostrar a cada integrante, en orden de alta, con «le deben {importe}», «debe {importe}» o «al día».
- **RF-10.3** El sistema deberá garantizar que la suma de todos los saldos de un grupo sea cero.
- **RF-10.4** Si el grupo no tiene integrantes, entonces el sistema deberá responder con M-10 al pedir saldos o el plan.

### RF-11 — Plan de liquidación (HU-7)

- **RF-11.1** Cuando el usuario pide el plan de liquidación, el sistema deberá mostrar una lista de transferencias «{A} le paga {importe} a {B}» que, si se hicieran, dejarían todos los saldos en cero.
- **RF-11.2** El sistema deberá calcular el plan repitiendo: el integrante que más debe le paga al que más le deben, por el menor de los dos importes; a igual importe, se elige el primero en orden de alta, tanto entre deudores como entre acreedores.
- **RF-11.3** El sistema deberá producir como máximo una transferencia menos que la cantidad de integrantes con saldo distinto de cero.
- **RF-11.4** Si todos los saldos son cero, entonces el sistema deberá responder con M-8.
- **RF-11.5** El sistema no deberá registrar ningún pago al mostrar el plan: el usuario registra los pagos cuando se hacen.

### RF-12 — Errores de datos (HU-8)

- **RF-12.1** Si los datos guardados no se pueden leer, están dañados o son de una versión de la aplicación más nueva que la instalada, entonces el sistema deberá informarlo con E-15 y no deberá modificar ni borrar nada.
- **RF-12.2** Si una operación no se puede guardar por completo (disco lleno, sin permisos, corte), entonces el sistema deberá informarlo con E-16 y dejar los datos como estaban antes de la operación.
- **RF-12.3** Si los datos están siendo usados por otra ejecución de la aplicación, entonces el sistema deberá rechazar la operación con E-17 sin cambiar nada.
- **RF-12.4** Cuando la aplicación se usa por primera vez, el sistema deberá crear los datos vacíos sin pedir configuración.

## 5. Mensajes

Todos los textos de la interfaz salen de este catálogo (principio 7).

| Id | Situación | Texto |
|---|---|---|
| M-1 | Grupo creado | «Grupo «{grupo}» creado.» |
| M-2 | Integrante agregado | «{nombre} se agregó a «{grupo}».» |
| M-3 | Integrante quitado | «{nombre} ya no está en «{grupo}».» |
| M-4 | Gasto registrado | «Gasto {G} registrado: {descripción}, {importe}, pagó {nombre}.» y una línea «  {nombre}: {parte}» por participante |
| M-5 | Gasto o pago eliminado | «{número} eliminado.» |
| M-6 | Pago registrado | «Pago {P} registrado: {A} le pagó {importe} a {B}.» |
| M-7 | Pago que supera la deuda | «Aviso: después de este pago, a {A} le deben {importe}.» |
| M-8 | Grupo a mano | «Todos están a mano.» |
| M-9 | Sin grupos | «Todavía no hay grupos.» |
| M-10 | Grupo sin integrantes | ««{grupo}» no tiene integrantes.» |
| M-11 | Sin gastos | ««{grupo}» no tiene gastos.» |
| M-12 | Sin pagos | ««{grupo}» no tiene pagos.» |
| E-1 | Importe inválido | «El importe debe estar entre 0,01 y 1000000,00 Bs, con hasta dos decimales y sin otros signos.» |
| E-2 | Nombre inválido | «El nombre debe tener de 1 a 40 caracteres, con al menos una letra o un número y sin saltos de línea.» |
| E-3 | Nombre repetido | «Ya existe «{nombre}».» |
| E-4 | No existe | «No existe «{nombre}».» |
| E-5 | Límite alcanzado | «Se alcanzó el máximo de {límite} {integrantes/gastos} en «{grupo}».» |
| E-6 | Baja con historial | «No se puede quitar a {nombre}: figura en gastos o pagos.» |
| E-7 | Descripción inválida | «La descripción debe tener de 1 a 80 caracteres, sin saltos de línea.» |
| E-8 | Participantes inválidos | «Indica al menos un participante, sin repetir.» |
| E-9 | Fecha inválida | «La fecha debe tener el formato DD/MM/AAAA, entre el 01/01/2000 y hoy.» |
| E-10 | Montos o porcentajes incompletos | «Indica un {monto/porcentaje} válido para cada participante, y solo para ellos.» |
| E-11 | Montos que no suman | «Los montos suman {suma}; la diferencia con {total} es {diferencia}.» |
| E-12 | Porcentajes que no suman | «Los porcentajes suman {suma} %; deben sumar 100 %.» |
| E-13 | Número inexistente | «No existe {número} en «{grupo}».» |
| E-14 | Pago a uno mismo | «Quien paga y quien recibe deben ser personas distintas.» |
| E-15 | Datos ilegibles | «No se pueden leer los datos ({motivo}). No se modificó nada.» |
| E-16 | No se pudo guardar | «No se pudo guardar ({motivo}). Los datos quedaron como estaban.» |
| E-17 | Datos en uso | «Los datos están en uso por otra ventana; vuelve a intentarlo.» |

## 6. Datos que se guardan

- **Grupo:** nombre y orden de creación.
- **Integrante:** grupo, nombre y orden de alta.
- **Gasto:** grupo, número `G`, descripción, importe, fecha, quién pagó, forma de reparto y la parte de cada participante (y el monto o porcentaje indicado, cuando aplica).
- **Pago:** grupo, número `P`, quién paga, quién recibe, importe y fecha.
- **Últimos números usados** de gastos y pagos por grupo (para no reutilizarlos).

Los saldos y el plan de liquidación no se guardan: se calculan cada vez.

## 7. Requisitos no funcionales

- **RNF-1 Exactitud:** ningún cálculo pierde ni crea centavos; todo se calcula con enteros.
- **RNF-2 Determinismo:** los mismos datos producen siempre los mismos repartos, saldos y plan.
- **RNF-3 Rapidez:** listar gastos, saldos y plan responde en menos de 1 segundo con un grupo de 50 integrantes y 10 000 gastos, en una computadora común de los últimos 5 años.
- **RNF-4 Integridad:** un gasto (con todas sus partes) o un pago se guarda completo o no se guarda.
- **RNF-5 Funcionamiento local:** no requiere internet, cuenta ni configuración.
- **RNF-6 Idioma:** todos los textos de la interfaz están en español y salen del catálogo de la sección 5.
- **RNF-7 Trazabilidad:** cada RF, RNF y CL tiene al menos un test automático que lo cita.

## 8. Casos límite

| # | Situación | Comportamiento esperado |
|---|---|---|
| CL-1 | 100,00 Bs entre 3 en partes iguales | 33,34 + 33,33 + 33,33; el centavo va al primero en orden de alta |
| CL-2 | 0,01 Bs entre 3 en partes iguales | 0,01 + 0,00 + 0,00 |
| CL-3 | 0,02 Bs entre 3 | 0,01 + 0,01 + 0,00 |
| CL-4 | 100,00 Bs al 33,33 % / 33,33 % / 33,34 % | 33,33 + 33,33 + 33,34 |
| CL-5 | Montos exactos que suman 0,01 menos que el total | Rechazado con E-11, diferencia 0,01 Bs |
| CL-6 | Quien pagó no participa | Su saldo sube por el importe completo |
| CL-7 | Pago mayor que la deuda | Se guarda y avisa con M-7 (RF-9.3) |
| CL-8 | Eliminar el único gasto de un grupo | Todos los saldos vuelven a 0 |
| CL-9 | Plan con un deudor y tres acreedores | Tres transferencias del deudor |
| CL-10 | Plan con todos a mano | M-8 |
| CL-11 | «Ana», « ána » y «ANA» en el mismo grupo; «Peña» y «Pena» | Las tres primeras son el mismo nombre; «Peña» y «Pena» son distintos |
| CL-12 | Gasto con fecha de mañana, del 31/12/1999 o «31/02/2026» | Rechazado con E-9 |
| CL-13 | Importes «1.234,50», «+5», «5 Bs», «5,001», «0», «1000000,01» | Rechazados con E-1 |
| CL-14 | Nombres «   », «!!!», de 41 caracteres o con un salto de línea | Rechazados con E-2 |
| CL-15 | Gasto cuyo único participante es quien pagó | Se acepta; ningún saldo cambia |
| CL-16 | 0,01 Bs entre 50 participantes | 0,01 para el primero en orden de alta y 0,00 para los otros 49 |
| CL-17 | Un participante al 100 %; porcentajes «33,33», «33.33» y «33,34» | Se aceptan; el reparto suma el total |
| CL-18 | Registrar gasto, saldos o plan en un grupo sin integrantes | Gasto rechazado con E-4; saldos y plan responden M-10 |
| CL-19 | Saldo acumulado mayor que 1 000 000,00 Bs | Se muestra completo, sin tope |
| CL-20 | Eliminar el único gasto en que figura un integrante | Después se le puede quitar del grupo |
| CL-21 | «Ana» en dos grupos distintos | Se permite |
| CL-22 | Saldos de +0,01 y −0,01 | Plan con una transferencia de 0,01 Bs |
| CL-23 | Dos deudores con la misma deuda | Paga primero el de antes en orden de alta |
| CL-24 | Eliminar G2 y registrar otro gasto | El nuevo es G3; G2 no se reutiliza |
| CL-25 | Datos de una versión más nueva, dañados, o un fallo a mitad de un guardado | E-15 o E-16, sin modificar nada |

## 9. Fuera de alcance

- Varias monedas y tipos de cambio.
- Gastos recurrentes, categorías, fotos de recibos.
- Varias personas que pagan un mismo gasto.
- Reparto «por partes».
- Editar gastos o pagos (solo eliminar).
- Renombrar integrantes; borrar o archivar grupos.
- Filtros y paginación de listados.
- Registrar automáticamente los pagos del plan de liquidación.
- Cuentas de usuario, sincronización, notificaciones, pagos en línea.
- Varias ejecuciones simultáneas (se rechazan, RF-12.3).
- Exportar o importar datos.

## 10. Criterios de finalización

1. Cada RF, RNF y CL tiene al menos un test automático que lo cita, y todos pasan (RNF-7).
2. Los repartos de los CL-1 a CL-4, CL-16 y CL-17 dan exactamente los importes de la tabla.
3. Un test con miles de importes y repartos al azar comprueba que todo reparto suma el total, que los saldos suman cero y que el plan los deja en cero.
4. RNF-3 se midió en las condiciones indicadas y se cumple.
5. Cada principio de la constitución se comprueba con la verificación indicada en [`constitution-proposal.md`](../../docs/constitution-proposal.md).
6. Todos los textos de la interfaz coinciden con la sección 5.

## 11. Supuestos

- La persona que usa la aplicación es de confianza y registra los datos de buena fe.
- Se usa una sola ejecución de la aplicación a la vez; si hay otra, se aplica RF-12.3.

## 12. Dudas abiertas

Ninguna. La duda de la versión 1 (identificar gastos) quedó resuelta con la numeración `G`/`P` (H-5, H-18).
