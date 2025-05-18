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
Aquí se calcula la pose relativa de la cámara respecto a la etiqueta detectada usando la función `solvePnP()` de `OpenCV`, que estima la posición y orientación de una cámara en base a puntos 3D conocidos (en el tag) y sus proyecciones 2D (en la imagen). Para ello, se han extraído los puntos correspondientes a las esquinas del AprilTag tanto en la imagen (que se extraen con la detección del AprilTag con el detector implementado en `pyapriltags`) como en coordenadas del mundo.

### cam_optical2cam
En este paso, se transforma del sistema de coordenadas ópticas al sistema de coordenadas del mundo con la matriz inversa de `tag2tag_optical`, como ya se ha comentado anteriormente.

### cam2robot
Como la cámara no está ubicada en el centro exacto del robot, se especifica una transformación rígida (traslación) desde el marco de la cámara al centro del robot. Para ello, se ha extraído esta información del fichero SDF del robot. La matriz generada tiene la siguiente forma:

![image](https://github.com/user-attachments/assets/c91a1841-df25-4ac9-8cac-c166860b7242)
