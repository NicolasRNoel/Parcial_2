# Clasificador de Monedas Colombianas — Simulación 3D

Simulación 3D autocontenida de una línea de clasificación de monedas trickleanas:
tolva → estación de visión artificial → banda transportadora → servomotor →
carro de carga con orugas que planifica su ruta hasta la meta esquivando obstáculos.

No necesita instalación, servidor ni dependencias locales. **Three.js se carga
desde CDN**, así que hace falta conexión a internet la primera vez.

## Cómo abrirlo

Doble clic en `clasificador_monedas_3d.html`. Se abre en el navegador y ya funciona.

## Qué hace

| Etapa | Detalle |
|---|---|
| **Tolva** | Embudo de dos tramos con compuerta giratoria que cicla, como el contador real |
| **Clasificar** | Haz que barre la cámara de visión. Mide Ø, espesor, tono y núcleo, y decide por **distancia al centroide más cercano** (*nearest-centroid*) |
| **Banda** | Superficie con textura que se desplaza, rodillos que giran y Accum-Feed: las monedas se encolan solas si van rápidas |
| **Servo** | Cuerpo, bocina y biela visibles. La pala lanza la moneda hacia el carro |
| **Carro** | Ruedas con orugas de tanque (eslabones instanciados sobre un circuito tipo estadio), plataforma con 5 contenedores y una carga de 18 monedas |

Cuando el carro se llena, **planifica su ruta con A\***, esquiva los seis obstáculos,
descarga en la diana de entrega y vuelve a la bahía de carga.

## Datos

Las medidas, masas, probabilidades y la capacidad salen de la configuración de un
sistema real de classification de monedas (denominaciones de 50, 100, 200, 500 y
1000 COP). La banda mide 9,4 unidades, equivalentes a 0,94 m, igual que la real.

**Escala:** 1 unidad ≈ 10 cm. Las monedas van ampliadas ×3,2 para que se lean en
cámara; con la proporción verdadera serían diminutas a esta distancia.

## Clasificación: sensibilidad

El par **$500 / $1.000** es el caso difícil: están a solo 0,4 mm de diámetro y
0,4 mm de grosor de separación, el margen más pequeño de todo el juego.

| Ruido en Ø | Ruido en grosor | Acierto | $500 se va a $1.000 |
|---|---|---|---|
| 0,12 mm | 0,055 mm | **100 %** | 0 % |
| 0,12 mm | 0,15 mm | 97,2 % | 6,4 % |
| 0,12 mm | 0,30 mm | 90,7 % | **22,2 %** |

El diámetro aguanta hasta ±0,4 mm. El grosor se rompe primero, y se rompe justo
en ese par. Lo que lo salva es la detección del **núcleo bimetal**: con ella el
acierto se mantiene al 100 % incluso con ruido alto.

> Si tu cámara real acaba en monocromo por iluminación, la detección del núcleo
> es la pieza que no se puede quitar.

## Consola de depuración

Abre la consola del navegador (F12) y usa el enganche `maquina`:

```js
maquina.ver()        // estado instantáneo
maquina.pausar(true) // congelar
maquina.verPlano(3)  // saltar a un plano de cámara (0-7)
maquina.avanzar(120) // avanzar 2 minutos de golpe
```

## Notas de implementación

- **A\*** sobre rejilla de 8 vecinos con las huellas de obstáculos infladas 1,15
  (el radio de giro del carro), prohibiendo cortar esquinas: cortar la diagonal
  entre dos celdas libres arrastraría al carro por la esquina del obstáculo.
- **Conducción de orugas:** el carro no interpola posición. Apunta al waypoint y
  decide entre pivotar sobre su eje o avanzar, así que las dos orugas van a
  distinta velocidad — cuando gira, una avanza y la otra retrocede.
- Si la meta cae dentro de un obstáculo, el planificador busca la celda libre más
  cercana en vez de rendirse.

---

Renderizado con [Three.js](https://threejs.org/) r128 y tone mapping ACES.