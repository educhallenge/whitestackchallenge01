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

Ambos archivos `.dockerignore` y  `Dockerfile` se copiaron en el mismo directorio donde estaba la aplicación. Luego se construye la imagen con el siguiente comando `docker build -t edual/tetris:1.0`

Validamos que la imagen se creó con el comando `docker images`

Finalmente se hizo login a mi cuenta de DockerHub y luego se hizo push para publicar la imagen en dicho repositorio público.

```
docker login -u edual
docker image push edual/tetris:1.0
```

## PASO 3: CREAR MANIFIESTOS K8S

En una virtual machine Ubuntu 22 en Google Cloud se instaló kubernetes con 1 solo nodo. Luego se creó el archivo `tetris-deploy.yaml`  que tiene la configuración de 1 deployment y 1 service tal cual se muestra a continuación

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tetris-deployment
  labels:
    app: tetris
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tetris
  template:
    metadata:
      labels:
        app: tetris
    spec:
      containers:
      - name: tetris
        image: edual/tetris:1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tetris-service
spec:
  type: NodePort
  selector:
    app: tetris
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30100
```
Luego se despliega esta configuración con el comando `kubectl apply -f tetris-deploy.yaml` . Se verifica con el comando `kubectl get svc`

También verificamos navegando a la IP pública de la máquina virtual con puerto 8080. Usando 2 navegadores distintos se probó exitosamente el modo multijugador de Tetris.


## PASO 4: CREAR UN HELMET CHART

En la virtual machine Ubuntu 22 en Google Cloud se borró el despliegue que se había hecho en el paso 3. Para ello se usaron los siguientes comandos:
```
kubectl delete deploy tetris-deployment
kubectl delete svc tetris-service
```
Se instaló Helm en dicha virtual machine. Luego se creó el template de un chart usando el comando `helm create tetrischart`.  Con dicho comando se crean muchos archivos automáticamente que no vamos a usar. Por ello borramos el archivo `values.yaml` (este archivo lo volveremos a crear en un siguiente paso)  y todos los archivos del directorio `templates`

Luego creamos el archivo `tetris-deploy.yaml` en el directorio `templates`. El contenido de dicho archivo es exactamente igual al mostrado en el paso 3.

Después usamos Helm para instalar la aplicación Tetris usando el comando `helm install mytetris ./tetrischart`

Verificamos con los comandos `helm ls` y  `kubectl get svc`

También verificamos navegando a la IP pública de la máquina virtual con puerto 8080. Usando 2 navegadores distintos se probó exitosamente el modo multijugador de Tetris.


## PASO 5: DESPLIEGUE DE LA WEBAPP EN UN CLUSTER K8S

En la virtual machine Ubuntu 22 en Google Cloud se creó el archivo `values.yaml` con el siguiente contenido
```
tetris_deploy:
    name: tetris-deployment
    label: tetris
    replicaCount: 1
    containers:
        name: tetris
        image:
          tag: "edual/tetris:1.0"
          containerPort: 8080

tetris_svc:
    name: tetris-service
    type: NodePort
    label: tetris
    ports:
        protocol: TCP
        port: 8080
        targetPort: 8080
        nodePort: 30100
```
Luego el archivo `tetris-deploy.yaml` que habíamos usado en el paso anterior lo modificamos para que no haya información "hard-coded" sino que todos los valores hagan referencia a una variable del archivo `values.yaml`.  A continuación mostramos el archivo `tetris-deploy.yaml` modificado:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.tetris_deploy.name}}
  labels:
    app: {{ .Values.tetris_deploy.label}}
spec:
  replicas: {{ .Values.tetris_deploy.replicaCount}}
  selector:
    matchLabels:
      app: {{ .Values.tetris_deploy.label}}
  template:
    metadata:
      labels:
        app: {{ .Values.tetris_deploy.label}}
    spec:
      containers:
      - name: {{ .Values.tetris_deploy.containers.name}}
        image: {{ .Values.tetris_deploy.containers.image.tag}}
        ports:
        - containerPort: {{ .Values.tetris_deploy.containers.image.containerPort}}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.tetris_svc.name}}
spec:
  type: {{ .Values.tetris_svc.type}}
  selector:
    app: {{ .Values.tetris_svc.label}}
  ports:
    - protocol: {{ .Values.tetris_svc.ports.protocol}}
      port: {{ .Values.tetris_svc.ports.port}}
      targetPort: {{ .Values.tetris_svc.ports.targetPort}}
      nodePort: {{ .Values.tetris_svc.ports.nodePort}}
```
Procedemos a actualizar el despliegue usando el siguiente comando `helm upgrade  mytetris ./tetrischart`

Verificamos navegando a la IP pública de la máquina virtual con puerto 8080. Usando 2 navegadores distintos se probó exitosamente el modo multijugador de Tetris.

## PASO 6: ACTUALIZACIÓN DE UNA VARIABLE DE ENTORNO EN LA WEAPP

En este paso usamos el mismo Linux local con sistema operativo Ubuntu 22 que se había usado para los pasos 1 y 2. Actualizamos el archivo para aceptar argumentos posicionales y asignarlos a las variables customhost  y customport y usar dichas variables para levantar el servicio web con Flask. El diff con las modificaciones realizadas se muestra a continuación:
```
diff application.py application.py.OLD
5d4
< import sys
105,110d103
< def main(customhost,customport):
<         app.debug = True
< #   socketio.run(app, host='0.0.0.0', port=8080)
<         socketio.run(app, customhost, customport)
<
<
112,126c105,106
<     try:
<        sys.argv[2]
<     except:
<        customport=8080
<     else:
<        customport=int(sys.argv[2])
<
<     try:
<        sys.argv[1]
<     except:
<        customhost='0.0.0.0'
<     else:
<        customhost=sys.argv[1]
<
<     sys.exit(main(customhost,customport))
---
>       app.debug = True
>       socketio.run(app, host='0.0.0.0', port=8080)
```

## PASO 7: ACTUALIZACIÓN DE LA WEBAPP DESPLEGADA


## PASO 8: DOCUMENTACIÓN DE PROCESO

La documentación se encuentra en este archivo README.md
