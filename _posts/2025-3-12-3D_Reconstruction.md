---
layout: post
title: 3D Reconstruction
---

## Obtención de las imágenes izquierda y derecha y de los puntos de interés
El primer paso consiste en detectar puntos de interés en las imágenes que puedan utilizarse para reconstruir el modelo tridimensional. En este caso, se ha optado por aplicar el detector de bordes Canny, una técnica clásica que permite extraer contornos relevantes de los objetos presentes en las imágenes.

Tras aplicar el filtro Canny a ambas imágenes (izquierda y derecha), se ha aplicado un filtro bilateral. Este tipo de filtro suaviza la imagen sin eliminar los bordes, lo que ayuda a eliminar ruido preservando las estructuras principales.

![Imágenes originales]({{ site.baseurl }}/Images/3DRec_Original-img.png)
*Imágenes capturadas por las cámaras de la escena.*

![Imágenes preprocesadas]({{ site.baseurl }}/Images/preprocessed_image.png)
*Puntos de interés*

Una vez procesadas las imágenes, se extraen las coordenadas de los píxeles que han sido marcados como bordes. Estos puntos se consideran los puntos característicos de la imagen izquierda, y sobre ellos se aplican los pasos siguientes del algoritmo.

## Búsqueda de correspondencias mediante el uso de la geometría epipolar
En un sistema estéreo, cada punto de una imagen tiene su punto correspondiente en la otra imagen. Sin embargo, no se conoce directamente cuál es. Para resolver esta ambigüedad, se hace uso de la geometría epipolar, que restringe la búsqueda del punto homólogo a una única línea (la línea epipolar) en la imagen derecha, reduciendo drásticamente el espacio de búsqueda.

Para cada punto detectado en la imagen izquierda:

1. Se convierte el punto desde coordenadas gráficas a coordenadas ópticas, y de ahí al espacio 3D mediante retroproyección.
2. A partir de este punto y de la posición de la cámara izquierda, se calcula el vector de proyección.
3. Se selecciona un punto adicional sobre la línea de visión del punto, y se proyectan ambos sobre la cámara derecha.
4. Estos puntos se transforman nuevamente a coordenadas g´raficas, y con ellos se calcula la pendiente y la ordenada de la línea epipolar.

Una vez generada la línea epipolar en la imagen derecha, se recorren los píxeles de dicha línea y se seleccionan únicamente aquellos que coinciden con píxeles marcados como bordes en la imagen derecha procesada con Canny. Estos puntos se consideran candidatos a correspondencia.

## Matching mediante correlación
Para determinar cuál de estos candidatos es el verdadero punto homólogo al de la imagen izquierda, se utiliza un enfoque basado en la correlación sobre la franja epipolar:
1. Se extrae una ventana de tamaño fijo (en este caso, 10x10 píxeles) centrada en el punto de interés de la imagen izquierda.
2. Para cada punto candidato en la imagen derecha, se extrae una ventana de igual tamaño y se calcula la correlación normalizada entre ambas con *cv2.matchTemplate*.
3. El punto con mayor correlación se considera el homólogo más probable.
