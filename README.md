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
 minikube start -p fastapi
 ```
