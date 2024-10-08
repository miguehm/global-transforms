---
title: Implementación en Python
---

Implementar la librería de OpenCV primero requiere instalarse mediante `pip`.

```bash
pip install opencv-python
```

## Mostrar Histograma de una Imagen por Canales de Color

Un histograma muestra la distribución de frecuencias de los niveles de intensidad de una imagen.

1. Importación de librerías:

```python
import cv2
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
```

2. Configuración de Matplotlib:

```python
matplotlib.use('TkAgg')
```

3. Carga de la imagen:

```python
imagen = cv2.imread('avatar.jpg')
```

4. Cálculo de histogramas para cada canal de color:

```python
hist_red = cv2.calcHist([imagen], [2], None, [256], [0, 256])
hist_green = cv2.calcHist([imagen], [1], None, [256], [0, 256])
hist_blue = cv2.calcHist([imagen], [0], None, [256], [0, 256])
```

5. Creación de un array para el eje `x` de los histogramas:

```python
x = np.arange(256)
```

6. Configuración de la figura para la visualización:

```python
plt.figure(figsize=(15, 10))
```

7. Visualización de los canales de color:

```python
plt.subplot(2, 3, 1)
plt.imshow(imagen[:, :, 2], cmap='Reds')
plt.title('Canal Red')
plt.colorbar()

plt.subplot(2, 3, 2)
plt.imshow(imagen[:, :, 1], cmap='Greens')
plt.title('Canal Green')
plt.colorbar()

plt.subplot(2, 3, 3)
plt.imshow(imagen[:, :, 0], cmap='Blues')
plt.title('Canal Blue')
plt.colorbar()
```

8. Visualización de los histogramas de cada canal:

```python
plt.subplot(2, 3, 4)
plt.bar(x, hist_red[:, 0], width=1.0, edgecolor='black')
plt.xlim([0, 256])
plt.title('Canal Red')
plt.xlabel('Intensidad de píxeles')
plt.ylabel('Número de píxeles')

plt.subplot(2, 3, 5)
plt.bar(x, hist_green[:, 0], width=1.0, edgecolor='black')
plt.xlim([0, 256])
plt.title('Canal Green')
plt.xlabel('Intensidad de píxeles')
plt.ylabel('Número de píxeles')

plt.subplot(2, 3, 6)
plt.bar(x, hist_blue[:, 0], width=1.0, edgecolor='black')
plt.xlim([0, 256])
plt.title('Canal Blue')
plt.xlabel('Intensidad de píxeles')
plt.ylabel('Número de píxeles')
```

9. Ajuste del diseño y visualización final:

```python
plt.tight_layout()
plt.show()
```

## Corrección Gamma

1. Importación de librerías:

```python
import cv2
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
from matplotlib.widgets import Slider
```

2. Método para crear un LUT:

```python
def create_lut(gamma: float):
    """
    Create a lut with gamma transformation
    """

    gamma_inv = 1.0/gamma
    table = np.arange(256)  # array with values 0..255

    for i in range(len(table)):
        value_normalized = table[i] / 255.0  # 0..1 range
        result = value_normalized ** gamma_inv
        result *= 255  # 0..255 range
        table[i] = np.round(result)

    return table
```

:::note

Un LUT (Look Up Table) es una tabla de consulta que se utiliza para transformar los valores de los píxeles de una imagen de acuerdo con una función específica, en este caso, una corrección gamma.

La función de corrección gamma se define como:

$$
\text{R(x, y)} = \left(\frac{\text{I(x, y)}}{255}\right)^{\frac{1}{\gamma}} 255
$$

Donde

- $\gamma$: Valor gamma positivo mayor a cero.
- $\text{I(x, y)}$: Imagen original.
- $\text{R(x, y)}$: Imagen resultante.

Podemos definir un arreglo LUT de 256 valores:

Indice | Valor
-------|------
0      | 0
1      | 0
2      | 0
3      | 0
...    | 0
255    | 0

Evaluamos la función en los valores de cada indice del arreglo. Si $\gamma = 1.7$

Indice | Valor
-------|------
0      | 0
1      | 38
2      | 48
3      | 55
...    | 0
255    | 255

:::

## Aplicar Corrección Gamma

```python
def get_gamma(image, gamma: float):
    """
    Return an image with gamma transformation
    """

    B, G, R = cv2.split(image)

    container_B = B
    container_G = G
    container_R = R

    lut = create_lut(gamma)

    for i in range(len(container_B)):
        value_B = container_B[i]
        container_B[i] = lut[value_B]

    for i in range(len(container_G)):
        value_G = container_G[i]
        container_G[i] = lut[value_G]

    for i in range(len(container_R)):
        value_R = container_R[i]
        container_R[i] = lut[value_R]

    image_merge = cv2.merge([container_B, container_G, container_R])

    return image_merge
```

1. Se separan cada uno de los canales y se guardan en un *container*.

```python
B, G, R = cv2.split(image)
```

2. Se genera la tabla LUT.

```python
lut = create_lut(gamma)
```

3. Cada píxel del container se reemplaza por el valor correspondiente en la LUT. Por ejemplo, si un píxel tiene un valor de 3, se busca el valor en la casilla 3 de la LUT y se reemplaza el valor del píxel original por este nuevo valor. 

Indice | Valor
-------|------
3      | 55

```python
for i in range(len(container_B)):
    value_B = container_B[i]
    container_B[i] = lut[value_B]
```

> Utilizar una tabla LUT fija la cantidad de evaluaciones de la función gamma a 256 (para una imagen de rango dinámico de 8 bits).

4. Se unen los canales
```python
image_merge = cv2.merge([container_B, container_G, container_R])
```

## Umbralización

```python
def apply_threshold(image, threshold: int, invert=False):
    """
    Return image with threshold and its mask
    """

    image_bw = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    mask = np.zeros(image_bw.shape, dtype=np.uint8)

    if invert:
        mask[image_bw < threshold] = 1
    else:
        mask[image_bw >= threshold] = 1

    B, G, R = cv2.split(image)

    channel_B = np.zeros(image_bw.shape, dtype=np.uint8)
    channel_G = np.zeros(image_bw.shape, dtype=np.uint8)
    channel_R = np.zeros(image_bw.shape, dtype=np.uint8)

    channel_B[mask == 1] = B[mask == 1]
    channel_G[mask == 1] = G[mask == 1]
    channel_R[mask == 1] = R[mask == 1]

    image_container = cv2.merge([channel_B, channel_G, channel_R])
    image_container = cv2.cvtColor(image_container, cv2.COLOR_BGR2RGB)

    mask = mask * 255

    return image_container, mask
```

1. Transformación de la imagen a escala de grises y creación de una matriz para la mascara.

```python
image_bw = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
mask = np.zeros(image_bw.shape, dtype=np.uint8)
```

2. Dependiendo del modo, la mascará abarcará cierta sección del umbral.

```python
if invert:
    mask[image_bw < threshold] = 1
else:
    mask[image_bw >= threshold] = 1
```

3. Separación de los canales, creación de las matrices contenedor y copia de valores solo si se encuentran dentro de la máscara.

```python
B, G, R = cv2.split(image)

channel_B = np.zeros(image_bw.shape, dtype=np.uint8)
channel_G = np.zeros(image_bw.shape, dtype=np.uint8)
channel_R = np.zeros(image_bw.shape, dtype=np.uint8)

channel_B[mask == 1] = B[mask == 1]
channel_G[mask == 1] = G[mask == 1]
channel_R[mask == 1] = R[mask == 1]
```

## Suma Ponderada

```python
def get_weighted_sum(channel1, channel2, value):
    return (value*channel1 + (1-value)*channel2).astype(np.uint8)


def get_weighted_images(image1, image2,
                        values: list = [0, 0.25, 0.5, 0.75, 1]):
    B1, G1, R1 = cv2.split(image1)
    B2, G2, R2 = cv2.split(image2)

    images = []
    for value in values:
        image = [get_weighted_sum(B1, B2, value),
                 get_weighted_sum(G1, G2, value),
                 get_weighted_sum(R1, R2, value)]
        image = cv2.merge(image)
        images.append(image)

    return images
```

1. Obtiene el valor de la suma ponderada de dos pixeles:

```python
def get_weighted_sum(channel1, channel2, value):
    return (value*channel1 + (1-value)*channel2).astype(np.uint8)
```

La función de suma ponderada empleada se define como:

$$
S = \omega x_1 + (1-\omega) x_2
$$

Donde

- $\omega$: Peso de la primera imagen comprendido entre $0$ y $1$.
- $x_1$: Valor del pixel de la primera imagen
- $x_2$: Valor del pixel de la segunda imagen

2. Obtener $n$ imágenes en base a un arreglo de pesos en `values`.

```python
def get_weighted_images(image1, image2,
                        values: list = [0, 0.25, 0.5, 0.75, 1]):
    B1, G1, R1 = cv2.split(image1)
    B2, G2, R2 = cv2.split(image2)

    images = []
    for value in values:
        image = [get_weighted_sum(B1, B2, value),
                 get_weighted_sum(G1, G2, value),
                 get_weighted_sum(R1, R2, value)]
        image = cv2.merge(image)
        images.append(image)

    return images
```
