# Docker en producción

Docker presenta enormes ventajas en entornos de producción debido a que nos  permite generar imágenes inmutables y portables que han sido validadas  en los procesos de integración continua. Además, el proceso de  distribución de imágenes es fácilmente automatizable gracias a la API  que implementa el demonio de Docker. Por último, gracias al aislamiento  entre contenedores, permite la combinación de microservicios en una  misma máquina para la consecución de diversos objetivos. Por ejemplo,  imagina una aplicación con una API pública que, entre otros, ofrece un  servicio de autenticación. Imaginemos otro componente que hace uso de la API pública para autenticar peticiones. Además de servidores dedicados a ofrecer la API pública, Docker facilita enormemente correr siempre el  segundo componente con un contenedor de API pública en la misma máquina  (y no visible públicamente). De esta manera, aunque nuestra API pública  sufra, por ejemplo, un ataque de denegación de servicio, el segundo  componente podrá seguir funcionando.

En los próximos meses vamos a realizar un curso sobre cómo gestionar  Docker en producción, pero en esta lección daremos alunas pautas para  aquellos que quieran o necesiten adquirir conocimientos en este campo.

## Retos que introduce Docker

Docker y otras tecnologías de contenedores no solucionan todos los problemas  que aparecen en una arquitectura basada en microservicios, es más, el  hecho de tener varios contenedores corriendo en la misma máquina hace  imposible el uso de servicios diseñador para el mundo de las máquinas  virtuales. Los principales problemas que quedan sin resolver por los  contenedores en un entorno de producción son:

- Service Discovery: debido a la cantidad de microservicios, estos  deben ser fáciles de localizar por los servicios que quieran contactar  con ellos. También debe ser sencillo (y seguro) conocer las credenciales para establecer dicha conexión.
- Balanceo de Carga: si un microservicio está corriendo en  varios contenedores, debe haber un endpoint único de acceso al  microservicio que balancee las peticiones entre los distintos  contenedores.
- Configuración de Red: algunos de estos servicios sólo pueden ser contactados por un subconjunto de nuestros microservicios. Este  problema suele solucionarse con la gestión de redes (y subredes) de tal  manera que servicios en redes diferentes no pueden verse el uno al otro.
- Persistencia: algunos servicios inevitablemente tienen  estado. Un contenedor puede ser fácilmente recreado en una nueva máquina si hay un fallo en el servidor donde se estaba ejecutando, pero lo  mismo no aplica a los datos persistentes que utilizaba dicho contenedor.
- Escalabilidad: nuestras herramientas de orquestación y  monitoreo debe escalar al número de contenedores desplegados en  producción.
- Personalización: la herramienta que utilicemos debe ser  fácilmente configurable con las sub-herramientas que sean más cómodas al equipo de operaciones.
- Logging y Monitoreo: debido a la gran cantidad de  contenedores desplegados en producción debemos utilizar herramientas de  monitoreo que nos faciliten identificar los contenedores donde se  producen fallos del sistema.
- Respuesta a fallo: es importante monitorizar nuestros  contenedores y desplegarlos en un nuevo servidor si hay una falla en la  máquina donde estaban ejecutando. 

Como hemos dicho, Docker no resuelve la mayoría de estas cuestiones,  ya que su principal objetivo es facilitar la creación de imágenes, su  distribución y su ejecución de una manera fiable. Es por esto que  existen hoy en día multitud de herramientas de **gestión de clusters** que vienen a solucionar en lo posible estos problemas.

## Gestión de Docker Cluster

#### Gestión de Cluster *On Premise*

Conocemos como soluciones *on premise* aquellas que podemos instalar en nuestros propios servidores. Destacamos las siguientes:

##### Kubernetes

Es la solución *open source* de Google. Es probablemente la  solución más completa en la actualidad, con soluciones elegantes para  service discovery, respuesta a fallo y la configuración de la red.  Ofrece integraciones para utilizar volúmenes persistentes tanto en *Amazon Web Services* (AWS) como en *Google Compute Engine*,  así como con soluciones de monitoreo (*cAdvisor*) y logging (*elasticsearch*). Su desarrollo es muy activo y también ha desarrollado soluciones para  la gestión de secretos y la gestión de control de acceso. 

##### Docker Swarm

Es la solución *open source* de Docker. Ha alcanzado una versión *production ready*, pero todavía tiene algunos problemas de estabilidad. Su principal  ventaja es que su API es compatible con la API del Docker Engine, por lo que todas las herramientas desarrolladas para Docker, que tiene el  ecosistema más activo, funcionan directamente sobre Docker Swarm. En la  Dockercon de Copenhague de 2017 se anunció una integración transparente  entre Docker Swarm y Kubernetes.

#### Gestión de Cluster *SaaS*

Conocemos como soluciones *SaaS* aquellas que están disponibles como un servicio en internet. Destacamos las siguientes:

##### GCE (Google Container Engine)

Es la solución *SaaS* de Google e internamente utiliza  Kubernetes. Una vez desplegado el cluster de kubernetes, la interacción  del usuario es directamente con el cluster usando la herramienta de  línea de comandos de kubernetes. Por tanto, no ofrece grandes beneficios sobre kubernetes, más allá de facilitar su configuración con otros  servicios del *Google Cloud Platform* como el logging o las métricas. Dispone de registro propio (*Google Container Registry*).

##### ECS (EC2 Container Service)

Es la solución *SaaS* de AWS e internamente utiliza una  tecnología propietaria pero desarrollada sobre Docker. Se integra  perfectamente con otros servicios de AWS como *Elastic Block Storage*, *Cloud Trail*, *Cloud Trail Logs*, *Identity and Access Management* o *Elastic Load Balancer*. Quizás sea la más difícil de utilizar como usuario de Docker, pero las  integraciones con otros servicios de AWS le dan un plus frente a otras  soluciones. Dispone de registro propio (*EC2 Container Registry*). En el congreso *re:Invent 2017* se anunció *AWS Fargate*, que permite la ejecución de contenedores sin gestión de infrastructure, y que puede marcar un antes y un después en cómo los usuarios ejecutan  contenedores en producción.

## Conclusión

Hemos visto como las **arquitecturas basadas en microservicios** agilizan los procesos de desarrollo del software en equipos de ingeniería de mediano y gran tamaño. También hemos visto como **Docker y los contenedores en general han dado un impulso definitivo a esta  metodología de desarrollo del software**, pero que aún quedan muchos  problemas por resolver en un entorno de producción a gran escala. Es aquí donde aparecen las **herramientas de gestión de clusters**, que vienen a solucionar la mayoría (si no todos) de estos problemas. Es un campo en constante evolución y donde hay un rico y activo ecosistema de compañías colaborando en encontrar las mejores soluciones a estos problemas. 

