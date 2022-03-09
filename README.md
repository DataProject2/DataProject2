# 							Demo IoT - Data Project 2 - Grupo 6

## 												Dispositivo IoT - iCare Solutions

![Icare](https://user-images.githubusercontent.com/84716641/157240802-f6557f7f-ab94-48a9-8fd9-d27914de1b2b.png)

_La salud, al momento, mejor_



### Arquitectura de datos:


![arquitectura](https://user-images.githubusercontent.com/84716641/157487670-12511677-c483-435b-bcf9-a22632f7fcde.png)



### Requisitos previos:

---



* Clonar este repositorio en Google Cloud Platform;

* Habilitar las APIs requeridas con los siguientes comandos:

  

```
gcloud services enable dataflow.googleapis.com
gcloud services enable cloudiot.googleapis.com
gcloud services enable cloudbuild.googleapis.com
```

- Crear entorno virtual de Python;

```python
virtualenv -p python3 <ENVIRONTMENT_NAME>
source <ENVIRONMENT_NAME>/bin/activate
```

- Instalar las dependencias necesarias.

```python
pip install -U -r setup_dependencies.txt
```



### PubSub:

---

- Ir a Cloud Console PubSub;
- Crear un tema y añadir una subscripción por defecto.



### Cloud Storage

---

- Ir a Cloud Console Cloud Storage;
- Crear un bucket con un nombre único



### IoT Core

---

Para esta demostración, usaremos Cloud Shell como un dispositivo IoT generador de datos;

- Ir a Cloud Console IoT Core;

- Crear un registro IoT escogiendo el tema PubSub creado anteriormente;
- Ir a Cloud Shell y generar una clave "RSA key with-self signed X509 certificate"
- Registrar un dispositivo y actualizar la clave _rsa_cert.pem_ en la sección de autenticación

Ahora ya hemos vinculado nuestro dispositivo con IoT Core.



### Big Query

---

- Ir a Cloud Console Big Query

- Crear un BigQuery Dataset

  

### Dataflow

---

- Ir a la carpeta 02_Dataflow para crear una Plantilla;
- Empaquetar el código python en una imagen de Docker y guardarlo en el registro de contenedores, ejecutando este comando;

```` 
gcloud builds submit --tag 'gcr.io/<YOUR_PROJECT_ID>/<YOUR_FOLDER_NAME>/<YOUR_IMAGE_NAME>:latest' .
````



- Crear la Plantilla Dataflow desde la imagen Docker:

````
gcloud dataflow flex-template build "gs://<YOUR_BUCKET_NAME>/<YOUR_TEMPLATE_NAME>.json" \
  --image "gcr.io/<YOUR_PROJECT_ID>/<YOUR_FOLDER_NAME>/<YOUR_IMAGE_NAME>:latest" \
  --sdk-language "PYTHON" 
````

- Para terminar, ejecutar el Dataflow job desde la Plantilla:

````
gcloud dataflow flex-template run "<YOUR_DATAFLOW_JOB_NAME>" \
    --template-file-gcs-location "gs://<YOUR_BUCKET_NAME>/<YOUR_TEMPLATE_NAME>.json" \
    --region "europe-west1"
````

![dataflowjob](https://user-images.githubusercontent.com/84716641/157240887-d24928a9-c58a-473d-b897-1c26c47361df.png)


### Cloud Functions

---

- Crear una Google Cloud Function para pasar los datos a Firestore;
- "main.py"

````python
import base64
import json
from google.cloud import firestore
from datetime import datetime

client = firestore.Client(project='lithe-window-342416')

def dataICareToFireStore(event, context):
    message = ''
    if 'data' in event:
        message = base64.b64decode(event['data']).decode('utf-8')
        ICaredata = json.loads(message)
        print(type(ICaredata))
        print(ICaredata["device_id"])

    doc = client.collection('device').document(ICaredata["device_id"])
    doc.set({
    'device_id': ICaredata["device_id"],
    'user_id': ICaredata["user_id"],
    'timeStamp': ICaredata["timeStamp"],
    'presion_arterial': ICaredata["presion_arterial"],
    'frec_cardiaca': ICaredata["frec_cardiaca"],
    'frec_respiratoria': ICaredata["frec_respiratoria"],
    'tension': ICaredata["tension"],
    'oxigeno_sangre': ICaredata["oxigeno_sangre"],
    })
    print(f'message: {message}')
````



- "requirements.txt"

````
google-cloud-firestore>=2.1.3
google-cloud-iot
````



### Enviar datos desde el dispositivo

- Ir a la carpeta 01_IoTCore y ejecutar el siguiente comando:

````
python edemDeviceData.py \
    --algorithm RS256 \
    --cloud_region europe-west1 \
    --device_id <YOUR_IOT_DEVICE_NAME> \
    --private_key_file rsa_private.pem \
    --project_id <YOUR_PROJECT_ID> \
    --registry_id <YOUR_IOT_REGISTRY>
````



### Verificar que los datos se almacenen correctamente y visualizar con Data Studio



- Ir a Big Query;


<img src="00_Images\Big Query.png" alt="Image" style="zoom:15%;" />



- FireStore



- Vincular ICare Dashboard


<img src="00_Images\dashboard.png" alt="Image" style="zoom:15%;" />


### Mobile App


![mobileapp](https://user-images.githubusercontent.com/84716641/157487766-6cf2029c-ff66-4808-bc9e-de6ea8aabb00.png)

![mobileapp2](https://user-images.githubusercontent.com/84716641/157487781-a1b6afdc-bad2-44f2-8d5f-b8a527a52158.png)



## Conoce a nuestro equipo

Compuesto por:

**Jose Manuel García**  [![Linkedin Badge](https://img.shields.io/badge/-JoseManuel-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/chiquillo/)](https://www.linkedin.com/in/jogacu/)


**Sergi Joan Sastre**  [![Linkedin Badge](https://img.shields.io/badge/-Sergi-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/sergisastre/)](https://www.linkedin.com/in/sergisastre/)

**Álvaro Chiquillo**  [![Linkedin Badge](https://img.shields.io/badge/-Alvaro-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/chiquillo/)](https://www.linkedin.com/in/chiquillo/)


**Héctor García**  [![Linkedin Badge](https://img.shields.io/badge/-Héctor-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/hhectorgarcia/)](https://www.linkedin.com/in/hhectorgarcia/)

**Diego Cortes**  [![Linkedin Badge](https://img.shields.io/badge/-Diego-blue?style=flat-square&logo=Linkedin&logoColor=white&link=https://www.linkedin.com/in/diego-cortes-gil/)](https://www.linkedin.com/in/diego-cortes-gil/)
