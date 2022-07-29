---
title: Modelos
icon: fa-solid fa-user-large
order: 2
math: true
---

## Encontrando los puntos en 3 dimensiones

Una vez que tenemos grabado el video a través de las cámaras, debemos encontrar las coordenadas 3D de los marcadores en base a los datos obtenidos. Para poder reconstruir la posición de un punto, este debe ser visto por al menos 2 cámaras, ya que en otro caso, no tenemos información de las 3 dimensiones. Si $C_1$ es la posición de la cámara 1, $C_2$ la de la segunda cámara. Estos datos son conocidos por como calibramos el sistema. Y de las imágenes podemos calcular los valores $I_1$ y $I_2$ que serian la posición del marcador en el plano relativo de la cámara 1 y 2 respectivamente.

Una vez que tenemos estos datos definidos. Si $P$ es la posición del marcador, entonces podemos escribir las ecuaciones.

$$C_1+k_1(I_1-C_1)=P $$ $$C_2+k_2(I_2-C_2)=P $$

donde $k_1$ y $k_2$ son incógnitas, estas dos ecuaciones se pueden igualar para obtener

$$ C_1+k_1(I_1-C_1)=C_2+k_2(I_2-C_2)$$

que es una ecuación con 2 incógnitas, pero recordando que son valores en $\mathbb{R}^3$, tenemos un sistema de 3 ecuaciones con 2 incógnitas, por lo que es posible resolverlo para encontrar la posición del marcador, con los datos.

Sin embargo para datos reales, la ecuación no siempre tiene solución, ya que habrá siempre ruido que evite tener datos exactos, por lo que se encuentran dos puntos $P_1$ y $P_2$ que cumplen 

$$C_1+k_1(I_1-C_1)=P_1 $$ $$C_2+k_2(I_2-C_2)=P_2 $$ $$(P_2-P_1)\dot(I_1-C_1)=0 $$ $$(P_2-P_1)\dot(I_2-C_2)=0 $$

Las primeras dos condiciones nos dicen que $P_1$ esta en la recta en la que la cámara 1 dice que esta el marcador, y que  $P_2$ en la recta en la cual la cámara 2 ve el marcador 2.  Las ultimas dos condiciones nos piden que la recta que une al punto $P_1$ y $P_2$ sea perpendicular a las dos rectas en la que las cámaras ven el marcador. Esto nos da los puntos mas cercanos en estas rectas, una vez que encontramos estos, podemos usar el punto medio entre $P_1$ y $P_2$ como la posición del marcador. 

Otro problema común, cuando hay varias marcadores, es saber definir cuales que marcador visto por una cámara corresponde al mismo marcador visto por otra cámara, para esto podemos checar la diferencia entre los puntos $P_1$ y $P_2$ generados, y si es muy grande, esto es un indicador de que no se esta observando el mismo marcador. 

## Guardando la información 

Una vez que se ha obtenido los datos de la posición de los marcadores, y se ha usado para generar la información acerca del esqueleto y su posición a través del tiempo. debemos guardar esta información de alguna manera. 

El formato mas común para esto es el formato BVH(Biovision Hierarchy), en el que se guarda primero el formato del esqueleto, esto se hace, definiendo primero una raíz, o punto de referencia, el cual suele ser ubicado en la cadera del esqueleto, a partir de el se definen las articulaciones en forma de árbol, cada articulación tiene una articulación padre, y se define su distancia original hacia su padre, lo que nos da una posición original, así como un tamaño para el hueso entre ambas articulaciones.

Una vez que tenemos definido el esqueleto, guardamos la información acerca del movimiento. de la siguiente manera. Primero definimos la cantidad de frames o imágenes que conformaran nuestra animación, y después definimos el tiempo que en segundos que ocurrirá entre cada frame. Después por cada frame guardamos información acerca de la posición del esqueleto en ese momento. Estos datos serán la posición de la raíz, y además para cada articulación, guardamos su rotación en los 3 ejes, Con esto podemos inferir la posición de cada uno de los huesos, simplemente concatenando las matrices de transformación de cada articulación con la de su padre.

## Ajuste del esqueleto

Una vez que los movimientos de los marcadores se ve razonable, el siguiente paso es asignarle una estructura de esqueleto para que sea controlado por el movimiento digitalizado, sin embargo, algunos problemas surgen, por ejemplo al usar los marcadores directamente para especificar la posición es que por el ruido, el suavizado e imprecisiones, las distancias entre las articulaciones del esqueleto no se van a mantener durante el tiempo lo que puede causar un efecto de que el esqueleto "patine" o que atraviese el suelo. 

Una razón de esto es que los marcadores no están exactamente en las articulaciones de la persona, si no afuera en la superficie, por lo que el punto digitalizado está desplazado de la articulación. 
Este desplazamiento puede ser calculado con la normal al plano formado por 3 marcadores consecutivos, consideremos el codo, si la muñeca tiene dos marcadores (usualmente para capturar la rotación) entonces la posición real de la muñeca puede ser interpolada a partir de esto y los marcadores muñeca-codo-hombro pueden ser usados para calcular la normal y así obtener la posición real del codo desplazando por una distancia previamente calculada (dependiendo del marcador y de la persona usando los marcadores) en dirección de la normal. El problema de esta solución es que cuando el brazo está extendido los marcadores son casi colineales. 
Ahora que las posiciones de las articulaciones son más consistentes con el esqueleto, ya pueden ser utilizadas para controlarlo. Para evitar tener coordenadas globales en el espacio se calculan las rotaciones de las articulaciones, por ejemplo, en un esqueleto jerárquico si se han capturado 3 marcadores consecutivos entonces el tercero es usado para calcular su rotación relativa a los dos anteriores

![Geometría para rotación relativa](/assets/images/rotacion-relativa.png)