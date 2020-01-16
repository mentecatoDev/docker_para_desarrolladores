Integración continua

a Integración Continua se refiere a la práctica de automatizar tareas de  modo que se ejecuten automáticamente cuando se produce un evento, por  ejemplo, una nueva versión de código en nuestro repositorio. Ejemplos de las tareas que pueden ser automatizadas son: compilación de  componentes, ejecución de pruebas unitarias, ejecución de pruebas de  integración, ejecución de pruebas de aceptación, obtención de métricas  de calidad de código… Algunos de los objetivos que persigue la  integración continua son:

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

## Docker Hub

Docker Hub es el registro público de Docker. Viene a ser lo que es Github en el mundo `git`, pero en el mundo Docker.
Permite configurar algunas tareas de integración continua, aunque está muy  enfocado a la automatización de construcción de imágenes de Docker.  Aunque hay muchas herramientas que permiten configurar tareas de  integración continua, en este curso vamos a utilizar Docker Hub por  varios motivos:

- Es una herramienta gratuita si generas imágenes públicas.
- Docker Hub es probablemente la herramienta más popular para distribuir  imágenes de Docker, y su conocimiento resulta muy conveniente.
- Aunque es una herramienta algo limitada, resulta suficiente para ilustrar las ideas que introducimos en este curso.

En el vídeo adjunto se harán tres demos usando Docker Hub:

- Una breve introducción a Docker Hub y alguna de sus principales características.
- Como automatizar el build de imágenes.
- Cómo automatizar la ejecución de tests.

## optimización de docker build

Cuando estamos trabajando en nuestra máquina local, Docker dispone de las capas de todas las ejecuciones previas de *docker build* que hemos realizado. En otras palabras, cuando trabajamos en nuestra  máquina local la caché de Docker está inicializada, recuerda la  ejecución de instrucciones *docker build* anteriores, y como  hemos visto en lecciones anteriores, optimiza enormemente la generación  de imágenes. Sin embargo, esto no suele ser así en entornos de  integración continua, donde normalmente cada tarea de integración (ya  sea el build de una imagen o la ejecución de tests) se ejecuta en una  nueva máquina independiente creada para este fin, y destruída cuando la  tarea acaba. Esto es así porque reusar las mismas máquinas entre tareas  de integración continua puede conllevar la aparición de estados  indeseados en la máquina, o la acumulación de basura que terminará  saturando los recursos de dicha máquina. En consecuencia, cuando una  tarea de integración continua se ejecuta, es bastante frecuente que la  cache de Docker no esté inicializada, lo que implica que los tiempos  para construir nuestras imágenes pueden crecer enormemente. Esto es un  problema importante porque idealmente las tareas de integración continua deben ejecutar cuanto más rápido mejor.

Por suerte Docker tiene una solución a este problema. Cuando ejecutamos un *docker build* podemos indicar que utilice la caché de una imagen ya creada. Dicho de otra manera, si estoy construyendo la imagen `openwebinars/docker-for-devs:latest`, puedo reusar la cache de la última versión de `openwebinars/docker-for-devs:latest` ejecutando estos commandos:

```bash
docker pull openwebinars/docker-for-devs:latest
docker build -t openwebinars/docker-for-devs:latest --cache-from=openwebinars/docker-for-devs:latest 
```

## Test de integración con Docker

El testeo de aplicaciones ha sido tradicionalmente tedioso ya que la  ejecución de nuestra aplicación requiere de la instalación de sus  dependencias, y además puede necesitar componentes externos como bases  de datos. Sabemos que Docker soluciona estos problemas, y que gracias a  la herramienta *docker-compose*, es prácticamente trivial el proceso de ejecución de una aplicación en local.

Pues bien, estas ideas pueden ser aplicadas al proceso de testeo de una aplicación. Lo primero que tenemos que hacer es *dockerizar* nuestros scripts de tests, de tal manera que podamos ejecutarlos haciendo uso de *docker-compose* al igual que hacemos con cualquier otra imagen de Docker.

El vídeo adjunto hace una demo de este proceso usando el directorio `docker-for-devs/e2e`.

Este enfoque para la ejecución de tests basado en Docker y Docker Compose presenta las siguientes ventajas:

- Ligero: Docker es ligero, por lo que el fichero *docker-compose.yml* puede contener tantos servicios como sea necesario y podrá ser  ejecutado en una sola máquina. Esto permite la ejecución de entornos  complejos para pruebas de integración y aceptación.
- Portable:  gracias a que el proceso está basado en Docker y Docker es portable, la  ejecución de pruebas puede realizarse en cualquier máquina,  independientemente de su sistema operativo o el hardware interno.
- Inmutable: gracias a que Docker es inmutable, la imagen creada y validada por los  procesos de integración continua, va a correr de la misma manera en  nuestros servidores de producción.

## otras herramientas de integración continua

Existen números herramientas para automatizar el *build* y las pruebas de aplicaciones, algunas puedes instalarlas en tu infraestructura, como:

- [Jenkins](https://jenkins-ci.org/)
- [Bamboo](https://www.atlassian.com/software/bamboo/)
- [Drone.io](https://drone.io/)

Y otras funcionan bajo el modelo SaaS, como:

- [Travis](https://travis-ci.org/)
- [Circleci](https://circleci.com/)

Como hemos visto en las lecciones anteriores, Docker ofrece su propia herramienta:

- [Docker Hub](https://hub.docker.com/)

