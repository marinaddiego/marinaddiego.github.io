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

En primer lugar, se decidió usar el centro de masas del contorno de la línea que sigue el coche. Para ello, se determinan los contornos de la máscara de la línea previamente obtenida mediante el filtro de color y se calculan los momentos de este contorno. Con el cálculo de los momentos, se determinan las coordenadas del centro de masa, que corresponde con un punto en una zona cercana al coche. Se trata de un punto que genera una gran cantidad de oscilaciones, difíciles de controlar.

El segundo punto que se prueba para el cálculo del error es el que corresponde con el punto más lejano de la máscara generada de la línea a seguir. En este caso, el coche se mueve de manera mucho más estable, siendo mucho más fácil de controlar y permitiendo al coche circular a una mayor velocidad de forma controlada. Es por ello, por lo que se decide tomar este punto para el cálculo del error para el controlador o controladores PID a implementar.

![Punto de cálculo de error]({{ site.baseurl }}/images/error_calc.png)
*Punto sobre el que se va a determinar el error para los controladores PID*

## Controlador PID
El manejo del coche (en principio el giro del volante, y más adelante la velocidad lineal del coche) se realizará a través de un controlador PID, cuya expresión matemática es:

$$u = -K_p \cdot e - K_i$$

. Para ello, se ha programado una función que, en base a las tres constantes $k_p$, $k_i$ y $k_d$, el error actual, el error anterior y el error acumulado proporcione el nuevo valor de la velocidad angular/lineal del coche. 
