---
title: Mocap
icon: fa-solid fa-camera
order: 1
---

# Posicionamiento de cámaras

Un problema que surge comúnmente en la captura de movimiento es el posicionamiento adecuado de las cámaras. Para saber las coordenadas de un punto de interés, es necesario que este sea visto por al menos dos cámaras (o más, dependiendo del sistema). Así, es de interés poder calcular las posiciones que maximicen la visibilidad de puntos. 

Cómo parte del proyecto, consultamos el paper Optimal Camera Placement for Motion Capture Systems. Dicha publicación propone una nueva manera de calcular posiciones óptimas para las cámaras en un entorno para la captura de movimiento.

## Métrica de error basada en oclusión

La **métrica de error basada en oclusión** presenta una forma de medir la visibilidad de un punto objetivo específico por al menos dos cámaras, para un conjunto de cámaras, cuando se presenta oclusión dinámica.

Para tomar en cuenta la gran mayoría de oclusores posibles, se considera un plano vertical que gira alrededor del punto objetivo. Así, entre todos todos los ángulos entre \(0\)° y \\(360\\)° que puede girar este plano respecto al punto, habrá ciertos intervalos de ángulos en los cuáles el punto no es visible por al menos dos cámaras. La métrica calcula la suma de las longitudes de estos intervalos. Informalmente, esta corresponde a la "suma de ángulos" en los cuáles no es visible el punto objetivo, y nos interesa minimizar esta cantidad.
 
![Oclusion](/assets/images/oclusion.png)

En la imagen anterior, observamos que la visibilidad del punto sólo cambia cuando el plano cruza el vector de la vista de una cámara. Notamos también que si existen $n$ cámaras, sus vectores de vista generan $2n$ regiones (es decir, intervalos de ángulos) de interés; por ejemplo, en la imagen, dos cámaras dividen el círculo centrado en el punto de interés en $4$ partes.

Además de esto, consideramos una **restricción de triangulación** para un par de cámaras: aunque ambas sean capaces de ver el punto de interés; sólo se toman en cuenta si el ángulo entre sus vectores de vista está entre 40° y 140°. Esto es porque si el ángulo es muy grande o muy pequeño, el error numérico al intentar triangular el punto es también muy grande. Asimismo, se descartan las cámaras si el punto está mas allá de el rango efectivo de estas.

La métrica anterior se puede calcular, en pseudocódigo, con el siguiente algoritmo:

    #n - cantidad de camaras
    #C[1..n] - posiciones de las cámaras
    #S[1..2n] - las longitudes de las regiones que generan 
    los vectores de vista de las camaras
    #Cf[] - posiciones de cámaras en C frente al oclusor
    #flag - es verdadera si en la región,
    el punto es visible por al menos dos cámaras
    Q - suma de las regiones en las que no es visible el punto.
    por al menos dos cámaras.
    -----------------------------------------------------
    calcular S 
	para i = 1 hasta 2n:
	    calcular Cf a partir de C y S[i]
	    flag = false
	    para j = 1 hasta longitud(Cf):
		    para k = j + 1 to longitud(Cf):
			    si Cf[j] y Cf[k] satisfacen la
			    restricción de triangulación:
				    flag = true
				    break
		si flag = false:
			Q += S[i]
	regresa S[i]

## Recocido simulado
Para aplicar la métrica anterior en el cálculo de posiciones óptimas, se utiliza el método de **recocido simulado**. Este es un método muy general que, de manera heurística, busca una aproximación al mínimo o máximo global de una función.

Este algoritmo considera una **temperatura**. Se inicia en una estado arbitrario; y se consideran varios estados siguientes válidos. Si alguno de estas mejora la función que queremos minimizar (o maximizar), cambiamos a este estado. De otra manera, aunque el siguiente estado empeore la función, con cierta probabilidad que depende de la temperatura, se cambia a este estado.

La temperatura está definida de tal manera que, entre más alta, es más probable que se tome una nueva posición. Así, se inicia con una temperatura alta, la cual va disminuyendo conforme pasa el tiempo. El objetivo de esta es "escapar" mínimos locales; es decir, estados que no son óptimos, pero que están rodeados de estados peores.

Volviendo al problema de posicionamiento de cámaras, consideramos los estados como el conjunto de posiciones de las cámaras, y la función a minimizar será una modificación sobre la *métrica de error basada en oclusión*. Dado un estado, tiene sentido considerar a los siguientes estados posibles como aquellos que cambian la posición de exactamente una cámara.

La primera modificación necesaria es considerar múltiples puntos. Supongamos que estos son $p_1,\ldots,p_m$. Entonces denotemos $Q_i$ como la métrica descrita antes, con el punto $p_i$ como objetivo. Nos interesa minimizar

$$f = \sum_{i = 1}^m Q_i.$$
 
Esto aún genera un problema. ¿Por qué? La métrica original no incentiva el poner una cámara en una región que no contenga ninguna cámara: al pasar de 0 cámaras a 1 cámara en la región, esta región sigue sin contener al menos un par de cámaras, así que su longitud se sigue considerando. Más aún, este cambio empeora la métrica si la región de la cual se toma la cámara a mover contiene sólo un par de cámaras.

Para esto definimos $\theta$ como un ***penalty***  asignado a puntos que no son visibles por un par de cámaras. Consideremos un punto $p_i$, y $C_i$ como el conjunto de puntos cuyos vectores de vista ven a $p_i$. Entonces, definimos el valor $E_i$ como la nueva métrica, dada por:

$$E_i = \begin{cases}
360 + \theta & |C_i| = 0\\
Q_i & |C_i| > 0.
\end{cases}$$

 Finalmente, queremos minimizar $f$, redefinida como

$$f = \sum_{i=1}^m E_i,$$

usando recocido simulado. El paper original sugiere usar $\theta = 360$ para obtener mejores resultados.





