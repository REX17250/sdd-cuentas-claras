# Aclaraciones — spec 001

Preguntas de la fase 2 para eliminar ambigüedades antes de redactar la [spec](spec.md). Rige la regla «usa tus recomendaciones»: se elige la opción recomendada y se registra el motivo. La numeración continúa la de la [constitución](../../docs/constitution-proposal.md) (D-1…D-5).

## Preguntas de aclaración (D-6…D-20)

### D-6. ¿Qué entra en el MVP?
- (a) **(Recomendada)** Grupos, integrantes, gastos con tres formas de reparto, pagos entre integrantes, saldos y un plan de liquidación (quién le paga a quién).
- (b) Lo de (a) sin el plan de liquidación: solo saldos.
- (c) Lo de (a) más varias monedas y gastos recurrentes.
- **Decisión:** (a).
- **Motivo:** el plan de liquidación es la respuesta que el usuario busca («¿cuánto le transfiero a quién?»); sin él, la app es una calculadora de saldos. Varias monedas y recurrencias duplican las reglas sin cambiar el problema central.

### D-7. ¿Quién usa la aplicación?
- (a) **(Recomendada)** Una sola persona por instalación, que lleva las cuentas de uno o varios grupos; los integrantes son nombres, no cuentas.
- (b) Cada integrante con su cuenta y contraseña.
- **Decisión:** (a).
- **Motivo:** coherente con D-1 (CLI local). Sin cuentas no hay autenticación ni datos personales más allá de un nombre.

### D-8. ¿Qué moneda y qué límites de importe?
- (a) **(Recomendada)** Una sola moneda, bolivianos (Bs), con dos decimales; cada importe entre 0,01 y 1 000 000,00 Bs.
- (b) Moneda configurable por grupo.
- (c) Varias monedas con tipo de cambio.
- **Decisión:** (a).
- **Motivo:** una moneda elimina conversiones y su redondeo; el máximo evita errores de tipeo (un cero de más) y deja margen para sumas grandes sin desbordes.

### D-9. ¿Qué formas de repartir un gasto?
- (a) **(Recomendada)** Tres: **en partes iguales**, **por montos exactos** y **por porcentajes**.
- (b) Solo en partes iguales.
- (c) Las tres más **por partes** (por ejemplo, 2 partes para quien comió doble).
- **Decisión:** (a).
- **Motivo:** cubren los casos reales más comunes (una cena, una compra con artículos de cada uno, un alquiler por tamaño de habitación). «Por partes» se puede expresar con porcentajes y queda para una versión futura.

### D-10. ¿Cómo se redondean los centavos que sobran?
- (a) **(Recomendada)** En partes iguales, cada participante recibe la parte entera en centavos y los centavos sobrantes se asignan de a uno, en el orden de alta de los participantes en el grupo. En porcentajes, cada parte se trunca a centavos y los sobrantes van a quienes tienen el mayor resto decimal (empates por orden de alta). En montos exactos no hay redondeo: deben sumar el total.
- (b) Los centavos sobrantes van siempre a quien pagó.
- (c) Se redondea cada parte al centavo más cercano y la diferencia se ignora.
- **Decisión:** (a).
- **Motivo:** la regla es determinista (el mismo gasto siempre da el mismo reparto), reparte la diferencia como máximo un centavo por persona y nunca pierde ni crea dinero, como exige el principio 5. La opción (c) hace que el reparto no sume el total.

### D-11. ¿Cuántas personas pagan un gasto?
- (a) **(Recomendada)** Exactamente una.
- (b) Varias, cada una con su monto.
- **Decisión:** (a).
- **Motivo:** cubre casi todos los casos; un gasto pagado entre dos se registra como dos gastos.

### D-12. ¿Quién tiene que participar en un gasto?
- (a) **(Recomendada)** Un subconjunto no vacío de los integrantes del grupo; quien paga puede no participar (pagó algo para otros).
- (b) Siempre todos los integrantes.
- **Decisión:** (a).
- **Motivo:** en un viaje no todos van a todas las actividades; obligar a todos rompe el uso real.

### D-13. ¿Cómo se manejan los integrantes?
- (a) **(Recomendada)** Se agregan por nombre, único dentro del grupo (sin distinguir mayúsculas ni tildes). Un integrante solo se puede quitar si su saldo es cero y no figura en ningún gasto ni pago. No se renombran en el MVP.
- (b) Se pueden quitar siempre, y sus deudas se reparten entre los demás.
- **Decisión:** (a).
- **Motivo:** quitar a alguien con historial haría que gastos pasados apunten a una persona inexistente; renombrar se puede añadir después sin cambiar las reglas.

### D-14. ¿Cómo se registran los pagos entre integrantes?
- (a) **(Recomendada)** Un pago va de un integrante a otro distinto, por cualquier importe válido; ajusta los saldos aunque supere lo que debía (el saldo cambia de signo) y la app avisa en ese caso.
- (b) Un pago no puede superar lo que el pagador debe.
- **Decisión:** (a).
- **Motivo:** en la vida real alguien puede adelantar dinero; prohibirlo obligaría a inventar gastos ficticios. El aviso evita errores de tipeo sin bloquear.

### D-15. ¿Qué es el plan de liquidación?
- (a) **(Recomendada)** Una lista de transferencias «A le paga X a B» que deja todos los saldos en cero, con como mucho una transferencia menos que el número de integrantes con saldo distinto de cero; calculada con una regla determinista (el mayor deudor paga al mayor acreedor, empates por orden de alta).
- (b) Las deudas directas de cada gasto, sin simplificar.
- (c) El número mínimo de transferencias posible.
- **Decisión:** (a).
- **Motivo:** reduce mucho las transferencias frente a (b) y es fácil de explicar y probar. Encontrar siempre el mínimo absoluto (c) es un problema combinatorio caro que no aporta a grupos pequeños.

### D-16. ¿Qué fecha tiene un gasto?
- (a) **(Recomendada)** La fecha que indique la persona o, si no indica ninguna, la de hoy; no se aceptan fechas futuras.
- (b) Siempre la de hoy.
- **Decisión:** (a).
- **Motivo:** los gastos de un viaje se suelen anotar después; una fecha futura casi siempre es un error de tipeo.

### D-17. ¿Se pueden corregir gastos y pagos?
- (a) **(Recomendada)** Se pueden **eliminar** (y volver a registrar bien); no se editan en el MVP.
- (b) Se pueden editar todos sus campos.
- (c) No se pueden corregir.
- **Decisión:** (a).
- **Motivo:** eliminar y volver a crear cubre la corrección con una sola regla; editar exige revalidar repartos y volver a calcular saldos sobre un registro existente.

### D-18. ¿Cuántos grupos y qué se hace con ellos?
- (a) **(Recomendada)** Varios grupos con nombre único; se crean y se listan. Borrar o archivar grupos queda fuera del MVP.
- (b) Un solo grupo por instalación.
- **Decisión:** (a).
- **Motivo:** una persona suele llevar el piso y un viaje a la vez; borrar grupos con historial es una acción delicada que puede esperar.

### D-19. ¿Qué límites de tamaño?
- (a) **(Recomendada)** Hasta 50 integrantes por grupo, 10 000 gastos por grupo; descripción de 1 a 80 caracteres; nombres de 1 a 40.
- (b) Sin límites.
- **Decisión:** (a).
- **Motivo:** límites claros hacen verificables los requisitos de rendimiento y de validación; superan con margen cualquier grupo real.

### D-20. ¿Qué pasa con los saldos de un integrante que no participa en nada?
- (a) **(Recomendada)** Su saldo es 0 y aparece en la lista de saldos; no aparece en el plan de liquidación.
- (b) No aparece en ninguna lista.
- **Decisión:** (a).
- **Motivo:** ver «0,00 Bs» confirma que esa persona está al día; el plan solo debe listar transferencias necesarias.

## Dudas que surgieron al redactar la spec

Se registran en la sección 12 de la [spec](spec.md) con la marca `[NECESITA ACLARACIÓN]`.
