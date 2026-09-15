# Laboratorio-3

**Universidad Militar Nueva Granada**

**Asignatura:** Instrumentación biomedica y biosensores

**Estudiantes:** Dubrasca Martínez, Mariana Leyton, Joshara Valentina Palacios

**Fecha:** 11 de septiembre del 2026

**Título de la práctica:** Cálculo ambulatorio del índice pletismográfico quirúrgico (PPG)

# **Introducción**


# **Metodología**

Para el desarrollo de esta práctica se utilizó como microcontrolador la ESP32 como unidad de adquisición y comunicación, un sensor óptico integrado MAX30102 para la adquisición de la señal fotopletismográfica, un computador con MATLAB para el procesamiento en tiempo real y el cálculo del índice pletismográfico, y el Arduino IDE para la programación del microcontrolador.

El MAX30102 es un sensor integrado que combina LEDs emisores (rojo e infrarrojo), fotodetector, amplificador de transimpedancia, cancelación de luz ambiental y un ADC de 18 bits internos, comunicándose con el microcontrolador mediante el protocolo I2C. La conexión con la ESP32 se realizó de la siguiente forma; el pin VIN del sensor se conecto al pin de 3.3V de la ESP32, el pin de tierra al de tierra de la ESP32, el pin de SDA se conecto al pin D21 de la ESP32 y el pin SCL se conecto al pin D22 de la ESP32.

***Sensor MAX30102***

<img width="329" height="368" alt="image" src= https://github.com/estmarianaleyton-cmyk/Laboratorio-3./blob/main/sensor.png>

La adquisición se realizó mediante la librería SparkFun MAX3010x Pulse and Proximity Sensor Library, configurando el sensor con una tasa de muestreo interna de 400 Hz. El canal infrarrojo (IR) se transmitió por puerto serial a 115200 baudios hacia MATLAB para su procesamiento.

## **Procesamiento digital de la señal**

Dado que la señal IR entregada por el MAX30102 conserva una componente DC de gran magnitud, la cual varía según la perfusión y el contacto del sensor con el tejido, se implementó en MATLAB un filtro pasa-altas digital de primer orden, con frecuencia de corte de 0.7 Hz para aislar la componente pulsátil antes de la detección de picos. Adicionalmente, dado que la polaridad de la señal cruda del MAX30102 es inversa a la convención habitual de la onda de pulso, se aplicó una inversión de signo por software para obtener la forma de onda con el pico sistólico orientado hacia arriba.

## **Algoritmo de detección de máximos y mínimos**

Se desarrolló un algoritmo de detección de picos en MATLAB, con las siguientes características:

- **Confirmación por ventana de tiempo (150 ms):** En este punto un valor solo es aceptado como latido válido si ningún valor posterior lo supera dentro de una venta de 150 ms. Tambien se cuenta con un periodo refractario de 500 ms, en donde se observa si el tiempo desde el ultimo pico confirmado es menor a 500 ms se descarta por completo. Esto permite descartar rebotes de ruido y la  muesca dicótica.

- **Umbral de amplitud adaptativo:** en lugar de utilizar un umbral fijo, se calcula como una fracción del promedio de amplitud de los últimos cinco latidos confirmados, lo cual permite que el algoritmo se ajuste automáticamente a distintos sujestos sin recalibración manual.
  
- **Calibarción inicial automática (primeros 20 s de reposo):** Se determina la amplitud de pulso basal específica del sujeto, a partir de la cual se calculan los límites "PPGA_max" y "PPGA_min" usados en la normalización del SPI.






# **Resultados**

## **Sujeto 1**

<img width="769" height="988" alt="image" src= https://github.com/estmarianaleyton-cmyk/Laboratorio-3./blob/main/Primer%20sujeto.Grafica.png>

Los valores obtenidos fueron: 

| Estado | SPI Promedio |
| --- | --- |
| Reposos inicial (0-40 s) | 47.8 |
| Cold Pressor Test (40-80 s) | 48.1 |
| Recuperación | 41.3 |

## **Sujeto 2**

<img width="769" height="988" alt="image" src= https://github.com/estmarianaleyton-cmyk/Laboratorio-3./blob/main/Primer%20sujeto.Grafica.png>

Los valores obtenidos fueron: 

| Estado | SPI Promedio |
| --- | --- |
| Reposos inicial (0-40 s) | 6.6 |
| Cold Pressor Test (40-80 s) | 42.5 |
| Recuperación | 32.8 |


# **Análisis de resultados**

