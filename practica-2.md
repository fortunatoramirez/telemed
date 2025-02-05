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
- **Frecuencia de Muestreo:** Debe ser suficientemente alta para capturar todas las variaciones significativas en la señal analógica, de acuerdo con el Teorema de Nyquist, que establece que la frecuencia de muestreo debe ser al menos el doble de la frecuencia máxima presente en la señal.

1.2 **Características del ADC en Arduino:**
El Arduino, como el modelo Uno, utiliza un ADC de 10 bits integrado, lo que significa que puede representar valores analógicos con 1024 niveles diferentes. Los pines analógicos en Arduino (A0 a A5 en el Uno) están conectados directamente a este ADC.

**Diagrama de Conversión ADC:**
```
  +----------------+         +----------+          +-----------------+           +----------------------+
  | Señal Analógica|         |  Sensor  |          | Convertidor ADC |           | Microcontrolador     |
  +----------------+         +----------+          +-----------------+           +----------------------+
         |                      |                           |                               |
         |---(entrada)------>|                         |---(digitaliza)------------>|---(procesa y analiza)--->
         |   Medición física   |  Transduce señal        |   Conversión A/D           |   Salida de datos
         |  (temperatura,     |  analógica a eléctrica  |   10 bits (1024 niveles)   |   (Utilizada en
         |  luz, etc.)        |                         |                            |   aplicaciones o 
         |                    |                         |                            |   almacenada)
         |                    |                         |                            |
  +----------------+         +----------+          +-----------------+           +----------------------+
  |  Mundo Real   |         |  Ej: Termistor,       |  Integrado en Arduino        |  Arduino Uno, etc.   |
  +----------------+         |  Fotocelda, etc.     |                              +----------------------+

```

**Tabla de Especificaciones del ADC de Arduino Uno:**
| Característica       | Detalle                       |
|----------------------|-------------------------------|
| Resolución           | 10 bits                       |
| Número de Canales    | 6 (A0 - A5)                   |
| Voltaje de Referencia| 5V (por defecto)              |
| Rango de Entrada     | 0V - 5V                       |
| Frecuencia de Muestreo Máxima | 10,000 muestras/segundo |

