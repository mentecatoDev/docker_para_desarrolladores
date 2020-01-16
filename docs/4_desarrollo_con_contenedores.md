# 4 Desarrollo con contenedores

 

## 4.1 Docker Compose y docker-compose.yml

Docker compose es otro proyecto open source que permite definir  aplicaciones multi-contenedor de una manera sencilla y declarativa. Es una herramienta ideal para gestionar entornos de desarrollo, pero también para configurar procesos de integración continua.

`docker-compose` es una alternativa más cómoda al uso de los comandos `docker run` y `docker build`, que resultan un tanto tediosos cuando trabajamos con aplicaciones de  varios componentes. Con Docker Compose se define un fichero  docker-compose.yml que tiene esta forma (tomado de  docker-for-devs/auto-build/docker-compose.yml): 

```
web:
  build: .
  ports:
   - "5000:5000"
  depends_on:
   - redis
redis:
  image: redis
```

donde estamos definiendo una aplicación que se compone de un contenedor definido desde un `Dockerfile` local, que escucha en el puerto 5000, y que hace uso de `redis` como un servicio externo. Dada esta definición, la manera de levantar la aplicación es simplemente: 

```bash
docker-compose up -d
```

`docker-compose` acepta distintos comandos, una lista completa puede encontrarse [aquí](https://docs.docker.com/compose/reference/). 

## 4.2 Comandos comunes de compose

Destacar los siguientes puntos sobre *docker-compose*

- `docker-compose up -d` levanta la aplicación en modo demonio, `docker-compose up` la levanta en primer plano, mostrando los logs de los distintos contenedores. La ejecución sucesiva del comando `docker-compose up -d` sólo recrea los contenedores que hayan cambiado su imagen o su definición. `docker-compose up -d` no hace el build cada vez que es invocado de las imágenes locales. Si  deseas actualizar tu aplicación en base a los últimos cambios de tu  código, tendrás  que ejecutar `docker-compose up --build -d`. Un truco para mejorar este proceso es montar tu código como un volumen en el fichero `docker-compose.yml`, de tal manera que tu container siempre ve los últimos cambios en tu  código fuente. Si quieres levantar solo uno o varios de los servicios en un compose, puedes añadir su nombre, por ejemplo `docker-compose up -d redis`.
- `docker-compose pull` actualiza las imágenes  definidas en el compose con la versión actual que haya en el registro.  En otras palabras, si alquien hace un push al registro, actualiza la  versión de estas imágenes en nuestra máquina. Con la opción `--parallel` hace el pull en paralelo. Como todo comando de *docker-compose* se puede hacer pull de un subconjunto de servicios: `docker-compose pull servicioA ServicioB`.
- `docker-compose build` reconstruye las imágenes de los servicios que tengan una sección de `build` definida. Opciones interesantes son: `--no-cache` para invalidad la caché, `--pull` para hacer pull de las imágenes base y `--build-arg key=val` para pasar argumentos. Como todo comando de *docker-compose* se puede hacer build de un subconjunto de servicios: `docker-compose build servicioA ServicioB`.
- `docker-compose push` pushea al registro la versión local de las imágenes con una sección de `build` definida. Como todo comando de *docker-compose* se puede hacer pull de un subconjunto de servicios: `docker-compose push servicioA ServicioB`.
- `docker-compose run` ejecuta un contenedor de uno de los servicios definido en el compose. La diferencia principal con `docker-compose up` es que permite definir el comando a ejecutar, así como otra información de contexto como variables de entorno, el `entrypoint`, volúmenes, el directorio de trabajo… Es uno de los comandos más útil  para el entorno del desarrollador. Por ejemplo, podemos definir un  servicio en nuestro *docker-compose.yml* con todas las dependencias necesarios para ejecutar nuestros comandos de desarrollo. Haciendo uso de `docker-compose run` podemos ejecutar comandos aleatorios en ese entorno, evitando la  necesidad de instalar todas las dependencias del entorno en la máquina  actual. El caso más común es tener un servicio para ejecutar tests, como veremos más adelante, pero podríamos tener para cualquier tipo de  tarea.
- `docker-compose rm` elimina los contenedores y otros recursos como redes, creados a partir de un compose.

*docker-compose* permite definir prácticamente todos los flags que soportan tanto el comando *docker run* como el *docker build*, pero *docker-compose* es mucho más fácil de utilizar. Las opciones más comunes son:

- *build*: para indicar que el container se construye desde un Dockerfile local. Puede tener subcampos como `context`, `dockerfile`, `cache_from` o `args`.
- *image*: para indicar que el container corre un imagen remota. También indica el nombre de la imagen que se crea si hay un campo `build`.
- *command*: para redefinir el comando que ejecuta el container en lugar del comando definido en la imagen.
- *environment*: para definir variables de entorno en el contenedor. Se pueden pasar haciendo referencia a un fichero usando la propiedad *env_file*. Si la variable no tiene un valor dado, su valor se cogerá del entorno de shell que ejecuta el `docker-compose up`, lo que puede ser útil para pasar claves, por ejemplo.
- *depends_on*: para definir relaciones entre contenedores.
- *ports*: para mapear los puertos donde el contenedor acepta conexiones.

[Aquí](https://docs.docker.com/compose/compose-file/) tenéis una lista completa y actualizada de las opciones que permite *docker-compose*.

## 4.3 Volúmenes

Cuando un contenedor es eliminado, la información contenida en él desaparece.  Para evitar este problema y que los datos generados en el interior de un contenedor no se eliminen cuando el contenedor termina podemos hacer uso de volúmenes de datos (*data volume*). Un volumen es un directorio dentro del contenedor que se asocia con un directorio del host, por lo que persiste a la finalización del contenedor. Un contenedor puede tener varios volúmenes, y un mismo volumen puede  montarse en varios contenedores para compartir información.

#### 4.3.1 Volúmenes de Datos

Los volúmenes de datos tienen las siguientes características:

- Cuando borramos el contenedor, no se elimina el volumen asociado.
- Nos permiten guardar e intercambiar información entre contenedores.
- No son gestionados por los storage drivers, por lo que las operaciones de entrada / salida son mucho más eficientes.

Los volúmenes de datos tienes su propia interfaz con la línea de comandos de *docker*:

- *docker volume create*: crea un nuevo volumen de datos.
- *docker volume ls*: muestra los volúmenes de datos de nuestra máquina.
- *docker inspect*: devuelve información relativa a un volumen.
- *docker volume rm*: elimina un volumen de datos. También se pueden eliminar automáticamente al eliminar un contenedor si ejecutamos `docker rm -f`.

Si hacemos un `docker inspect` de un contenedor con volúmenes asociados esta información aparece en el campo `Mounts`.
Por último podemos comprobar que aunque borremos el contenedor, el volumen no se borra.

#### 4.3.2 Volúmenes del Host

Una aplicación particular de los volúmenes es la posibilidad de  montar en el contenedor un directorio ya existente en el host. En este  caso hay que tener en cuenta que si el directorio de montaje del  contenedor ya existe, no se borra su contenido, simplemente se monta  encima. Veamos un ejemplo:

```
$ docker run -it -v `pwd`:/data alpine sh
# cd data
# ls
```

#### 4.3.3 Gestión de Volúmenes con *docker-compose*

El siguiente ejemplo ilustra la gestión de volúmenes con *docker-compose*:

```
version: '3.4'
services:
  mysql:
    image: mysql
    volumes:
      - mysql:/var/lib/mysql
      - logs:/var/log/mysql
      - /etc:/etc
  mysql:
    image: log-analizer
    volumes:
      - logs:/var/log:ro
volumes:
  data:
  logs:
```

## 4.4 Redes

Docker nos permite crear diferentes redes virtuales para nuestras necesidades, ya bien para unir o segmentar diferentes contenedores. De esta manera,  podemos separar contenedores por seguridad en redes diferentes, o  unirlos en la misma por conveniencia o por conectar sus servicios entre  sí.

Por defecto, Docker nos ofrece tres tipos de redes diferentes. La  primera, bridge, es donde arrancarían todos nuestros contenedores por  defecto. Es una red que crea un puente entre la interfaz de red del  contenedor que arrancamos y una interfaz de red virtual que se crea en  nuestro equipo cuando instalamos Docker. La siguiente sería host. Host  lo que hace es copiar la configuración de red del host, es decir, del  servidor o máquina donde está Docker en el contenedor que estamos  arrancando. Si arrancamos un contenedor aquí y ejecutamos la revisión de la configuración de red, veremos que es la misma que la de la máquina  en la que lo estamos corriendo. Y después tenemos la red none, que  utiliza el driver null, que lo que hace es eliminar toda la  configuración de red de nuestro contenedor. Si creamos un contenedor  aquí, solo tendremos loopback, solo tendremos la dirección 127.0.0.1, y  no podremos conectar ningún sitio más.

Cuando instalamos Docker, veremos que nos crea una interfaz llamada  docker0. Tiene una dirección IP privada, probablemente esta, si no da  colisión con ninguna otra dirección IP que tengáis configurada. Y cuando creáis algún tipo de contenedor que se conecta a la red bridge, lo que  hace es recibir por DHCP una dirección IP de este rango. Podéis conectar a través de él. Todos vuestros contenedores harán NAT a través de esta  IP, y a través de la IP de salida de la máquina host o servidor en la  que tenéis Docker instalado. Esta es vuestra red por defecto. De la  misma manera, podéis conectar desde aquí a través de esta interfaz y por esta dirección IP a las direcciones IP de los contenedores que tenéis  corriendo en esta red.

Esta red bridge no es la única que podéis tener, podéis crear más  para separar vuestros contenedores en diferentes redes. Para ello,  tendréis que ejecutar el siguiente comando: “docker network create”.  Tenéis que elegir el driver del tipo de red que queréis crear. Lo más  probable es que sea una red bridge. Y el nombre de la red. Veréis dos  cosas, una que tenemos una red nueva con el nombre red1, driver bridge y un identificador. Otra, que si hacemos un ifconfig, vemos también que  tenemos una interfaz virtual de red nueva en la que tenemos las siglas  “br” de bridge y la ID de nuestra red. Si vemos esa interfaz en  concreto, vemos que tenemos una nueva dirección IP. Dentro de este rango de red, aparecerán todos los contenedores que nosotros ejecutemos en  red1. Y tanto red1 como bridge serán redes separadas. Los contenedores  que tengamos en bridge, la red bridge por defecto, y los contenedores  que tengamos en red1, la otra red bridge que hemos creado no se podrán  comunicar entre sí.

Aparte, todo lo que arranquéis con Docker Compose, tendrá una red  privada para él. Docker Compose creará una red cada vez que levantéis  una infraestructura completa. Como veis, es bastante sencillo manejar  las redes de Docker. Podéis segmentar todas las aplicaciones que  corráis, por seguridad o por el sistema que utilicéis para conectarlas.  Por último, podéis usar *docker-compose* para crear nuevas redes o lanzar los contenedores en redes específicas.

```
version: '3.4'

services:
  proxy:
    image: busybox
    networks:
      - outside
  app:
    image: busybox
    networks:
      - default
      - inside

networks:
  outside:
    external: true
  default:
  inside:
     driver: bridge
    enable_ipv6: true
```

Las redes tienen su propia interfaz con la línea de comandos de *docker*:

- *docker network create*: crea una nueva red.
- *docker network ls*: muestra las redes de nuestra máquina.
- *docker inspect*: devuelve información relativa a una red.
- *docker network rm*: elimina una red.

## 4.5 Casos de uso

A continuación destacamos algunos casos de uso comunes usando *docker-compose*.

##### 4.5.1 Service Discovery.

Por defecto, *docker-compose* crea una red para correr los servicios definidos en el fichero *docker-compose.yml* del tipo *bridge* y con el nombre del directorio actual. Dentro de dicha red, los  contenedores son accesibles con el nombre del servicio. En el vídeo  podemos ver una demo.

##### 4.5.2 Variables de Entorno.

En los valores de todos y cada uno de los campos del  fichero *docker-compose.yml* podemos hacer uso de la notación `${VERSION:-value}`, para tomar el valor de la variable `$VERSION`, o el valor `value` si la variable `$VERSION` no está definida. Por ejemplo, podríamos usarlo para parametrizar la versión de go que queremos usar:

```
example:
    image: golang:${GO_VERSION:-1.9}
    command: go test ./...
```

##### 4.5.3 Montar Directorio de Trabajo.

Aunque el proceso de hacer *build* de una imagen está muy  optimizado gracias al uso de la caché de Docker, aunque solo sea para  mandar el contexto a la api de Docker, suele tardar algunos segundos  como poco. Por tanto, construir una imagen de Docker para cambio de  código que hagamos puede resultar ineficiente.

La solución es es montar el directorio de trabajo actual el  contenedor que estemos ejecutando. Por ejemplo, el siguiente servicio  corre los tests de una aplicación Go sin necesidad de construirlo en  cada ejecución:

```
  test:
    image: golang:1.9
    working_dir: /go/src/app
    volumes:
      - ${PWD}:/go/src/app
    command: go test ./...
```

##### 4.5.4 Montar Docker Socket

Otro caso muy común es cuando necesitamos que un contenedor acceda a  la API de Docker. Una solución que nosotros no recomendamos es el uso de *Docker in Docker* (*DinD*), un contenedor que necesita correr en modo privilegiado y que puede crear contenedores dentro del. *DinD* no es del todo estable, nosotros recomendamos usar el mismo docker que  está corriendo en el host. Para ello, nuestro contenedor solo necesitará tener la línea de comandos de Docker instalada, y montar el socket  donde Docker publica su api, `/var/run/docker.sock`. Como ejemplo, el siguiente servicio lista los contenedores del host:

```yaml
  docker:
    image: docker:17.10
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    entrypoint: docker
    command: ps
```

##### 4.5.5 Aplicación Django, con celery y beat

El siguiente ejemplo muestra el *docker-compose.yml* de una aplicación Django, integrada con *celery* y con un *beat* para ejecutar tareas periódicas:

```
version: '3.4'

services:
  api:
    image: app
    build: .
    command: gunicorn -b 0.0.0.0:80 -w 8 app.wsgi

  worker:
    image: app
    entrypoint: celery -A app 
    command: worker -c 8 -P prefork -O fair

  beat:
    image: app
    entrypoint: celery -A app 
    command: beat
```

Como vemos, en este caso estamos usando la misma imagen de Docker para los tres servicios. Este es un ejemplo  donde puede resultar conveniente usar la misma imagen para distintos  servicios. Es verdad que estamos instalando alguna dependencia no  estrictamente necesaria, para las dependencias de los tres componentes  son muy parecidas, y todo parece más sencillo de gestionar si tenemos  una sola imagen. Además, el uso de la caché hará que los workflows de  desarrollo sean más eficientes.

##### 4.5.6 Importar compose file

Por defecto Docker Compose toma como entrada dos ficheros, *docker-compose.yml* y otro opcional, *docker-compose.override.yml*. Se supone que *docker-compose.yml* es la configuración base, mientras que *docker-compose.override.yml* redefine esos servicios para un entorno de desarrollo local, o incluso  crear nuevos servicios. Cuando un servicio está definido en ambos  ficheros, sus campos se mazclan. Las reglas para mezclar estos dos  ficheros son bastante intuitivas, para una explicación detallada del  cómo se mezclan ambos ficheros podéis acceder [aquí](https://docs.docker.com/compose/extends/#adding-and-overriding-configuration). Ese es el comportamiento por defecto, pero podemos añadir más *compose files* con la opción `-f`. Docker Compose mezcla estos ficheros en el orden en que aparecen en el comando. 

Como ejemplo tenemos el siguiente *docker-compose.yml*:

```
web:
  image: app
  depends_on:
    - db
db:
  image: postgres:latest
```

y definir un *docker-compose.override.yml* para desarrollo:

```
web:
  build: .
  volumes:
    - '.:/code'
  ports:
    - 8883:80
  environment:
    DEBUG: 'true'

db:
  command: '-d'
  ports:
    - 5432:5432
```

y un *docker-compose.prod.yml* para producción:

```
web:
  ports:
    - 80:80
  environment:
    PRODUCTION: 'true'
```

##### 4.5.7 Extender un Servicio.

También podemos crear un servicio que extiende a otro. Por ejemplo,  podemos re-escribir el compose de la aplicación Django anterior como:

```
  worker:
    image: app
    entrypoint: celery -A app 
    command: worker -c 8 -P prefork -O fair

  beat:
    extends: worker
    command: beat
```

## 4.6 Integración Continua con Docker

La Integración Continua se refiere a la práctica de automatizar  tareas de modo que se ejecuten automáticamente cuando se produce un  evento, por ejemplo, una nueva versión de código en nuestro repositorio. Ejemplos de las tareas que pueden ser automatizadas son: compilación de componentes, ejecución de pruebas unitarias, ejecución de pruebas de  integración, ejecución de pruebas de aceptación, obtención de métricas  de calidad de código… Algunos de los objetivos que persigue la  integración continua son:

- Facilitar la integración entre distintos los distintos componentes de nuestra aplicación.
- Automatizar la construcción de nuestra aplicación.
- Automatizar la ejecución de tests en la construcción de nuestra aplicación.

La integración continua aporta aporta innumerables ventajas en el  objetivo de conseguir un software de alta calidad, algunas de las cuales son:

- Detectar rápidamente posibles conflictos entre desarrolladores.
- Garantizar el correcto funcionamiento de nuestra aplicación antes de desplegar una nueva versión.
- Agiliza la corrección de los errores detectados.
- Permite resolver problemas de integración a lo largo de todo el proceso de  desarrollo del software y no sólo al final del mismo, evitando  situaciones caóticas previas a la fecha de entrega.

Aunque el concepto de integración continua es muy amplio, en este curso lo vamos a delimitar a:

- **Optimización de Docker Build**: cómo utilizar la caché de Docker para acelerar el build de nuestras imágenes. 
- **Test de Integración con Docker**: cómo utilizar Docker para facilitar la automatización del testeo de nuestras aplicaciones.