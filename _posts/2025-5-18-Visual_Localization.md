---
layout: post
title: Marker Based Visual Localization
---

## Inicialización
El primer paso para conseguir la autolocalización del robot es la captura de la imagen de la cámara y la detección de los AprilTags en dicha imagen, así como la identificación de a cuáles corresponden. Para ello, se ha utilizado la librería `pyapriltags`, que implementa un detector de AprilTags, así como la carga del fichero YAML con la información de los tags presentes en la escena.

Una vez capturada la imagen, se detectan los AprilTags en el fotograma actual y se extrae la información sobre la posición del AprilTag respecto a la escena. Esta información se utilizará más adelante para la estimación de la posición.

## Estimación de la posición
Para realizar la estimación de la posición, se ha implementado la siguiente transformación:

![image](https://github.com/user-attachments/assets/a810846f-5406-4765-ab92-9bc30155944d)

### world2tag
Esta transformación se extrae directamente de la información estática del YAML, que contiene la posición y orientación (yaw) conocidas de cada AprilTag en el mundo. Para ello, se construye una matriz con la siguiente forma:

![image](https://github.com/user-attachments/assets/50a67d0f-1ada-43fb-a2a8-bce8e8fe7540)

### tag2tag_optical
Los AprilTags utilizan un sistema de coordenadas diferente al que emplean las cámaras. De este modo, para los AprilTags:

* +X es horizontal
* +Y es vertical
* +Z sale hacia fuera

Mientras que para la cámara:

* +X apunta a la derecha
* +Y apunta hacia abajo
* +Z apunta hacia adelante

Por ello, es necesario realizar una corrección, aplicando primero una rotación sobre el eje Z y luego otra sobre el eje X. El resultado de estas transformaciones será la posición del AprilTag en el sistema de coordenadas de la cámara. La inversa de esta transformación servirá más adelante para convertir de coordenadas ópticas a coordenadas del mundo.

### tag_optical2cam_optical
Aquí se calcula la pose relativa de la cámara respecto a la etiqueta detectada. Para ello, se emplea la función `solvePnP()` de `OpenCV`, que estima la posición y orientación de la cámara a partir de puntos 3D conocidos (en el tag) y sus proyecciones 2D (en la imagen).

Se han extraído los puntos correspondientes a las esquinas del AprilTag, tanto en la imagen (obtenidos con el detector implementado en `pyapriltags`) como en coordenadas del mundo. El resultado de `solvePnP()` es la matriz RT `cam_optical2tag_optical`, por lo que para obtener `tag_optical2cam_optical`, se calcula la matriz inversa de la obtenida previamente.

### cam_optical2cam
En este paso, se transforma del sistema de coordenadas ópticas al sistema de coordenadas del cuerpo de la cámara, utilizando la matriz inversa de `tag2tag_optical`, como se ha comentado anteriormente.

### cam2robot
Como la cámara no está ubicada en el centro exacto del robot, se especifica una transformación rígida (traslación) desde el marco de la cámara al centro del robot. Esta información se ha extraído del fichero SDF del robot. La matriz generada tiene la siguiente forma:

![image](https://github.com/user-attachments/assets/c91a1841-df25-4ac9-8cac-c166860b7242)

## Estimación final de la pose
Una vez estimada la matriz `world2robot`, se puede obtener la estimación de la pose del robot. La posición x e y del robot corresponderán, respectivamente, con los dos primeros valores de la última columna de `world2robot`. Por otro lado, para calcular el ángulo de orientación del robot, debe realizarse el siguiente cálculo: 

![image](https://github.com/user-attachments/assets/2f270336-e8b2-412d-ba7e-7803b6236159)

## Selección del AprilTag
En ocasiones, se detecta más de un AprilTag con la cámara. Para que el algoritmo sea lo más robusto posible, se utiliza únicamente la estimación de la pose del AprilTag que se encuentre más cercano al robot en su posición anterior. Esto se debe a que, cuanto más lejos se encuentre el AprilTag, mayor será el error en la estimación de la posición del robot.

Para ello, se calcula la distancia entre cada AprilTag detectado y se utiliza únicamente aquel que esté más próximo al robot.

## Odometría
Por otro lado, habrá momentos en los que no haya ningún AprilTag visible para el robot. Para poder seguir estimando su posición, se emplea odometría, que proporciona una estimación aproximada de la posición del robot sin necesidad de visión.

Adicionalmente, se ha recurrido a la odometría en otro caso: cuando se detecta un AprilTag, pero su distancia al robot es muy elevada. Esta decisión se ha tomado porque, al realizar la autolocalización visual con AprilTags lejanos, se observaba un error considerable en la estimación, alejándose notablemente de la posición real.

## Resultados
En este vídeo se muestran los resultados del algoritmo de autilocalización visual implementado:

<iframe width="740" height="473" src="https://www.youtube.com/embed/SjcH_R2s9D4?si=oyjZ3k7CDTwcrJv_" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
