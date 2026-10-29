# Epiciclos de Fourier

Fronteras de países dibujadas solo con círculos que giran. Las series de Fourier se animan con [p5.js](https://p5js.org/).

**Demo en vivo:** https://curve-approximation-indol.vercel.app/

La animación no muestra una imagen del mapa. El borde de cada país se convierte en un recorrido cerrado, la transformada de Fourier lo descompone en círculos (epiciclos) y estos, encadenados y girando cada uno a su ritmo, vuelven a trazar la frontera con la precisión que elijas.

## Características

- **7 países:** Colombia, México, España, Estados Unidos, Argentina, Japón e Italia.
- **Control de precisión:** de 16 círculos hasta los 4.096, con error cero.
- **Métricas en vivo:** número de círculos activos (N) y error medio respecto a la frontera real.
- **Islas:** cada isla grande tiene su propia cadena de epiciclos y todas cierran la vuelta al mismo tiempo.
- **Controles:** velocidad (0,1× a 3×), pausar o reanudar, reiniciar, mostrar u ocultar los epiciclos y la frontera original.
- **Explicación interactiva** ("¿Qué es esto?"): un recorrido paso a paso con demos animadas, desde un solo círculo hasta la matemática completa.
- Todo se calcula en el navegador. No hay backend.

### Modos de precisión

| Modo    | Círculos      | Resultado                     |
|---------|---------------|-------------------------------|
| Dibujo  | 16            | Forma muy aproximada          |
| Boceto  | 64 a 256      | Contorno reconocible          |
| Detalle | 512 a 1.024   | Alta precisión                |
| 0 Error | 2.048 a todos | Casi perfecto; exacto al 100 % |

### Atajos de teclado

| Tecla   | Acción             |
|---------|--------------------|
| `Space` | Pausar / reanudar  |
| `R`     | Reiniciar el dibujo |

## Cómo funciona

1. **Contornos:** se lee el GeoJSON del país y se toma el borde exterior de cada polígono. Los huecos interiores, como el Vaticano o San Marino, se ignoran, y las islas de menos del 0,65 % del territorio principal se descartan.
2. **Proyección:** las coordenadas se proyectan con la **proyección azimutal equivalente de Lambert**, centrada en el país.
3. **Muestreo:** se reparten **4.096 puntos** equidistantes entre todos los contornos, en proporción a su longitud, con un mínimo de 64 por isla.
4. **Transformada:** cada punto se trata como un número complejo *z = x + iy* y se calcula la DFT:

   $$c_k = \frac{1}{M}\sum_{n=0}^{M-1} z_n \, e^{-2\pi i k n / M}$$

   Se usa una FFT de base 2 y, para longitudes que no son potencia de 2, el algoritmo de **Bluestein**.
5. **Reconstrucción:** el recorrido se reconstruye como suma de giros, con *t* entre 0 y 1:

   $$z(t) = \sum_k c_k \, e^{2\pi i k t}$$

   Cada término es un círculo de radio |c<sub>k</sub>|, velocidad *k* y ángulo inicial arg(c<sub>k</sub>). Los círculos se ordenan de mayor a menor radio.
6. **Error medio:** promedio de |z<sub>n</sub> − ẑ<sub>n</sub>| dividido por la diagonal del mapa, × 100.

## Estructura del proyecto

```
.
├── index.html          # App completa: HTML, CSS, matemática de Fourier y render con p5.js
└── geojson/            # Fronteras de cada país (Natural Earth 1:50m)
    ├── argentina.geojson
    ├── colombia.geojson
    ├── italy.geojson
    ├── japan.geojson
    ├── mexico.geojson
    ├── spain.geojson
    └── united-states.geojson
```

La lógica matemática (`<script id="fourier-math">`) no toca el DOM, así que puede probarse fuera del navegador.

## Ejecutar en local

No hay dependencias que instalar ni paso de build. Lo único necesario es un servidor estático, porque los GeoJSON se cargan con `fetch` y no funcionan abriendo el archivo con `file://`.

```bash
git clone https://github.com/jcamilo1599/curve-approximation.git
cd curve-approximation

# Cualquiera de estas opciones:
npx serve .
# o
python3 -m http.server 8000
```

Luego abre `http://localhost:8000` (o el puerto que indique el servidor).

> p5.js se carga desde el CDN de jsDelivr (`p5@1.11.11`), así que hace falta conexión a internet.

## Agregar un país

1. Descarga el GeoJSON del país desde [Natural Earth](https://www.naturalearthdata.com/) y guárdalo en `geojson/<slug>.geojson`.
2. Agrega una opción al selector en `index.html`:

   ```html
   <option value="<slug>">Nombre del país</option>
   ```

## Despliegue

El sitio está publicado en **Vercel** como sitio estático: https://curve-approximation-indol.vercel.app/

Para desplegar tu propia copia, importa el repositorio en Vercel (o en cualquier hosting estático, como GitHub Pages o Netlify) sin comando de build y con la raíz del proyecto como directorio de salida.

## Créditos

- **Fronteras:** [Natural Earth](https://www.naturalearthdata.com/), datos de dominio público a escala 1:50 millones.
- **Dibujo:** [p5.js](https://p5js.org/), con licencia LGPL 2.1.
- **Idea:** Joseph Fourier (1768–1830).

Este proyecto no está afiliado a Natural Earth ni a p5.js.
