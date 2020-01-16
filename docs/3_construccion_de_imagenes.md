# 3 Construcción de imágenes

## 3.1 Docker Build y Dockerfile

Una **imagen** se corresponde con la información necesaria para arrancar un  contenedor, y básicamente se compone de:

- **Sistema de archivos**
- **Metadatos** 
  - **comando a ejecutar**
  - **variables de entorno**, 
  - **volúmenes del contenedor**
  - **los puertos que utiliza nuestro contenedor**
  - **etc.**

La **manera recomendada de construir una imagen** es utilizar un fichero `Dockerfile`:

> `Dockerfile`: fichero con un conjunto de instrucciones que indican cómo construir  una imagen de Docker.

Las instrucciones principales que pueden utilizarse en un Dockerfile son:

- `FROM `: para definir la imagen base de nuestro contenedor
- `RUN `: para ejecutar un comando en el contexto de la imagen
- `ENTRYPOINT `: para definir el entrypoint que ejecuta el contenedor al arrancar
- `CMD `: para definir el comando que ejecuta el contenedor al arrancar
- `WORKDIR `: para definir el directorio de trabajo en el contenedor
- `ENV=`: para definir variables de entorno
- `EXPOSE `: para definir puertos donde el contenedor acepta conexiones
- `VOLUME `: para definir volúmenes en el contenedor
- `COPY  `: para copiar ficheros dentro de la imagen. También se usa para *multi-stage builds*

Para una lista completa de las instrucciones disponibles ir a la documentación oficial.

Un ejemplo de un dockerfile para una aplicación Flask en python podría ser: 

```yaml
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get install -y python-pip python-dev
WORKDIR /app
ENV DEBUG=True
EXPOSE 80
VOLUME /data
COPY . /app
RUN pip install -r requirements.txt
ENTRYPOINT ["python"]
CMD ["app.py"]
```

## 3.2 La caché de Docker

La **construcción de una imagen** de Docker dado un Dockerfile puede ser un  **proceso costoso** ya que puede implicar la instalación de un número elevado de librerías, y al mismo tiempo es un proceso **bastante repetitivo** porque sucesivos builds del mismo Dockerfile suelen ser similares entre sí. Es por eso que Docker introduce el concepto de la **caché** para **optimizar** el proceso de **construcción de imágenes**.

- La **primera optimización** que hace la cache de Docker es la **descarga de la imagen base** de nuestro Dockerfile.
  - Docker descargará la imagen base siempre que la misma no se encuentre ya descargada en la máquina que hace el *build*. Esta optimización parece obvia ya que estas  imágenes pueden tener un tamaño de cientos de MB, pero hay que tener  cuidado ya que si la versión remota de la imagen cambia, Docker seguirá  utilizando la versión local. Por tanto, si queremos ejecutar nuestro Dockerfile con la nueva versión de la imagen base deberemos hacer un `docker pull` manual de la imagen base, o ejecutar `docker build --pull`.

Como hemos comentado anteriormente, una **imagen de Docker** tiene una estructura interna bastante **parecida a un repositorio de git**. Lo que conocemos como *commits* en git lo denominamos **capas** de una imagen en Docker. Por lo tanto, **una imagen** (o repositorio) **es una sucesión de capas** en un Registro de Docker, donde cada capa almacena un *diff* respecto de la capa anterior. Esto es importante de cara a optimizar nuestros *Dockerfiles*, como veremos en la siguiente sección.

Por ahora bastará saber que **cada instrucción** de nuestro `Dockerfile` **creará una y sólo una capa de nuestra imagen**. Por lo tanto, la caché de Docker funciona a nivel de instrucción. En otras palabras, si una línea del Dockerfile no cambia, en lugar de recomputarla, Docker asume que la capa que genera esa instrucción es la misma que la ejecución anterior del Dockerfile. Por lo tanto, si tenemos una instrucción tal como:

```yaml
RUN apt-get update && apt-get install -y git
```

que no ha cambiado entre dos build sucesivos, los comandos `apt-get` no se ejecutarán, sino que **se reusará la capa que generó el primer build**. Por tanto, aunque antes de ejecutar el segundo build haya una nueva versión del paquete git, la imagen construida a partir de este Dockerfile tendrá la versión de git anterior, la que se instaló en el primer build de este Dockerfile. Podemos desactivar el uso de la caché ejecutando `docker build --no-cache`. 

Es importante destacar los siguientes aspectos sobre la caché de Docker: 

- La **cache de Docker es local**, es decir, si es la primera vez que haces el build de un Dockerfile en una máquina dada, todas las  instrucciones del Dockerfile serán ejecutadas, aunque la imagen ya haya  sido construida en un Registro de Docker.
- **Si una instrucción ha cambiado** y no puede utilizar la caché, la caché queda invalidada y las siguientes instrucciones del `Dockerfile` serán ejecutadas sin hacer uso de la caché.
- El comportamiento de las instrucciones `ADD` y `COPY` **es distinto** en cuanto al comportamiento de la caché. Aunque estas instrucciones no cambien, **invalidan la caché si el contenido de los  ficheros que se están copiando ha sido modificado**.

## 3.3 Comandos comunes para gestionar imágenes

Estos son los **comandos** más comunes para el **manejo de imágenes**:

- `docker build`: nos permite crear una imagen a partir de un Dockerfile
- `docker login`: autentica la cli de docker contra un registro, por defecto *Docker Hub*
- `docker pull/push`: descarga/carga una imagen a la que tengamos acceso desde un registro
- `docker image ls`: lista las imágenes que están disponibles en nuestra máquina; igual a como lo hace `docker images`
- `docker inspect`: muestra información detallada de una  imagen. Se puede acceder a un campo particular con el comando docker  inspect -f '{{.Size}}' imagen.
- `docker image rm`: elimina una imagen. 

## 3.4 Buenas prácticas

### 3.4.1 Usar `.dockerignore`

El build de una `image` se ejecuta a partir de un `Dockerfile` y de un directorio, que se conoce con el nombre de **contexto**. Este  directorio suele ser el mismo que el directorio donde se encuentra el `Dockerfile`, por lo que si ejecutamos la instrucción: 

```
ADD app.py /app/app.py
```

Estamos añadiendo a la imagen el fichero `app.py` del contexto, es decir, el fichero `app.py` que se encuentra en el directorio donde está el `Dockerfile`. Dicho directorio se comprime y se manda al Docker Engine para construir la imagen, pero puede que tenga ficheros que no son necesarios. Es por eso que este directorio puede tener un fichero `.dockerignore`, que de una manera similar a fichero `.gitignore`, indica los ficheros que no deben ser considerados como parte del contexto del build. 

### 3.4.2 Reducir el tamaño de las imágenes al mínimo

La imagen Docker **sólo debe contener lo estrictamente necesario** para ejecutar la aplicación. Con el objetivo de reducir complejidad,  dependencias, tamaño de las imágenes, tiempos de "build" de una imagen, se debe evitar la instalación de paquetes sólo por el hecho de que puedan ser útiles para depurar un contenedor. Como ejemplo, **no incluir editores de texto en las imágenes**. 

Otro opción muy práctica es el uso de imágenes base pequeñas, por ejemplo, haciendo uso de "alpine". 

Por ejemplo, comparar las imágenes de `ubuntu` y `alpine`. 

### 3.4.3 Ejecutar sólo un proceso por contenedor

Salvo raras excepciones, es **recomendable correr sólo un proceso por contenedor**. Esto permite reutilizar contenedores más fácilmente, que sean más fáciles de escalar, y da lugar a sistemas más desacoplados. Por ejemplo sacar la lógica de *logging* a un contenedor independiente (Docker tiene soluciones *ad-hoc* para esto). 

### 3.4.4 Minimizar el número de capas de la imagen

Como se ha dicho anteriormente, cada capa de una imagen se corresponde con una instrucción del `Dockerfile`. Compare el `Dockerfile`: 

```yaml
RUN apt-get update
RUN apt-get install -y bzr
RUN apt-get install -y cvs
RUN apt-get install -y git
RUN apt-get install -y mercurial
```

con este otro: 

```yaml
RUN apt-get update && apt-get install -y \
    bzr \ 
    cvs \ 
    git \ 
    mercurial \ 
    apt-get clean
```

Ambos son igualmente legibles, pero el primero genera 5 capas, y el segundo sólo una, que además ejecuta un `apt-get clean` que reduce el tamaño de dicha capa. Recordar que **cada instrucción de **`Dockerfile` **genera una capa en la imagen final**. Por tanto, si hacemos un `apt-get install` en una instrucción, y un `apt-get clean` en otra instrucción, habremos dejado una capa con todos los ficheros que luego el `apt-get clean` borra. 

¿Nueva tecnología? [BuildKit](https://www.youtube.com/watch?v=kkpQ_UZn2uo&t=1023s)

### 3.4.5 Optimizar el uso de la cache

**Optimizar el uso de la cache añadiendo al principio del Dockerfile las instrucciones que menos cambian** (como la instalación de librerías), y  dejando para el final las que más cambian (como el copiado del código  fuente).

Como ejemplo comparar los `Dockerfile`'s:(https://github.com/OpenWebinarsNet/docker-for-devs) 

[`docker-for-dev/flask-alpine`](https://github.com/OpenWebinarsNet/docker-for-devs/tree/master/flask-alpine) y [`docker-for-dev/flask-build-cache`](https://github.com/OpenWebinarsNet/docker-for-devs/tree/master/flask-build-cache). 

El primero cachea la instalaciones de las dependencias pip siempre que no añadamos nuevas dependencias al fichero `requirements.txt`, antes de  añadir el código fuente. Sin embargo, el segundo, aunque genere menos capas, no reusa la instalación de las dependencias porque `ADD * /app`  invalida la cache en cuanto hay un cambio en nuestro código fuente.

### 3.4.6 Parametrizar los Dockerfiles usando argumentos

Se aumenta la reusabilidad de los `Dockerfile`'s entre distintos entornos y aplicaciones parametrizando los `Dockerfile`'s con argumentos. Los  argumentos son valores que se pasan como parámetros a cada "build" (aunque pueden tener valores por defecto), y que se pueden utilizar en las  instrucciones del `Dockerfile`. Por ejemplo, el `Dockerfile`:


```yaml
FROM ubuntu ARG user=root ARG password RUN echo $user $password 
```
puede ser parametrizado de la siguiente manera:

```bash
$ docker build -t imagen –build-arg password=secret . 
```

### 3.4.7 Utilizar multi-stage builds

Los multi-stage es una funcionalidad introducida recientemente y que ayuda a **crear imágenes muy pequeñas**. Permiten resetear el sistema de  ficheros de la imagen que se está construyendo, cambiar a otro sistema de fichero, pero importar ficheros de la imagen anterior.

Tenemos un ejemplo en [`docker-for-dev/go-multi-stage`](https://github.com/OpenWebinarsNet/docker-for-devs/tree/master/go-multi-stage). 

El primer `FROM` inicializa el sistema de ficheros con una imagen que lleva *Go* instalado. En esa imagen añadimos el directorio actual con todo su contexto y hacer el build de nuestro programa *Go*. Luego viene  una nueva instrucción `FROM` que inicializa el sistema de ficheros con una imagen *alpine* sin nada instalado. La instrucción `COPY` copia el binario  generado en el stage anterior y lo copia en la imagen actual. El  resultado es una imagen muy pequeña, ya que no lleva el compilador de *Go* incluido, solo lleva el binario que necesitamos.

## 3.5 [Resumen de la estructura de un fichero `Dockerfile`](https://www.ramoncarrasco.es/es/content/es/kb/150/resumen-de-estructura-fichero-dockerfile)

- El fichero debe llamarse `Dockerfile`, respetando mayúsculas y minúsculas

- Las líneas de comentario van precedidas del símbolo `#`

- El flujo típico de un fichero `Dockerfile` es 

  + Selección de imagen base
  + Descarga e instalación de dependencias
  + Comandos a ejecutar al arrancar el contenedor

- Instrucciones: 

  + `FROM <imagen> [AS <fase>] ` especifica  la imagen a utilizar como base. Se puede especificar un nombre de fase para los casos en los que se utilizan construcciones multi-fase.

  + `RUN <comando>` ejecuta el comando en el momento de la creación de la imagen

  + `CMD <comando>` ejecuta el comando en el momento de ejecución del contenedor. Parámetros: 	
- `<comando>` es el comando a ejecutar en el momento de iniciar el contenedor. Es un objeto de tipo *array*, que contiene un primer elemento que se identificará como el comando a ejecutar, y todos los sucesivos elementos serán los parámetros a pasar a dicho comando. Por ejemplo, si en línea de comando quisiésemos ejecutar `npm start` para arrancar Node, `<comando>` tendría el valor `["npm","start"]`.
  
- `WORKDIR <ruta>` especifica la carpeta de trabajo dentro del contenedor. A partir del momento en que se ejecuta esta  instrucción, todos los comandos siguientes se ejecutarán de forma  relativa a `<ruta>`. Si `<ruta>` no  existe, se creará automáticamente. Esta es una buena práctica cara a  evitar conflictos de ficheros con la imagen base utilizada.
  
- `COPY [--from=<fase>] <origen> <destino>` copia los ficheros de `<origen>` en `<destino>`. Los parámetros son relativos al contexto de construcción (la carpeta donde se encuentra `Dockerfile`):
    + `<origen>` se encuentra en el sistema de archivos fuera del contenedor
    + `<destino>` se encuentra en el sistema de archivos dentro del contenedor
    + `--from=` cuando se especifica el modificador `from`, `COPY` busca los ficheros en la imagen referenciada por `<fase>` en vez del sistema de archivos del host.

Ejemplo.-
```dockerfile
# Imagen base
FROM node:alpine

# Carpeta de trabajo
WORKDIR /usr/app

# Copiado de archivos
COPY ./ ./

# Instalación de dependencias
RUN npm install

# Comando por defecto del contenedor
CMD ["npm", "start"]
```
