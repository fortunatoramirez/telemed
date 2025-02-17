### Práctica 4: Comunicación en Tiempo Real entre ESP32 y Servidor Node.js

#### Objetivo:

Desarrollar una aplicación utilizando Node.js y socket.io para establecer una comunicación en tiempo real entre un servidor y un dispositivo ESP32. El ESP32 enviará valores leídos de una entrada analógica al servidor para su visualización y análisis.

---

### Capítulo 1. Fundamentos Teóricos

#### 1. Comunicación WebSocket

Explorar cómo la tecnología WebSocket proporciona un canal de comunicación bidireccional y full-duplex sobre un único socket persistente, lo que es fundamental para aplicaciones que requieren una baja latencia, como la comunicación en tiempo real entre un servidor y un dispositivo hardware.

#### 1.2 Tecnología Socket.io

Introducción a socket.io, una biblioteca de JavaScript para aplicaciones web en tiempo real que permite la comunicación bidireccional entre los navegadores web y los servidores. Se discutirá la versión 1.7.2 y su relevancia para la compatibilidad con el ESP32.

---

### Capítulo 2. Desarrollo

#### 2.1 Configuración del Hardware

**Materiales Necesarios:**

- ESP32.
- Sensor analógico (ejemplo: potenciómetro).
- Cable USB para programación y comunicación.

**Procedimiento:**

1. **Conectar el Sensor al ESP32:**
   - Conecte el sensor analógico al pin adecuado del ESP32.
   - Asegúrese de que el sensor esté correctamente alimentado.

2. **Programación del ESP32:**
   - Utilice el IDE de Arduino para cargar el sketch que leerá los datos del sensor y los enviará al servidor utilizando la biblioteca SocketIoClient.

#### 2.2 Configuración del Servidor

**Desarrollo del Servidor en Node.js:**

- Instalación de Node.js y configuración del entorno.
- Creación de un servidor utilizando Express y socket.io para manejar conexiones WebSocket.
- Implementación de la lógica para recibir datos del ESP32 y enviar comandos de respuesta si fuera necesario.

**Código del Servidor:**

```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

io.on('connection', (socket) => {
    console.log('Nuevo dispositivo conectado.');

    socket.on('data', (data) => {
        console.log(`Datos recibidos del ESP32: ${data}`);
        // Procesamiento o respuesta
    });

    socket.on('disconnect', () => {
        console.log('Dispositivo desconectado.');
    });
});

server.listen(3000, () => {
    console.log('Servidor escuchando en puerto 3000');
});
```

### Capítulo 3. Resultados

#### Documentación de Resultados

- **Visualización de Datos:** Muestre cómo los datos enviados por el ESP32 se visualizan en tiempo real en el servidor.
- **Análisis de Conectividad:** Discuta la estabilidad y la eficacia de la conexión WebSocket entre el ESP32 y el servidor.

### Capítulo 4. Conclusiones

#### Reflexiones Sobre la Práctica

- Evaluar la eficacia de la comunicación en tiempo real mediante WebSocket y socket.io.
- Discutir los desafíos encontrados durante la integración del ESP32 con el servidor Node.js y cómo se superaron.

### Capítulo 5. Anexos

#### Materiales Complementarios

- **Códigos Completos:** Se incluyen los códigos fuente completos y bien documentados tanto para el servidor Node.js como para el ESP32.
- **Documentación de la Bitácora de Prácticas:** Se sugiere registrar todas las observaciones y ajustes realizados durante la práctica.
