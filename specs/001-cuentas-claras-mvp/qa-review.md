# Revisión de QA — Spec 001 (MVP de cuentas-claras)

- **Documento revisado:** spec 001, versión 1 (2026-10-25)
- **Referencia:** [constitución](../../docs/constitution.md) y [aclaraciones](clarifications.md)
- **Fecha de la revisión:** 2026-10-25
- **Alcance:** las secciones 1 a 5 solo detectan, sin proponer soluciones. Las resoluciones (fase 4) están en la sección 6 y se aplican en la versión 2 de la spec.
- **Nota de método:** esta revisión la hizo la misma IA que redactó la spec. Para reducir ese sesgo, cada requisito se leyó buscando una segunda interpretación posible, un caso que lo rompa y una regla de la constitución que lo contradiga.

Los números de requisito (RF-x.y, RNF-x, CL-x, M-x, E-x) se refieren a la **versión 1** de la spec.

## 1. Ambigüedades

- **H-1 — RF-1.2:** no dice si se aceptan separadores de miles («1.234,50»), signos («+5»), espacios o la unidad («10 Bs»). Con coma y punto como decimales a la vez, «1.234» puede leerse como mil doscientos treinta y cuatro o como uno coma doscientos treinta y cuatro.
- **H-2 — RF-1.5:** «tildes» no está definido: no se sabe si la «ñ» equivale a la «n», si la diéresis («ü») cuenta, ni si los espacios repetidos en el interior («Ana  María») hacen distinto un nombre.
- **H-3 — RF-1.6:** no dice si los 40 caracteres se cuentan antes o después de quitar espacios, ni si se aceptan saltos de línea, tabuladores o nombres sin ninguna letra ni número («!!!»).
- **H-4 — RF-8.1:** «por fecha» no dice si el orden es ascendente o descendente ni cómo se ordenan dos gastos del mismo día.
- **H-5 — RF-8.2 / RF-9.4:** «su número» no dice si la numeración es por grupo, si se reutiliza un número tras eliminar, ni si gastos y pagos comparten numeración («eliminar el 3» podría ser un gasto o un pago).
- **H-6 — RF-5.2:** «orden de alta en el grupo» puede confundirse con el orden en que se escriben los participantes al registrar el gasto.
- **H-7 — RF-7.1:** no dice si se acepta un porcentaje de 0, negativo o mayor que 100.
- **H-8 — RF-7.3:** «mayor resto» no está definido (¿resto en fracción de centavo? ¿antes o después de truncar?), y «el importe por su porcentaje» no dice cómo se multiplica sin decimales.
- **H-9 — RF-9.3:** «lo que quien paga le debía al grupo» no está definido como cantidad; tampoco dice qué pasa si quien paga ya tenía saldo a favor antes del pago, ni si se avisa cuando quien recibe pasa a deber.
- **H-10 — RF-10.2:** no dice el orden de la lista de saldos ni cómo se escriben los saldos negativos (signo, palabras).
- **H-11 — RF-11.2:** no dice cómo se desempata cuando dos integrantes deben (o les deben) lo mismo, aunque D-15 lo decidió y RNF-2 exige determinismo.
- **H-12 — RF-4.6 / RF-4.7:** el formato de fecha que escribe el usuario no está definido, ni hay una fecha mínima; «hoy» solo aparece en los supuestos, no en un requisito.
- **H-13 — RF-8.1:** con 10 000 gastos, listar todo con la parte de cada participante produce una salida enorme; no se dice si hay paginación o filtros.
- **H-14 — RF-1.4:** el ejemplo «1 234,50 Bs» usa un espacio como separador de miles, pero RF-1.2 no acepta escribirlo así; no dice qué espacio es ni cómo se muestran los negativos.

## 2. Contradicciones entre requisitos

- **H-15 — RF-1.3 frente a CL-2 y RF-6.1:** RF-1.3 rechaza «un importe» de cero, pero CL-2 asigna partes de 0,00 Bs; RF-6.1 exige «un monto válido» para cada participante sin decir si 0 lo es.
- **H-16 — RF-3.4 / RF-3.5:** exige «saldo 0 y que no figure en ningún gasto ni pago», pero quien no figura en nada siempre tiene saldo 0: la condición doble no dice qué comprobar primero ni qué mensaje dar. E-6 solo menciona el historial.
- **H-17 — RF-9.4 frente a RF-8:** «listar y eliminar pagos igual que los gastos» hereda la numeración de RF-8, lo que choca con H-5: un mismo número podría identificar dos cosas.
- **H-18 — Sección 12 frente a RF-8.2:** la duda abierta pregunta cómo se identifica un gasto, pero RF-8.2 ya lo resuelve («por su número»). La duda está cerrada en un lugar y abierta en otro.
- **H-19 — Sección 6 frente a RF-9.1:** «Datos que se guardan» incluye la fecha del pago, pero ningún RF dice de dónde sale ni qué fechas se aceptan.
- **H-20 — Criterios de finalización frente a RNF:** el criterio 1 exige un test por RF y CL, pero ninguno cubre los RNF (rendimiento, integridad), y RF-1.1 («una sola moneda») no describe un comportamiento verificable.

## 3. Casos límite no cubiertos

- **H-21:** gasto cuyo único participante es quien pagó (no cambia ningún saldo); ¿se acepta?
- **H-22:** 0,01 Bs entre 50 participantes (49 partes de 0,00 Bs).
- **H-23:** un solo participante al 100 %, y porcentajes como 33,33 + 33,33 + 33,34 escritos con distintos separadores.
- **H-24:** grupo sin integrantes: registrar un gasto, pedir saldos o el plan.
- **H-25:** saldos acumulados mayores que 1 000 000,00 Bs (el límite es por importe, no por saldo): cómo se muestran.
- **H-26:** pago con fecha futura o inválida, o entre integrantes de grupos distintos.
- **H-27:** eliminar un gasto que era lo único que impedía quitar a un integrante (después debe poder quitarse).
- **H-28:** el mismo nombre de integrante en dos grupos distintos (¿se permite?).
- **H-29:** dos ejecuciones de la aplicación a la vez sobre los mismos datos.
- **H-30:** datos guardados ilegibles, de una versión más nueva de la aplicación, o un error del disco a mitad de una operación.
- **H-31:** plan de liquidación cuando un integrante tiene saldo de 0,01 Bs (la transferencia mínima).

## 4. Conflictos con la constitución

- **H-32 — Principio 5 (dinero exacto) frente a RF-7:** los porcentajes con dos decimales y «el importe por su porcentaje» invitan a usar coma flotante; la spec no fija una forma entera de calcularlo.
- **H-33 — Principio 6 (persistencia controlada):** RNF-4 dice que se guarda «completo o no se guarda», pero ningún RF dice qué ve el usuario si falla el guardado o si los datos no se pueden leer.
- **H-34 — Principio 7 (idioma) y principio 4 (tests):** la tabla de mensajes no cubre la mayoría de los rechazos (nombre largo, límite de 50 integrantes, participante inexistente, fecha futura, límite de gastos, pago a uno mismo…), así que no hay texto que centralizar ni que probar.
- **H-35 — Principio 2 (spec antes que código):** el criterio de finalización 4 («se cumplen los 7 principios») no dice cómo se comprueba, aunque la constitución ya lo define.

## 5. Resumen

35 hallazgos: 14 ambigüedades, 6 contradicciones, 11 casos límite y 4 conflictos con la constitución. Los de mayor impacto son H-8 y H-32 (cálculo de porcentajes sin coma flotante), H-5 y H-17 (identificación de gastos y pagos), H-9 (aviso de pago) y H-30 y H-33 (fallos de datos).

## 6. Resolución (fase 4)

Cada hallazgo se resuelve en la versión 2 de la spec con la opción más simple que lo cierra. «→» indica dónde quedó.

| Hallazgo | Resolución | En la v2 |
|---|---|---|
| H-1 | Importe = dígitos, con un único separador decimal opcional (coma o punto) seguido de 1 o 2 dígitos. Sin separador de miles, signo, espacios ni unidad. «1,5» es 1,50. | RF-1.2, RF-1.3, CL-13 |
| H-2 | Nombres equivalentes: sin distinguir mayúsculas; ignorando el acento agudo de las vocales y la diéresis de la «u»; la «ñ» es distinta de la «n»; los espacios internos repetidos cuentan como uno. | Definiciones, RF-1.5, CL-11 |
| H-3 | «Nombre limpio» = sin espacios al inicio ni al final y con los internos reducidos a uno. De 1 a 40 caracteres limpios; sin caracteres de control; con al menos una letra o un número. | RF-1.6, RF-1.7, CL-14 |
| H-4 | Gastos y pagos: por fecha ascendente y, en el mismo día, por número ascendente. | RF-8.1, RF-9.5 |
| H-5, H-17 | Numeración por grupo y separada: gastos `G1, G2…` y pagos `P1, P2…`. Un número eliminado no se reutiliza. | Definiciones, RF-4.1, RF-9.1 |
| H-6 | Se aclara «orden de alta en el grupo, no el orden en que se escriben». | RF-5.2, RF-7.3 |
| H-7 | Cada porcentaje es mayor que 0 y como máximo 100, con hasta dos decimales. | RF-7.1 |
| H-8, H-32 | Se calcula en enteros: un porcentaje se expresa en centésimas de punto (100 % = 10 000). Parte = ⌊importe en centavos × centésimas ÷ 10 000⌋; resto = (importe en centavos × centésimas) mod 10 000. Sobrantes de a uno al mayor resto, desempate por orden de alta. | RF-7.3, RF-7.4 |
| H-9 | «Deuda antes del pago» = el saldo negativo de quien paga, en positivo, o 0 si no debía. Si el pago la supera, se guarda y se avisa con el nuevo saldo a favor de quien paga. No se avisa por quien recibe. | RF-9.3 |
| H-10 | Saldos en orden de alta; se muestran en palabras: «le deben 20,00 Bs», «debe 20,00 Bs», «al día». | RF-10.2 |
| H-11 | Desempate por orden de alta, tanto entre deudores como entre acreedores. | RF-11.2 |
| H-12 | Fecha en formato `DD/MM/AAAA`, no anterior al 01/01/2000 ni posterior a hoy; «hoy» es la fecha local de la computadora, recibida una sola vez por operación. | Definiciones, RF-4.6, RF-4.7 |
| H-13 | Sin paginación en el MVP: se lista todo; el rendimiento se cubre con RNF-3. Filtros, fuera de alcance. | RF-8.1, sección 9 |
| H-14 | Los importes se muestran sin separador de miles («1234,50 Bs»); los negativos no se muestran con signo sino con palabras (RF-10.2). | RF-1.4 |
| H-15 | El **importe de un gasto o pago** va de 0,01 a 1 000 000,00. Un **monto exacto** por participante va de 0,01 al importe. Las **partes calculadas** (iguales o porcentajes) pueden ser 0,00 por redondeo. | RF-1.3, RF-6.1, CL-2 |
| H-16 | La condición única para quitar a un integrante es no figurar en ningún gasto (como pagador o participante) ni pago; con eso su saldo es 0. | RF-3.4, RF-3.5 |
| H-18 | Se quita la duda: resuelta por H-5. | Sección 12 |
| H-19, H-26 | El pago lleva fecha con las mismas reglas que el gasto; quien paga y quien recibe deben ser integrantes del mismo grupo. | RF-9.1, RF-9.2 |
| H-20, H-35 | Los criterios cubren RNF y citan la verificación de la constitución; RF-1.1 se reescribe como comportamiento verificable. | RF-1.1, sección 10 |
| H-21 | Se acepta; no cambia ningún saldo. | CL-15 |
| H-22 | Se acepta: 0,01 para el primero y 0,00 para los otros 49. | CL-16 |
| H-23 | Se aceptan: 100 % a uno; «33,33», «33.33» y «33,34» valen igual. | CL-4, CL-17 |
| H-24 | Un gasto en un grupo sin integrantes se rechaza porque quien pagó no existe; saldos y plan dicen que el grupo no tiene integrantes. | RF-10.4, CL-18 |
| H-25 | Los saldos no tienen tope; se muestran completos. | CL-19 |
| H-27 | Tras eliminar el gasto, el integrante se puede quitar. | CL-20 |
| H-28 | Se permite: los nombres son únicos solo dentro de un grupo. | RF-3.2, CL-21 |
| H-29 | Una sola ejecución a la vez es un supuesto; si los datos están ocupados por otra ejecución, la operación se rechaza sin cambiar nada. | RF-12.3, sección 11 |
| H-30, H-33 | Nuevo RF-12: datos ilegibles o de versión desconocida, y fallos al guardar, se informan sin modificar nada. | RF-12 |
| H-31 | Se acepta una transferencia de 0,01 Bs. | CL-22 |
| H-34 | La tabla de mensajes se completa con un texto por cada rechazo. | Sección 5 |

Todos los hallazgos quedan resueltos; la versión 2 no tiene dudas abiertas.
