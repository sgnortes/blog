---
layout: post
title:  "Ordenación de enjambres de robots"
date: 2019-04-07 18:49:36 +0200
categories: robótica
permalink: /ordenacion-de-enjambres-de-robots.html
---
<div align="justify">
<p>
Este fin de semana realizando un trabajo para la asignatura de Sistemas Multirobot, tuve que investigar a cerca de comportamientos de ordenación bioinspirados. Para ello me leí el paper referenciado al final del post, que me pareció interesantísimo y me gustaría compartir.
</p>
<h2>Introducción</h2>
<p>
Investigaciones recientes han demostrado que los insectos sociales ordenan sus nidos siguiendo patrones sofisticados. Estos patrones surgen espontáneamente de las interacciones dinámicas que tienen los insectos durante el proceso de deposición y retirada del nido.

Durante el proceso ningún plano espacial o representación global son requeridos, ni decisiones tomadas de forma jerárquica son realizadas. Mediante la interacción  con otros individuos y con el entorno, los individuos actúan siguiendo sus propios objetivos y conocimiento relativo al espacio explorado.

El sistema de ordenamiento de los insectos sociales posee muchas características atractivas como son la escalabilidad, la flexibilidad y robustez. Debido a ello, modelos abstractos basados en estos comportamientos han sido aplicados en muchas áreas como son:
</p>
<ul>
	<li>La exploración</li>
	<li>Ordenamiento colectivo</li>
	<li>Minería de datos</li>
	<li>Análisis de datos numéricos</li>
</ul>
<p>
El ordenamiento de objetos llevado a cabo por enjambres de robots es una tarea complicada, que hace uso de mecanismos de auto-organización, toma de decisiones colectiva y reconocimiento de patrones. 
</p>
<h2>Estudios anteriores</h2>
<p>
Modelos abstractos relativos a estos comportamientos fueron propuestos por Deneubourg et al. Sus modelos se basaban en sencillas reglas que se usaban para determinar las probabilidades de coger y dejar objetos. La mayor desventaja de estos modelos es la complejidad de obtener la densidad de objetos locales. El uso mínimo de sensores impone también un reto en la obtención de resultados similares a los vistos en insectos reales. Este método fue extendido para ordenar más de tres tipos de objetos. Para ello usaron una cámara que grababa el comportamiento de los robots desde arriba. Mediante la cámara se podían identificar los robots, las orientaciones y posiciones de los objetos, e información global a cerca del entorno. Este enfoque fue extendido añadiendo una cámara a cada robot, que únicamente permite a los robots compartir datos de imágenes con sus vecinos, pero también permite a los robots estimar tamaños de clusters (agrupaciones). Con ambos enfoques se obtienen buenos resultados, pero es complicado aplicar esto en robots reales, particularmente en enjambres de robots, debido a los sensores complejos y sofisticados aplicados.
</p>
<h2>Estudio propuesto</h2>
<p>
Para afrontar este problema, en este paper se describe un método de ordenación basado en el uso de información local obtenido por sensores sencillos acoplados a robots. Al igual que en el estudio de Deneubourg et al, la nueva propuesta también usa algoritmos basados en reglas de comportamiento simples. 
</p>
<h2>Algoritmo</h2>
<p>
La mejor manera de explicar el algoritmo propuesto es poniendo ejemplos, para ello me basaré en el escenario de simulación propuesto en la investigación.
</p>
<p>
El escenario usado consiste en tres franjas, dos negras en los extremos y una blanca en medio. Hay dos grupos de robots, un grupo de color verde y otro rojo. Cada grupo se posicionará en una de las franjas negras. Para ello se organizan en tiempo real interactuando con otros robots y el entorno. Este último es desconocido para los robots y podría consistir en otra disposición de las franjas negras. Los robots únicamente usan información local y no global, por tanto no tienen planos espaciales y tampoco pueden atender decisiones tomadas de forma jerárquica por un superior.
</p>
<center>
<img src="/assets/image/ordenacion-de-enjambre-de-robots/escenario.jpg" alt="Disposición inicial y final en el escenario" width="60%" height="40%">
</center>
<p>
El comportamiento de los robots es el siguiente:
</p>
<ul>

<li>Los robots se disponen aleatoreamente en el escenario.</li>
<li>Estos comienzan a moverse buscando las franjas negras.</li>
<li>Cuando un robot encuentra una franja negra se detiene y se comunica con robots cercanos a él.</li>
<li>Los robots cercanos se desplazan a su posición y se detienen</li>
<li>Los robots detenidos se comunican entre ellos y cuando en una franja se determina una mayoría de robots pertenecientes a un grupo, se obliga a los robots del grupo contrario a salir de su franja y adentrarse en la blanca.</li>
<li>Esto fuerza a que cada vez haya una mayoría en una franja de las franjas negras.</li>
<li>Es importante destacar que los robots se detienen en las líneas que delimitan las franjas negras con la blanca generando una barrera. Esta distribución permite que únicamente los robots de un mismo grupo puedan pasar e impida el paso a los de la otra agrupación. Cuando el número de robots de un grupo situados en una franja supere un umbral, estos podrán adentrarse en la franja.</li>
</ul>
<p>
El algoritmo de control se divide en los siguientes módulos:
</p>
<ul>
	<li>Contador de robots: los robots parados cuentan el número de robots distintos situados en la misma franja comunicándose con ellos.</li>
	<li>Minoría de robots que se van: después de contar el número de robots existentes en una franja, ellos saben mediante sus identificadores a qué grupo pertenecen y si representan una minoría. Si lo son dejan la barrera y se adentran en la franja blanca.</li>
	<li>Formación de barrera: cuando un robot que se mueve se comunica con un robot cercano parado, este se mueve circularmente alrededor de él en busca de la franja negra, hasta encontrarla y detenerse en la barrera, formando parte de esta.</li>
	<li>Incorporación en la franja negra: cuando la mayoría de robots supera un umbral, los robots del mismo grupo podrán adentrarse dentro de la franja de color negro.</li>
</ul>
<p>
La comunicación entre robots llevada a cabo para realizar la ordenación, recibe el nombre de “gossip communication”, que yo he traducido como “cuchicheo”. La comunicación será llevada a cabo por pares de robots que comparten información local, siempre y cuando ambos robots se encuentren dentro de un rango de comunicación. El objetivo del cuchicheo es contar el número de robots pertenecientes a un grupo en los enjambres creados. 
</p>
<p>
Cada robot que se pare en el límite de una franja se comunica con los robots que se encuentran en su alcance de comunicación, es decir sus vecinos. En la comunicación establecida a pares entre el robot parado y los otros robots (que pueden o no estar parados) se intercambia información local. La información intercambiada consiste del número de robots pertenecientes a cada grupo situados en el límite de la franja. Esta información la percibe el propio robot o se la transmite a través de cuchicheos. Pasados varios cuchicheos cada robot situado en una franja conoce la cantidad de robots pertenecientes a un grupo u otro.
</p>
<center>
<img src="/assets/image/ordenacion-de-enjambre-de-robots/esquema_de_comunicacion.jpg" alt="Esquema que ejemplifica la comunicación mediante cuchicheos" width="60%" height="40%">
</center>
<p>
En el ejemplo anterior se muestra la comunicación entre robots situados en el límite de la franja. Se pueden percibir tres grupos:
</p>
<ul>
	<li>S1 = G1.</li>
	<li>S2 = G2, G3, R1, R2, G4, R3 y G5.</li>
	<li>S3 = R4, G6, G7.</li>
</ul>
<p>
El prefijo G indica que el robot pertenece al grupo verde y el R al rojo. En el grupo S1 hay un color dominante que es el verde, en el S2 el verde domina y en S3 también. Los robots rojos serán expulsados de la barrera , volverán a la franja blanca y se reorganizarán. Es cierto que los robots rojos pueden volver al mismo límite anterior, pero la probabilidad de que esto ocurra a medida que pase el tiempo será menor, fomentando así su desplazamiento al otro límite.
Cuando el número de robots situados en el límite sea superior a un umbral estos podrán desplazarse dentro de la franja negra.
</p>
<h2>Resultados</h2>
<center>
<img src="/assets/image/ordenacion-de-enjambre-de-robots/resultados.jpg" alt="Resultados" width="70%" height="70%">
</center>
<p>
De los experimentos realizados puede observarse que cuanto mayor sea el enjambre que deseamos ordenar, mayor tiempo invertirá el algoritmo. El tiempo de ejecución también dependerá de la distribución de robots verdes y rojos que deseemos usar. Cuanto más dominante sea un grupo menos tarda el algoritmo.
</p>
<p>
También podemos deducir que la distribución que mejor resultados de ordenación da es aquella en la que se emplean cantidad de robots equitativas en ambos grupos.
</p>
<h2>Futuras líneas de investigación</h2>
<p>
En un futuro podríamos optar por ordenar objetos y no robots. Es decir, los robots se desplazarían buscando objetos, los cogerían y en función de la clase los dejarían en una u otra zona, posteriormente seguirían buscando. Los resultados obtenidos en la investigación nos hacen deducir que el número de robots empleados para clasificar cada clase deberá ser igual.
</p>
<p>
El sistema de comunicación basado en cuchicheos podríamos ponerlo a prueba simulando perturbaciones en la comunicación. Esta situación podría dar lugar a la investigación de nuevas metodologías de comunicación que sean más robustas.
</p>
<p>
Por último, todo lo anterior podríamos implementarse en robots reales para ver su desempeño y comportamiento en el mundo real.
</p>
</div>
<h3>Referencia</h3>
<p>- H.  Ding et al, <i>Sorting in Swarm Robots Using Communication-Based Cluster Size Estimation</i> (2014), (available at https://www.researchgate.net/publication/266078839).
</p>