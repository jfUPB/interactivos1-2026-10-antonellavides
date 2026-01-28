# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01

> Luego de ver juntos los videos y conversar sobre ellos, define en tus propias palabras las siguientes preguntas.

#### *¿Qué es un sistema físico interactivo?*

Son sistemas que utilizan elementos fisicos y software para interactuar con las personas o algún espacio.

#### *¿Cómo podrías aplicar lo que has visto en tu perfil profesional?*

Los sistemas interactivos abren las posibilidades para poder expresar el de manera más creativa y descubrir multiples resultados a traves de ellos. Esto me haría obtener más conocimiento y tener un perfil profesional más completo.

### Actividad 02

> Luego de este ejercicio, define en tus propias palabras las siguientes preguntas-

#### *¿Qué es el diseño/arte generativo?*

Se refiere a cualquier practica artistica en la que el artista utiliza un sistema basado en reglas que no diseña la salida sino el producir multiples salidas.

#### *¿Cómo podrías aplicar lo que has visto en tu perfil profesional?*

Al automatizar y optimizar algunos procesos se pueden llegar a lograr resultados muy buenos y llevar a otro nivel todo lo que haga.

### Actividad 03

#### *Sacude el micro:bit. ¿Qué pasa?*

El circulo en p5js cambia a color verde y muestra la letra C

#### *Presiona el botón Send Love. ¿Qué pasa?*

El microbit muestra una carita feliz, un corazón y un mensaje de love

### Actividad 4

> Explica detalladamente.

#### *¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()?*



## Bitácora de aplicación 

### Actividad 5

> Crea un programa en p5.js que muestre un círculo en la pantalla. Utiliza los botones A y B del micro:bit para controlar la posición en x del círculo en el canvas de p5.js. Explica detalladamente cómo funciona el sistema físico interactivo que has creado.

#### Programa p5.js
[Control de movimiento con micro:bit: p5.js](https://editor.p5js.org/antonellavides/full/TKwpeHvDO)
```js
let port;
let connectBtn;
let x = 200;

function setup() {
  createCanvas(400, 400);
  background(220);
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);
}

function draw() {
  background(220);

  if (port.availableBytes() > 0) {
    let data = port.read(1);
    if (data === "A") {
      x -= 10;
    } else if (data === "B") {
      x += 10;
    }
  }

  x = constrain(x, 25, width - 25);
  fill("pink");
  ellipse(x, height / 2, 50, 50);

  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
  } else {
    connectBtn.html("Disconnect");
  }
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
  } else {
    port.close();
  }
}
```
#### Programa microbit
[Control de movimiento con micro:bit: microbit](https://editor.p5js.org/antonellavides/full/TKwpeHvDO)
```python
from microbit import *

uart.init(baudrate=115200)
display.show(Image.BUTTERFLY)

while True:
    if button_a.was_pressed():
        uart.write('A')
    if button_b.was_pressed():
        uart.write('B')
```

#### *Funcionamiento* :shipit:

- Creo la variable del puerto (port) para comunicarme con el micro:bit y también una variable x para guardar la posición horizontal del círculo.
- createCanvas dibuja el espacio donde va el círculo.
- createSerial() crea el objeto para la conexión serial.
- Hago un botón que me deja conectar y desconectar el micro:bit.
- draw() se repite muchas veces por segundo para que la pagina se vaya actualizando para redibujar los fotogramas del circulo.
- Si es A muevo el círculo a la izquierda y si es B lo muevo a la derecha.
- 
 #### *Extra* :octocat:
 
**uart:** es el módulo que permite al micro:bit comunicarse por puerto serial USB.

**.init():** inicializa la comunicación.

**baudrate=115200:** define la velocidad a la que se envían los datos en bits por segundo.

## Bitácora de reflexión








