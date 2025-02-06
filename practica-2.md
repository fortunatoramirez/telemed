### Práctica 2: Lecturas Continuas mediante Comunicación Serial: PC ↔ Dispositivo

#### Objetivo:
Introducir a los estudiantes a la captura de datos en tiempo real desde un dispositivo conectado a la computadora, utilizando la comunicación serial para leer valores de una señal analógica.

---

### Capítulo 1. Fundamentos Teóricos

#### 1. Conversión Analógica-Digital (ADC)

La conversión analógica-digital es un proceso que permite a los microcontroladores leer señales analógicas, como las provenientes de sensores, y convertirlas en valores digitales que pueden ser procesados y utilizados por software. Este proceso es fundamental para aplicaciones en robótica, mediciones ambientales y cualquier sistema que requiera interacción con el mundo físico.


1.1 **Principios Básicos de ADC:**
La conversión ADC implica muestrear una señal analógica a intervalos regulares y cuantificar cada muestra en valores digitales. La precisión de esta conversión depende de dos factores principales: la resolución del ADC y la frecuencia de muestreo.

- **Resolución del ADC:** Refiere al número de bits que el ADC utiliza para representar una entrada analógica en forma digital. Por ejemplo, una resolución de 10 bits significa que el ADC puede producir 1024 (2^10) niveles discretos de salida, lo que permite una gradación fina de la señal de entrada.

  ### Tabla de Comparación de Resoluciones de ADC
  
  | Resolución (bits) | Niveles de Salida (2^n) | Rango de Voltaje (0-5V) | Incremento de Voltaje (V) | Ejemplo de Uso                            |
  |-------------------|-------------------------|-------------------------|---------------------------|-------------------------------------------|
  | 1 bit             | 2                       | 0V - 5V                 | 5 V                       | Detección de presencia/no-presencia       |
  | 2 bits            | 4                       | 0V - 5V                 | 1.6667 V                  | Control de intensidad básica (bajo, medio, alto) |
  | 4 bits            | 16                      | 0V - 5V                 | 0.3333 V                  | Control de dispositivos con múltiples estados  |
  | 8 bits            | 256                     | 0V - 5V                 | 0.0196 V                  | Control de iluminación y dispositivos de audio  |
  | 10 bits           | 1024                    | 0V - 5V                 | 0.0049 V                  | Monitoreo ambiental y sensores de precisión     |

  ### Tabla de Mapeo de Niveles de ADC de 2 bits

  | Valor Digital (binario) | Valor Digital (decimal) | Voltaje Correspondiente |
  |-------------------------|-------------------------|-------------------------|
  | 00                      | 0                       | 0 V                     |
  | 01                      | 1                       | 1.6667 V                |
  | 10                      | 2                       | 3.3333 V                |
  | 11                      | 3                       | 5 V                     |


- **Frecuencia de Muestreo:** Debe ser suficientemente alta para capturar todas las variaciones significativas en la señal analógica, de acuerdo con el Teorema de Nyquist, que establece que la frecuencia de muestreo debe ser al menos el doble de la frecuencia máxima presente en la señal.

1.2 **Ejemplo de las características del ADC de un dispositivo**
  - | Característica       | Detalle                       |
    |----------------------|-------------------------------|
    | Resolución           | 10 bits                       |
    | Número de Canales    | 6 (A0 - A5)                   |
    | Voltaje de Referencia| 5V (por defecto)              |
    | Rango de Entrada     | 0V - 5V                       |
    | Frecuencia de Muestreo Máxima | 10,000 muestras/segundo |


### Capítulo 2. Desarrollo

Para realizar la lectura continua de datos desde un dispositivo mediante comunicación serial. Se detallarán tanto la configuración del hardware como la programación del Dispositivo y la aplicación en Python para capturar los datos.

#### 2.1 Configuración del Hardware

**Materiales Necesarios:**
- Dispositivo basado en microcontrolador.
- Potenciómetro o cualquier sensor analógico que se conecte al pin A0.
- Cable USB para conectar el Arduino a la computadora.

**Procedimiento:**
1. **Conexión del Sensor:**
   - Conecte un extremo del potenciómetro al pin A0 del Arduino.
   - Conecte los otros dos extremos a 5V y GND en el Arduino para proporcionar alimentación al potenciómetro.

2. **Verificación:**
   - Asegúrese de que todas las conexiones estén seguras y correctas para evitar lecturas erráticas o daños en el Arduino.

#### 2.2 Código del Microcontrolador (Arduino)

El código para el Arduino se encarga de leer el valor del pin A0 cada vez que recibe un comando 'r' desde la computadora, y luego envía ese valor de vuelta a través del puerto serial.

```c++
char car;
int val;

void setup() {
  Serial.begin(9600);  // Inicia la comunicación serial a 9600 baudios
}

void loop() {
  if (Serial.available() > 0) {
    car = Serial.read();  // Lee el carácter enviado desde la computadora
    if (car == 'r') {
      val = analogRead(A0);  // Lee el valor analógico del pin A0
      Serial.println(val);  // Envía el valor de vuelta a la computadora
    }
  }
}
```

**Explicación del Código:**
- `Serial.begin(9600);` Inicia la comunicación serial a 9600 baudios.
- `Serial.available() > 0` Comprueba si hay datos disponibles en el puerto serial.
- `Serial.read();` Lee el carácter recibido.
- `analogRead(A0);` Realiza la lectura analógica del pin A0.
- `Serial.println(val);` Envía el valor leído de vuelta a la computadora.

#### 2.3 Código de la Aplicación en Python

El script en Python se encargará de enviar el comando 'r' al Arduino, recibir el valor analógico convertido y mostrarlo en la consola. Este proceso se repite indefinidamente cada 0.5 segundos.

```python
import serial
import time

print("Conectando al Arduino...")
arduino = serial.Serial('COM6', 9600, timeout = 3.0)
print("Conectado al Arduino.")

while True:
    arduino.write("r".encode())  // Envía el comando 'r' al Arduino
    val = arduino.readline().strip()  // Lee la línea que contiene el valor enviado por Arduino
    if not val:
        continue
    val = int(val)  // Convierte el valor leído a entero
    print(val)  // Muestra el valor en la consola
    time.sleep(0.5)  // Espera 0.5 segundos antes de enviar otro comando
```

**Explicación del Código:**
- `serial.Serial('COM6', 9600, timeout = 3.0)` Inicia la conexión serial con el Arduino.
- `arduino.write("r".encode())` Envía el carácter 'r' al Arduino, solicitando una lectura.
- `arduino.readline().strip()` Lee la respuesta del Arduino.
- `time.sleep(0.5)` Pausa el bucle durante medio segundo para evitar saturar el puerto serial y dar tiempo al Arduino para procesar la lectura.

### Capítulo 3. Resultados

#### Documentación de Resultados
En esta fase de la práctica, los estudiantes reemplazarán el potenciómetro inicialmente utilizado por una fotoresistencia con una resistencia en serie para medir los cambios en la variación de la luz. Esto permite observar cómo el sensor responde a diferentes condiciones de iluminación en tiempo real. 

**Configuración del Sensor:**
- **Fotoresistencia y Resistencia en Serie:** Conecte la fotoresistencia en serie con una resistencia fija a uno de los pines analógicos del Arduino (A0). El otro extremo de la fotoresistencia se conecta a 5V y el extremo libre de la resistencia a GND. Esto forma un divisor de voltaje, cuyo voltaje medio (en el pin A0) varía con la intensidad de la luz.

**Resultados Esperados:**
- **Gráficos de Variación Lumínica:**
  - Capture y documente gráficos o tablas que muestren cómo varía la señal leída del sensor en respuesta a cambios en la intensidad de la luz. Utilice datos recogidos durante un periodo de tiempo donde la iluminación varía (por ejemplo, al encender y apagar luces o exponiendo el sensor a la luz natural cambiante).
  
- **Análisis de Sensibilidad del Sensor:**
  - Documente cómo el sensor responde a cambios sutiles en la iluminación. ¿Puede detectar cambios menores o solo cambios significativos en la intensidad de la luz?
  
- **Comparación con el Potenciómetro:**
  - Discuta las diferencias en la respuesta del circuito cuando se utiliza el potenciómetro versus la fotoresistencia. Esto puede incluir diferencias en la estabilidad y predictibilidad de las lecturas.

### Capítulo 4. Conclusiones

#### Reflexiones Sobre la Práctica

- **Efectividad de la Comunicación Serial y Captura de Datos:**
  - Evalúe la efectividad de la comunicación serial en la transmisión continua de datos desde la fotoresistencia al ordenador. Reflexione sobre cualquier desafío encontrado, como el ruido o la latencia en las lecturas.

- **Sensibilidad y Aplicabilidad del Sensor de Luz:**
  - Comente sobre la sensibilidad de la fotoresistencia y su aplicabilidad en situaciones reales. ¿Qué limitaciones podrían existir para su uso en aplicaciones prácticas?

- **Comparativa de Sensores:**
  - Proporcione una reflexión final sobre las ventajas y desventajas de usar un potenciómetro frente a una fotoresistencia en términos de aprendizaje y aplicaciones prácticas.

### Capítulo 5. Anexos

#### Materiales Complementarios

- **Códigos Completos:**
  - Proporcione los códigos fuente completos y bien comentados tanto del Arduino como del script en Python. Asegúrese de incluir cualquier cambio realizado para adaptar el sistema a la fotoresistencia.
  
- **Documentación de la Bitácora de Prácticas:**
  - Registre detalladamente todas las sesiones de prácticas, observaciones y ajustes realizados. Esto facilitará la evaluación del proceso y servirá como referencia para futuras prácticas o investigaciones.



