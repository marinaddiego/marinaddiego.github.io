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

![Línea epipolar]({{ site.baseurl }}/Images/epipolar_line.png)
*Visualización de punto de interés en la imagen de la cámara izquierda (punto verde) y línea epipolar correspondiente sobre la imagen de la cámara derecha (línea roja).*

Una vez generada la línea epipolar en la imagen derecha, se recorren los píxeles de dicha línea y se seleccionan únicamente aquellos que coinciden con píxeles marcados como bordes en la imagen derecha procesada con Canny. Estos puntos se consideran candidatos a correspondencia.

![Puntos candidatos]({{ site.baseurl }}/Images/candidatos.png)
*Visualización de los puntos candidatos a ser homólogos sobre la línea epipolar (puntos verdes sobre la línea epipolar roja).*

## Matching mediante correlación
Para determinar cuál de estos candidatos es el verdadero punto homólogo al de la imagen izquierda, se utiliza un enfoque basado en la correlación sobre la franja epipolar:
1. Se extrae una ventana de tamaño fijo (en este caso, 20x20 píxeles) centrada en el punto de interés de la imagen capturada por la cámara izquierda en escala de grises.
2. Para cada punto candidato en la imagen derecha, se extrae una ventana de igual tamaño en la imagen capturada por la cámara derecha en escala de grises y se calcula la correlación normalizada entre ambas con *cv2.matchTemplate*.
3. El punto con mayor correlación se considera el homólogo más probable.

![Punto homólogo]({{ site.baseurl }}/Images/homologo.png)
*Punto homólogo estimado sobre la imagen capturada por la cámara de la derecha*

## Triangulación
Una vez identificado el punto homólogo en la imagen derecha mediante el proceso de correlación, se procede a calcular la posición tridimensional del punto observado por ambas cámaras. Para ello, primero se obtiene la dirección de los rayos de visión que pasan por ambos puntos (el original de la imagen izquierda y su homólogo en la imagen derecha). Esto se realiza mediante los siguientes pasos:
1. Conversión de coordenadas gráficas a ópticas: las coordenadas del punto de máxima correlación en la imagen derecha se transforman a coordenadas ópticas.
2. Retroproyección: se obtiene el punto 3D en la dirección del rayo proyectado desde la cámara derecha hacia el punto seleccionado en la imagen.
3. Cálculo de vectores de proyección: para ambas cámaras (izquierda y derecha), se calcula el vector de dirección desde la posición de la cámara hasta el punto proyectado. Este vector representa la dirección del rayo que atraviesa el punto en cuestión.
4. Normalización de los vectores: ambos vectores de proyección se normalizan para asegurar que su longitud no influya en los cálculos posteriores.
Idealmente, los dos rayos proyectados desde cada cámara deberían encontrarse en un único punto del espacio 3D. Sin embargo, debido a errores de calibración, ruido en las imágenes o discretización, es poco probable que los rayos se crucen exactamente.
Por esta razón, se calcula el punto más cercano entre las dos líneas, utilizando una solución basada en álgebra vectorial. El procedimiento es el siguiente:
1. Producto vectorial de los vectores de proyección: se calcula el vector perpendicular al plano definido por los dos rayos, el cual se utilizará para determinar la distancia mínima entre ambas líneas.
2. Parámetros de intersección: se resuelve un sistema que permite encontrar los parámetros t y u que definen los puntos más cercanos sobre cada línea (una desde la cámara izquierda y otra desde la derecha).
3. Puntos más cercanos: se calculan los puntos 3D individuales sobre cada rayo, y luego se promedia su posición para obtener el punto triangulado.
4. Para aportar realismo y coherencia visual a la nube de puntos generada, se asigna un color al punto 3D. Este color se obtiene directamente del píxel correspondiente en la imagen izquierda (aunque también se puede considerar una media entre ambas cámaras). Esto permite visualizar la escena con información fotométrica además de geométrica.

Finalmente, el punto reconstruido se añade a la escena mediante la función ``GUI.ShowNewPoints``, lo que permite su representación visual en el entorno virtual del simulador.

## Resultados
El resultado de la reconstrucción tridimensional de la escena se puede comprobar en el siguiente vídeo:

<iframe width="740" height="473" src="https://www.youtube.com/embed/wMiUv4usiJU?si=AjbRa5B-V3QNhaGg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Se piede comprobar que se trata de una recpnstruccón muy densa, por lo que se ha modificado la solución para que se realice la reconstrucción en tan solo uno de cada diez puntos de interés, reduciendo en una gran cantidad el tiempo de reconstrucción, permitiendo aún así la distinción de los diferentes objetos de la escena.

<iframe width="740" height="473" src="https://www.youtube.com/embed/6BmCFAGwdJ8?si=6I0bFCCKAKm3X7TQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
