

**Correcciones CI/CD** 

**Saray Cerquera Vallejo**  
**Natalia Ramirez Ocampo**

**Prof. Cesar Augusto Palacios**  
**Administración de Sistemas Informáticos**  
**Devops**

**Universidad Nacional de Colombia – Sede Manizales**

**Facultad de Administración**  
**Departamento de Informática y Computación**

**Manizales, Colombia**  
**2026 \- 1**

**Errores encontrados** 

**Primer error**   
Se encontró un error al intentar correrlo por primera vez

Ejecutar tests  
Run pytest test\_app.py  
/home/runner/work/.../script.sh: line 1: pytest: command not found  
Error: Process completed with exit code 127\.

—----------------------------------------------------------------------------------------------------

Al revisar en requeriments.txt no tenía pytest, por eso no ejecutaba, por ende, se le agregó. 

flask==3.0.0  
psutil==5.9.8  
pytest==8.0.0

—--------------------------------------------------------------------------------------------------

Segundo Error

El test\_home está esperando que sea “running”  y al app está retornando un “ok” 

\==================== FAILURES \============================

test\_home

def test\_home():  
    client \= app.app.test\_client()  
    response \= client.get('/')  
    assert response.status\_code \== 200  
    data \= response.get\_json()  
    assert data\["status"\] \== "running"  
    AssertionError: assert 'ok' \== 'running'

    \- running  
    \+ ok

test\_app.py:8: AssertionError

test\_health

—-----------------------------------------------------------

@app.route(\*/\*)  
def home():  
return jsonify({"status": "ok", "service": “devops-api"})|

—-----------------------------------------------------------

La corrección es cambiarle el estado de “running” por “ok”

import app

def test\_home():  
   client \= app.app.test\_client()  
   response \= client.get('/')  
   assert response.status\_code \== 200  
   data \= response.get\_json()  
   assert data\["status"\] \== "ok"

—---------------------------------------------------------

**Tercer Error** 

test\_healthy está esperando un uptime\_seconds pero al parecer no está

\====================================== FAILURES====================================

test\_health

def test\_health():  
    client \= app.app.test\_client()  
    response \= client.get('/health')  
    assert response.status\_code \== 200  
    data \= response.get\_json()  
    assert "uptime\_seconds" in data  
AssertionError: assert 'uptime\_seconds' in {'cpu\_percent': 33.3, 'memory\_percent': 6.1, 'status': 'healthy'}

test\_app.py:15: AssertionError

—--------------------------------------------------------------------

@app.route('/health')  
def health():  
    cpu \= psutil.cpu\_percent()  
    mem \= psutil.virtual\_memory().percent  
    return jsonify({  
        "cpu\_percent": cpu,  
        "memory\_percent": mem,  
        "status": "healthy" if cpu \< 80 and mem \< 80 else "unhealthy"  
    })

—-----------------------------------------------------------  
Se corrigió, agregando uptime y todo lo que requería(import time,etc)

start\_time \= time.time()

@app.route('/')  
def home():  
    return jsonify({"status": "ok", "service": "devops-api"})

@app.route('/health')  
def health():  
    cpu \= psutil.cpu\_percent()  
    mem \= psutil.virtual\_memory().percent  
    uptime \= int(time.time() \- start\_time)

    return jsonify({  
        "cpu\_percent": cpu,  
        "memory\_percent": mem,  
        "uptime\_seconds": uptime,  
        "status": "healthy" if cpu \< 80 and mem \< 80 else "unhealthy"  
    })

**Cuarto Error**  
está definido como test\_metrics, pero en la ruta esta como metric (falta la s)  
\=========================== FAILURES \========================  
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ test\_metrics \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

def test\_metrics():  
    client \= app.app.test\_client()  
    response \= client.get('/metrics')  
\>   assert response.status\_code \== 200  
E   assert 404 \== 200  
E   \+ where 404 \= \<WrapperTestResponse streamed \[404 NOT FOUND\]\>.status\_code  
test\_app.py:21: AssertionError  
\=========================== short test summary info \============================  
FAILED test\_app.py::test\_metrics \- assert 404 \== 200  
\+ where 404 \= \<WrapperTestResponse streamed \[404 NOT FOUND\]\>.status\_code  
\===================== 1 failed, 2 passed in 0.16s \=====================

@app.route('/metric')  
def metric():  
    cpu \= psutil.cpu\_percent()  
    mem \= psutil.virtual\_memory().percent  
    return f"""\# HELP app\_cpu\_percent CPU usage percentage  
\# TYPE app\_cpu\_percent gauge  
app\_cpu\_percent {cpu}  
\# HELP app\_memory\_percent Memory usage percentage  
\# TYPE app\_memory\_percent gauge  
app\_memory\_percent {mem}  
**"""**  
**—-------------------------------------------------------------------------------------------**

La corrección fue agregada  

@app.route('/metrics')  
def metric():  
    cpu \= psutil.cpu\_percent()  
    mem \= psutil.virtual\_memory().percent  
    return f"""\# HELP app\_cpu\_percent CPU usage percentage  
\# TYPE app\_cpu\_percent gauge  
app\_cpu\_percent {cpu}  
\# HELP app\_memory\_percent Memory usage percentage  
\# TYPE app\_memory\_percent gauge  
app\_memory\_percent {mem}  
"""

**—---------------------------------------------------------------------------**

**Quinto Error**  
Al intentar acceder al [http://localhost:5000/health](http://localhost:5000/health) aparece un error que indica que el contenedor se cerró o no abrió correctamente 

Run curl \-s http://localhost:5000/health   
Error: Process completed with exit code 56\. 

Se encontró que el puerto en [app.py](http://app.py), señalaba al 5001 en lugar del 5000

if \_\_name\_\_ \== '\_\_main\_\_':  
    app.run(host='0.0.0.0', port=5001)  
—-----------------------------------------------------------------------------------------------------  
La corrección corresponde a cambiar el “port” por 5000

if \_\_name\_\_ \== '\_\_main\_\_':  
    app.run(host='0.0.0.0', port=5000)

  —--------------------------------------------------------------------------------------------------  
**Otros errores encontrados**

docker-compose.yml, el puerto del contenedor estaba cambiado a 5001, por ende no apuntaba al correcto (5000) luego se corrigió

api:  
    build: .  
    ports:  
      \- "5000:5001"

  api:  
    build: .  
    ports:  
      \- "5000:5000"  
—--------------------------------------------------------  
en prometheus.yml se usaba el localhost en lugar del nombre definido para la api ("api") y además ("metric") en lugar de ("metrics")
scrape\_configs:  
  \- job\_name: 'devops-api'  
    static\_configs:  
      \- targets: \['localhost:5000'\]  
    metrics\_path: '/metric'

scrape\_configs:  
  \- job\_name: 'devops-api'  
    static\_configs:  
      \- targets: \['api:5000'\]  
    metrics\_path: '/metrics'  
—------------------------------------------------------------

Apunta a otro repositorio, se cambió 

antes:   
\!\[CI/CD\]([https://github.com/cesarpalacios/devops-ci-workshop/actions/workflows/ci.yml/badge.svg](https://github.com/cesarpalacios/devops-ci-workshop/actions/workflows/ci.yml/badge.svg))

después:   
\!\[CI/CD\](https://github.com/nramirezoc/devops-ci-workshop/actions/workflows/ci.yml/badge.svg)

