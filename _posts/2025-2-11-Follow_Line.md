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

<iframe width="740" height="473" src="https://www.youtube.com/embed/-8n43gd51KA?si=6KdQbGRxQdNPnALk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Proporcionando robustez al coche
Antes de continuar con un aumento de la velocidad lineal del coche, se implementa una medida de robustez al coche, de manera que, cuando este no "encuentre" la línea roja, sea capaz de encontrarla y no se choque con las paredes. Para ello, cuando "no haya línea" en la imagen, el coche reducirá su velocidad lineal a 1 y mantendrá la velocidad angular que tenía hasta el momento. de esta manera, el coche continuará girando y, en el momento en el que reencuentre la línea, reajustará su velocidad angular para continuar siguiéndola.

## Coche holonómico: velocidad lineal variable
Una vez ya el coche es capaz de recorrer correctamente el circuito simple a una velocidad constante baja, se propone aumentar la velocidad para disminuir el tiempo en el que recorre el circuito. Al aumentar la velocidad lineal y mantenerla constante, el control del coche en las curvas se complica, saliéndose el coche de ellas y recorriéndolas con una gran oscilación. Por ello, se implementa un controlador PD para la velocidad lineal, de manera que, al aumentar el error, es decir, al iniciar una curva, la velocidad lineal se reduzca. Así, la velocidad lineal el coche oscilará entre 5 y 17, alcanzando los 17 en las rectas y disminuyendo hasta los 5 en las curvas más cerradas. Además, con este controlador permite que en las curvas menos cerradas el coche pueda circular a una velocidad más elevadas que en aquellas que sean más cerradas, aumentando la velocidad del coche. Además de ajustar las nuevas constantes para el controlador PD, se reajustan las constantes para el controlador PID de la velocidad angular. Con todo esto, el coche es capaz de realizar la vuelta al circuito simple en un tiempo mínimo de 61.10 s. En este caso, la oscilación del coche en las curvas es algo más marcada que cuando la velocidad lineal era más baja y constante, sin embargo, estas no son muy pronunciadas. En el siguiente vídeo, se pueden ver los resultados:

<iframe width="740" height="473" src="https://www.youtube.com/embed/7MuRxSBSpG4?si=0VE3fD6hQrlsAkIL" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

**Tiempo en otros circuitos**:
* Montreal: 172.80 s
* Montmelo: 100.61 s
* Nurburgring: 92.91 s

## Coche con dinámica Ackermann
Se comienzan las primeras pruebas con el coche con dinámica Ackermann. Lo primero que destaca de este, es que se trata de un coche mucho más "nervioso" que el holonómico y, por lo tanto, mucho más difícil de controlar. Así, la primera prueba que se va a realizar sobre este coche es tratar de realizar una vuelta al circuito simple.

### Velocidad lineal constante
En primer lugar, se pueba a realizar una vuelta completa al circuito simple con una velocidad constante y baja, usando las mismas técnicas que para el coche holonómico y ajustando los parámetros. Así, se trata de realizar una vuelta con una velocidad constante de 15. Sin embargo, el coche no consigue realizar la vuelta, debido al elevado grado de oscilación que presenta, chocándose contra las paredes e incluso dándose la vuelta por completo en varias ocasiones.

Para conseguir que de la vuelta al circuito, aunque sea a baja velocidad, se prueba a usar dos velocidades constantes: una para las rectas de 15 y otra para las curvas de 5. Así, en función de la magnitud del error que se produce, se detecta si el coche se encuentra en una recta o en una curva y se aplica la velocidad correspondiente comentada. Con esto, el coche es capaz de dar la vuelta completa al circuito, sin embargo, el tiempo que le toma es de 239.60 segundos (aproximadamente 4 minutos), es decir, le toma demasiado tiempo. A pesar de ello, se puede considerar un coche que presenta pocas oscilaciones.

<iframe width="740" height="473" src="https://www.youtube.com/embed/ORe3N3q8qzc?si=RjhJp2gUhl8nZvtz" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Velocidad lineal variable
Para conseguir aumentar algo más la velocidad del coche e intentar que se reduzcan las oscilaciones de este, se intenta implementar una solución en la que la velocidad lineal sea variable. Así, al igual que para el coche holonómico, se añade un segundo controlador PD para la velocidad lineal, que la disminuye cuando el coche entra en alguna curva, siendo esta velociad menor en curvas más pronunciadas.

Se comienza estableciendo una velocidad máxima de 25 y una velocidad mínima de 7, velocidades lineales entre las que oscilará el coche a lo largo del circuito. además, para disminuir el nivel de oscilaciones respecto a la velocidad angular del coche, se reformula el error que se toma para el cálculo de la salida del controlador PID/PD de las velocidades angular y lineal, de manera que, en vez de tomarse directamente el error actual del coche, se toma como error el promedio de los últimos 5 errores. Este nuevo cálculo del error permite amortiguar las oscilaciones y, por lo tanto, mejorar la velocidad del coche. Adicionalmente, se limita el giro máximo del coche a un intervalo entre -0.5 y +0.5, de manera que las oscilaciones sean lo más suaves posibles sin limitar el movimiento del coche a través del circuito. Con esta solución, a penas reduce el tiempo de vuelta del circuito en 50 segundos, hasta dar la vuelta en un tiempo de 217.56 segundos, siguiendo siendo aún así un tiempo demasiado elevado. A esto, se le añade el problema de la gran cantidad de oscilaciones que se genera en el control del coche, tal y como se puede ver en el vídeo.

<iframe width="740" height="473" src="https://www.youtube.com/embed/-R-06ukl2EM?si=4eNgkY-MMEmNsGD9" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
