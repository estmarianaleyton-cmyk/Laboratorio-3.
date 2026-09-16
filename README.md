# Laboratorio-3

**Universidad Militar Nueva Granada**

**Asignatura:** Instrumentación biomedica y biosensores

**Estudiantes:** Dubrasca Martínez, Mariana Leyton, Joshara Valentina Palacios

**Fecha:** 15 de septiembre del 2026

**Título de la práctica:** Cálculo ambulatorio del índice pletismográfico quirúrgico (PPG)

# **Introducción**

El índice pletismográfico quirúrgico (SPI) es una métrica empleada para proporcionar una aproximación sobre el balance existente entre nocicepción y analgesia durante procedimientos llevados a cabo bajo anestesia general. Este índice se calcula basándose en características de la onda fotopletismográfica (PPG), una señal que representa las variaciones en el volumen sanguíneo periférico provocadas por cada latido cardíaco. Como los estímulos nociceptivos pueden causar alteraciones sobre la actividad del sistema nervioso autónomo, y en particular sobre la respuesta vascular periférica, el análisis de esta señal nos proporcionará una información indirecta sobre esta respuesta. La escala del SPI varía entre 0 y 100 puntos, siendo valores más altos indicativos de mayor respuesta nociceptiva. 

En esta práctica se propone el diseño e implementación de un sistema ambulatorio de adquisición y procesado de señal PPG mediante el sensor óptico MAX30-102 y una ESP32. Mediante MATLAB, se procesará la señal de forma digital para calcular el SPI mencionado anteriormente. A partir de la señal procesada se implementarán técnicas de filtrado y de búsqueda de máximos y mínimos necesarias para la extracción de las características del pulso. Por último, se estudiará el comportamiento del índice en condiciones de reposo, durante la realización del Cold Pressor Test (CPT) y durante la fase de recuperación para visualizar las variaciones fisiológicas ocasionadas por el estímulo analizar las capacidades y limitaciones del sistema implementado. De este modo se relacionan los conceptos de instrumentación biomédica y procesado de señales con una aplicación médica que permite monitorizar la respuesta nociceptiva. 

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

## **Cálculo del índice pletismográfico quirúrgico (SPI)**

Desde cada pulso detectado se calcularon dos variables descriptivas de la señal: el Intervalo entre pulsos cardíacos (Heartbeat interval o HBI) y la Amplitud de la onda pletismográfica (PPGA). El HBI fue calculado como la diferencia temporal entre dos picos sucesivos de la señal PPG. Por su parte, la PPGA fue calculada como la diferencia entre el máximo de cada pulso y el mínimo del mismo intervalo. El algoritmo impone un intervalo de HBI entre 500 y 1200 ms para excluir valores no representativos desde el punto de vista fisiológico y realiza un procesamiento de normalización sobre ambas variables. 

El cálculo de la normalización se realizó acotando primero el HBI dentro del intervalo aceptado y llevándolo a una escala entre 0 y 100. De forma similar se normaliza la PPGA usando valores máximos y mínimos (PPGA_min y PPGA_max) que son automáticamente determinados durante los primeros 20 segundos del período de reposo de cada sujeto. De esta manera se tienen en cuenta las variaciones individuales en la amplitud de la señal debidas, por ejemplo, a la perfusión o a la presión con que el dedo hace contacto con el sensor. 

El valor del SPI se calculó por latido utilizando la siguiente expresión implementada en MATLAB: 

SPI = 100 − ( 0.7 ⋅ PPGAnorm + 0.3 ⋅ HBInorm )

donde PPGAnorm corresponde a la amplitud de pulso normalizada y HBInorm al intervalo entre pulsos normalizado. El valor obtenido se mostró en tiempo real junto con el HBI, la frecuencia cardíaca y la PPGA, y posteriormente se almacenó para analizar su evolución durante las diferentes etapas del experimento.

## **Aplicación del Cold Pressor Test (CPT)**

Para la aplicación de un estímulo nociceptivo, se usó la maniobra Cold Pressor Test (CPT) tal como se describe en la guía de laboratorio. La captura duró 120 segundos, donde los 0-40 segundos fue considerado periodo de reposo inicial (Los primeros 20s de calibración), los 40-80 segundos fue cuando se aplicó CPT y los 80-120 segundos fue el periodo de recuperación. 

Los primeros 40 segundos consistieron en que el voluntario se mantuvo en estado de reposo mientras se tomaba como referencia la señal PPG inicial o base. Una vez que el cronometro llegó a los 40 segundos el voluntario colocó su mano en el agua fria por 40 segundos. Cuando el cronometro marcó 80 segundos se mostró otro mensaje indicando levantar la mano y volver a las condiciones de base para tener el periodo de recuperación. El SPI se calculó durante cada latido para las tres etapas y luego se graficó utilizando tiempo como eje x para apreciar mejor los cambios ocasionados por el estímulo. 

Una vez terminada la adquisición, los valores obtenidos del SPI fueron separados por condiciones automáticamente y se promedió el SPI de reposo inicial, el SPI durante CPT y el SPI del periodo de recuperación. Se utilizó un promedio movil de 5 latidos para obtener una curva mas "suave" permitiendo apreciar mejor la tendencia del índice sin estar afectados por el cambio brusco de un solo latido. 

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

<img width="769" height="988" alt="image" src= https://github.com/estmarianaleyton-cmyk/Laboratorio-3./blob/main/Segundo%20sujeto.Grafica.png>

Los valores obtenidos fueron: 

| Estado | SPI Promedio |
| --- | --- |
| Reposos inicial (0-40 s) | 6.6 |
| Cold Pressor Test (40-80 s) | 42.5 |
| Recuperación | 32.8 |


# **Análisis de resultados**

## **Comparación con los valores de SPI observados en cirugía**

Dentro de la guia de laboratorio se señala que el rango objetivo para una analgesia intraoperatoria adecuada es SPI 20-50, y que deben evitarse incrementos mayores a 10 unidades. Al compararlo con los dos sujetos se pueden observar varias cosas.

- **Sujeto 1:** las tres fases se mantuvieron dentro o muy cerca del rango clínico de "analgesia adecuada", pero sin diferencia real entre reposo y CPT, con una variación de apenas 0.3 unidades. Si este fuera un paciente real en cirugía, este resultado se interpretaría como "sin respuesta nociceptiva significativa al estímulo", lo cual sabemos que no es cierto, el sujeto sí recibió un estímulo doloroso real. Esto revela un falso negativo: la señal era demasiado ruidosa para que el cambio fisiológico real se reflejara en el promedio.
  
- **Sujeto 2:** la línea base en reposo está muy por debajo del rango clínico típico, lo cual es razonable, ya que ese rango de referencia se definió para pacientes anestesiados, mientras que el suejto estaba despierto y relajado, con un tono simpático basal distinto. Durante el CPT, el SPI promedio subió a 42.5, y los picos individuales alcanzaron valores de 85-100, un incremento momentáneo de más de 80 puntos sobre la línea base, muy por encima del umbral de alerta. Si se tratara de un paciente quirúrgico real, esto se interpretaría como un evento de estrés nociceptivo severo que ameritaría refuerzo inmediato de analgesia.

Esto nos demuestra que el sistema desarrollado puede reproducir cualitativamente el comportamiento esperado del SPI clínico (Sujeto 2), pero es altamente dependiente de la calidad de la señal. Con contacto inestable, el sistema puede fallar en detectar un evento nociceptivo real (Sujeto 1), algo que un monitor clínico validado y con procesamiento de señal más robusto minimizaría.

## **Alcance y limitaciones del sistema para cuantificar el dolor percibido**

- **Sensibilidad extrema a artefactos de movimiento:** La diferencia más marcada entre ambos sujetos no fue la respuesta fisiológica en sí, sino la estabilidad del contacto del dedo con el sensor. En ambos sujetos, el pico más alto de todo el registro ocurrió justo en la transición de CPT a la recuperación (t≈80-88s), el momento en que el sujeto retira la mano del agua y se reacomoda, esto sugiere que ese pico refleja en buena parte un artefacto de movimiento, no una respuesta nociceptiva pura.

- **Variabilidad interindividual del nivel basal:** El SPI de reposo del Sujeto 1 es más de 7 veces el del Sujeto 2. Esta diferencia probablemente combina tono simpático basal distinto, como el nivel de relajación y la ansiedad anticipatoria, con diferencias en la calidad de la señal evidenciando que un umbral fijo (como el 20-50 clínico) no es directamente trasladable a un sistema de bajo costo sin calibración clínica validada.

- **El SPI mide balance autonómico, no dolor directamente:** Como toda la literatura de SPI reconoce, el índice se basa en la interacción simpático-parasimpática (frecuencia cardíaca y vasoconstricción periférica), que también responde a ansiedad, temperatura, o esfuerzo cognitivo — no exclusivamente a nocicepción. En un sujeto consciente, esos factores confusores son mucho más difíciles de aislar que en un paciente anestesiado e inmóvil, que es el contexto para el que el SPI fue diseñado originalmente.

- **Diferencias técnicas frente al monitor clínico validado:** Este sistema usa autocalibración simple por sujeto (primeros 20s), detección de picos por software sin validación cruzada con ECG, y un solo punto de medición óptica, mientras que el monitor comercial (GE Healthcare) incluye años de calibración clínica, rechazo de latidos ectópicos, y validación en miles de pacientes. Por tanto, los valores absolutos del sistema deben interpretarse con cautela; su mayor utilidad aquí es cualitativa, no como una medida clínicamente equivalente.

# **Discusión**
1. ¿Cómo se relacionan las variaciones del volumen sanguíneo periférico con el balance autonómico?

Las variaciones del volumen sanguíneo periférico están relacionadas principalmente con la actividad del sistema nervioso autónomo, debido a que el sistema simpático regula el tono de los vasos sanguíneos periféricos. Ante un estímulo doloroso o estresante, aumenta la actividad simpática, produciendo vasoconstricción periférica. Como consecuencia, disminuye temporalmente el volumen de sangre en los tejidos periféricos y también puede disminuir la amplitud de la onda fotopletismográfica (PPGA).

Por el contrario, cuando predomina una menor activación simpática, se favorece la vasodilatación y aumenta el flujo sanguíneo periférico, lo que puede producir una mayor amplitud de la señal PPG. Por esta razón, la fotopletismografía permite obtener información indirecta sobre los cambios en el tono vascular y en el balance autonómico.

2. ¿Cómo se compara el SPI con otros índices comúnmente empleados en cirugía, como el ANI y el índice de perfusión?

- El SPI combina dos variables: el intervalo entre latidos (HBI) y la amplitud de la onda pletismográfica (PPGA). Su valor se encuentra entre 0 y 100 y valores más altos representan una mayor respuesta de estrés/nocicepción. En la literatura se describe la forma general del cálculo como una combinación ponderada de las versiones normalizadas de HBI y PPGA.
- El ANI en cambio se obtiene principalmente a partir del ECG y la variabilidad de la frecuencia cardíaca (HRV), especialmente de componentes asociados con la actividad parasimpática. Su escala también va de 0 a 100 pero la interpretación es inversa respecto al SPI: valores bajos de ANI se asocian con menor actividad parasimpática y mayor respuesta nociceptiva mientras que valores altos indican mayor predominio parasimpático.
- El índice de perfusión (PI) es una medida de la fuerza relativa de la señal pulsátil respecto a la componente no pulsátil de la señal de un oxímetro. Se utiliza principalmente como indicador de la perfusión periférica por lo que no constituye por sí mismo un índice específico de nocicepción. 
- Por lo tanto el SPI tiene como ventaja que combina información cardíaca y vascular relacionada con la respuesta autonómica el ANI se concentra principalmente en la regulación autonómica cardíaca mientras que el PI se enfoca en la perfusión periférica. Ninguno debe interpretarse como una medición directa y absoluta del dolor.

# **Conclusión**

La practica permitió establecer un sistema ambulatorio para la adquisición de una señal fotopletismográfica utilizando el sensor MAX30102 y su posterior procesamiento en MATLAB para la estimación del índice pletismográfico quirúrgico (SPI). Los resultados mostraron que existe variación en el comportamiento del índice entre la condición basal, prueba del Cold Pressor Test y condición de recuperación. Las diferencias presentadas por los resultados de los sujetos analizados pudieron deberse a variaciones de la respuesta fisiológica de cada sujeto y la calidad de la señal captada, esto debido a que se presentó un aumento en el SPI durante el CPT en el Sujeto 2 y fue menor en el Sujeto 1, lo cual afectó la estimación del índice. Con estos resultados podemos asociar la variación en la señal PPG con la respuesta autonómica frente a un estímulo nociceptivo y además evidenciar la importancia del procesamiento digital para la obtención de indicadores fisiológicos a partir de una señal biomédica; no obstante, los artefactos producidos por movimiento, contacto con el sensor y otros fenómenos fisiológicos deberán ser eliminados o tomados en cuenta para una correcta interpretación. Para un siguiente paso se debería mejorar la robustez del sistema para luego realizar una validación frente a métodos clínicos para establecer si el sistema es capaz de medir o estimar la respuesta nociceptiva. 

# **Referencias**
- Bonhomme, V., Uutela, K., Hans, G., Maquoi, I., Born, J., Brichant, J., Lamy, M., & Hans, P. (2010). Comparison of the Surgical Pleth IndexTM with haemodynamic variables to assess nociception–anti-nociception balance during general anaesthesia. British Journal of Anaesthesia, 106(1), 101–111. https://doi.org/10.1093/bja/aeq291
- Oh, S. K., Won, Y. J., & Lim, B. G. (2023). Surgical pleth index monitoring in perioperative pain management: usefulness and limitations. Korean Journal Of Anesthesiology, 77(1), 31-45. https://doi.org/10.4097/kja.23158
