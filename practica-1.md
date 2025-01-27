### Práctica 1: Comunicación Serial: PC → Dispositivo

### **Objetivo:**
Introducir a los estudiantes al concepto y la práctica de la comunicación serial entre una computadora y un microcontrolador (Arduino), enseñando los fundamentos del manejo de señales digitales y el intercambio de datos a través de protocolos de comunicación establecidos.

---

#### Capítulo 1. Fundamentos teóricos

#### 1. Comunicación Serial: 
La comunicación serial es un método esencial en la transferencia de datos en muchos sistemas electrónicos y computacionales, utilizado ampliamente tanto en comunicaciones industriales como personales. La información se envía secuencialmente, bit a bit, a través de un único canal de comunicación, lo que simplifica el diseño del sistema y reduce los costos.

**Trama de Datos Serial:**
La trama de datos serial es el formato fundamental en el que se organiza la información para su transmisión, compuesta por varios elementos críticos que facilitan la detección y corrección de errores, así como la sincronización entre el emisor y el receptor.

- **Bit de Inicio:** Un bit único que señala el comienzo de una nueva trama de datos, usualmente establecido en '0'.
- **Bits de Datos:** Usualmente entre 5 y 9 bits que representan el dato o carácter transmitido.
- **Bit de Paridad (Opcional):** Proporciona un método simple de verificación de errores sumando paridad par o impar.
- **Bits de Parada:** Uno o dos bits que indican el fin de la trama de datos, típicamente en '1'.

**Trama Serial:**
```
 +---+---+---+---+---+---+---+---+---+---+
 | S | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | P | E |
 +---+---+---+---+---+---+---+---+---+---+---+
   |   |                               |   |   |
Inicio Datos                        Paridad Parada
```
**Tabla de Configuración Común de Tramas:**
| Configuración | Bits de Datos | Bit de Paridad | Bits de Parada | Ejemplo de Uso         |
|---------------|---------------|----------------|----------------|------------------------|
| 7E1           | 7             | Par            | 1              | Comunicación antigua   |
| 8N1           | 8             | Ninguno        | 1              | Uso estándar en PCs    |
| 8E1           | 8             | Par            | 1              | Transmisión confiable  |
| 8O1           | 8             | Impar          | 1              | Detección de errores   |

#### 2. Baudios:
El término "baudios" se refiere a la velocidad a la que los datos son transmitidos o recibidos en la comunicación serial. Un baudio representa la transmisión de un símbolo por segundo, que puede consistir en varios bits, dependiendo del esquema de codificación utilizado.

**Importancia de la Velocidad de Baudios:**
- **Sincronización:** Asegura que el emisor y el receptor operen a la misma velocidad para evitar errores de transmisión.
- **Eficiencia:** Velocidades de baudios más altas permiten una transmisión de datos más rápida, pero pueden requerir hardware más sofisticado y susceptibilidad a errores en entornos ruidosos.

#### 3. Protocolos de Codificación:
Los protocolos de codificación definen cómo los caracteres y símbolos son convertidos en señales que pueden ser transmitidas a través de medios de comunicación serial.

- **ASCII (American Standard Code for Information Interchange):** Codificación estándar utilizada para la representación de textos en computadoras y otros dispositivos de comunicación. Cada carácter se representa por un número binario único.
  
- **UTF-8/UTF-16:** Estos son métodos de codificación más avanzados que permiten la representación de caracteres de casi todos los alfabetos y símbolos del mundo, extendiendo la capacidad del modelo ASCII tradicional. UTF-8 es particularmente popular en la web debido a su eficiencia en el manejo de caracteres estándar del inglés mientras que proporciona la capacidad de representar caracteres de otros idiomas.

**Tabla de Codificaciones Comunes:**
| Codificación | Bits por Carácter | Uso Común                             |
|--------------|-------------------|---------------------------------------|
| ASCII        | 7 o 8             | Texto en inglés y símbolos básicos    |
| UTF-8        | 8 a 32            | Texto en web y soporte internacional  |
| UTF-16       | 16 o más          | Documentos con requerimientos complejos de idioma |

Estos elementos de la comunicación serial, desde la estructura de la trama de datos hasta los protocolos de codificación, son fundamentales para el diseño y la implementación de sistemas de comunicación, permitiendo a los dispositivos de diferentes orígenes y capacidades comunicarse de manera efectiva.

**Conceptos Utilizados en el Desarrollo de Software:**
1. **Carácter vs. Cadena:**  
   - Un carácter es una unidad simple que puede ser leída más rápidamente por el microcontrolador en comparación con una cadena, que es un conjunto de caracteres y necesita más tiempo para ser procesada.

2. **Salidas Digitales:**  
   - Referencia a los pines en un Arduino configurados para emitir un alto (5V) o bajo (0V) para controlar dispositivos como LEDs.

3. **Implementación del Protocolo Serial en Arduino:**  
   - Utiliza buffers de memoria para almacenar temporalmente los caracteres recibidos hasta que estén listos para ser procesados.

---

#### Capítulo 2. Desarrollo

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

---

#### Capítulo 3. Resultados
- Documenta con capturas de pantalla, imágenes y detalles de cada caso:
   - Encendido y apagado de LEDs.
   - Comportamientos con entradas distintas a "ON" y "OFF".
   - Cambios observados al usar un carácter en lugar de una cadena.
   - Efectos de eliminar o modificar el `delay()`.

---

#### Capítulo 4. Conclusiones
- Reflexiona sobre la eficacia de la comunicación serial, la codificación de caracteres y la respuesta del hardware a diferentes tipos de entradas.

---

#### Anexos
- Incluye los códigos completos en Python y Arduino, así como la modificación para el control con un solo carácter.

---

**Agrega secciones y extiende cada sección, según cosideres necesario.**
**Agrega el reporte a la bitácora de prácticas.**
