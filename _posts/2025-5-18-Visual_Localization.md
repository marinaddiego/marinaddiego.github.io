---
layout: post
title: Marker Based Visual Localization
---

## Inicialización
El primer paso para conseguir la autolocalización del robot es la captura de la imagen de la cámara y la detección de los AprilTags en la imagen, así como a cuáles corresponden. Para ello, se ha usado la librería `pyapriltags`, que implementa un detector de AprilTags, así como la carga del fichero YAML con la información de los tags presentes en la escena.

## Estimación de la posición
Para realizar la estimación de la posición, se ha implementado la siguiente transformación:

![image](https://github.com/user-attachments/assets/a810846f-5406-4765-ab92-9bc30155944d)
