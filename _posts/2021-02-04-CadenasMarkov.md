---
published: true
title: Introduction to Hidden Mmarkov Models
date: 2019-02-15
layout: single
author_profile: false
read_time: true
tags: [Statistics , HMM] 
categories: [statistics]
excerpt: " statistics, HMM"
comments : true
toc: true
toc_sticky: true
---






# Cadenas de Markov



Cuando hablamos de un proceso estocástico en el sentido más general,
tenemos una familia de variables aleastorias (*X*<sub>*j*</sub>), donde
j es un conjunto de índices, definidas en un espacio de probabilidad
(*Ω*, *F*, *P*). Si el proceso estocástico representa la evolución
temporal de algún sistema aletorio, quiere decir que cada variable
aleatoria *X*<sub>*j*</sub> está indexado en tiempo, así decimos que el
proceso es en tiempo discreto en el caso que J={0,1,2…} o en tiempo
continuo si J=\[0,+∞\], aunque se pueda considerar conjunto más
generales.

Ahora, sea S un conjunto finito o numerable, al que llamamos espacio de
estados del proceso y que definimos como los posibles valores que pueden
tomar las variables aleatorias ($X\_{j}\\space j \\in J$)

De manera informal, podemos decir que una **cadena de Markov** es un
proceso con la propiedad de que, condicionado a su valor presente, su
futuro es independiente del pasado.

Una **cadena de Markov** es un proceso estocástico en el que si el
estado actual *X*<sub>*n*</sub> y los estados previos
*X*<sub>1</sub>..., *X*<sub>*n* − 1</sub> son conocidos, la probabilidad
del estado futuro X\_{n+1} no depende de los estados anteriores
*X*<sub>1</sub>..., *X*<sub>*n* − 1</sub> y solamente depende del estado
actual *X*<sub>*n*</sub>

Para n=1,2.. y para cualquier sucesión de estados
*s*<sub>1</sub>, ..., *s*<sub>*n* + 1</sub>  



# Modelado sistemas dinámicos en tiempo discreto





## El ascensor social



Para muchos, el nivel de movilidad intergeneracional de la sociedad -
denominado ascensor social- es visto como una medida de hasta qué punto
existe la igualdad de oportunidades económicas y sociales. El nivel de
movilidad intergeneracional capta el grado de igualdad en sus
oportunidades en la vida: hasta qué punto se reflejan las circunstancias
de una persona durante la infancia en su éxito en la vida posterior, o
por el contrario, hasta qué punto los individuos pueden llegar a
conseguilro en virtud de su propio talento, motivación y suerte.

La manera más intuitiva de ver el alcance de la movilidad
intergeneracional es ver dónde acaban en la distribución de ganancias o
ingresos los niños de las familias con más o menos recursos cuando son
adultos. Esto se puede mostrar con una matriz de transición que muestra
los moviminetos de la distribución de la renta entre generaciones.



### Matriz de transición de la cadena de Markov discreta



El siguiente cuadro contiene la matriz de transición del ascensor social
de los hijos nacidos en 1970. Las filas representan los cuartiles de los
ingresos de los padres cuando sus hijos tenían 16 años. Las columnas
representan los cuartiles de los ingresos de los hijos a la edad de 30
años en 2000.

<table class="table" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="empty-cells: hide;border-bottom:hidden;" colspan="1">
</th>
<th style="border-bottom:hidden;padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="4">

<div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">

Cuartil de ingresos de los hijos

</div>

</th>
</tr>
<tr>
<th style="text-align:left;">
Cuartil de ingresos medios parentales
</th>
<th style="text-align:right;">
Q4
</th>
<th style="text-align:right;">
Q3
</th>
<th style="text-align:right;">
Q2
</th>
<th style="text-align:right;">
Q1
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Q4
</td>
<td style="text-align:right;">
0.38
</td>
<td style="text-align:right;">
0.25
</td>
<td style="text-align:right;">
0.21
</td>
<td style="text-align:right;">
0.16
</td>
</tr>
<tr>
<td style="text-align:left;">
Q3
</td>
<td style="text-align:right;">
0.29
</td>
<td style="text-align:right;">
0.28
</td>
<td style="text-align:right;">
0.26
</td>
<td style="text-align:right;">
0.17
</td>
</tr>
<tr>
<td style="text-align:left;">
Q2
</td>
<td style="text-align:right;">
0.22
</td>
<td style="text-align:right;">
0.26
</td>
<td style="text-align:right;">
0.28
</td>
<td style="text-align:right;">
0.24
</td>
</tr>
<tr>
<td style="text-align:left;">
Q1
</td>
<td style="text-align:right;">
0.11
</td>
<td style="text-align:right;">
0.22
</td>
<td style="text-align:right;">
0.24
</td>
<td style="text-align:right;">
0.43
</td>
</tr>
</tbody>
</table>

Muestra un ejemplo de matriz de transición para el Reino Unido para
niños nacidos el 1970. Divide la distribución de cada generación en
cuatro cuartiles de medida igual (que contienen cada uno el 25 por
ciento de la gente) y muestra el movimiento que hay entre cuartiles
entre generaciones. En una sociedad totalmente móvil, la cuarta parte de
los niños de cada grupo de ingresos acabaría en cada uno de los cuatro
cuartiles de la distribución de los ingresos por adultos, de forma que
cada celda de la matriz contendría un 0,25. En caso de no tener
movilidad, todos los niños estarían en el mismo cuartil que sus padres y
la falta de movimiento entre los cuartiles se mostraría por un 1 en la
diagonal y 0 en el resto de elementos de la matriz.

``` r
x <- c(0.38, 0.25, 0.21, 0.16,
       0.29, 0.28, 0.26, 0.17,
       0.22, 0.26, 0.28, 0.24,
       0.11, 0.22, 0.24, 0.43)
labels <- c("Q4", "Q3", "Q2", "Q1")
P <- matrix(data=x, byrow=TRUE, nrow=4, dimnames=list(labels, labels))

P
```

    ##      Q4   Q3   Q2   Q1
    ## Q4 0.38 0.25 0.21 0.16
    ## Q3 0.29 0.28 0.26 0.17
    ## Q2 0.22 0.26 0.28 0.24
    ## Q1 0.11 0.22 0.24 0.43



### Definición cadena de Markov



Creamos la cadena de Markov discreta definida por la matriz de
transiciones P. Nuestra cadena de Markov presenta las probabilidades de
transición por filas y no por columnas.

``` r
library(markovchain)
```

    ## Package:  markovchain
    ## Version:  0.8.5-4
    ## Date:     2021-01-07
    ## BugReport: https://github.com/spedygiorgio/markovchain/issues

``` r
mcP <- new("markovchain", states=labels, byrow=TRUE, transitionMatrix=P, name="Escala social")
mcP
```

    ## Escala social 
    ##  A  4 - dimensional discrete Markov Chain defined by the following states: 
    ##  Q4, Q3, Q2, Q1 
    ##  The transition matrix  (by rows)  is defined as follows: 
    ##      Q4   Q3   Q2   Q1
    ## Q4 0.38 0.25 0.21 0.16
    ## Q3 0.29 0.28 0.26 0.17
    ## Q2 0.22 0.26 0.28 0.24
    ## Q1 0.11 0.22 0.24 0.43

``` r
summary(mcP)
```

    ## Escala social  Markov chain that is composed by: 
    ## Closed classes: 
    ## Q4 Q3 Q2 Q1 
    ## Recurrent classes: 
    ## {Q4,Q3,Q2,Q1}
    ## Transient classes: 
    ## NONE 
    ## The Markov chain is irreducible 
    ## The absorbing states are: NONE



#### Diagrama de la cadena



Cada estado se representa por un círculo, y las transiciones entre
estados con flechas acompañadas por la correspondiente probabilidad de
transición

``` r
library(diagram)
```

    ## Loading required package: shape

``` r
plot(mcP, package="diagram", cex= .6)
```

![](CadenasMarkov_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->



#### Distribución estacionaria



Un vector estacionario es un vector de probabilidad y es el vector
propio de la matriz de transición P asociado al valor propio *λ* = 1

Cuando la cadena de Markov tiene un comportamiento a largo plazo
definido éste viene dado por el vector estacionario.

Dada nuestra matriz de transición P de la cadena de Markov finita,
aperiódica (período=1) e irreducible, existe una única solución y se
cumpple la siguiente igualdad que indica una distribución estacionario o
de equilibrio

Donde *π*<sub>*j*</sub> es la distribució estacionaria y
*π*<sub>*i*</sub> el vector de probabilidad.

Mediante el siguiente código obtenemos el resultado pertenecienta a la
distribución estacionaria

``` r
# Calculo de valores y vectores propios mediante la traspuesta de P
V <- eigen(t(P))
vaps <- V$values
veps <- V$vectors
# Calculo distribucion estacionaria
SS <- abs(veps[,1])
SS <- SS/sum(SS)
SS
```

    ## [1] 0.2502453 0.2525585 0.2474415 0.2497547



### Probabilidad sobre generaciones



A partir de las cadenas de Markov definidas para el caso, se procederá a
responder a planteamientos sobre las probabilidades según generaciones.

> Se quiere conocer cuántas generaciones son necesarias para que la
> probabilidad que un hijo nacido en una familia de ingresos del cuarto
> cuartil (Q4) esté a los 30 añs en el primer cuartil (Q1) y que sea
> superior al 22%

Para conocer las generaciones necesarias para que se de dicha
probabilidad entre los estados que representan los cuartiles de ingresos
realizamos el cálculo mediante las Ecuaciones de Chapman-Kolmogorov. Nos
permite obtener las probabilidades de transición de n pasos (en este
caso generacione).

Como apunte previo al desarrollo de las ecuaciones, este cálculo es
posible puesto que consideramos la cadena de Markov homogénea, es decir,
la probabilidad de pasar de un cuartil a otro cuartil de otra generación
no varía a lo largo del tiempo.

Multiplicamos la matriz de un paso por sí misma, esto es la matriz P de
un paso , y posteriormente la matriz P de dos pasos.

``` r
# Matriz P de un paso
p2 <- mcP*mcP
p2
```

    ## Escala social 
    ##  A  4 - dimensional discrete Markov Chain defined by the following states: 
    ##  Q4, Q3, Q2, Q1 
    ##  The transition matrix  (by rows)  is defined as follows: 
    ##        Q4     Q3     Q2     Q1
    ## Q4 0.2807 0.2548 0.2420 0.2225
    ## Q3 0.2673 0.2559 0.2473 0.2295
    ## Q2 0.2470 0.2534 0.2498 0.2498
    ## Q1 0.2057 0.2461 0.2507 0.2975

``` r
# Matriz P de dos pasos
p3 <- mcP*p2
p3
```

    ## Escala social 
    ##  A  4 - dimensional discrete Markov Chain defined by the following states: 
    ##  Q4, Q3, Q2, Q1 
    ##  The transition matrix  (by rows)  is defined as follows: 
    ##          Q4       Q3       Q2       Q1
    ## Q4 0.258273 0.253389 0.246355 0.241983
    ## Q3 0.255436 0.253265 0.246991 0.244308
    ## Q2 0.249780 0.252606 0.247650 0.249964
    ## Q1 0.237414 0.250965 0.248779 0.262842

Observamos que son necesarias 2 generaciones para que la probabilidad de
pasar del Q4 al Q1 sea superior al 22%. En este caso es de 24.2%

> Se quiere conocer cuántas generaciones tienen que pasar para que la
> probabilidad que un hijo esté a los 30 años en cualquier de los
> cuartiles no dependa de la clase inicial

La siguiente igualdad implica que la probabilidad de transición en n
etapas se estabiliza y cualquiera que sea el estado de partida, la
distribución estacionaria contiene la probabilidad de que un hijo esté a
los 30 años en cualquiera de los cuartiles (por consiguiente, en
cualquiera de los estados):

Como en el apartado previo, podemos observar el comportamiento de matriz
P en n pasos.

A partir de la quinta generación la probabilidad de transición no
depende del estado de partida, ya que observamos que va asemejándose a
la distribución estacionaria.

``` r
# Probabilidades de los estados a largo plazo
p4 <- mcP*p3
p5 <- mcP*p4
p6 <- mcP*p5
p6
```

    ## Escala social 
    ##  A  4 - dimensional discrete Markov Chain defined by the following states: 
    ##  Q4, Q3, Q2, Q1 
    ##  The transition matrix  (by rows)  is defined as follows: 
    ##           Q4        Q3        Q2        Q1
    ## Q4 0.2504143 0.2525781 0.2474218 0.2495858
    ## Q3 0.2503608 0.2525720 0.2474281 0.2496391
    ## Q2 0.2502391 0.2525579 0.2474423 0.2497608
    ## Q1 0.2499655 0.2525260 0.2474739 0.2500346
