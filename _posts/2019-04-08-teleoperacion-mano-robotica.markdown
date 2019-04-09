---
layout: post
title:  "Teleoperación de mano robótica"
date: 2019-04-08 18:49:36 +0200
categories: robótica
permalink: /teleoperacion-mano-robotica.html
---
<div align="justify">
<h2>Teleoperación de mano robótica</h2>
<p>
En este post os expondré cómo he realizado el control de una mano robótica teleoperada.
</p>
<p> 
Para ello comenzaré exponiendo los principales tipos de control usados en teleoperación. Escogeré uno y lo aplicaré en una mano que diseñaré. Finalmente concluiré el post evaluando el comportamiento y proponiendo futuras mejoras.
</p>
<h2>Principales tipos de control</h2>
<p>
Los principales tipos de control que conozco son los siguientes:
</p>
<ul>
<li>
<p>
El <b>control unilateral</b> consiste en un maestro que da una serie de órdenes a un esclavo y este último las ejecuta usando su propio controlador. Este controlador no proporciona información de realimentación del esclavo al maestro, por tanto, el maestro no podrá percibir las interacciones que el esclavo realice con su entorno.
</p> 
</li>
<li>
<p>
El <b>control supervisado</b> se caracteriza por ser usado en aplicaciones en las que queramos que el esclavo goce de autonomía. Suele usarse en aplicaciones en las que el retardo sea bastante considerable. Con este control el maestro ordena al esclavo la realización de una tarea definida a alto nivel. El esclavo la realiza con cierto grado de autonomía y realimenta al maestro con información relevante de la operación que ayude en la toma de nuevas decisiones.
</p>
</li>
<li>
<p>
Por último, en el control bilateral el maestro también proporciona órdenes a un esclavo y recibe realimentación de este. Ahora bien, se usa en aplicaciones que tienen bastante menor retraso. Habrán por tanto dos controladores, uno maestro y uno esclavo interconectados. Los tres tipos de controladores bilaterales los desarrollaré a continuación.
</p>
<p>
El <b>control bilateral posición-posición</b> es un sistema bastante estable en su funcionamiento. En régimen permanente la fuerza aplicada del operador sobre el maestro es similar a la realizada por el esclavo sobre el entorno. Ahora bien, como desventaja destacaremos que en régimen transitorio los esfuerzos percibidos por el operador no se asemejan a los que realiza el esclavo sobre el entorno.
</p>
<p>
En lo que respecta al <b>control bilateral fuerza-posición</b>, destacaría que es el más usado en aplicaciones de teleoperación. A diferencia del anterior, es menos estable pero reproduce fielmente la reacción del esclavo sobre el entorno en el operador tanto en régimen permanente como en el transitorio. Por tanto el operador puede percibir en cualquier momento las fuerzas del entorno con el que interactúa. 
</p>
<p>
El <b>control bilateral presión-posición</b> es una modificación del de fuerza-posición. La diferencia reside en que en el de presión-posición no realimentamos en el maestro la fuerza percibida en un punto del esclavo, en cambio, realimentamos los pares de las articulaciones del esclavo. Esto último en el caso de la mano permitirá que la mano maestra perciba esfuerzos cuando cualquier punto de la mano esclava interaccione con los alrededores y no solamente cuando lo haga un punto escogido de la mano. De este modo la percepción del entorno por parte del maestro será más realista.
</p>
</li>
</ul>
<p>
El control que implementaré en la mano teleoperada será el esquema de control presión-posición. He optado por este porque de cara a futuras aplicaciones con nuestro diseño queremos que el maestro le transmita al operario de la manera más fiel posible la interacción en tiempo real que está teniendo el esclavo con el entorno remoto.
</p>
<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/esquema_control_presion_posicion_un_dedo.jpg" alt="Control bilateral presión-posición" width="60%" height="40%">
</center>
<h2>
Descripción del sistema teleoperado
</h2>
<p>
El sistema de teleoperación que plantearé estará compuestos por una mano maestra y otra esclava. Ambas serán cinemáticamente iguales.
</p>
<h2>
Creación del sistema
</h2>
<p>
Lo primero que haré será modelar un dedo que nos sirva para ambas manos. Partiendo de ese dedo generaremos ambas manos robóticas. 
</p>
<p>
Para modelarlos usaremos la aplicación de simmechanics que se encuentra dentro de Matlab. El archivo lo encontraréis en la carpeta de git adjuntada al final del post. 
</p>
<p>
Antes de desarrollar los esquemas de control es importante obtener referencias fiables de lo que deseamos conseguir. En mi caso quiero que la mano maestra ejecute un movimiento de cierre que la esclava trate de reproducir. 
</p>
<p>
El esquema de control presión-posición usa fuerzas del operario como entrada. Yo he considerado que dichas fuerzas son las que se aplicarán sobre cada articulación de la mano maestra. Ahora bien, su obtención a ojo es complicada. Por tanto decidí implementar con Matlab y simmechanics un control por compensación de gravedad. En él especifiqué en radianes las posiciones deseadas que quería que tuvieran las articulaciones de los dedos cuando se cierran y usando sensores de par obtuve lecturas de fuerza hasta que los dedos alcanzaran dichas posiciones. A continuación muestro el esquema de simulink con el que obtuve las referencias.
</p>
<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/diagrama_control_compensacion_gravedad.jpg" alt="Diagrama de control de compensación de gravedad" width="70%" height="40%">
</center>
<p>
Dado que he considerado que todos los dedos de la mano son iguales, las fuerzas ejercidas sobre cada articulación para realizar el movimiento de cierre serán las mismas. Las fuerzas las almacené en los vectores signal1, signal2, signal3 y signal4, correspondientes a cada articulación.Los datos están presentes en DatosTeleoperacionBilateral.mat.
</p>
<p>
Dentro del archivo ControlCompensacionGravedad.m encontrarás el código del controlador.
</p>
<xmp>
%Posición articular deseada
qd = [0; 0; pi/2; pi/2];

%Posición articual actual
q = u(1:4);

%Velocidad articular actual
qv = u(5:8);

%Error de posición 
Q = qd - q

%Ganancias del controlador
Kp = 10;
Kv = 10;

%Vector de gravedad
g = dedoRobot.gravload(transpose(q));

%tau

tau = Kp*Q - Kv*qv + transpose(g);

salida = tau;
</xmp>

<h2> Implementación del control</h2>
<p>
La mano está compuesta por cuatro dedos, por tanto implementaré un control bilateral presión-posición a cada uno de ellos. La composición de cada uno de los esquemas de control forma el control de la mano completa. El diagrama simulink creado para controlar un dedo se muestra a continuación.
<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/esquema_control_bilateral_presion_posicion.jpg" alt="Diagrama de control de un dedo" width="80%" height="70%">
</center>
</p>

<p>
Al anterior esquema le añadiremos los retardos existentes en la comunicación para asemejar más su funcionamiento a la realidad. El esquema implementado lo muestro a continuación.
</p>


<p>
Los valores de Km y Ks los obtuve empíricamente probando el sistema y viendo que valores proporcionaban la mejor respuesta. El valor de Kp he supuesto que es 0 porque he querido asegurar que el esclavo sigue en todo momento el movimiento del maestro sin interacción de fuerzas externas. En caso de que desee observar el comportamiento del  sistema ante la interacción del esclavo con algún elemento, el valor Kp lo cambaré en función del tipo de interacción. Para esta práctica supondre que Kp siempre valdrá 0. Los bloques dedoMaestro y dedoEsclavo representan el modelo de un dedo de la mano maestra y su análogo en la mano esclava.
</p>
<p>

Por cada dedo de la mano esclava que controlaré implementaré el esquema anterior. Por tanto, la mano está compuesta por cuatro esquemas bilaterales que hemos almacenado en subsistemas, cada uno representativo de un dedo. A continuación muestro el modelo de simulink que representa la mano.

</p>

<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/esquema_control_mano_robotica.jpg" alt="Esquema de control de la mano" width="100%" height="60%">
</center>

<p>
Las señales de entrada a cada uno de los bloques corresponden con las señales de fuerza correspondientes a cada articulación de los dedos de la mano.
</p>
<h2>
Resultados
</h2>
<p>

La teleoperación de la mano esclava sin retardo en las comunicaciones es excelente, los movimientos de la mano maestra se reproducen a la perfección en la esclava. Ahora bien, a medida que aumentamos el retardo el comportamiento se vuelve más inestable y los movimientos de la mano esclava se asemejan menos a la maestra.
<p>
A continuación muestro los resultados ante distintos retardos. La mano maestra está a la derecha y la esclava a la izquierda. El primer gif muestra el comportamiento cuando no existe retardo, en el segundo el retardo es de 0.3s y en el último de 0.5s.

<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/simulacion_0s_retraso.gif" alt="Simulación" width="40%" height="40%">
</center>

<br>

<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/simulacion_03_retraso.gif" alt="Simulación" width="40%" height="40%">
</center>
<br>
<center>
<img src="/assets/image/teleoperacion-de-mano-robotica/simulacion_05s_retraso.gif" alt="Simulación" width="40%" height="40%">
</center>

</p>
</p>
<h2>
Conclusiones
</h2>
<p>
Considero que el sistema de teleoperación creado para simular el control de la mano es bastante satisfactorio. En este post  he simulado únicamente un control de movimiento de cierre sin obstáculo alguno. Una futura mejora sería que la mano esclava interactuara con objetos y que el operador con la mano maestra sintiese la interacción. Por tanto, haber optado por un control bilateral presión-posición ha sido un acierto. Esto se debe a que gracias a él podremos realimentar las fuerzas que siente la mano esclava con el fin de que la mano maestra las sienta y por ende también el operador que la maneje. 
</p>
<p>
Otra futura mejora sería emplear como canal de comunicación entre el maestro y el esclavo variables de onda para paliar las inestabilidades causadas por los retrasos en el control bilateral de cada dedo. He intentado hacer uso de este método de comunicación pero la implementación no ha funcionado como se esperaba retrasando bastante la ejecución de la simulación. Por tanto espero en un futuro dar con la tecla e implementarlo correctamente.
</p>
</div>
