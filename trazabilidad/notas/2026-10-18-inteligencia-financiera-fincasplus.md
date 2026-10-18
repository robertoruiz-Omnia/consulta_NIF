---
tipo: nota
fecha: 2026-10-18
titulo: Inteligencia financiera sobre FincasPlus (nota de concepto)
ambito: interno
estado: borrador
aplicaciones: [fincasplus-elite]   # ajustar slug a la tarjeta existente; añadir la de Afterbanks si tiene tarjeta
frentes: []                        # pendiente de asignar
fuentes: [reflexión interna, 18/10/2026]
---

# Inteligencia financiera sobre FincasPlus (nota de concepto)

Idea en fase de concepto, recogida el 18/10/2026 para comentarla con el equipo. No hay decisión tomada ni compromiso con IDS. Se documenta para fijar el planteamiento y las preguntas abiertas antes de decidir si entra en el plan.

## 1. Resumen

El software de administración de fincas está concentrando la IA en gestión documental, lectura de facturas, incidencias y redacción de actas. Son terrenos donde cualquier modelo generalista compite y donde los productos del sector acabarán pareciéndose. La tesorería de las comunidades no está en esa carrera y es, además, el terreno donde los datos que hacen falta (cuotas, remesas, devoluciones, extractos, histórico contable) sólo los tiene el software del despacho.

Se propone un servicio de Omnia para los usuarios de FincasPlus que sean clientes de Omnia, que a partir de la contabilidad íntegra de cada comunidad ofrezca cuatro piezas sobre una base común:

1. Previsión de tesorería a 30, 60 y 90 días, con simulador de derramas.
2. Conciliación bancaria asistida del cuaderno 43.
3. Detección de gastos anómalos.
4. Vista de cartera: estado de todas las comunidades del despacho, con lista de excepciones.

FincasPlus sigue siendo el sistema de registro (contabilidad, remesas, presupuesto, conciliación válida). Omnia es la capa que calcula, predice y explica. El servicio lo presta y lo cobra Omnia; el beneficio se ve dentro de FincasPlus y en Omnia.

## 2. Punto de partida

- FincasPlus ya recibe los movimientos bancarios mediante su conexión con Afterbanks, que IDS ofrece a sus clientes como suscripción aparte. Omnia no necesita conectarse al banco.
- FincasPlus aplica reglas de conciliación habituales del cuaderno 43 (importe, IBAN de contrapartida, referencia SEPA, texto del concepto).
- Omnia ya presta a IDS el servicio de identificación de facturas de proveedor. Ese flujo de vuelta hacia FincasPlus existe y es el punto de apoyo natural para todo lo que sigue.
- Las devoluciones de remesa ya quedan en FincasPlus atadas a comunero y a cargo, con su motivo SEPA. El histórico de morosidad por tipo de cargo ya está construido dentro del programa.
- La interconexión IDS - Omnia (sincronizador) está en especificación. Este caso de uso sirve de prueba de estrés para decidir qué entidades viajan y con qué profundidad temporal.

## 3. La pieza común: el apunte previsto

Las cuatro funciones beben de un mismo concepto, el apunte bancario previsto: un movimiento que todavía no ha llegado al banco pero del que se conoce importe (exacto o con banda), fecha prevista, contrapartida, forma de pago (y por tanto el concepto común del cuaderno 43 que se espera: 03 recibo domiciliado, 04 transferencia, 14 devolución) y origen.

Los apuntes previstos nacen de tres orígenes:

- **Facturas identificadas.** Cuando Omnia devuelve a FincasPlus una factura leída, devuelve también el apunte o apuntes previstos. Un presupuesto de obra con calendario de pagos (por ejemplo 30 / 60 / 10) genera varios apuntes, uno por plazo.
- **Series recurrentes aprendidas del histórico contable.** Suministros domiciliados cuya factura llega a la vez o después del cargo, seguros, impuestos, comisiones. Se predicen por su propia historia: periodicidad, estacionalidad (agua con piscina en verano, calefacción central de noviembre a marzo), deriva del importe por subidas de tarifa.
- **Lo emitido por la comunidad.** Cuotas y derramas con su calendario de remesa, descontando la devolución esperada según el histórico de la comunidad y el tipo de cargo (una derrama de obra obligatoria no tiene la misma morosidad que una cuota ordinaria).

Cuando llega el movimiento real y se casa con el previsto, la diferencia (fecha real frente a prevista, concepto real frente a esperado, importe) vuelve a Omnia. Así el modelo aprende cómo cobra realmente cada proveedor y cómo paga cada comunidad sin que nadie lo configure.

## 4. Las cuatro piezas

### 4.1 Previsión de tesorería y simulador de derramas

- Saldo proyectado a 30, 60 y 90 días por cuenta y por comunidad, con la banda de incertidumbre que aporten las series estimadas y las devoluciones esperadas.
- Indicadores: liquidez real frente a fondo de reserva exigible (10% del último presupuesto ordinario según la LPH desde el Real Decreto-ley 19/2021; mínimo del 5% de los gastos comunes presupuestados en Cataluña según el Codi Civil), meses de gasto ordinario que cubre el saldo. El fondo de reserva es una cifra contable que muchas comunidades tienen dotada "en papel" y no en banco; el indicador útil es la liquidez real.
- Alerta temprana: aviso con antelación de que una comunidad entrará en negativo o por debajo del fondo si no mueve la cuota o adelanta una derrama.
- Simulador de derramas. Entrada: importe del gasto, calendario de pagos del proveedor, colchón mínimo que se quiere mantener en banco. Salida: propuestas de número de plazos, importes y fechas de cobro a los comuneros tales que el saldo proyectado no baje del colchón, descontada la parte que previsiblemente no se cobrará a tiempo según el histórico de devoluciones de esa comunidad para ese tipo de cargo. Cada escenario con su explicación en lenguaje natural, utilizable ante la junta.
- La herramienta propone y justifica; nunca decide. La derrama la aprueba la junta.

### 4.2 Conciliación asistida del cuaderno 43

- Con la contabilidad íntegra, las reglas del 43 se aplican mejor en Omnia que en el programa, porque además de las reglas se dispone de los apuntes previstos y del histórico de cada proveedor. Se pasa de buscar a qué factura corresponde un movimiento a comprobar si ha llegado el movimiento que se esperaba.
- Omnia devuelve a FincasPlus cada movimiento con la conciliación propuesta y una confianza. Por encima de un umbral FincasPlus la aplica; por debajo la deja como propuesta. El administrador puede deshacerla siempre.
- Cada propuesta lleva su motivo ("coincide con la factura 2026/114 de Fontanería López por proveedor e importe") y se rechaza en un clic. Los rechazos vuelven a Omnia y son el mejor dato de mejora.
- El mayor valor está en el residuo que las reglas no resuelven: conceptos que no se parecen a la factura, pagos parciales, dos facturas en un movimiento, transferencias manuales de comuneros con textos libres.
- Los despachos sin Omnia siguen conciliando como hasta ahora.

### 4.3 Detección de gastos anómalos

Por capas, de lo más simple a lo más valioso:

- Factura duplicada (13 facturas de agua en un año). Encaja con lo que ya hace la identificación de facturas.
- Factura que falta (9 en vez de 12): posible impago, corte de suministro o factura no pasada al programa.
- Importe fuera de la banda histórica, corregida por estacionalidad. La referencia es la realidad de la propia comunidad, no un parámetro.
- Cambio de precio de un proveedor.
- Comparación entre comunidades parecidas de la cartera: lo que se paga por mantenimiento de ascensor, limpieza o seguro. Da al despacho un argumento ante la junta y una palanca ante el proveedor.
- Posible extensión futura: comparación agregada y anonimizada entre despachos (benchmark sectorial). Requiere una conversación aparte sobre propiedad de los datos y con Legal.

### 4.4 Vista de cartera

- El administrador tiene herramientas para mirar una comunidad y ninguna para mirar las 150 a la vez.
- Lo que se usa no es un panel por comunidad sino la lista de excepciones de la cartera: comunidades que entran en negativo en 60 días, comunidades por debajo del fondo, derramas con cobro por debajo de lo previsto, cargos anómalos. El panel de comunidad es donde se profundiza cuando una excepción llama la atención.
- Agregaciones por bloque, por zona o por cliente/propietario si el despacho trabaja así.

## 5. Reparto entre FincasPlus y Omnia

| Pieza | FincasPlus (IDS) | Omnia |
|---|---|---|
| Contabilidad, remesas, presupuesto | Sistema de registro | Lectura con histórico vía sincronizador |
| Movimientos bancarios | Los recibe de Afterbanks | Los recibe de FincasPlus, en bruto y conciliados |
| Identificación de facturas | Recibe la factura y los apuntes previstos | Lee la factura y genera los apuntes previstos |
| Conciliación | Aplica o deja en propuesta; verdad contable | Propone con confianza y motivo |
| Previsión, simulador, anomalías, cartera | Puede mostrar alertas | Calcula, explica y presenta |
| Aprendizaje | Devuelve el movimiento real casado | Ajusta modelos por proveedor y comunidad |

## 6. Flujos que necesita el sincronizador

Tres flujos de IDS hacia Omnia, que corresponden a cosas que ya existen en FincasPlus:

1. **Bruto bancario.** Movimientos tal como llegan de Afterbanks. Frecuente, hacia adelante. Da frescura y saldo real; alimenta la alerta temprana y la clasificación provisional.
2. **Conciliado y contabilidad.** Movimientos casados con su apunte, facturas con vencimientos, remesas con devoluciones (fecha, importe, comunero, motivo SEPA), deuda por comunero, presupuesto por partida con periodicidad. Con histórico de dos o tres ejercicios como mínimo; puede ir más despacio. Alimenta los modelos.
3. **Lo previsto por la comunidad.** Cuotas y derramas emitidas, calendario de remesa, condiciones pactadas con proveedores de obra.

Y dos de Omnia hacia IDS:

4. **Apuntes previstos**, como ampliación del bloque de respuesta de la identificación de facturas.
5. **Conciliación propuesta**, movimiento a movimiento, con confianza y motivo; y recepción del rechazo o corrección del administrador.

La frescura del conjunto la marca la cadena Afterbanks, FincasPlus, sincronizador, Omnia. Para la previsión a 30/60/90 la latencia de uno o dos días es irrelevante; para la alerta temprana conviene conocer la frecuencia real de refresco de cada tramo y valorar un sincronizador por eventos cuando entra un lote de movimientos.

## 7. Decisiones de diseño propuestas

- Separar lo que el administrador sabe y la IA no puede adivinar (condiciones pactadas con un proveedor de obra, aplazamientos negociados), que se parametriza, de cómo cobra realmente cada proveedor, que se aprende del histórico y sólo se toca si se quiere. Si todo hay que configurarlo, el servicio no se usa.
- El modelo financiero (flujo de caja) se hace con reglas explicables. La IA entra en lo que las reglas no saben: tasa de devolución por comunero y tipo de cargo, estacionalidad por comunidad, clasificación de movimientos, lectura de calendarios de pago en presupuestos, y la capa de conversación y explicación.
- Omnia no monta una conciliación propia que compita con la del programa: toda clasificación de Omnia es provisional hasta que FincasPlus la confirma, y se sustituye por la conciliada en cuanto llega. Una sola verdad contable.
- La morosidad por comunero se usa agregada para la tesorería. Mostrar probabilidades de impago por vecino roza el perfilado y requiere criterio de Legal antes de enseñarlo.

## 8. Encaje con IDS

- No se les quita nada de su programa. Se les piden datos que ya tienen y se les devuelve trabajo hecho (apuntes previstos, conciliación propuesta) dentro de su propia pantalla.
- La pieza más sensible es la conciliación: pedir que la haga Omnia convierte su pantalla de conciliación en una pantalla de confirmación. Puede recibirse como alivio de mantenimiento o como entrada en su cocina. El planteamiento por estados y umbral deja la contabilidad en sus manos.
- El servicio tiene un requisito previo que es de IDS, no de Omnia: la suscripción a Afterbanks. Conviene explicitarlo como requisito del servicio, igual que una versión mínima del programa, y valorar si se empaqueta. A IDS le interesa porque empuja su propia suscripción.
- Se puede presentar como mejora de su integración de facturas, no como una integración nueva.

## 9. Posible orden

1. Asegurar los datos: sincronizador con histórico suficiente de contabilidad, remesas y devoluciones.
2. Apunte previsto en la respuesta de identificación de facturas, y previsión de tesorería básica (saldo proyectado, fondo, meses de cobertura).
3. Conciliación asistida y simulador de derramas, que comparten el apunte previsto.
4. Anomalías y vista de cartera, que son análisis sobre lo mismo.

## 10. Preguntas abiertas para el equipo

- ¿Entra en el plan y con qué prioridad frente a lo ya comprometido con IDS?
- ¿Qué profundidad temporal de histórico puede dar FincasPlus a través del sincronizador? ¿Se contempla ya el histórico de remesas y devoluciones?
- ¿El formato de respuesta de la identificación de facturas está cerrado con IDS o se puede ampliar sin romperles nada?
- ¿Con qué frecuencia refresca FincasPlus desde Afterbanks y cabe un sincronizador por eventos?
- ¿Cómo se plantea a IDS la conciliación asistida: como propuesta dentro de su pantalla o como conciliación que Omnia devuelve hecha?
- ¿Se empaqueta el requisito de Afterbanks o se exige por separado?
- ¿Qué parte de la morosidad por comunero se puede mostrar y a quién? (Legal)
- ¿Interesa a medio plazo la comparación agregada entre despachos?

## Fuentes

- Reflexión interna del 18/10/2026.
- Inventario de aplicaciones de IDS (14/10/2026), bloques de escritorio y servicios en la nube.
- Página de la wiki de IDS sobre integraciones con Omnia (06/10/2026).
