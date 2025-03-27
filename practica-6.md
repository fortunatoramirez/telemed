

### 🛠️ **Control remoto de un brazo robótico por gestos usando MediaPipe, Socket.IO y Arduino**

---

## 🎯 Objetivo general:

Diseñar un sistema distribuido de control por gestos para un brazo robótico, utilizando visión por computadora con MediaPipe, comunicación cliente-servidor mediante Socket.IO y control físico a través de Arduino.

---

## 🧩 Estructura de la práctica

1. **Parte 1: Detección de la mano y envío de comandos al servidor (Cliente A)**
2. **Parte 2: Configuración del servidor central con Node.js y socket.io**
3. **Parte 3: Cliente Python que recibe los comandos y controla el brazo robótico (Cliente B)**
4. **Parte 4: Pruebas del sistema distribuido y mejora de la precisión gestual**

---

# 🔹 PARTE 1: Detección de la mano y envío de comandos al servidor (Cliente A)

---

### 🎯 Objetivo de esta parte

Crear un cliente Python que utilice MediaPipe para detectar la posición de la mano del usuario en tiempo real y enviar comandos de control (por ejemplo, ángulos de servos) a un servidor utilizando Socket.IO.

---

### 📦 Requisitos previos

- Tener Python instalado (idealmente 3.8 o superior).
- Tener pip actualizado.
- Conexión a internet para instalar librerías.
- Una cámara web funcional.

---

### 🧪 Instalación de librerías necesarias

Desde consola o terminal:

```bash
pip install mediapipe opencv-python socketIO-client
```

---

### 📁 Estructura esperada de archivos

```
cliente_mediapipe/
│
├── cliente_gestos.py      ← Código principal de esta parte
```

---

### 📄 Código: `cliente_gestos.py`

```python
import cv2
import mediapipe as mp
from socketIO_client import SocketIO

# Configuración del servidor (asegúrate que coincida con tu servidor)
SERVER_IP = 'localhost'  # cámbialo si el servidor está en otra máquina
SERVER_PORT = 3000

# Inicializar conexión con el servidor
# socketIO = SocketIO(SERVER_IP, SERVER_PORT)

# Inicializar MediaPipe
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(max_num_hands=1, min_detection_confidence=0.7)
mp_drawing = mp.solutions.drawing_utils

cap = cv2.VideoCapture(0)

def map_range(value, in_min, in_max, out_min, out_max):
    return int((value - in_min) * (out_max - out_min) / (in_max - in_min) + out_min)

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    frame = cv2.flip(frame, 1)
    rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(rgb)

    if results.multi_hand_landmarks:
        landmarks = results.multi_hand_landmarks[0].landmark

        # Ejemplo simple: usar la posición Y del dedo índice para controlar 4 servos
        finger_tip_y = landmarks[8].y  # dedo índice
        mapped_value = map_range(finger_tip_y, 0, 1, 0, 180)

        # Crear comandos para los 4 servos (todos el mismo valor como prueba)
        angles = {
            'servo0': mapped_value,
            'servo1': mapped_value,
            'servo2': mapped_value,
            'servo3': mapped_value
        }

        print("Enviando:", angles)
        # socketIO.emit('control_servos', angles)

        # Dibujar mano en pantalla
        mp_drawing.draw_landmarks(frame, results.multi_hand_landmarks[0], mp_hands.HAND_CONNECTIONS)

    cv2.imshow("Detección de Mano - Cliente A", frame)
    if cv2.waitKey(1) & 0xFF == 27:
        break

cap.release()
cv2.destroyAllWindows()
```

---

### 📌 Notas:

- Este cliente **no controla directamente el brazo robótico** aún.
- Solo **detecta la mano y envía valores angulares** al servidor.
- Los 4 servos reciben el mismo valor, solo para pruebas iniciales.
- Asegúrate de que el servidor esté corriendo antes de iniciar este cliente.

---

# 🔹 PARTE 2: Servidor central con Node.js y Socket.IO

---

### 🎯 Objetivo de esta parte

Configurar un servidor en Node.js utilizando la versión `1.7.2` de la biblioteca `socket.io`, que reciba los comandos enviados por el cliente con MediaPipe y los retransmita a los demás clientes conectados (como el cliente que controla el brazo robótico).

---

### 📦 Requisitos previos

- Tener instalado **Node.js** (versión 14 o superior recomendada).
- Tener `npm` disponible (se instala junto con Node.js).
- Tener conexión a internet para instalar las dependencias.

---

### 📁 Estructura esperada de archivos

```
servidor/
│
├── server.js            ← Código principal del servidor
├── package.json         ← Se generará con `npm init`
```

---

### 🧪 Paso 1: Crear y configurar el proyecto

Desde terminal o consola:

```bash
mkdir servidor
cd servidor
npm init -y
npm install socket.io@1.7.2
```

---

### 📄 Código: `server.js`

```js
const app = require('http').createServer();
const io = require('socket.io')(app);

const PORT = 3000;
app.listen(PORT);
console.log(`Servidor corriendo en el puerto ${PORT}`);

io.on('connection', function (socket) {
    console.log('Cliente conectado:', socket.id);

    socket.on('disconnect', function () {
        console.log('Cliente desconectado:', socket.id);
    });

    socket.on('control_servos', function (data) {
        console.log('Comando recibido:', data);
        
        // Reenviar a todos los demás clientes (excepto el emisor)
        socket.broadcast.emit('control_servos', data);
    });
});
```

---

### 🟢 ¿Qué hace este servidor?

- Escucha conexiones en el puerto `3000`.
- Cuando un cliente envía el evento `'control_servos'`, **reenvía ese mensaje a todos los demás clientes** usando `socket.broadcast.emit`.
- Imprime todo lo que recibe por consola, útil para depurar.

---

### 🧪 Paso 2: Ejecutar el servidor

Desde la carpeta `servidor/`:

```bash
node server.js
```

Deberías ver:

```
Servidor corriendo en el puerto 3000
```

Cuando conectes el cliente de la Parte 1, verás:

```
Cliente conectado: /#abc123...
Comando recibido: { servo0: 90, servo1: 90, ... }
```

---

### 💡 Notas:

- Este servidor **no necesita saber quién es el cliente MediaPipe ni quién es el del Arduino**. Simplemente recibe y reenvía.
- En la Parte 3 se conectará otro cliente (Cliente B) que sí moverá el brazo.
- `socket.broadcast.emit()` evita que el emisor reciba sus propios datos, útil para mantener tráfico limpio.

---

# 🔹 PARTE 3: Cliente Python que recibe comandos y controla el brazo (Cliente B)

---

### 🎯 Objetivo de esta parte

Construir un cliente Python que:

- Se conecta al servidor con socket.io
- Escucha los comandos enviados por otro cliente (como el de MediaPipe)
- Envía esos datos al Arduino por el puerto serial, haciendo que los servos se muevan

---

### 📦 Requisitos previos

- Arduino UNO con el brazo robótico de 4 servos conectado a los pines 4, 5, 6 y 7
- Python 3.8+
- Arduino IDE (para subir el programa al Arduino)
- Biblioteca `pyserial` y `socketIO-client` en Python

---

### 🧪 Paso 1: Instalar las librerías necesarias

```bash
pip install pyserial socketIO-client
```

---

### 🧪 Paso 2: Subir el siguiente programa al Arduino

Este programa recibe comandos por el puerto serial en formato JSON, por ejemplo:

```json
{ "servo0": 90, "servo1": 45, "servo2": 120, "servo3": 60 }
```

#### 📄 Código para Arduino:

```cpp
#include <Servo.h>
#include <ArduinoJson.h>

Servo servos[4];
int pins[4] = {4, 5, 6, 7};

void setup() {
  Serial.begin(9600);
  for (int i = 0; i < 4; i++) {
    servos[i].attach(pins[i]);
    //servos[i].write(90);  // posición inicial
  }
  servos[0].write(30);
  servos[1].write(40);
  servos[2].write(0);
  servos[3].write(120);
}

void loop() {
  if (Serial.available()) {
    StaticJsonDocument<128> doc;
    DeserializationError error = deserializeJson(doc, Serial);

    if (!error) {
      for (int i = 0; i < 4; i++) {
        String key = "servo" + String(i);
        if (doc.containsKey(key)) {
          int angle = doc[key];
          angle = constrain(angle, 0, 180);
          servos[i].write(angle);
        }
      }
    }
  }
}

```

> 💡 Usa `ArduinoJson` versión 6, ya incluida desde el gestor de librerías del IDE de Arduino.

---

### 🧪 Paso 3: Crear el cliente Python (cliente_brazo.py)

Este cliente:
- Se conecta al servidor
- Escucha eventos `'control_servos'`
- Envía los datos directamente al Arduino

#### 📄 Código: `cliente_brazo.py`

```python
import serial
import json
from socketIO_client import SocketIO

# Configura el puerto serial (ajusta el puerto a tu sistema)
arduino = serial.Serial('COM3', 9600)  # En Linux o Mac puede ser '/dev/ttyUSB0'

# Configuración del servidor
SERVER_IP = 'localhost'
SERVER_PORT = 3000

def on_connect():
    print("Conectado al servidor.")

def on_disconnect():
    print("Desconectado del servidor.")

def on_servo_data(data):
    print("Recibido:", data)
    try:
        json_data = json.dumps(data)
        arduino.write((json_data + '\n').encode('utf-8'))
    except Exception as e:
        print("Error al enviar al Arduino:", e)

# Conexión al servidor
socketIO = SocketIO(SERVER_IP, SERVER_PORT)
socketIO.on('connect', on_connect)
socketIO.on('disconnect', on_disconnect)
socketIO.on('control_servos', on_servo_data)

print("Esperando datos...")
socketIO.wait()  # Mantiene el script corriendo
```

---

### 🔁 Flujo final hasta ahora

```
[Cliente A - MediaPipe]
        ↓ socket.io
[Servidor Node.js]
        ↓ socket.io
[Cliente B - Arduino]
        ↓ serial
[Brazo robótico se mueve]
```

---

### 🧪 Prueba del sistema distribuido (mínima)

1. Ejecuta el **servidor** (`node server.js`)
2. Ejecuta el **cliente del brazo** (`cliente_brazo.py`)
3. Ejecuta el **cliente MediaPipe** (`cliente_gestos.py`)
4. Observa cómo el brazo reacciona al mover tu mano

---

### 📌 Notas:

- Asegúrate de que `COM3` o el puerto que uses esté correcto.
- Si hay problemas de comunicación serial, puede ayudar reiniciar Arduino o agregar un pequeño `delay(100)` en el `loop()` de Arduino.
- El cliente del brazo no tiene que hacer procesamiento visual. Solo ejecuta.

---

¡Perfecto! Ahora que ya tienes los tres componentes esenciales funcionando (detección de mano, servidor central, y brazo controlado por cliente remoto), vamos a la última etapa de la práctica.

---

# 🔹 PARTE 4: Pruebas del sistema distribuido y mejora de la precisión gestual

---

### 🎯 Objetivo de esta parte

- Integrar todo el sistema para probarlo en conjunto.
- Analizar el comportamiento del brazo según los movimientos de la mano.
- Mejorar la calidad del control gestual con filtros y asignaciones más precisas.
- Aplicar correcciones lógicas si es necesario (por ejemplo, evitar movimientos bruscos o invertir ejes).

---

### 🧪 Paso 1: Prueba del sistema completo

1. **Sube el código al Arduino.**
2. Ejecuta el **servidor Node.js** con:

```bash
node server.js
```

3. Ejecuta el **cliente Python del brazo**:

```bash
python cliente_brazo.py
```

4. Ejecuta el **cliente con MediaPipe**:

```bash
python cliente_gestos.py
```

5. Observa cómo el brazo robótico **reacciona en tiempo real** a tus movimientos de mano.

---

### 📈 Paso 2: Ajustes de sensibilidad

Si el brazo se mueve muy rápido o se sacude, puedes **filtrar el movimiento** de esta manera:

#### En `cliente_gestos.py`, agrega un filtro de suavizado:

```python
# Variables globales
prev_angle = 90
alpha = 0.5  # entre 0 (más lento) y 1 (respuesta inmediata)

# Dentro del loop, después de mapear el valor:
mapped_value = int(alpha * mapped_value + (1 - alpha) * prev_angle)
prev_angle = mapped_value
```

Esto suaviza los cambios bruscos y da una sensación más estable.

---

### 🧩 Paso 3: Asignación por dedo (opcional)

Actualmente todos los servos reciben el mismo valor. Puedes diferenciarlo con más precisión usando otros dedos o gestos. Por ejemplo:

```python
# Usar distintos dedos para distintos servos
angles = {
    'servo0': map_range(landmarks[8].y, 0, 1, 0, 180),   # dedo índice
    'servo1': map_range(landmarks[12].y, 0, 1, 0, 180),  # dedo medio
    'servo2': map_range(landmarks[16].y, 0, 1, 0, 180),  # dedo anular
    'servo3': map_range(landmarks[20].y, 0, 1, 0, 180),  # dedo meñique
}
```

Esto da un control más preciso (cada dedo controla un motor distinto).

---

### 🎯 Posibles mejoras adicionales

- Invertir dirección del movimiento si algún servo gira al revés:
```python
map_range(y, 0, 1, 180, 0)
```

- Enviar comandos **cada 100 ms** en lugar de cada frame (para evitar sobrecarga):

```python
import time
last_send = time.time()

# dentro del if
if time.time() - last_send > 0.1:
    socketIO.emit('control_servos', angles)
    last_send = time.time()
```

---


## ✅ Cambios sugeridos para el control del brazo:

| Servo | Descripción         | Mano        | Control                       | Sentido         |
|-------|---------------------|-------------|-------------------------------|-----------------|
| 0     | Rotación base       | Izquierda   | Movimiento **X** de la palma  | **Invertido**   |
| 1     | Falange inferior    | Izquierda   | Movimiento **Y** de la palma  | **Invertido**   |
| 2     | Falange superior    | Derecha     | Movimiento **Y** de la palma  | **Normal**      |
| 3     | Pinza               | Derecha     | Distancia entre pulgar e índice | **Normal**    |

---

## 📄 Código actualizado: `cliente_gestos_final.py`

```python
import cv2
import mediapipe as mp
import math
import time
# from socketIO_client import SocketIO  # Descomenta si vas a enviar

# socketIO = SocketIO('localhost', 3000)

mp_hands = mp.solutions.hands
hands = mp_hands.Hands(
    max_num_hands=2,
    min_detection_confidence=0.7,
    min_tracking_confidence=0.7
)
mp_drawing = mp.solutions.drawing_utils
cap = cv2.VideoCapture(0)

def map_range(value, in_min, in_max, out_min, out_max):
    value = max(min(value, in_max), in_min)
    return int((value - in_min) * (out_max - out_min) / (in_max - in_min) + out_min)

def distance(p1, p2):
    return math.hypot(p1.x - p2.x, p1.y - p2.y)

def draw_landmark_point(frame, landmark, shape, color=(0, 255, 255)):
    h, w, _ = frame.shape
    cx, cy = int(landmark.x * w), int(landmark.y * h)
    if shape == "circle":
        cv2.circle(frame, (cx, cy), 8, color, -1)
    elif shape == "cross":
        cv2.drawMarker(frame, (cx, cy), color, cv2.MARKER_CROSS, 10, 2)

def clasificar_manos(hands_result):
    mano_izquierda = None
    mano_derecha = None
    for idx, hand in enumerate(hands_result.multi_handedness):
        label = hand.classification[0].label
        if label == "Left":
            mano_izquierda = hands_result.multi_hand_landmarks[idx]
        elif label == "Right":
            mano_derecha = hands_result.multi_hand_landmarks[idx]
    return mano_izquierda, mano_derecha

last_send = time.time()

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    frame = cv2.flip(frame, 1)
    rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(rgb)

    if results.multi_hand_landmarks:
        left, right = clasificar_manos(results)

        servo0 = 90  # rotación base
        servo1 = 90  # brazo inferior
        servo2 = 90  # brazo superior
        servo3 = 90  # pinza

        if left:
            lm_left = left.landmark
            # Servo 0: rotación base (invertida)
            servo0 = map_range(lm_left[0].x, 0, 1, 180, 0)
            # Servo 1: brazo inferior (invertida)
            servo1 = map_range(lm_left[0].y, 0, 1, 0, 180)
            # Visual
            draw_landmark_point(frame, lm_left[0], "circle", (0, 255, 0))  # palma izquierda
            mp_drawing.draw_landmarks(frame, left, mp_hands.HAND_CONNECTIONS)

        if right:
            lm_right = right.landmark
            # Servo 2: brazo superior → Y de la palma derecha
            servo2 = map_range(lm_right[0].y, 0, 1, 180, 0)

            # Servo 3: pinza → distancia entre pulgar e índice
            dist = distance(lm_right[4], lm_right[8])
            servo3 = map_range(dist, 0.02, 0.25, 180, 0)

            # Visual
            draw_landmark_point(frame, lm_right[0], "circle", (255, 0, 255))  # palma derecha
            draw_landmark_point(frame, lm_right[4], "circle", (255, 255, 0))  # pulgar
            draw_landmark_point(frame, lm_right[8], "circle", (255, 255, 0))  # índice
            mp_drawing.draw_landmarks(frame, right, mp_hands.HAND_CONNECTIONS)

        angles = {
            'servo0': servo0,
            'servo1': servo1,
            'servo2': servo2,
            'servo3': servo3
        }

        print("Ángulos:", angles)

        if time.time() - last_send > 0.1:
            # socketIO.emit('control_servos', angles)
            last_send = time.time()

    cv2.imshow("Control por gestos - Ambas manos (final)", frame)
    if cv2.waitKey(1) & 0xFF == 27:
        break

cap.release()
cv2.destroyAllWindows()
```

---

### ✅ Resumen final de control:

| Servo | Función           | Control con...         | Landmark usado | Sentido       |
|-------|-------------------|-------------------------|----------------|---------------|
| 0     | Rotación base     | Mano izquierda (X)      | `lm[0]`        | Invertido     |
| 1     | Brazo inferior     | Mano izquierda (Y)      | `lm[0]`        | Invertido     |
| 2     | Brazo superior     | Mano derecha (Y)        | `lm[0]`        | Normal        |
| 3     | Pinza              | Distancia pulgar/índice | `lm[4]`, `lm[8]` | Normal      |







