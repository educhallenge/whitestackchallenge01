PROCEDIMIENTO PASO A PASO DEL CHALLENGE 01

PASO 0: PROBAR ACCESOS

En un Linux local se copió el archivo "challenger-015.yaml" y luego se ejecutó el siguiente comando :

 export KUBECONFIG="/home/eduardo/whitestack/challenger-015.yaml"

En dicho Linux se usó "kubectl get pods" y se verificó exitosamente la comunicación con el Kubernetes remoto.


PASO 1: COMPILAR Y PROBAR EL CODIGO DE MANERA LOCAL

En un Linux local se ejecutó "git clone https://github.com/aitorperezzz/tengen-tetris.git" para clonar la aplicación Tetris.

En dicho Linux se ejecutó la aplicación de la siguiente manera "python3 application.py" y se verificó que el modo multijugador funcionaba correctamente


PASO 2: CREAR UNA IMAGEN DE DOCKER DE LA WEBAPP

