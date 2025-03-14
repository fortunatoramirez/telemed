### Práctica 5: Monitoreo y Control en Tiempo Real con ESP32 y Node.js

#### Objetivo
Desarrollar una aplicación que permita leer señales analógicas a través de un ESP32 y mostrar estos datos en tiempo real en una interfaz web. Adicionalmente, la interfaz permitirá enviar comandos para controlar un LED en el ESP32. Esta práctica integrará tecnologías como Node.js, socket.io, y la programación de microcontroladores para crear un sistema interactivo de monitoreo y control.

#### Estructura del Proyecto
El proyecto estará organizado en varios archivos distribuidos en carpetas específicas para facilitar la gestión y el desarrollo del código. A continuación se presenta la estructura propuesta del proyecto:

```
SERVER/
│
│
├── public/               # Carpeta para archivos estáticos servidos por Express
│   ├── index.html        # Página HTML principal para la interfaz de usuario
│   ├── style.css         # Hoja de estilos CSS para estilizar la página web
│   └── index.js          # Scripts de JavaScript para manejar la lógica del lado del cliente
│
├── main.js               # Script del servidor Node.js utilizando Express y socket.io
├── package.json          # Configuración y dependencias de Node.js
```

### Descripción de Archivos y Carpetas

- **node_modules/**: Contiene todas las bibliotecas de Node.js necesarias para el proyecto, las cuales son instaladas a través de npm.
- **public/**: Esta carpeta contiene todos los archivos que serán accesibles públicamente en la web. Incluye el HTML, CSS, y JavaScript necesario para la interfaz de usuario.
- **main.js**: Este es el archivo principal del servidor que configura el servidor HTTP y maneja las conexiones WebSocket para la comunicación en tiempo real.
- **package.json**: Define las dependencias del proyecto y otros metadatos necesarios para configurar y ejecutar la aplicación Node.js.
- **telecom_p5_p1.ino**: Contiene el código que se ejecutará en el ESP32, incluyendo la lógica para la lectura de datos analógicos y la respuesta a comandos del servidor para controlar dispositivos como LEDs.

### Paso 1: Configuración del Servidor Node.js (Archivo `main.js`)

El servidor Node.js utilizará Express para servir contenido estático y socket.io para manejar la comunicación WebSocket. La configuración básica es:

```javascript
// main.js
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

app.use(express.static('public'));

io.on('connection', (socket) => {
    console.log('Nuevo dispositivo conectado.');

    socket.on('data', (data) => {
        console.log(`Datos recibidos del ESP32: ${data}`);
        // Procesamiento o respuesta
    });

    socket.on('desde_cliente', (data) => {
        console.log(`Datos recibidos desde la página: ${data}`);
        io.sockets.emit("desde_servidor_comando",data)
    });

    socket.on('FORTUNATO', (data) => {
        console.log(`FORTUNATO: ${data}`);
        io.sockets.emit("desde_servidor_comando",data)
    });
    socket.on('VILLEGAS', (data) => {
        console.log(`VILLEGAS: ${data}`);
        io.sockets.emit("desde_servidor_comando",data)
    });
    socket.on('LAS_CHICAS_SUPER_PODEROSAS', (data) => {
        console.log(`LAS_CHICAS_SUPER_PODEROSAS: ${data}`);
        io.sockets.emit("LAS_CHICAS_SUPER_PODEROSAS_DS",data)
    });
    socket.on('ARMANDO', (data) => {
        console.log(`ARMANDO: ${data}`);
        io.sockets.emit("ARMANDO_DS",data)
    });
    socket.on('ME_PUSE_HASTA_EL_BASH', (data) => {
        console.log(`ME_PUSE_HASTA_EL_BASH: ${data}`);
        io.sockets.emit("ME_PUSE_HASTA_EL_BASH_DS",data)
    });

    socket.on('disconnect', () => {
        console.log('Dispositivo desconectado.');
    });

});

server.listen(3000, () => {
    console.log('Servidor escuchando en puerto 3000');
});
```

### Paso 2: Código ESP32 

El código para el ESP32 configurará el dispositivo para conectarse a la red WiFi y al servidor WebSocket, enviará datos de sensores y recibirá comandos para controlar un LED.

```cpp
#include <WiFi.h>
#include <SocketIoClient.h>
#define ONBOARD_LED 2

const char*       ssid     = "Nombre-de-la-Red"; 
const char*       password = "Contraseña";
const char*       server   = "ip-del-servidor";
const uint16_t    port     = 3000; //Puerto del servidor

String    mensaje;
uint64_t  now         = 0;
uint64_t  timestamp   = 0;

int valor;
float valor_f;

SocketIoClient socketIO;

void setup() {
  Serial.begin(115200);
  conectar_WiFiSTA();
  socketIO.begin(server, port);
  socketIO.on("desde_servidor_comando",procesar_mensaje_recibido);

  pinMode(ONBOARD_LED, OUTPUT);
  pinMode(12, OUTPUT);

}

void loop() {
  now = millis();
  if(now - timestamp > 1000)
  {
    timestamp = now;

    valor = analogRead(34);
    valor_f = ((float)valor/4095)*200;
    
    mensaje = "\""+String(valor_f)+"\"";
    socketIO.emit("FORTUNATO",mensaje.c_str());
  }
  socketIO.loop();
}

void conectar_WiFiSTA()
{
   delay(10);
   Serial.println("");
   WiFi.mode(WIFI_STA);
   WiFi.begin(ssid, password);
   while (WiFi.status() != WL_CONNECTED) 
   { 
     delay(100);  
     Serial.println('.'); 
   }
   Serial.println("");
   Serial.print("Conectado a STA:\t");
   Serial.println(ssid);
   Serial.print("My IP Address:\t");
   Serial.println(WiFi.localIP());
}

void procesar_mensaje_recibido(const char * payload, size_t length) {
 Serial.printf("Mensaje recibido: %s\n", payload);
 String paystring = String(payload);
 if(paystring == "ON")
 {
  digitalWrite(ONBOARD_LED,HIGH);
  digitalWrite(12,HIGH);

 }
 else if(paystring == "OFF")
 {
  digitalWrite(ONBOARD_LED,LOW);
  digitalWrite(12,LOW);

 }
}

```

### Paso 3: Interfaz Web (Archivos `index.html`, `style.css`, `index.js`)

La interfaz web incluirá elementos para mostrar datos y botones para enviar comandos al ESP32. El diseño es responsivo y visualmente atractivo.

**Archivo `index.html`**:
```html
<!DOCTYPE html>

<!-- ESTO ES UN COMENTARIO-->

<html>

    <head>
        <title> Mi página</title>
        <script type="text/javascript" src="socket.io/socket.io.js"></script>
        <script type="text/javascript" src="https://www.google.com/jsapi"></script>
        <script type="text/javascript" src="index.js"></script>
        <link rel="stylesheet" type="text/css" href="style.css">
    </head>

    <body>
        <h2>Fortunato - Monitoreo y Control</h2>
        <div id="div_otro"></div>
        <div id="las_chicas_super_poderosas"></div>
        <div id="mphb"></div>
        <div id="armando"></div>
        <div id="villegas"></div>
        <!-- 
        <div id="my_div"></div>
        -->
        <br><br>
        <div>
            <input type="button" class="button button1" value="ENCENDER LED" onclick="encender()">
            <input type="button" class="button button1" value="APAGAR LED" onclick="apagar()">
            <br>
            <br> 
            <input type="text" id="txt_comando">
            <input type="button" class="button button1" value="ENVIAR COMANDO" onclick="enviar_comando()">
        </div>
    </body>

</html>
```

**Archivo `style.css`**:
```css
/*Mi estilo*/
.button1{
    background-color: white; 
    color: black; 
    border: 2px solid #4CAF50;
  }
.button1:hover {background-color: #3e8e41}

.button1:active {
    background-color: #3e8e41;
    box-shadow: 0 5px #666;
    transform: translateY(4px);
}
```

**Archivo `index.js`**:
```javascript
var socket = io.connect();
var explodedValues = [4];

socket.on('desde_servidor_otro', function(data){
    //console.log(data);
    //data_JSON = JSON.parse(data)
    var cadena = `<div> <strong> Otro:  <font color="orange">` + data + `</strong> </div>`;
    document.getElementById("div_otro").innerHTML = cadena;
    explodedValues[3] = parseFloat(data);
    drawVisualization();
});

socket.on('LAS_CHICAS_SUPER_PODEROSAS_DS', function(data){
    //console.log(data);
    //data_JSON = JSON.parse(data)
    var cadena = `<div> <strong> &#128536; Las Chicas Súper Poderosas:  <font color="orange">` + data + `</strong> </div>`;
    document.getElementById("las_chicas_super_poderosas").innerHTML = cadena;
    //explodedValues[0] = parseFloat(data);
    //drawVisualization();
});

socket.on('ME_PUSE_HASTA_EL_BASH_DS', function(data){
    //console.log(data);
    //data_JSON = JSON.parse(data)
    var cadena = `<div> <strong> &#128526; Me Puse Hasta el Bash:  <font color="blue">` + data + `</strong> </div>`;
    document.getElementById("mphb").innerHTML = cadena;
    //explodedValues[1] = parseFloat(data);
    //drawVisualization();
});

socket.on('ARMANDO_DS', function(data){
    //console.log(data);
    //data_JSON = JSON.parse(data)
    var cadena = `<div> <strong> &#128584; Me Puse Hasta el Bash:  <font color="black">` + data + `</strong> </div>`;
    document.getElementById("armando").innerHTML = cadena;
    //explodedValues[2] = parseFloat(data);
    //drawVisualization();
});

socket.on('VILLEGAS_DS', function(data){
    //console.log(data);
    //data_JSON = JSON.parse(data)
    var cadena = `<div> <strong> &#128584; Villegas:  <font color="green">` + data + `</strong> </div>`;
    document.getElementById("villegas").innerHTML = cadena;
    //explodedValues[3] = parseFloat(data);
    //drawVisualization();
});

function encender()
{
    socket.emit("desde_cliente","ON");
}

function apagar()
{
    socket.emit("desde_cliente","OFF");
}

function enviar_comando()
{
    var comando = document.getElementById('txt_comando').value;
    socket.emit('desde_cliente',comando);

}


function drawVisualization() {
    // Create and populate the data table from the values received via websocket
    var data = google.visualization.arrayToDataTable([
        ['Tracker', '1',{ role: 'style' }],
        ['Temp[C]', explodedValues[0],'color: orange'],
        ['Temp[F]', explodedValues[1],'color: blue'],
        ['Hum[%]', explodedValues[2],'color: black'],
        ['Otro',   explodedValues[3],'color: green']
    ]);
    
    // use a DataView to 0-out all the values in the data set for the initial draw
    var view = new google.visualization.DataView(data);
    view.setColumns([0, {
        type: 'number',
        label: data.getColumnLabel(1),
        calc: function () {return 0;}
    }]);
    
    // Create and draw the plot
    var chart = new google.visualization.BarChart(document.getElementById('div_grafica'));
    
    var options = {
        title:"Monitoreo",
        width: 1200,
        height: 300,
        bar: { groupWidth: "95%" },
        legend: { position: "none" },
        animation: {
            duration: 0
        },
        hAxis: {
            // set these values to make the initial animation smoother
            minValue: 0,
            maxValue: 200
        }
    };
    
    var runOnce = google.visualization.events.addListener(chart, 'ready', function () {
        google.visualization.events.removeListener(runOnce);
        chart.draw(data, options);
    });
    
    chart.draw(view, options);
}

google.load('visualization', '1', {packages: ['corechart'], callback: drawVisualization});
```

### Paso 4: Ejecución y Pruebas

1. **Ejecuta el servidor Node.js**:
   - Navega al directorio del servidor y ejecuta `node main.js` para iniciar el servidor.
2. **Programa el ESP32**:
   - Sube el código `archivo.ino` al ESP32 utilizando el Arduino IDE.
3. **Accede a la interfaz web**:
   - Abre un navegador y visita `http://ip-del-servidor:3000`.

