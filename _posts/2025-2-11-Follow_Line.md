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
El manejo del coche (en principio el giro del volante, y más adelante la velocidad lineal del coche) se realizará a través de un controlador PID. Así, se ha definido una función genérica que, en base a las tres constantes (proporcional, integral y derivativa), el error actual, el acumulado y el anterior, determina los nuevos valores de velocidad angular/lineal. Durante las diferentes pruebas se han ajustado los valores de las tres constantes hasta encontrar la combinación que proporciona unos mejora resultados. Para ello, se ha realizado primeramente el ajuste de la constante proporcional del controlador hasta que la oscilación del controlador es constante, a lo que le ha seguido el ajuste de la constante derivativa hasta que las oscilaciones quedan reducidas a lo mínimo posible. Finalmente, se ajusta la constante integral, si es necesario.

## Coche holonómico: velocidad lineal constante
La primera prueba que se realiza es el control del coche holonómico a una velocidad constante baja de 5, con un controlador PID que realiza correcciones de la velocidad angular del coche. En este caso, el coche consigue completar, en el caso más rápido, el circuito simple en un tiempo de 110.33 s (1 minuto 50 s) con una baja cantidad de oscilaciones. Se puede comprobar un resultado el el siguiente vídeo:

<iframe width="560" height="315" src="https://www.youtube.com/embed/-8n43gd51KA?si=6KdQbGRxQdNPnALk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Proporcionando robustez al coche
