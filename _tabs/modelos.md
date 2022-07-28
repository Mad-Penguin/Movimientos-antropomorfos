---
title: Modelos
icon: fa-solid fa-user-large
order: 2
---

## Ajuste del esqueleto

Una vez que los movimientos de los marcadores se ve razonable, el siguiente paso es asignarle una estructura de esqueleto para que sea controlado por el movimiento digitalizado, sin embargo, algunos problemas surgen, por ejemplo al usar los marcadores directamente para especificar la posición es que por el ruido, el suavizado e imprecisiones, las distancias entre las articulaciones del esqueleto no se van a mantener durante el tiempo lo que puede causar un efecto de que el esqueleto "patine" o que atraviese el suelo. 

Una razón de esto es que los marcadores no están exactamente en las articulaciones de la persona, si no afuera en la superficie, por lo que el punto digitalizado está desplazado de la articulación. 
Este desplazamiento puede ser calculado con la normal al plano formado por 3 marcadores consecutivos, consideremos el codo, si la muñeca tiene dos marcadores (usualmente para capturar la rotación) entonces la posición real de la muñeca puede ser interpolada a partir de esto y los marcadores muñeca-codo-hombro pueden ser usados para calcular la normal y así obtener la posición real del codo desplazando por una distancia previamente calculada (dependiendo del marcador y de la persona usando los marcadores) en dirección de la normal. El problema de esta solución es que cuando el brazo está extendido los marcadores son casi colineales. 
Ahora que las posiciones de las articulaciones son más consistentes con el esqueleto, ya pueden ser utilizadas para controlarlo. Para evitar tener coordenadas globales en el espacio se calculan las rotaciones de las articulaciones, por ejemplo, en un esqueleto jerárquico si se han capturado 3 marcadores consecutivos entonces el tercero es usado para calcular su rotación relativa a los dos anteriores

![Geometría para rotación relativa](/assets/images/rotacion-relativa.png)