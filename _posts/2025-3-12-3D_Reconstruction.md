---
layout: post
title: 3D Reconstruction
---

## Obtención de las imágenes izquierda y derecha y obtención de puntos de interés
En primer lugar, se capturan las imágenes de las dos cámaras, la cámara izquierda y la cámara derecha, de la escena y se realiza su preprocesamiento. El preprocesamiento consistirá en la obtención de aquellos puntos de interés para realizar la reconstrucción 3D, en concreto, los bordes de los objetos capturados por las cámaras. De esta manera, se aplica el algortimo de Canny que permite capturar estos bordes, y, a continuación, con el objetivo de eliminar detalles, se le aplica a la imagen de bordes un filtro bilateral. El resultado del preprocesamiento es el conjunto de puntos de interés de las imágenes, que serán los utilizados para la reconstrucción tridimensional de la escena.

![Imágenes originales]({{ site.baseurl }}/Images/3DRec_Original-img.png)
*Imágenes capturadas por las cámaras de la escena.*

![Imágenes preprocesadas]({{ site.baseurl }}/Images/preprocessed_image.png)
*Puntos de interés*





