---
layout: post
title:  "Visión artificial en Android"
date:   2017-3-19 13:38:45
description: Visión artificial para pasar los planos de dibujo técnico a un sistema tridimensional
categories:
- blog
- BoofCV
- Android
- OpenGL ES
- Java
- Lambda expresions
---

La primera parte del proyecto, que es añadir documentación para enseñar dibujo técnico, está cubierta parcialmente (falta añadir documentación y arreglar algunos fallos). En cambio la segunda parte de la aplicación, para pasar una lámina de diedrico a un sistema tridimensional por medio de visión artificial, no está muy terminada.

Como he puesto ya en este blog, la aplicación de DiedricoApp lleva desarrollandose hace un año (más o menos), y empezó siendo una aplicación para pasar una lamina de diedrico de un plano a un sistema tridimensional, pero el código no estaba muy depurado y tenía muchos errores que no se habían solucionado. Más tarde, deje este objetivo a un lado y me puse a programar la aplicación para substituir al tradicional libro de texto. Ahora toca sacar todo este trabajo del cajón y mejorar el código para que realmente haga lo que tenía que hacer.

El estado en el que se encontraba el tema de pasar a 3D era bastante pésimo, la documentación era mala y las cosas no llegaban de funcionar. Los principales problemas eran:
* Los algoritmos utilizados para recopilar los puntos y las lineas interesantes no funcionan muy bien, no terminan que coger todos los puntos.
* No se filtran los resultados. En consecuencia hay mucho ruido en los análisis.
* El usuario tiene que decirle al programa que hay en la imagen, CUANDO DEBERÍA DE SER AL CONTRARIO.
* Los puntos que son erroneos (le falta una proyección o están fuera de la lámina) no son eliminados.
* La UX es nefasta, otra vez más.

Con lo que me puse manos a la obra para solucionar todos estos problemas. Todas las contribuciones se encuentran en el [repositorio del proyecto](https://github.com/DiedricoApp/DiedricoApp).

La librería que utilizo para visión artifial se llama BoofCV y es una herramienta desarrollada en Java, con la licencia Apache 2.0, que proporciona muchas herramientas para analizar imagenes en tiempo real. Para conseguir pasar de un dibujo a un sistema en 3D el programa pasa por los siguiente pasos:
1. Se echa una foto (utilizando la cámara del móvil por medio de la API de google).
2. Se reescala la imagen para que luego los análisis sean más rápidos y no gasten tantos recursos.
3. Por medio de BoofCV se aplica un filtro (Thresholding) que pasa la imagen a escala de grises, con el objetivo de eliminar todas las imperfecciones de la imagen.
4. Se analizan todos los puntos y las lineas interesantes. Esto lo hace BoofCV [aquí](http://boofcv.org/index.php?title=Example_Detect_Interest_Points) y [aquí](http://boofcv.org/index.php?title=Example_Detect_Lines).
5. El análisis de imagenes deja muchas lineas que son pequeñas y que no sirven, y hacen falta eliminarlas.
6. En el análisis se pillan muchos puntos cerca de las lineas (sobretodo en la linea de tierra), y las tenemos que eliminar.
7. Eliminar los grupos de puntos y dejar solo un representante.
8. Eliminar los puntos que no tengan su proyección vertical y horizontal.
9. Por medio de matemáticas se determina la posición con respecto de la linea de tierra.
10. Se pasan las coordenadas a OpenGL ES para representarlo.

Para eliminar los puntos cerca de la linea de tierra he utilizado una cosa en Java que es muy reciente, que son las expresiones Lambda (Lambda expressions) y sirven para hacer funciones anónimas. El código es el siguiente:

```
                //Also, boofcv finds a lot of useless points near the landLine, so we have to delete them
                points = Stream.of(points)
                        .filter((PointVector point) -> point.getPointY() < yEdge - 15 || point.getPointY() > yEdge + 15)
                        .toList();
```

Como se puede ver, las expresiones lambda su muy útiles y dejan el código muy limpio. Todo el código se encuentra el el repositorio. En los próximos dias me dedicaré a mejorar los análisis para que el programa sea capaz de leer la lámina entera sin necesidad de que le ayude una persona.

amil101@debian:~$ EXIT
