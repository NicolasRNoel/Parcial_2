# 🪙 Sistema Automatizado de Clasificación de Monedas y Minitanque Autónomo Todoterreno

Este proyecto integra Visión Computacional, Mecatrónica, Robótica Móvil y Telemetría en Tiempo Real para la clasificación, embalaje y transporte de monedas colombianas (diseño nuevo y antiguo).

---

## 📐 Arquitectura General del Sistema

El proceso logístico e industrial se divide en 4 etapas principales:

1. **Clasificación por Visión Computacional:** Identificación y separación de monedas según su denominación y familia mediante algoritmos de procesamiento de imágenes.
2. **Sistema de Embalaje:** Transferencia de monedas hacia vasos/empaques mediante banda transportadora y servomecanismos para la colocación de la tapa.
3. **Transporte Autónomo (Minitanque):** Recepción de empaques en una carrocería tipo balde y desplazamiento hacia la meta a través de un circuito dinámico con obstáculos y pendientes de hasta $30^\circ$[cite: 1].
4. **Monitoreo y Asistencia:** Dashboard con métricas del proceso en tiempo real y asistente inteligente (Chatbot) que narra los eventos del sistema por voz[cite: 1].

---

## 🚜 Minitanque Todoterreno: Especificaciones Mecánicas y Eléctricas

El vehículo de carga está diseñado para superar terrenos irregulares y pendientes pronunciadas ($30^\circ$) de forma autónoma[cite: 1].

### 1. Motores Elegidos: N20 con Reductora Metálica
Se descartaron los motores DC convencionales de plástico (motores TT) en favor de los **Motores N20 con caja reductora metálica (6V - 150 a 200 RPM)** por las siguientes razones:
* **Engranajes de Acero/Latón:** Soportan el estrés mecánico continuo al ascender rampas con carga sin riesgo de barrer los dientes del piñón.
* **Alto Torque en Menor Tamaño:** Entregan un torque de hasta **$1.8\text{ a }2.2\text{ kg}\cdot\text{cm}$** por motor a un voltaje nominal de 7.4V.
* **Eficiencia Térmica y Eléctrica:** Menor consumo de corriente bajo carga alta en comparación a motores TT retenidos.

---

### 2. Desglose Detallado de Peso y Potencia (Presupuesto de Masa)

Para garantizar la viabilidad en pendientes de $30^\circ$, se realizó el cálculo estricto de la masa del sistema:

| Componente / Elemento | Masa Estimada | Notas / Función |
| :--- | :---: | :--- |
| **Chasis Estructural** (PLA/PETG 3D) | $160\text{ g}$ | Estructura principal y carrocería tipo balde |
| **Orugas Triangulares + Sprockets** | $110\text{ g}$ | Sistema de tracción con banda de goma de alto agarre |
| **Baterías (2x Li-Ion 18650 + Portabaterías + BMS)** | $100\text{ g}$ | Fuente de energía a 7.4V nominales (8.4V pico) |
| **2x Motores N20 Metálicos** | $30\text{ g}$ | Motorreductores de tracción |
| **Electrónica Principal** (ESP32 + PCB/Breadboard) | $35\text{ g}$ | Unidad de procesamiento dual-core y control |
| **Driver de Motores** (TB6612FNG) | $10\text{ g}$ | Control PWM puente H eficiente basado en MOSFET |
| **Sensores** (MPU-6050 + Sensor Ultrasonido) | $15\text{ g}$ | Medición de inclinación y detección de muros[cite: 1] |
| **Cableado y Tornillería** | $40\text{ g}$ | Ensamblaje mecánico e interconexión |
| **MASA EN VACÍO DEL ROBOT** | **$500\text{ g}$** | **Peso propio del Minitanque** |
| **Carga Útil (Payload)** | **$100\text{ g}$** | **Vaso cargado con monedas colombianas** |
| **MASA TOTAL EN OPERACIÓN ($m_{\text{total}}$)** | **$600\text{ g}$** | **Masa a mover sobre la rampa de $30^\circ$** |

---

### 3. Estrategia para Sobrellevar el Peso y la Inclinación ($30^\circ$)

Para evitar que los $600\text{ g}$ detengan o deslicen el robot en la rampa, se implementaron 4 estrategias combinadas:

1. **Tracción por Oruga Triangular (Triangular Tracks):**
   * **Ángulo de Ataque Elevado:** Permite al frente del robot iniciar el ascenso sin golpear la base de la rampa.
   * **Distribución de Carga:** Reparte los $600\text{ g}$ sobre una mayor área de contacto, evitando deslizar hacia abajo en inclinaciones de $30^\circ$.
2. **Alimentación a 7.4V con Driver MOSFET (TB6612FNG):**
   * Las 2 baterías 18650 en serie alimentan los motores N20 a 7.4V. El driver TB6612FNG minimiza la caída de voltaje interna (a diferencia de módulos antiguos como el L298N que pierden hasta 2V en forma de calor), entregando el máximo torque disponible.
3. **Compensación Dinámica con IMU MPU-6050 (I2C):**
   * El sensor MPU-6050 lee continuamente el ángulo de cabeceo (*Pitch*).
   * **En Plano ($0^\circ$):** El ESP32 opera los motores al $50\% - 60\%$ PWM para ahorrar energía.
   * **En Pendiente ($\ge 15^\circ \text{ a } 30^\circ$):** Al detectar la inclinación, el algoritmo incrementa dinámicamente la modulación hasta el $90\% - 100\%$ PWM, asegurando la fuerza ascensional requerida para la carga de $100\text{ g}$.
   * **En Descenso:** Se aplica un freno dinámico mediante PWM inverso para evitar caídas descontroladas.
4. **Evasión de Obstáculos:**
   * Sensores de distancia frontales le permiten al ESP32 rodear los muros y bloques de ladrillo del circuito sin colisionar[cite: 1].

---

## 🖥️ Telemetría y Asistente de Voz

* **Dashboard en Tiempo Real:** Visualización gráfica de la velocidad de la banda, conteo por denominación de moneda y estatus de navegación del minitanque[cite: 1].
* **Chatbot NARRADOR de Voz:** Módulo interactivo que describe verbalmente el estado del proceso, la detección de obstáculos y la llegada exitosa a la meta[cite: 1].

---

## 🛠️ Tecnologías Utilizadas

* **Lenguajes:** Python (Visión Computacional / Dashboard), C++ (Firmware ESP32).
* **Librerías / Frameworks:** OpenCV, Adafruit MPU6050, WebSockets / MQTT.
* **Hardware Principal:** ESP32, IMU MPU-6050, Motores N20 Metálicos, Driver TB6612FNG, Orugas Triangulares, Baterías Li-Ion 18650 (7.4V).

---

## 📌 Diagrama de Flujo del Proceso
