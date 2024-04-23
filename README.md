# PROCEDIMIENTO PASO A PASO DEL CHALLENGE 01

## PASO 0: PROBAR ACCESOS

En un Linux local con sistema operativo Ubuntu 22 se copió el archivo "challenger-015.yaml" y luego se ejecutó el siguiente comando :

 `export KUBECONFIG="/home/eduardo/whitestack/challenger-015.yaml"`

En dicho Linux se usó `kubectl get pods` y se verificó exitosamente la comunicación con el Kubernetes remoto.


## PASO 1: COMPILAR Y PROBAR EL CODIGO DE MANERA LOCAL

En un Linux local con sistema operativo Ubuntu 22 se ejecutó `git clone https://github.com/aitorperezzz/tengen-tetris.git` para clonar la aplicación Tetris.

En dicho Linux se ejecutó la aplicación de la siguiente manera `python3 application.py` y se verificó que el modo multijugador funcionaba correctamente


## PASO 2: CREAR UNA IMAGEN DE DOCKER DE LA WEBAPP

En un Linux local con sistema operativo Ubuntu 22 se creó el archivo `.dockerignore` para ignorar los archivos GIT , docker y el directorio __pycache__.  También se creó el siguiente archivo `Dockerfile` para "dockerizar" la aplicación

```
FROM python:3.10.12
LABEL Maintainer="Eduardo Aliaga"

WORKDIR /home/eduardo/tengen-tetris
COPY . .
RUN pip install -r requirements.txt
CMD [ "python", "./application.py"]
EXPOSE 8080/tcp
```

Ambos archivos `.dockerignore` y  `Dockerfile` se copiaron en el mismo directorio donde estaba la aplicación. Luego se construye la imagen con el siguiente comando `docker build -t edutetris:1.0 edual/tetris:1.0`

Validamos que la imagen se creó con el comando `docker images`

Finalmente se hizo login a mi cuenta de DockerHub y luego se hizo push para publicar la imagen en dicho repositorio público.

```docker login -u edual
docker image push edual/tetris:1.0```


## PASO 3: CREAR MANIFIESTOS K8S



## PASO 4: CREAR UN HELMET CHART


## PASO 5: DESPLIEGUE DE LA WEBAPP EN UN CLUSTER K8S


## PASO 6: ACTUALIZACIÓN DE UNA VARIABLE DE ENTORNO EN LA WEAPP


## PASO 7: ACTUALIZACIÓN DE LA WEBAPP DESPLEGADA


## PASO 8: DOCUMENTACIÓN DE PROCESO

