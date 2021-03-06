# Microservicio en Java
Este proyecto de compone de:
- Microservicio **ms-devops** en Java, ubicación carpeta **app**
- Manifiestos creación de objetos en clúster de Kubernetes, ubicación carpeta **kubernetes**
- Pipeline de Azure, archivo **azure-pipelines.yml**
Se ha usado la herramienta API Management para publicar el punto de acceso. Ver la sección **Endpoint**

## Microservicio **ms-devops**
El microservicio está desarrollado siguiendo la metodología TDD, posee las siguientes características:
- Lenguaje de programación **Java**
- Gestión de proyectos y dependencias con **Maven**
- Pruebas automáticas con **Junit**

## Manifiestos creación de objetos en clúster de Kubernetes
El archivo **deployment.yml** permite crear un objeto en k8s de tipo deployment, con dos pods con la imagen de docker **ms-devops**, la cuál está registrada en **acrdevopsbp.azurecr.io**.
El archivo **service.yml** permite crear un servicio de tipo balanceador de carga

## Pipeline de Azure
Se ha usado la herramienta Azure Devops para la creación del pipeline. El proyecto público se encuentra [aquí](https://dev.azure.com/pedroalexanderdev/devops-bp)-
El pipeline tiene 2 stages: **Build** y **Deploy to Dev**
### Build
Esta stage consta de las siguientes tareas:
- Análisis de código estático con SonarClod y publicación de [reportes publicos](https://sonarcloud.io/project/overview?id=pedroalexanderdev_devops-bp).
- Testing con Junit 
- Contrucción código
- Creación de la imagen **ms-devops**.  Publicación en ACR **acrdevopsbp.azurecr.io**
- Publicación de artefactos, archivos de kubernetes
### Deploy
- Descarga de artefactos, archivos de kubernetes
- Creación de secreto descarga de imágenes desde acr
- Despliegue de clúster de k8s en base a los manifiestos
- 
# Endpoint
El aplicativo de microservicio tiene endpoint el host **apimanagement-bp.azure-api.net**. Url completa: https://apimanagement-bp.azure-api.net/DevOps
## Características
- Se usa como API-Key **2f5ae96c-b558-4c7b-a590-a501ae1c3f6c**
- La API publicada está gestionada con API Management de Azure
- El acceso para obtención de token se lo realiza mediante Azure Active Directory (AAZ). Se ha creado el usuario **bp@pedroalexanderdevhotmail.onmicrosoft.com** para ejemplificar este proceso. Una vez AAZ nos devuelve el token, se puede invocal el servicio.
- Tokens de tipo JWT
## Test del Endpoint
Los siguientes pasos han sido probados en sistema operativo linux, se necesita instalar python3:
- Crear el archivo **get-tokens-for-user.sh** y agregar el siguiente código
```sh
#!/bin/sh
JWT=$(curl -s POST 'https://login.microsoftonline.com/d25969d2-52f4-4199-acc3-c83825f4e462/oauth2/v2.0/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Cookie: fpc=AtuZbYPyObJIjIHqTao-KQ79gGDLAgAAAI5-5dkOAAAA; stsservicecookie=estsfd; x-ms-gateway-slice=estsfd' \
--data-urlencode 'grant_type=password' \
--data-urlencode 'password=Huga92531' \
--data-urlencode 'client_id=5de43e34-9dba-4f07-bcaa-c78e91c30f82' \
--data-urlencode 'scope=api://d511491d-1396-47df-89be-09973aa4f0e1/access.api' \
--data-urlencode 'username=bp@pedroalexanderdevhotmail.onmicrosoft.com' | \
python3 -c "import sys, json; print(json.load(sys.stdin)['access_token'])")
echo "$JWT"
```
Los comandos del archivo nos devuelve un token emitido por AAZ para poder conectarnos al endpoint.
Para realizar las peticiones HTTP ejecutar:
```sh
HOST=apimanagement-bp.azure-api.net
```
```sh
JWT=$(sudo sh get-tokens-for-user.sh)
```
```sh
curl -X POST -H "X-Parse-REST-API-Key: 2f5ae96c-b558-4c7b-a590-a501ae1c3f6c" -H "X-JWT-KWY: ${JWT}" -H "Content-Type: application/json" -d '{ "message" : "This is a test", "to": "Juan Perez", "from": "Rita Asturia", "timeToLifeSec" : 45 }' https://${HOST}/DevOps
```

Se mostrará el resultado:
```sh
{"message":"Hello Juan Perez your message will be send"}
```

