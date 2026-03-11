# Unidad 4

## Bitácora de proceso de aprendizaje

### Actividad 01
> Registra tus observaciones a medida que vamos analizando juntos el caso de estudio.

#### Observación sobre el hardware

- El micro:bit está configurado para enviar datos por UART a 115200 baudios.

- El formato de los datos enviados es una cadena con cuatro valores separados por comas:

`xValue` → acelerómetro en eje X

`yValue` → acelerómetro en eje Y

`aState` → estado del botón A (0 o 1)

`bState` → estado del botón B (0 o 1)

- La frecuencia de envío es de 10 Hz (cada 100 ms).

***Observación sobre la infraestructura de software***

- El reto principal es leer los datos del puerto serial en la computadora y traducirlos a parámetros visuales dentro del sketch.

- El repositorio de integración provee la infraestructura para conectar el flujo de datos del micro:bit con p5.js (usando Node.js o librerías como serialport).

***Observación sobre herramientas disponibles***

- Contamos con SerialTerminal para inspeccionar los datos que envía el micro:bit y verificar que llegan correctamente, esto nos permite confirmar que el protocolo de comunicación funciona antes de integrarlo con el sketch.

***Retos identificados***

**Lectura de datos seriales:** asegurar que el software reciba y procese las cadenas enviadas por el micro:bit.

**Parseo de datos:** convertir la cadena "x,y,a,b" en valores numéricos y booleanos utilizables en p5.js.

**Mapeo creativo:** decidir cómo los valores del acelerómetro y botones afectan la pieza de arte generativo (posición, color, velocidad, tamaño).

**Sincronización:** mantener la fluidez visual a pesar de la frecuencia de actualización de 10 Hz.

## Bitácora de aplicación 

### Actividad 02
> Te piden que construyas un sistema físico interactivo que integre el hardware del caso de estudio con una pieza de arte generativo, pero esta vez el hardware tiene un firmware diferente que no puedes cambiar, además debes usar la arquitectura de software que ya existe.
> Registra en tu bitácora la documentación necesaria para reconstruir tu solución.



## Bitácora de reflexión


