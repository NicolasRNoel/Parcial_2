# 🪙 Sistema Automatizado de Clasificación de Monedas y Minitanque Autónomo Todoterreno

Este proyecto integra Visión Computacional, Mecatrónica, Robótica Móvil y Telemetría en Tiempo Real para la clasificación, embalaje y transporte de monedas colombianas.

---

## 📐 Arquitectura General del Sistema

El proceso logístico e industrial se divide en 4 etapas principales:

1. **Clasificación por Visión Computacional:** Detección e identificación en tiempo real de las monedas colombianas (tanto de la familia antigua como de la nueva) clasificadas según su valor comercial mediante un modelo de visión artificial basado en **YOLO (You Only Look Once)**.
2. **Sistema de Embalaje en Vasos:** Dosificación de las monedas clasificadas directamente dentro de vasos sobre la banda transportadora, seguido del sellado automático del vaso colocando su respectiva tapa mediante un servomecanismo.
3. **Transporte Autónomo (Minitanque):** Recepción de los vasos cargados en la carrocería tipo balde y desplazamiento a través del circuito hacia la meta. *(Las especificaciones técnicas de tracción, peso y control se detallan en la sección siguiente)*.
4. **Monitoreo y Asistente Expositivo:** Dashboard en tiempo real para visualizar métricas del proceso y un **Chatbot de Voz** cuyo objetivo exclusivo es exponer y explicar la presentación general del proyecto ante los evaluadores o público.

---

## 🚜 Minitanque Todoterreno: Especificaciones Mecánicas y Eléctricas

El vehículo de carga está diseñado para superar terrenos irregulares y pendientes pronunciadas (30°) de forma autónoma.

### 1. Motores Elegidos: N20 con Reductora Metálica
Se descartaron los motores DC convencionales de plástico (motores TT) en favor de los **Motores N20 con caja reductora metálica (6V - 150 a 200 RPM)** por las siguientes razones:
* **Engranajes de Acero/Latón:** Soportan el estrés mecánico continuo al ascender rampas con carga sin riesgo de barrer los dientes del piñón.
* **Alto Torque en Menor Tamaño:** Entregan un torque de hasta **1.8 a 2.2 kg·cm** por motor a un voltaje nominal de 7.4V.
* **Eficiencia Térmica y Eléctrica:** Menor consumo de corriente bajo carga alta en comparación a motores TT retenidos.

---

### 2. Desglose Detallado de Peso y Potencia (Presupuesto de Masa)

Para garantizar la viabilidad en pendientes de 30°, se realizó el cálculo estricto de la masa del sistema:

| Componente / Elemento | Masa Estimada | Notas / Función |
| :--- | :---: | :--- |
| **Chasis Estructural** (PLA/PETG 3D) | 160 g | Estructura principal y carrocería tipo balde |
| **Orugas Triangulares + Sprockets** | 110 g | Sistema de tracción con banda de goma de alto agarre |
| **Baterías (2x Li-Ion 18650 + Portabaterías + BMS)** | 100 g | Fuente de energía a 7.4V nominales (8.4V pico) |
| **2x Motores N20 Metálicos** | 30 g | Motorreductores de tracción |
| **Electrónica Principal** (ESP32 + PCB/Breadboard) | 35 g | Unidad de procesamiento dual-core y control |
| **Driver de Motores** (TB6612FNG) | 10 g | Control PWM puente H eficiente basado en MOSFET |
| **Sensores** (MPU-6050 + Sensor Ultrasonido) | 15 g | Medición de inclinación y detección de muros |
| **Cableado y Tornillería** | 40 g | Ensamblaje mecánico e interconexión |
| **MASA EN VACÍO DEL ROBOT** | **500 g** | **Peso propio del Minitanque** |
| **Carga Útil (Payload)** | **100 g** | **Vaso cargado con monedas colombianas** |
| **MASA TOTAL EN OPERACIÓN ($m_{\text{total}}$)** | **600 g** | **Masa a mover sobre la rampa de 30°** |

---

### 3. Estrategia para Sobrellevar el Peso y la Inclinación (30°)

Para evitar que los 600 g detengan o deslicen el robot en la rampa, se implementaron 4 estrategias combinadas:

1. **Tracción por Oruga Triangular (Triangular Tracks):**
   * **Ángulo de Ataque Elevado:** Permite al frente del robot iniciar el ascenso sin golpear la base de la rampa.
   * **Distribución de Carga:** Reparte los 600 g sobre una mayor área de contacto, evitando deslizar hacia abajo en inclinaciones de 30°.
2. **Alimentación a 7.4V con Driver MOSFET (TB6612FNG):**
   * Las 2 baterías 18650 en serie alimentan los motores N20 a 7.4V. El driver TB6612FNG minimiza la caída de voltaje interna (a diferencia de módulos antiguos como el L298N que pierden hasta 2V en forma de calor), entregando el máximo torque disponible.
3. **Compensación Dinámica con IMU MPU-6050 (I2C):**
   * El sensor MPU-6050 lee continuamente el ángulo de cabeceo (*Pitch*).
   * **En Plano (0°):** El ESP32 opera los motores al 50% - 60% PWM para ahorrar energía.
   * **En Pendiente ($\ge 15^\circ \text{ a } 30^\circ$):** Al detectar la inclinación, el algoritmo incrementa dinámicamente la modulación hasta el 90% - 100% PWM, asegurando la fuerza ascensional requerida para la carga de 100 g.
   * **En Descenso:** Se aplica un freno dinámico mediante PWM inverso para evitar caídas descontroladas.
4. **Evasión de Obstáculos con Sensor Ultrasónico HC-SR04:**
   * **Sensor Seleccionado:** Sensor Ultrasónico HC-SR04.
   * Se eligió el HC-SR04 debido a su cono de detección angular (~15° a 30°), el cual permite identificar los muros y bloques de ladrillo de la pista incluso cuando el minitanque está inclinado sobre la rampa. Proporciona lecturas precisas a distancias de 2 cm a 400 cm con baja carga de procesamiento en el ESP32 y un consumo de corriente mínimo que cuida la autonomía de las baterías.
---

## 🖥️ Asistente de Voz

* **Chatbot Narrador del Proyecto (Explicación por Voz):** Módulo interactivo con síntesis de voz que actúa como guía explicativo del sistema. Su función es **describir la arquitectura, objetivos, funcionamiento técnico y fases del proyecto**

---

## 🛠️ Tecnologías Utilizadas

* **Lenguajes:** Python (Visión Computacional / Dashboard), C++ (Firmware ESP32).
* **Librerías / Frameworks:** OpenCV, Adafruit MPU6050, WebSockets / MQTT.
* **Hardware Principal:** ESP32, IMU MPU-6050, Motores N20 Metálicos, Driver TB6612FNG, Orugas Triangulares, Baterías Li-Ion 18650 (7.4V).

---

## 📌 Simualcion completa del proyecto final
file:///D:/Documents/Proyecto%20predeterminado/clasificador_monedas_3d.html
