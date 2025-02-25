---
layout: post
title: Follow Line
---

## Captura de imágenes y preprocesado
El primer paso que se ha seguido para realizar el seguimiento visual de la línea roja ha sido realizar la captura de la imagen a través de la cámara del coche y preprocesarla. El preprocesamiento ha consistido en aplicar un filtro de color, transformando en primer lugar el fotograma capturado a un espacio HSV y estableciendo aquellos valores de tonalidad, saturación y brillo que corresponden con colores rojos. Además, se ha realizado una erosión y dilatación del resultado para eliminar el posible ruido que pueda generarse en el filtrado.

![Filtro de color]({{ site.baseurl }}/images/Preprocesado1.png)
*Resultado de aplicar un filtro de color al fotograma capturado*

## Determinación del punto para el cálculo del error
Una vez obtenida la máscara que determina la línea que va a seguir el coche, se continúa con la extracción del punto a partir del cual se va a calcular el error para el controlador PID que se va a implementar. 

En primer lugar, se decide usar el centro de masas del contorno de la línea que sigue el coche. Para ello, se determinan los contornos de la máscara de la línea previamente obtenida mediante el filtro de color y se calculan los momentos de este contorno. Con el cálculo de los momentos, se determinan las coordenadas del centro de masa, que corresponde con un punto en una zona cercana al coche. Se trata de un punto que genera una gran cantidad de oscilaciones, difíciles de controlar.

El segundo punto que se prueba para el cálculo del error es el que corresponde con el punto más lejano de la máscara generada de la línea a seguir. En este caso, el coche se mueve de manera mucho más estable, sin embargo, no se desplaza de manera centrada sobre la línea, sino que en la mayoría del tiempo lo hace a un lado de esta.

Finalmente, se decide tomar un punto intermedio entre los previamente comentados. Así, se toma un punto desplazado un 75% hacia el punto más lejano de la máscara y un 25% hacia el punto del centro de masas, obteniendo un equilibrio entre baja oscilación y desplazsamiento centrado en la línea a seguir.
