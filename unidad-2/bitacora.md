# Unidad 2

## Bitácora de proceso de aprendizaje

 ### Actividad 01

> Analizando un programa con una máquina de estados simple
 
```python
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)

class Pixel:
    def __init__(self,_x,_y,_interval):
        self.event_queue = []
        self.timers = []
        self.myTimer = self.createTimer("Timeout",_interval)
        
        self.posx = _x
        self.posy= _y
        self.interval= _interval

        self.estado_actual = None
        self.transicion_a(self.estado_waitInON)

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

# COMIENZO COMO TAL LA APLICACIÓN: estructura completa maquina de estados

    def estado_waitInON(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.posx, self.posy,9)
            self.myTimer.start()
        if ev == "Timeout":
            self.transicion_a(self.estado_waitInOFF)

    def estado_waitInOFF(self,ev):
        if ev == "ENTRY":
            display.set_pixel(self.posx, self.posy,0)
            self.myTimer.start()
        if ev == "Timeout":
           self.transicion_a(self.estado_waitInON)
         

    
pixel1 = Pixel(0,0,400) 
pixel1 = Pixel(1,0,356) 


while True:
    pixel1.update()
    utime.sleep_ms(20)
```

> Responde lo siguiente

***¿Cuáles son los estados en el programa?***

`estado_waitInOn`

`estado_waitInOff`

***¿Cuáles son los eventos en el programa?***

`Entry`

`Exit`

`Timeout`

***¿Cuáles son las acciones en el programa?***

`display.set_pixel(self.posx, self.posy, 9)`

`self.myTimer.start()`

`display.set_pixel(self.posx, self.posy, 0)`

`self.myTimer.start()`

### Actividad 02

> Implementando un semáforo con máquinas de estados

> Vas a realizar una modificación. Cuando el semáforo esté en verde, si se presiona el botón A, el semáforo debe cambiar inmediatamente a amarillo (sin esperar a que termine el tiempo de verde). El evento que se debe postear es “A” (post_event(“A”)).

`MAIN.PY`

```python
from microbit import*
import utime
from fsm import Timer, FSMTask, ENTRY

class Semaforo(FSMTask):
    def __init__(self, _x, _y, _timeInRed, _timeInGreen, _timeInYellow):
        super().__init__()
        self.x = _x
        self.y = _y
        self.timeInRed = _timeInRed
        self.timeInGreen = _timeInGreen
        self.timeInYellow = _timeInYellow
        self.myTimer=self.add_timer("Timeout", self.timeInRed)
        self.transition_to(self.estado_waitInRed)

    def clear(self):
        display.set_pixel(self.x,self.y,0)
        display.set_pixel(self.x,self.y+1,0)
        display.set_pixel(self.x,self.y+2,0)
    
    def estado_waitInRed(self, ev):
         if ev == ENTRY:
            self.clear()
            display.set_pixel(self.x,self.y,9)
            self.myTimer.start(self.timeInRed)
         if ev == "Timeout":
            display.set_pixel(self.x,self.y,0)
            self.transition_to(self.estado_waitInGreen) 

    def estado_waitInGreen(self, ev):
         if ev == ENTRY:
            self.clear()
            display.set_pixel(self.x,self.y+2,9)
            self.myTimer.start(self.timeInGreen)
         if ev == "Timeout":
            display.set_pixel(self.x,self.y+2,0)
            self.transition_to(self.estado_waitInYellow)
         if ev == "A":
            display.set_pixel(self.x,self.y+2,0)
            self.transition_to(self.estado_waitInYellow)

    def estado_waitInYellow(self, ev):
        if ev == ENTRY:
            self.clear()
            display.set_pixel(self.x,self.y+1,9)
            self.myTimer.start(self.timeInYellow)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y+1,0)
            self.transition_to(self.estado_waitInRed)


semaforo1 = Semaforo(0,0,2000,1000,500)

while True:

    # Input processing
    if button_a.was_pressed(): semaforo1.post_event("A")
    
    semaforo1.update()
    utime.sleep_ms(20)
```

`FSM.PY`

```python
import utime

ENTRY = "ENTRY"
EXIT  = "EXIT"

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active and utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
            self.active = False
            self.owner.post_event(self.event)


class FSMTask:
    def __init__(self):
        self._q = []
        self._timers = []
        self._state = None

    def post_event(self, ev):
        self._q.append(ev)

    def add_timer(self, event, duration):
        t = Timer(self, event, duration)
        self._timers.append(t)
        return t

    def transition_to(self, new_state):
        if self._state:
            self._state(EXIT)
        self._state = new_state
        self._state(ENTRY)

    def update(self):
        for t in self._timers:
            t.update()
        while self._q:
            ev = self._q.pop(0)
            if self._state:
                self._state(ev)
```


> Construye la máquina de estados que modela el problema usando PlantUML.

<img width="500" height="500" alt="Screenshot 2026-02-09 182141" src="https://github.com/user-attachments/assets/727cec75-267f-4b8b-8a84-45bbbf107a7e" />


### Actividad 03
***¿Cómo es posible estructurar una aplicación usando una máquina de estados para poder atender varios eventos de manera concurrente?***

Se estructura creando objetos independientes, cada uno con su propia máquina de estados, cola de eventos y temporizadores. El ciclo principal llama continuamente a `update()`, permitiendo procesar eventos sin bloquear el programa. Aunque hay un solo hilo, esto genera concurrencia cooperativa, ya que varias tareas avanzan casi al mismo tiempo.

## Bitácora de aplicación 

### Actividad 04
>Temporizador interactivo con máquina de estados

***PlantUML***


<img width="500" height="500" alt="Screenshot 2026-02-12 211900" src="https://github.com/user-attachments/assets/a406f1f7-a2d0-45ac-919b-e44ec5753e06" />


***Codigo Microbit***

```python
from microbit import *
import utime
import music

# -------------------------------------------------
# DISPLAY
# -------------------------------------------------
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()

SKULL = Image(
    "09090:"
    "99999:"
    "99999:"
    "09990:"
    "00900"
)

# -------------------------------------------------
# TIMER (NO MODIFICAR)
# -------------------------------------------------
class Timer:
    def __init__(self, owner, event, duration):
        self.owner = owner
        self.event = event
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def update(self):
        if self.active and utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
            self.active = False
            self.owner.post_event(self.event)

# -------------------------------------------------
# FSM
# -------------------------------------------------
class Task:

    def __init__(self):
        self.queue = []
        self.timers = []

        self.timer = self.createTimer("Timeout", 1000)

        self.n = 20
        self.state = None
        self.transition(self.config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.queue.append(ev)

    def update(self):
        for t in self.timers:
            t.update()

        while self.queue:
            self.state(self.queue.pop(0))

    def transition(self, new_state):
        self.state = new_state
        self.state("ENTRY")

    # ---------------- ESTADOS ----------------

    # CONFIGURACION
    def config(self, ev):
        if ev == "ENTRY":
            display.show(FILL[self.n])

        elif ev == "A" and self.n < 25:
            self.n += 1
            display.show(FILL[self.n])

        elif ev == "B" and self.n > 15:
            self.n -= 1
            display.show(FILL[self.n])

        elif ev == "S":
            self.transition(self.countdown)

    # CUENTA REGRESIVA
    def countdown(self, ev):
        if ev == "ENTRY":
            self.timer.start()

        elif ev == "Timeout":
            self.n -= 1
            display.show(FILL[self.n])

            if self.n == 0:
                self.transition(self.end)
            else:
                self.timer.start()

    # FIN
    def end(self, ev):
        if ev == "ENTRY":
            display.show(SKULL)
            music.play(music.WAWAWAWAA)

        elif ev == "A":
            self.n = 20
            self.transition(self.config)

# -------------------------------------------------
# LOOP PRINCIPAL
# -------------------------------------------------
task = Task()

while True:

    if button_a.was_pressed():
        task.post_event("A")

    if button_b.was_pressed():
        task.post_event("B")

    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    task.update()
```


## Bitácora de reflexión

### Actividad 05
> Modificar el temporizador interactivo de modo que puedas controlarlo también desde p5.js además de los botones del micro:bit.

***Codigo Microbit***

```python
from microbit import *
import utime
import music

uart.init(115200)

# -------------------------------------------------
# DISPLAY
# -------------------------------------------------
def make_fill_images(on='9', off='0'):
    imgs = []
    for n in range(26):
        rows = []
        k = 0
        for y in range(5):
            row = []
            for x in range(5):
                row.append(on if k < n else off)
                k += 1
            rows.append(''.join(row))
        imgs.append(Image(':'.join(rows)))
    return imgs

FILL = make_fill_images()

SKULL = Image(
    "09090:"
    "99999:"
    "99999:"
    "09990:"
    "00900"
)

# -------------------------------------------------
# TIMER
# -------------------------------------------------
class Timer:
    def __init__(self, owner, event, duration):
        self.owner = owner
        self.event = event
        self.duration = duration
        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def update(self):
        if self.active and utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
            self.active = False
            self.owner.post_event(self.event)

# -------------------------------------------------
# FSM
# -------------------------------------------------
class Task:

    def __init__(self):
        self.queue = []
        self.timers = []

        self.timer = self.createTimer("Timeout", 1000)

        self.n = 20
        self.state = None
        self.transition(self.config)

    def createTimer(self, event, duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.queue.append(ev)

    def update(self):
        for t in self.timers:
            t.update()

        while self.queue:
            self.state(self.queue.pop(0))

    def transition(self, new_state):
        self.state = new_state
        self.state("ENTRY")

    # -------- ESTADOS --------

    def config(self, ev):
        if ev == "ENTRY":
            display.show(FILL[self.n])

        elif ev == "A" and self.n < 25:
            self.n += 1
            display.show(FILL[self.n])

        elif ev == "B" and self.n > 15:
            self.n -= 1
            display.show(FILL[self.n])

        elif ev == "S":
            self.transition(self.countdown)

    def countdown(self, ev):
        if ev == "ENTRY":
            self.timer.start()

        elif ev == "Timeout":
            self.n -= 1
            display.show(FILL[self.n])

            if self.n == 0:
                self.transition(self.end)
            else:
                self.timer.start()

    def end(self, ev):
        if ev == "ENTRY":
            display.show(SKULL)
            music.play(music.WAWAWAWAA)

        elif ev == "A":
            self.n = 20
            self.transition(self.config)

# -------------------------------------------------
# LOOP
# -------------------------------------------------
task = Task()

while True:

    # botones físicos
    if button_a.was_pressed():
        task.post_event("A")

    if button_b.was_pressed():
        task.post_event("B")

    if accelerometer.was_gesture("shake"):
        task.post_event("S")

    # mensajes desde p5.js
    msg = uart.read()
    if msg:
        for c in msg:
            letra = chr(c)
            if letra in ["A", "B", "S"]:
                task.post_event(letra)

    task.update()

```

***Codigo p5.js***

```js
let serial;
let portName = "COM3"; 

function setup() {
  createCanvas(400, 200);

  serial = new p5.SerialPort();
  serial.open(portName);
}

function draw() {
  background(30);
  fill(255);
  textSize(16);

  text("Control del temporizador", 80, 60);
  text("A → UP", 140, 100);
  text("B → DOWN", 130, 130);
  text("S → ARM", 140, 160);
}

function keyPressed() {

  if (key === 'A') {
    serial.write('A');
  }

  if (key === 'B') {
    serial.write('B');
  }

  if (key === 'S') {
    serial.write('S');
  }
}

```

***Explicacion***

Resolví el reto extendiendo el temporizador interactivo para que pudiera recibir eventos tanto desde los botones del micro:bit como desde `p5.js`, sin modificar la arquitectura de máquina de estados. Para lograrlo, utilicé la comunicación serial para recibir las letras `A`, `B` y `S` enviadas desde `p5.js`, simulando las acciones de subir, bajar y activar el temporizador. Cuando el micro:bit recibe estos caracteres, los convierte en eventos y los envía a la máquina de estados mediante `post_event()`, manteniendo intacta la lógica de los estados, el uso de la clase Timer y el funcionamiento general de la aplicación.


