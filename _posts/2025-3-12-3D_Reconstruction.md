---
layout: post
title: 3D Reconstruction
---

## Obtención de las imágenes izquierda y derecha y preprocesado
En primer lugar, se capturan las imágenes de las dos cámaras, la cámara izquierda y la cámara derecha, de la escena y se realiza su preprocesamiento. El preprocesamiento consistirá en la obtención de aquellos puntos de interés para realizar la recosntrucción 3D, en contreto, los bordes de los objetos capturados por las cámaras. De esta manera, se aplica el algortimo de Canny que permite capturar estos bordes.

![Imágenes originales]({{ site.baseurl }}/Images/3DRec_Original-img.png)
*Imágenes capturadas por las cámaras de la escena.*

