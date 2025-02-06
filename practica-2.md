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
  | Característica       | Detalle                       |
  |----------------------|-------------------------------|
  | Resolución           | 10 bits                       |
  | Número de Canales    | 6 (A0 - A5)                   |
  | Voltaje de Referencia| 5V (por defecto)              |
  | Rango de Entrada     | 0V - 5V                       |
  | Frecuencia de Muestreo Máxima | 10,000 muestras/segundo |


