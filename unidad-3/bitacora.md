# Unidad 3

## Bitácora de proceso de aprendizaje

### Actividad 03
> Toma nota durante el análisis de la implementación del temporizador en p5.js. Registra tus observaciones, dudas, y cualquier idea que te surja al revisar el código.

***Java script***
- En java script no se utiliza el self, no utiliza self en el cuerpo, utiliza self
- En la maquina de estados empezan a aparecer los metodos
- Las estructuras de datos se manipulas un poco pero no cambian muchas cosas
  
---

- Mientras la lista no este vacia vamos a sacar eventos y los vamos a enviar al estado - mientras en la cola hayan datos se van a sacar datos y luego se los envio al estado
- La forma en que se crean los metodos cambian
  
***Funciones anonimas -> c chart java script***
- Lee cada uno de los eventos
- Se identifican por `=>`
- Izquierda -> Argumentos de la funcion
- Derecha -> Las llaves que envuelven el cuerpo de la funcion de lo que debe hacer
  
---

- Set up crea canvas
- Renderer set que quiero dibujar en el codigo -> poner en el contexto esto
- Renderer cual de las funciones vamos a pintar en el frame
- De ahi para abajo en el codigo son las diferentes ideas de animaciones/dibujos
- Si coloco 0 en fill se pone negro
- El draw se llama automaticamente en cada frame
- Los eventos tienen funciones especiales para llamarlos -> ej: A B S
  
---

- Hay que detenerlo y desarmarlo
- Entregar enunciado y programa para
- En el index va la biblioteca serial 
- en vez de usar is -> was

***Codigos***

``fsm.js``
```js
const ENTRY = "ENTRY";
const EXIT = "EXIT";

class Timer {
  constructor(owner, eventToPost, duration) {
    this.owner = owner;
    this.event = eventToPost;
    this.duration = duration;
    this.startTime = 0;
    this.active = false;
  }

  start(newDuration = null) {
    if (newDuration !== null) this.duration = newDuration;
    this.startTime = millis();
    this.active = true;
  }

  stop() {
    this.active = false;
  }

  update() {
    if (this.active && millis() - this.startTime >= this.duration) {
      this.active = false;
      this.owner.postEvent(this.event);
    }
  }
}

class FSMTask {
  constructor() {
    this.queue = [];
    this.timers = [];
    this.state = null;
  }

  postEvent(ev) {
    this.queue.push(ev);
  }

  addTimer(event, duration) {
    let t = new Timer(this, event, duration);
    this.timers.push(t);
    return t;
  }

  transitionTo(newState) {
    if (this.state) this.state(EXIT);
    this.state = newState;
    this.state(ENTRY);
  }

  update() {
    for (let t of this.timers) {
      t.update();
    }
    while (this.queue.length > 0) {
      let ev = this.queue.shift();
      if (this.state) this.state(ev);
    }
  }
}
```

``index.html``
```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Sketch</title>

    <link rel="stylesheet" type="text/css" href="style.css">

    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.js"></script>
  </head>

  <body>
    <script src="fsm.js"></script>
    <script src="sketch.js"></script>
  </body>
</html>
```

``sketch.js``
```js
const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};


class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);

  }

  get currentState() {
    return this.state;
  }

  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
    }
    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };


  estado_armed = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.start();
    } else if (ev === EVENTS.TICK) {
      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;
        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }
    } else if (ev === EXIT) {
      this.myTimer.stop();
    }

  };

  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  }
}

let temporizador;
const renderer = new Map();

function setup() {
  createCanvas(windowWidth, windowHeight);
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());
}

function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();
}

function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```

### Actividad 04
> Registra el código que implementaste para estas modificaciones tanto para la parte de p5.js como para la parte del micro:bit.

***Intrucciones***
>Vas a modificar el código anterior adicionando:
>Si está corriendo el temporizador, al presionar la tecla A se debe pausar el conteo. Si se vuelve a presionar la tecla A, el conteo se reanuda en donde se había pausado.
Si el temporizador está corriendo, si se presiona una secuencia de teclas A-B-A, el temporizador debo volver a modo de configuración.
Vas a controlar el temporizador que está funcionando en p5.js usando el micro:bit, además de las teclas del computador. Recuerda que solo deberás usar del micro:bit los botones A y B y el acelerómetro.

#### Abrir la funcionalidad para conectar el micro:bit al temporizador en p5.js

***Modifico sketch.js***

>  Agregar la librería p5.webserial antes de sketch.js -> Objeto serial, botón para conectar, lectura de datos, conversión a eventos FSM.

``sketch.js``
```js
const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};


class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);

  }

  get currentState() {
    return this.state;
  }

  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
    }
    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };


  estado_armed = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.start();
    } else if (ev === EVENTS.TICK) {
      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;
        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();
        }
      }
    } else if (ev === EXIT) {
      this.myTimer.stop();
    }

  };

  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  }
}

let temporizador;
const renderer = new Map();

// -------------------------
// SERIAL
// -------------------------
let serial;
let connectButton;

function setupSerial() {
  serial = new p5.WebSerial();

  serial.on("connected", () => console.log("Conectado al puerto"));
  serial.on("data", onSerialData);
  serial.on("error", (err) => console.error(err));
}

function createConnectButton() {
  connectButton = createButton("Conectar micro:bit");
  connectButton.position(20, 20);
  connectButton.mousePressed(connectSerial);
}

async function connectSerial() {
  if (!serial.isOpen()) {
    await serial.requestPort();
    await serial.open({ baudRate: 115200 });
    console.log("Puerto abierto");
  }
}

function onSerialData() {
  let data = serial.readStringUntil("\n");
  if (!data) return;

  data = data.trim();
  console.log("Recibido:", data);

  if (data === "A") temporizador.postEvent(EVENTS.DEC);
  if (data === "B") temporizador.postEvent(EVENTS.INC);
  if (data === "S") temporizador.postEvent(EVENTS.START);
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  setupSerial();           //NUEVO
  createConnectButton();   //NUEVO
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());
}

function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();
}

function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```

***Codigo Modificado***

``sketch.js``
```js
const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
  RESET: "RESET"
};

const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};


class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;
    this.sequence = []; // NUEVO

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);

  }

  get currentState() {
    return this.state;
  }

  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
    }
    else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };

 estado_armed = (ev) => {

  if (ev === ENTRY) {
    this.myTimer.start();
  }

  else if (ev === EVENTS.TICK) {
    if (this.remainingSeconds > 0) {
      this.remainingSeconds--;
      if (this.remainingSeconds === 0) {
        this.transitionTo(this.estado_timeout);
      } else {
        this.myTimer.start();
      }
    }
  }

  else if (ev === EVENTS.DEC) {  // A = pausa
    this.myTimer.stop();
    this.transitionTo(this.estado_paused);
  }

  else if (ev === EVENTS.INC) {  // B = parte de secuencia
    this.checkSequence(EVENTS.INC);
  }

  else if (ev === EXIT) {
    this.myTimer.stop();
  }

  this.checkSequence(ev);
};

  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  }
}

estado_paused = (ev) => {

  if (ev === EVENTS.DEC) {  // A reanuda
    this.transitionTo(this.estado_armed);
  }

  else if (ev === EVENTS.INC) {
    this.checkSequence(EVENTS.INC);
  }

  this.checkSequence(ev);
};

let temporizador;
const renderer = new Map();

// -------------------------
// SERIAL
// -------------------------
let serial;
let connectButton;

function setupSerial() {
  serial = new p5.WebSerial();

  serial.on("connected", () => console.log("Conectado al puerto"));
  serial.on("data", onSerialData);
  serial.on("error", (err) => console.error(err));
}

function createConnectButton() {
  connectButton = createButton("Conectar micro:bit");
  connectButton.position(20, 20);
  connectButton.mousePressed(connectSerial);
}

async function connectSerial() {
  if (!serial.isOpen()) {
    await serial.requestPort();
    await serial.open({ baudRate: 115200 });
    console.log("Puerto abierto");
  }
}

function onSerialData() {
  let data = serial.readStringUntil("\n");
  if (!data) return;

  data = data.trim();
  console.log("Recibido:", data);

  if (data === "A") temporizador.postEvent(EVENTS.DEC);
  if (data === "B") temporizador.postEvent(EVENTS.INC);
  if (data === "S") temporizador.postEvent(EVENTS.START);
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  setupSerial();           //NUEVO
  createConnectButton();   //NUEVO
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  renderer.set(temporizador.estado_config, () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed, () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());
}

function draw() {
  temporizador.update();
  renderer.get(temporizador.currentState)?.();
}

function drawConfig(val) {
  background(20, 40, 80);
  fill(255);
  textSize(120);
  text(val, width / 2, height / 2);
  textSize(18);
  fill(200);
  text("A(-) B(+) S(start)", width / 2, height / 2 + 100);
}

function drawArmed(val, total) {
  background(20, 20, 20);
  let pulse = sin(frameCount * 0.1) * 10;

  noFill();
  strokeWeight(20);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, 250);

  stroke(255, 150, 0);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 250, 250, -HALF_PI, angle - HALF_PI);

  fill(255);
  noStroke();
  textSize(100 + pulse);
  text(val, width / 2, height / 2);
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(100);
  text("¡TIEMPO!", width / 2, height / 2);
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```


## Bitácora de aplicación 

Nota del profesor: esta sección no tiene evidencia del trabajo de evaluación. No asistió a su sustentación




## Bitácora de reflexión




