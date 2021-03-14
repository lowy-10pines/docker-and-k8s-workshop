## HEALTHCHECK
Este comando tiene dos formas:
- `HEALTHCHECK [OPTIONS] CMD command`: Chequea la salud del contenedor corriendo el comando pasado en CMD dentro del contenedor
- `HEALTHCHECK NONE`: Deshabilita cualquier HEALTHCHECK heredado de la imagen base

La instrucción HEALTHCHECK indica a Docker como chequear la salud de un contenedor que está ejecutandose. Esto ayuda a detectar casos donde el proceso ha dejado de responder pero sigue ejecutandose (por ejemplo un webserver que está atascado en un loop infinito y no puede manejar nuevas conexiónes). 

Cuando se especifica un HEALTHCHECK, se agrega un campo health status ademas del status normal. Este comienza siendo `starting`, apenas un check pasa cambia a `healthy`, y si varios checks fallan, despues de un numero de reintentos, pasa a `unhealthy`

Las opciones que pueden pasarse en `[OPTIONS]` son:

* --interval=DURACION (default: 30s)
* --timeout=DURACION (default: 30s)
* --start-period=DURACION (default: 0s)
* --retries=N (default: 3)

Puede haber solo una instrucción HEALTHCHECK en un Dockerfile. Si se agregan varias, solo la ultima tomará efecto.

El estado de salida del comando que se pasa con CMD indica el status del contenedor, siendo:
0: éxito - el contenedor está saludable y listo para usarse
1: unhealthy - el contenedor no está funcionando correctamente
2: reservado - no usar este codigo de salida

## ONBUILD

ONBUILD agrega a la imagen un trigger que será ejecutado cuando esta sea utilizada como base de otra imagen. Este se ejecutará en el contexto del build de la nueva imagen, tal y como si se hubiese insertado después de la instrucción FROM del nuevo Dockerfile.

Cualquier instrucción puede registrarse como ONBUILD, exceptuando ONBUILD. Y puede no ejecutar instrucciones FROM o MAINTAINER.

Esto es util cuando estás creando una imagen que será utilizada como base para construir otras imagenes, por ejemplo, si la imagen prepara un entorno para compilar otras aplicaciones o es un daemon que puede customizarse.
Hacer esto es una mejor solución que, por ejemplo, proveer a quienes quieran tomar tu imagen como base de un "Dockerfile demo" que puedan copiar y pegar, ya que ademas de introducir potenciales errores, es dificil de mantener y se mezcla con el codigo especifico de la aplicación.

Las instrucciones ONBUILD se limpian luego de construir la imagen hija, asi que no pueden pasarse a imagenes "nietas". 
También, pueden crearse varias instrucciones ONBUILD, estas serán ejecutadas en el orden en que se registraron.

## VOLUME

VOLUME ["/data"]
La instrucción VOLUME crea un punto de montaje con el nombre especificado y marca que tiene un volumen externo montado para el host nativo y otros contenedores. El valor puede ser un array JSON `VOLUME ["/var/log/"]`, o un string con varios argumentos `VOLUME /var/log` o `VOLUME /var/log /var/db`.

El comando docker run inicializa el nuevo volumen con cualquier data que exista en la localización especificada dentro de la imagen base.

Se deben tener las siguientes cosas en mente al crear volumenes desde el Dockerfile:

* Volumenes en contenedores Windows: El destino de un volumen dentro del contenedor debe ser, o bien un directorio nuevo o vacío, o bien estar en otro dive que no sea C:

* Modificando el volumen desde el Dockerfile: Si cualquier paso del build modifica la data dentro del volumen *despues* de que se haya generado, estos cambios se descartan

* Formateo JSON: La lista es parseada como un array JSON. Deben usarse comillas dobles (") en lugar de simples (')

* El directorio host se declara en runtime: El punto de montaje es, por naturaleza, dependiente del host. Esto es para preservar la portabilidad de las imagenes, dado que un directorio en el host puede no estar disponible en todos los hosts. Por esta razon, no se puede montar un directorio host desde el Dockerfile. La instrucción VOLUME no permite especificar un directorio del host, debe especificarse el punto de montaje al crear o correr el contenedor.


## Bibliografía
Información recopilada de la documentación oficial https://docs.docker.com/engine/reference/builder/ 