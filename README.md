# liberando-productos-yilisramirez
Práctica del módulo Liberando Productos DevOps - Yilis Ramirez

Esta práctica tiene como objetivo implementar una serie de mejoras al código para despIegarlo en producción. Para ello se debe tener en cuenta los siguientes prerequisitos:

| Software | URL |
| ------ | ------ |
| Pyhton >=3.8.5 | [Install Python](https://www.python.org/downloads/release/python-385/) |
| Docker | [Install Docker](https://docs.docker.com/engine/install/ubuntu/) | 
| Minikube | [Install Minikube](https://minikube.sigs.k8s.io/docs/start/) | 
| Kubectl | [Install Kubectl](https://kubernetes.io/es/docs/tasks/tools/) | 
| Helm | [Install Helm](https://helm.sh/docs/intro/install/) |

# Requerimientos
- Añadir un nuevo endpoint `/bye`  a la aplicación <b>Fast API</b>  y que devuelva como mensaje `Bye Bye`
![endpoint bye](https://user-images.githubusercontent.com/39458920/168478541-a9957406-09ef-4ebf-a3c6-4e887ba06966.JPG)

- Creación de test unitarios para el nuevo endpoint

```bash 
pytest 
```

```bash
=========================================================== test session starts ===========================================================
platform linux -- Python 3.8.10, pytest-7.1.1, pluggy-1.0.0 -- /usr/bin/python3
cachedir: .pytest_cache
rootdir: /home/yilis/Desktop/liberando-productos-yilisramirez, configfile: pytest.ini, testpaths: src/tests/
plugins: cov-3.0.0, asyncio-0.18.3, anyio-3.6.1
asyncio: mode=auto
collected 3 items                                                                                                                         

src/tests/app_test.py::TestSimpleServer::read_health_test PASSED                                                                    [ 33%]
src/tests/app_test.py::TestSimpleServer::read_main_test PASSED                                                                      [ 66%]
src/tests/app_test.py::TestSimpleServer::read_bye_test PASSED                                                                       [100%]

============================================================ 3 passed in 1.51s ============================================================
```
```bash 
pytest --cov
```

```bash
---------- coverage: platform linux, python 3.8.10-final-0 -----------
Name                          Stmts   Miss Branch BrPart     Cover   Missing
----------------------------------------------------------------------------
src/application/__init__.py       0      0      0      0   100.00%
src/application/app.py           32      4      2      0    88.24%   25, 29-31
src/tests/__init__.py             0      0      0      0   100.00%
src/tests/app_test.py            33      0      2      0   100.00%
----------------------------------------------------------------------------
TOTAL                            65      4      4      0    94.20%

Required test coverage of 80.0% reached. Total coverage: 94.20%
```
```bash 
pytest --cov --cov-report=html 
```

```bash
src/tests/app_test.py::TestSimpleServer::read_health_test PASSED                                                                                                                     [ 33%]
src/tests/app_test.py::TestSimpleServer::read_main_test PASSED                                                                                                                       [ 66%]
src/tests/app_test.py::TestSimpleServer::read_bye_test PASSED                                                                                                                        [100%]

---------- coverage: platform linux, python 3.8.10-final-0 -----------
Coverage HTML written to dir coverage_html_report

Required test coverage of 80.0% reached. Total coverage: 94.20%
==================================================================================== 3 passed in 2.86s =====================================================================================
```
![coverage report](https://user-images.githubusercontent.com/39458920/168476884-3de12ff7-b1d6-403a-a96d-3fc2eabd0772.JPG)

- Creación del Helm chart para desplegar la aplicación en Kubernetes. Para ello inicializamos un minikube que lleva por nombre `fastapi`
 ```bash 
 minikube start -p monitoring-demo
 ```
 Configuramos las alarmas de Alert-manager en slack, añadiendo la app webhook y conectandolo al canal `#yilis-ramirez-prometheus-alarms` con el objetivo de monitorizar el consumo de uso de la CPU enviando alertas con impacto critical y severity, y tambien informará cuando estas esten ya restablecidas.
 ```bash
 slack_configs:
      - api_url: 'https://hooks.slack.com/services/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' # <--- AÑADIR EN ESTA LÍNEA EL WEBHOOK CREADO
        send_resolved: true
        channel: '#yilis-ramirez-prometheus-alarms'
 ```
  Añadimos el repositorio de `Helm prometheus community`
 
 ```bash
 helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
 helm repo update
```
Desplegamos el chart de kube-prometheus-stack del helm añadido anteriormente con los valores definidos en el fichero `custom_values_prometheus.yaml`
```bash
helm -n monitoring upgrade --install prometheus prometheus-community/kube-prometheus-stack -f custom_values_prometheus.yaml --create-namespace --wait --version 34.1.1
```

Verificamos que los pods han sido creados correctamente para el stack de monitoring
```bash
kubectl get po -n monitoring -o wide
NAME                                                     READY   STATUS    RESTARTS   AGE   IP             NODE      NOMINATED NODE   READINESS GATES
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   0          11m   172.17.0.6     fastapi   <none>           <none>
prometheus-grafana-65fcd9964b-ndmf8                      3/3     Running   0          11m   172.17.0.5     fastapi   <none>           <none>
prometheus-kube-prometheus-operator-7f7b975f97-hk8dh     1/1     Running   0          11m   172.17.0.4     fastapi   <none>           <none>
prometheus-kube-state-metrics-94f76f559-6cm5r            1/1     Running   0          11m   172.17.0.3     fastapi   <none>           <none>
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0          11m   172.17.0.7     fastapi   <none>           <none>
prometheus-prometheus-node-exporter-qx7xr                1/1     Running   0          11m   192.168.49.2   fastapi   <none>           <none>
```
Añadimos el repositorio de helm bitnami, que es el que permite el despliegue en HA (alta disponibilidad)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
Habilitamos el metric server al cluster creado anteriormente `fastapi`, que es quien va a permitir autoescalar
```bash
minikube addons enable metrics-server -p monitoring-demo
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
🌟  The 'metrics-server' addon is enabled
```
Creación de un helm chart de la aplicación
```bash
helm dep up fast-api-webapp
```
Desplegamos la aplicación y observamos los pods estan siendo creados

```bash
helm -n fast-api upgrade my-app --install --create-namespace fast-api-webapp
kubectl -n fast-api get po -w
```
```bash
NAME                                      READY   STATUS              RESTARTS   AGE
my-app-fast-api-webapp-5cb74848bd-bdlrq   0/1     Init:0/1            0          20s
my-app-mongodb-7d7d86796b-rp9gn           0/2     ContainerCreating   0          20s
my-app-mongodb-7d7d86796b-rp9gn           0/2     Running             0          60s
my-app-mongodb-7d7d86796b-rp9gn           1/2     Running             0          61s
my-app-mongodb-7d7d86796b-rp9gn           2/2     Running             0          67s
my-app-fast-api-webapp-5cb74848bd-bdlrq   0/1     PodInitializing     0          70s
my-app-fast-api-webapp-5cb74848bd-bdlrq   0/1     Running             0          2m5s
my-app-fast-api-webapp-5cb74848bd-bdlrq   1/1     Running             0          2m10s
```
Simulamos una prueba de estres para visualizar el autoescalado.
Para ello nos conectamos al siguiente pod.
```bash
export POD_NAME=$(kubectl get pods --namespace fast-api -l "app.kubernetes.io/name=fast-api-webapp,app.kubernetes.io/instance=my-app" -o jsonpath="{.items[0].metadata.name}")
```
Accedemos a una shell interactiva
```bash
kubectl -n fast-api exec --stdin --tty $POD_NAME -- /bin/sh
```
Una vez dentro de la shell, instalamos los binarios necesarios
```bash
apk update && apk add git go
```
Descargamos el siguiente repositorio y accedemos a èl.

```bash
git clone https://github.com/jaeg/NodeWrecker.git
cd NodeWrecker
go build -o extress main.go
```
Y lanzamos la prueba de estres dentro del pod
```bash
./extress -abuse-memory -escalate -max-duration 10000000
```
Comprobamos que recibimos notificaciones del alert manager configurado en slack
![monitoring alert1](https://user-images.githubusercontent.com/39458920/168493253-de63d480-1404-4003-950a-7a903c908fbf.JPG)

Hacemos un port-forward al servicio de monitorización de Grafana e importamos el dashboard encontrado  
[aqui,](https://github.com/KeepCodingCloudDevops5/liberando-productos-yilisramirez/blob/main/custom_dashboard.json) para monitorizar:
- El número de llamadas a los endpoints
- El número de veces que la aplicación ha arrancado
```bash
kubectl -n monitoring port-forward svc/prometheus-grafana 3000:3000
```
Para ello debemos simular una serie de llamadas a los diferentes endpoints, por lo que tendremos que acceder al <b>Fast API Swagger</b> haciendo localhost al puerto 8081 

```bash
kubectl -n fast-api port-forward svc/my-app-fast-api-webapp 8081:8081
```
![endpoint bye](https://user-images.githubusercontent.com/39458920/168493250-b4859cf7-ec42-4e6a-92d9-54e48bc95c98.JPG)

Una vez importado el dashboard en Grafana podemos visualizar las diferentes llamadas a los endpoints:
![dashboardendpoint](https://user-images.githubusercontent.com/39458920/168553853-9ae6bfa1-8c4a-4401-a989-2acdf9da01ae.JPG)


- Creación de un pipeline en Gitbhub actions para la ejecución de test unitarios de la aplicación
![unit-tests](https://user-images.githubusercontent.com/39458920/168586697-d1704774-30a9-44e5-80f2-9154a9a30882.JPG)

- Creación de un pipeline en Gitbhub actions para la creación de una imagen Docker y push al registro de GHCR
 ![docker build y push](https://user-images.githubusercontent.com/39458920/168586722-438f03ed-5f36-4847-b999-0ae0503fc2ff.JPG)
