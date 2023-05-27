# NodeRed-Flow4G12

En esta cuarta práctica de NodeRED vamos a realizar una estación climatológica básica utilizando Mosquitto. Para ello se requiere lo siguiente:

1. Docker, con los servicios de Mosquitto y NodeRED instalados

- Nota: Debido a algunos errores por los autores humanos (fuera del alcance del autor de este documento), el archivo

    ~~~
    compose.yaml
    ~~~
    
    será modificado. Si aún no lo tiene, comience por [aquí](https://github.com/Vanadiox/NodeRed-Flow1G12); sólo hasta la creación de dicho archivo y las carpetas. Aún no inicie los servicios. 

    Si ya ha tenido el archivo desde el principio, comience por detener los contenedores con el siguiente comando en la terminal:

    ~~~
    docker stop $(docker ps -a -q)
    ~~~

    Ahora detenga Docker Compose. Para ello vaya a la carpeta donde se encuentra el archivo, que en este caso es ~/DockerCompose/

    ~~~
    docker compose down
    ~~~

    Esto no sólo detendrá Docker Compose, sino también va a eliminar los contenedores que ya se han realizado; paso que es necesario, aunque _per se_ sólo se necesita eliminar mosquitto. 

    Ahora, hay que eliminar la imgene de Mosquitto, para ello ejecute:

    ~~~
    docker image ls -a
    ~~~

    Anote el id de la imagen de Mosquitto y ejecute lo siguiente:

    ~~~
    docker rmi [mosquitto id]
    ~~~

    Ahora, los usuarios que ya tenían el archivo compose.yaml y los que lo acaban de crear, editen dicho archivo, modifiquen estas líneas:

    ~~~
    image: eclipse-mosquitto
    #volumes:
      #- ~/DockerVolumes/Mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf
    ~~~

    Debe quedar así: 

    ~~~
    image: eclipse-mosquitto
    volumes:
      - ~/DockerVolumes/Mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf
    ~~~

    Y ya que estamos aquí, modifiquemos la sección de MySQL. Las líneas a modificar son las siguientes: 

    ~~~
    volumes:
      #- ~/DockerVolumes/MySQL/config/my.cnf:/etc/mysql/my.cnf
      - ~/DockerVolumes/MySQL/data:/var/lib/mysql
    ~~~

    Ahora debe quedar así: 

    ~~~
    volumes:
      - ~/DockerVolumes/MySQL/config/my.cnf:/etc/mysql/my.cnf
      - ~/DockerVolumes/MySQL/data:/var/lib/mysql
    ~~~

    En general, debe verse así: 

    ![Imagen 1]()

    Ahora, hay que agregar dos archivos más. El primero se llama **mosquitto.conf** y este se ubicará en

    ~~~
    ~/DockerVolumes/Mosquitto/config/
    ~~~

    Para ello, dirígase a esa ubicación, abra una terminal y escriba lo siguiente: 

    ~~~
    touch mosquitto.conf
    ~~~

    El archivo debería tener esta ruta: 

    ~~~
    ~/DockerVolumes/Mosquitto/config/mosquitto.conf
    ~~~

    Ahora abra este archivo con su editor de texto de favorito y agregue las siguientes lineas (En caso de no usar nano o vim o cualquier editor basado en consola, cierre la terminal):

    ~~~
    #Configuración de Mosquitto

    # Configuración de escucha
    listener 1883
    allow_anonymous true
    ~~~

    Guarde y cierre. Ahora, a hacer lo mismo con el archivo **my.cnf** de MySQL. Para ello, vaya al siguiente directorio:

    ~~~
    ~/DockerVolumes/MySQL/config/
    ~~~

    De la misma forma, abra una terminal en esta ubicación y teclee

    ~~~
    touch my.cnf
    ~~~

    Ahora, misma dinámica: abra el archivo con su editor de texto favorito (sino usa la consola, ciérrela) e ingrese el código que se encuentra en [este enlace](https://github.com/codigo-iot/servidor-IoT-basico-docker-compose/blob/main/my.cnf)

    Guarde y cierre. 

    El archivo debería tener esta ruta:

    ~~~
    ~/DockerVolumes/MySQL/config/my.cnf
    ~~~

    ![Imagen 2]()

    Ahora, hay que levantar de nuevo el archivo de compose.yam. Para ello, de nuevo vaya a 

    ~~~
    ~/DockerCompose/
    ~~~

    y en consola ejecute lo siguiente:

    ~~~
    docker compose up -d
    ~~~

    Espere a que termine y ya podrá acceder a NodeRED para continuar. 

## Instrucciones:

Por ahora, cree un _flow_ nuevo, haciendo clic en el botón **+** 

![Imagen 3]()

Ahora hay que agregar:

- Un módulo _mqtt in_

![Imagen 4]()

- Un módulo _json_

![Imagen 5]()

- Dos módulos _function_

![Imagen 6]()

- Dos módulos _gauge_

![Imagen 7]()

- Y un módulo _chart_

![Imagen 8]()

Ordénelos de la siguiente manera:

![Imagen 9]()

Y unalos como se muestra a continuación:

![Imagen 10]()

Ahora para configurar el módulo _mqtt in_ de doble clic sobre este y saldrá una ventana como esta: 

![Imagen 11]()

En el campo _Topic_ teclee lo siguiente:

~~~
codigoIoT/mqtt/clima
~~~

Luego vaya al ícono de lápiz, aparecerá otra ventana. En el campo _Name_ teclee

~~~
mosquitto
~~~

Del mismo modo, en el campo _server_. Se debe ver así: 

![Imagen 12]()

Luego, de clic en el botón rojo que dice _Update_. Será llevado a la ventana anterior y se debe ver así:

![Imagen 13]()

De click en _Done_. Para el módulo json, haga doble clic y en la pestaña _action_ seleccione 

~~~
Always convert to JavaScript Object. 
~~~

![Imagen 14]()

De clic en _Done_. Para los nodos de _function_ se realizará lo siguiente: En el primero, haga doble clic. En el campo _Name_ teclee "Temperatura". Y en la pestaña _On Message_ coloque lo siguiente: 

~~~
msg.payload = msg.payload.temp;
msg.topic = "Temperatura";
return msg;
~~~

![Imagen 15]()

De clic en Done y en el otro nodo _function_, el _Name_ es "Humedad". En la pestaña _On Message_ meta esto:

~~~
msg.payload = msg.payload.hum;
msg.topic = "Humedad";
return msg;
~~~

![Imagen 16]()

De clic en _Update_ y hasta aquí estará todo listo. Para los 3 nodos restantes hay que crear una pestaña nueva y 2 grupos. En la siguiente imágen se muestra dónde se encuentra la opción _dashboard_ para colocar dichos objetos. 

![Imagen 17]()

Una vez abierto el _dashboard_, añada una pestaña con el botón "**+ tab**" y una vez añadida la pestaña ponga el cursor sobre la pestaña y de clic en _edit_ para cambiar el nombre por "ClimaLocal". Ahora añada dos grupos con el botón "**+ group**". Cambie el nombre de ambos; uno de ellos se llamará "Indicadores" y el otro "Grafica". El producto final debe verse así: 

![Imagen 18]()

Ahora centre su atención en los últimos 3 nodos. 2 de ellos son del tipo _gauge_, los cuales estarán dentro del grupo indicadores. Para ello, de clic sobre uno de ellos. En el menú desplegable seleccione **[Clima local] Indicadores**.

![Imagen 19]()

En el campo _Label_ escriba: 

~~~
Temp
~~~

En _Units_ escriba:

~~~
°C
~~~

Como este nodo mostrará información relacionada a temperatura, los valores que se utilizarán serán de entre 10 a 40. Ya usted decidirá si desea esos valores. Para ello, en el apartado _Range_ escriba su valor mínimo y máximo. Luego seleccione sus colores, de nuevo, indicando un color para el, mínimo, medio y máximo. Y en _Sectors_, usted decidirá en qué valores se hace el cambio. En este ejemplo se utilizaron los valores 22 y 32. 

![Imagen 20]()

Haga clic en _Done_ para aceptar los cambios. Ahora haga lo mismo con el otro nodo _Gauge_. Recuerde, este otro nodo es para la humedad. Este valor va entre 0 a 100; no hay más, ya que es un valor universal. En la siguiente imágen se muestra cómo yo lo configuré. 

![Imagen 21]()

Por último, el nodo _chart_. Este nodo estará en el grupo "Grafica" y en _Label_ se llamará "Historico". Quedaría de la siguiente manera:

![Imagen 22]()

De clic en _Update_ y ya está listo para hacer el _Deploy_. Vaya a esta [dirección](http://localhost:1880/ui/) para ver sus gráficas. 

- Nota: Si usted detuvo los _flows_ anteriores, el gráfico debería aparecer sin problemas. En caso de que usted no lo haya hecho, le aparecerán los _flows_ anteriores. Para poder visualizar su trabajo usted verá al lado izquierdo del nombre del flow un menú tipo hamburguesa. De clic ahí y seleccione el _flow_ que desea ver, en este caso es el _flow_ 4. 

![Imagen 23]()

Por el momento, se verá así: 

![Imagen 24]()

Ahora, hay que enviar un mensaje por medio de MQTT. Para ello vaya a la terminal de comandos e identifique cuál es el servidor de MQTT que usted tiene en ejecución. Para ello teclee 

~~~
docker container ls -a
~~~

Aparecerán los contenedores que tiene disponible. Centre su atención en las primeras dos columnas, llamadas _id_  e _image_. Vea que primero se muestran una serie de números y letras, seguido de un nombre. Fije su atención en la fila que dice _eclipse-mosquitto_ y a la izquierda se encuentra su id. Copie la _id_ y peguela en el siguiente comando:

~~~
docker exec -it [id] mosquitto_pub -h localhost -t [topic] -m '{"temp":x,"hum":y}'
~~~

- Donde _id_ es el id del contenedor
- _topic_ es el tema que utilizamos, que es **codigoIoT/mqtt/clima**
- "temp":x, donde la x es el valor, comprendido entre el mínimo y máximo que usted puso en el _gauge_ "temp". Para este nodo los valores van entre 10 y 40
- "hum":y, del mismo modo, es el porcentaje de humedad, que va de 0 a 100

El comando completo que usted deberá utilizar se vería algo como esto: 

~~~
docker exec -it 24d37aa0f028 mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":18,"hum":60}'
~~~

Introduzca los valores correspondientes a su caso y de enter. Los valores se deberían reflejar en su gráfica: 

![Imagen 25]()

# Y LISTO

Usted ha configurado una estación climatológica básica

![Imagen 26]()

## Referencias: 

[Flow 1](https://github.com/Vanadiox/NodeRed-Flow1G12)

[Estación climatológica básica: Código IoT](https://github.com/codigo-iot/servidor-IoT-basico-docker-compose)

## Créditos

Manual creado por [Vanadiox (Kevin H)](https://github.com/Vanadiox)

[Código IoT](https://github.com/codigo-iot)