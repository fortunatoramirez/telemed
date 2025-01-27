### Práctica 1: Comunicación Serial PC -> Dispositivo → Arduino

**Objetivo:**  
Introducir a los estudiantes al concepto y la práctica de la comunicación serial entre una computadora y un microcontrolador (Arduino), enseñando los fundamentos del manejo de señales digitales y el intercambio de datos a través de protocolos de comunicación establecidos.

#### Fundamentos de la Comunicación Serial

**Protocolos y Formatos de Comunicación:**
1. **Comunicación Serial:**  
   - **Trama de Datos:** Es la estructura básica de la información transmitida que incluye bits de inicio, datos, paridad y stop. Cada parte tiene su función específica para asegurar una transmisión correcta y la integridad de los datos.
   - **Diagrama de una Trama:**  
     ![Diagrama de Trama Serial](URL_a_un_diagrama_aquí)  

2. **Baudios:**  
   - Define la velocidad de transmisión en bits por segundo (bps). Un baudio representa un símbolo por segundo, que puede ser más de un bit de información.

3. **Protocolos de Codificación:**  
   - **ASCII:** Utilizado comúnmente para la codificación de caracteres, donde cada carácter tiene un equivalente numérico binario.
   - **UTF-8/UTF-16:** Codificaciones que permiten representar caracteres más allá de los estándares ingleses, incluyendo caracteres internacionales.

**Conceptos Utilizados en el Desarrollo de Software:**
1. **Carácter vs. Cadena:**  
   - Un carácter es una unidad simple que puede ser leída más rápidamente por el microcontrolador en comparación con una cadena, que es un conjunto de caracteres y necesita más tiempo para ser procesada.

2. **Salidas Digitales:**  
   - Referencia a los pines en un Arduino configurados para emitir un alto (5V) o bajo (0V) para controlar dispositivos como LEDs.

3. **Implementación del Protocolo Serial en Arduino:**  
   - Utiliza buffers de memoria para almacenar temporalmente los caracteres recibidos hasta que estén listos para ser procesados.

#### Desarrollo

**Código de la Aplicación en Python en la PC:**
1. Abre Visual Studio Code y crea un nuevo archivo llamado `control_leds.py`.
2. **Bloques de Código:**
   ```python
   import serial
   print("Conectando al Arduino...")
   arduino = serial.Serial('COM6', 9600, timeout = 3.0)
   print("Conectado al Arduino.")
   while True:
       comando = input('Introduzca el comando: ')
       arduino.write(comando.encode())
   ```
   - `serial.Serial('COM6', 9600)`: Inicializa la conexión serial con Arduino.
   - `input()`: Captura comandos del usuario.
   - `comando.encode()`: Convierte el string a bytes para la transmisión serial.

**Código para el Microcontrolador (Arduino):**
1. Abre Arduino IDE y configura el entorno para tu placa Arduino.
2. **Bloques de Código:**
   ```arduino
   char car;
   String cad;

   void setup() {
       Serial.begin(9600);
       pinMode(LED_BUILTIN, OUTPUT);
       pinMode(5, OUTPUT);
   }

   void loop() {
       if (Serial.available()) {
           cad = Serial.readStringUntil('\n');
           if (cad == "ON") {
               digitalWrite(LED_BUILTIN, HIGH);
               digitalWrite(5, HIGH);
           } else if (cad == "OFF") {
               digitalWrite(LED_BUILTIN, LOW);
               digitalWrite(5, LOW);
           }
           Serial.print(cad);
       }
       delay(200);
   }
   ```
   - `Serial.readStringUntil('\n')`: Lee datos hasta que encuentra un salto de línea.
   - `digitalWrite()`: Activa o desactiva un LED.

**Modificación: Cambio de Cadena por Carácter**
- Modifica el código para leer un solo carácter en lugar de una cadena. Observa la diferencia en la velocidad de lectura y procesamiento.
   ```arduino
   car = Serial.read();
   if (car == 'O') {  // Asumiendo 'O' para "ON"
       digitalWrite(LED_BUILTIN, HIGH);
       digitalWrite(5, HIGH);
   } else if (car == 'F') {  // Asumiendo 'F' para "OFF"
       digitalWrite(LED_BUILTIN, LOW);
       digitalWrite(5, LOW);
   }
   ```

#### Resultados
- Documenta con capturas de pantalla, imágenes y detalles de cada caso:
   - Encendido y apagado de LEDs.
   - Comportamientos con entradas distintas a "ON" y "OFF".
   - Cambios observados al usar un carácter en lugar de una cadena.
   - Efectos de eliminar o modificar el `delay()`.

#### Conclusiones
- Reflexiona sobre la eficacia de la comunicación serial, la codificación de caracteres y la respuesta del hardware a diferentes tipos de entradas.

#### Anexos
- Incluye los códigos completos en Python y Arduino, así como la modificación para el control con un solo carácter.

---
**Agrega secciones y extiende cada sección, según cosideres necesario.**
**Agrega el reporte a la bitácora de prácticas.**
