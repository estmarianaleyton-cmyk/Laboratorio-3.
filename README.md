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

<img width="449" height="168" alt="image" src= https://github.com/estmarianaleyton-cmyk/Laboratorio-2./blob/main/Frecuencia%20de%20corte.png>

La adquisición se realizó mediante la librería SparkFun MAX3010x Pulse and Proximity Sensor Library, configurando el sensor con una tasa de muestreo interna de 400 Hz. El canal infrarrojo (IR) se transmitió por puerto serial a 115200 baudios hacia MATLAB para su procesamiento.
