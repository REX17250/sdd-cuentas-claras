# Propuesta de constitución — decisiones y verificación

Fase 1. Rige la regla «usa tus recomendaciones»: se elige la opción recomendada y se registra el motivo.

## Decisiones (D-1…D-5)

### D-1. ¿Qué tipo de aplicación es el MVP?
- (a) **(Recomendada)** Una aplicación de línea de comandos local, de una sola persona que lleva las cuentas del grupo.
- (b) Una aplicación web con cuentas para cada integrante.
- (c) Una API para una futura app móvil.
- **Decisión:** (a).
- **Motivo:** el valor del producto está en las reglas de dinero (repartos, redondeos, liquidación), no en la interfaz. Una CLI local deja todo el esfuerzo en esas reglas y evita autenticación, despliegue y datos personales en un servidor.

### D-2. ¿Cómo se representan los importes?
- (a) **(Recomendada)** Enteros de centavos en todas las capas; solo la interfaz convierte a texto con dos decimales.
- (b) `Decimal` con dos decimales.
- (c) Números de coma flotante redondeados al mostrar.
- **Decisión:** (a).
- **Motivo:** los enteros no tienen errores de representación, son triviales de sumar y comparar, y obligan a que cada redondeo sea explícito. Con coma flotante, 0,1 + 0,2 no es 0,3; con `Decimal`, el redondeo se puede esconder en el contexto.

### D-3. ¿Qué persistencia?
- (a) **(Recomendada)** Una base SQLite local con transacciones y migraciones.
- (b) Un archivo JSON reescrito completo en cada cambio.
- (c) Sin persistencia (todo en memoria).
- **Decisión:** (a).
- **Motivo:** un gasto toca varias filas (el gasto y la parte de cada participante) y debe guardarse completo o no guardarse; SQLite viene en la biblioteca estándar y da transacciones sin código extra. Un JSON obligaría a reimplementar la escritura atómica y escala peor con miles de gastos.

### D-4. ¿Qué nivel de tests?
- (a) **(Recomendada)** Un test por RF y por CL como mínimo, con el identificador en el nombre o en la docstring, y la suite completa antes de cada commit.
- (b) Tests solo del dominio.
- (c) Tests al final del proyecto.
- **Decisión:** (a).
- **Motivo:** un error de un centavo en un reparto es exactamente el tipo de fallo que solo atrapa un test; citar el identificador hace la trazabilidad comprobable.

### D-5. ¿Idioma del código y de la interfaz?
- (a) **(Recomendada)** Código en inglés; interfaz y documentación en español.
- (b) Todo en español.
- (c) Todo en inglés.
- **Decisión:** (a).
- **Motivo:** el usuario final habla español; el código en inglés sigue las convenciones de Python y de sus bibliotecas.

## Cómo se verifica cada principio

| Principio | Verificación |
|---|---|
| 1. Stack mínimo | Un test analiza los `import` de todo el paquete y falla si aparece un módulo que no es de la biblioteca estándar ni del propio proyecto. |
| 2. Spec antes que código | Revisión de cada PR: todo comportamiento nuevo cita un RF; un test de trazabilidad comprueba que cada RF de la spec aparece en algún test. |
| 3. Dominio sin E/S | Un test analiza los `import` de la capa de dominio y falla si importa `sqlite3`, la capa de interfaz, la de persistencia, `datetime.date.today` o `time`. |
| 4. Tests obligatorios | El test de trazabilidad (principio 2) y `pytest -q` en verde en cada commit. |
| 5. Dinero exacto | Un test busca `float(` y literales con decimales en el dominio; tests de propiedad comprueban que cada reparto suma el total con miles de importes. |
| 6. Persistencia controlada | Un test comprueba que solo la capa de persistencia importa `sqlite3`; tests que simulan un error a mitad de una operación y verifican que no queda nada a medias. |
| 7. Idioma | Revisión de los mensajes centralizados; un test comprueba que los textos de la interfaz solo salen del módulo de mensajes. |
