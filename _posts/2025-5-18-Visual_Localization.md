---
layout: post
title: Marker Based Visual Localization
---

## Inicialización
El primer paso para conseguir la autolocalización del robot es la captura de la imagen de la cámara y la detección de los AprilTags en la imagen, así como a cuáles corresponden. Para ello, se ha usado la librería `pyapriltags`, que implementa un detector de AprilTags, así como la carga del fichero YAML con la información de los tags presentes en la escena.

Una vez capturada la imagen, se detectan los AprilTags en el fotograma actual y se extrae la información sobre la posición del AprilTag respecto a la escena. Esta información nos servirá más adelante para la estimación de la posición.

## Estimación de la posición
Para realizar la estimación de la posición, se ha implementado la siguiente transformación:

![image](https://github.com/user-attachments/assets/a810846f-5406-4765-ab92-9bc30155944d)

### world2tag
Esta transformación se extrae directamente de la información estática del YAML, que contiene la posición y orientación (yaw) conocida de cada AprilTag en el mundo. Para ello, se obtiene una matriz con la siguiente forma:

![image](https://github.com/user-attachments/assets/50a67d0f-1ada-43fb-a2a8-bce8e8fe7540)

### tag2tag_optical
Los AprilTags tienen un sistema de coordenadas diferente al que usan las cámaras. De esta manera, para los AprilTags:

* +X es horizontal
* +Y es vertical
* +Z sale hacia fuera

Mientras que para la cámara:

* +X apunta a la derecha
* +Y apunta hacia abajo
* +Z apunta hacia adelante

Por ello, es necesario realizar una corrección, haciendo un giro sobre el eje Z y luego otro sobre el eje X. El resultado de la aplicación de estas será la posición del AprilTag en el sistema de coordenadas de la cámara. La inversa de esta transformación servirá para transformar de coordenadas ópticas a coordenadas del mundo más adelante.

### tag_optical2cam_optical
Aquí se calcula la pose relativa de la cámara respecto a la etiqueta detectada. Para ello, se usa la función `solvePnP()` de `OpenCV`, que estima la posición y orientación de la cámara en base a puntos 3D conocidos (en el tag) y sus proyecciones 2D (en la imagen). Para ello, se han extraído los puntos correspondientes a las esquinas del AprilTag tanto en la imagen (que se extraen con la detección del AprilTag con el detector implementado en `pyapriltags`) como en coordenadas del mundo. El resultado de la función `solvePnP()` es la matriz RT `cam_optical2tag_optical`, por lo que para obtener `tag_optical2cam_optical`, se calcula la matriz inversa a la previamente obtenida.

### cam_optical2cam
En este paso, se transforma del sistema de coordenadas ópticas al sistema de coordenadas del mundo con la matriz inversa de `tag2tag_optical`, como ya se ha comentado anteriormente.

### cam2robot
Como la cámara no está ubicada en el centro exacto del robot, se especifica una transformación rígida (traslación) desde el marco de la cámara al centro del robot. Para ello, se ha extraído esta información del fichero SDF del robot. La matriz generada tiene la siguiente forma:

![image](https://github.com/user-attachments/assets/c91a1841-df25-4ac9-8cac-c166860b7242)

## Estimación final de la pose
Una vez estimada la matriz `world2robot`, se puede obtener la estimación de la pose del robot. La posición x e y del robot corresponderán, respectivamente, con los dos primeros valores de la última columna de `world2robot`. Por otro lado, para calcular el ángulo del robot, debe realizarse el siguiente cálculo: 

![image](https://github.com/user-attachments/assets/2f270336-e8b2-412d-ba7e-7803b6236159)

## Selección del AprilTag
En ocasiones, más de un AprilTag es detectado con la cámara. Para que el algoritmo sea lo más robusto posible, se utiliza tan solo la estimación de la pose del AprilTag que se encuentre más cercano al robot en su posición anterior, ya que cuanto más lejos se encuentre el AprilTag, mayor error habrá en la estimación de la posición del robot. Para ello, se calcula la distancia entre cada AprilTag detectado y se utiliza tan solo aquel que se encuentre más cercano al robot.

## Odometría
Por otro lado, habrá momentos en los que no haya ningún AprilTag visible al robot. Para poder seguir detectando la posición de este, se utiliza odometría, que proporciona la posición aproximada del robot sin necesidad de visión. 

Adicionalmente, se ha usado odometría en otro momento: cuando se detecta un AprilTag, pero su distancia al robot es muy elevada. Esto se ha decidido hacer así porque al realizar la autolocalización visual cuando los AprilTags estaban lejanos al robot se observaba un mayor error en la localización, alejándose bastante de la posición real.

