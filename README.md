# RA2 SBD · Dashboard IoT en Streamlit con datos en S3 (AWS Academy)

**Versión:** `v1.0-entrega`
**Contexto:** Proyecto de arquitectura en AWS y visualización de datos para el curso de especialización en Sistemas de Big Data (RA2).

Este repositorio contiene la implementación de un sistema de ingesta y el despliegue de un cuadro de mando interactivo para la monitorización de telemetría IoT (sensores de temperatura y CO2). Todo el ecosistema está alojado y configurado sobre servicios de Amazon Web Services (almacenamiento privado en S3 y servidor web en EC2) utilizando Streamlit.

---

## Objetivo del Proyecto
El proyecto implementa una **arquitectura de datos IoT** orientada a la generación, almacenamiento y consumo:
1. **Ingesta:** Generación de datos simulados de sensores IoT (JSON) y subida a un bucket S3 privado mediante script.
2. **Despliegue:** Configuración de una instancia EC2 (Ubuntu) para alojar la aplicación web.
3. **Visualización:** Creación de un Dashboard interactivo con **Streamlit** que consume los datos de S3 de forma segura mediante variables de entorno y muestra gráficas, tablas y mapas.

---

## Estructura del repositorio

```text
ra2-awsacademy-s3-streamlit-starter-template/
├── app/                                  # Código fuente de la aplicación web
│   ├── services/                         # Módulos auxiliares de conexión y procesado
│   │   ├── preprocessing.py              # Limpieza, conversión de fechas y estructuración con Pandas
│   │   └── s3_loader.py                  # Conexión y descarga del archivo JSON desde el bucket S3 privado
│   └── dashboard.py                      # Script principal de la interfaz interactiva de Streamlit
├── docs/                                 # Documentación original y evidencias del proyecto
│   ├── decisiones.md                     # Registro de decisiones técnicas adoptadas
│   ├── entrega.md                        # Guía de pasos para la entrega
│   ├── enunciado.md                      # Documento con los requisitos de la práctica
│   ├── evidencias.md                     # Archivo para la justificación y capturas de pantalla
│   ├── pistas.md                         # Sugerencias de desarrollo
│   └── rubric.md                         # Criterios de evaluación
├── img/                                  # Directorio destinado a almacenar las capturas de evidencia
├── notebooks/                            # Entorno para la creación e ingesta de datos sintéticos
│   ├── ingesta_json_S3.ipynb             # Notebook con Faker para simular métricas IoT y subirlas a S3
│   └── README.md                         # Notas o instrucciones específicas de la ingesta
├── scripts/                              # Automatización y despliegue en la instancia EC2
│   ├── ec2_setup.sh                      # Script de configuración inicial para el entorno Ubuntu
│   ├── healthcheck.sh                    # Script para la comprobación del estado del servicio
│   └── run_streamlit_nohup.sh            # Orquestador para ejecutar Streamlit en segundo plano
├── README.md                             # Documentación principal del proyecto (Este archivo)
└── requirements.txt                      # Dependencias necesarias (Streamlit, pandas, plotly, boto3, etc.)
```

---

## Arquitectura del proyecto

![Arquitectura proyecto](./img/arquitectura_proyecto.png)

---

## Pasos de Ejecución (Reproducibilidad)

A continuación se detallan los pasos exactos realizados para construir y desplegar este proyecto, cubriendo desde la generación de datos hasta la puesta en producción.

### Paso 1: Generación y subida de datos a S3 (Ingesta)
Se ha creado un notebook en Python (ubicado en `notebooks/`) que utiliza la librería `Faker` para simular un entorno IoT realista:
* Genera lecturas para una "flota" de 5 sensores únicos (`SENS-01` a `SENS-05`).
* Las fechas (`timestamp`) se distribuyen aleatoriamente a lo largo de los últimos 7 días para poder visualizar la evolución temporal.
* El archivo resultante (`iabd03_sensores.json`) se sube automáticamente a la ruta `data/sensores/` del bucket privado en S3 usando `boto3`.
* Por eficiencia, el notebook elimina el archivo `.json` local una vez confirmada la subida exitosa.

### Paso 2: Configuración del entorno EC2 y Conexión SSH
Se ha aprovisionado una instancia EC2 (Ubuntu) con el Security Group configurado para permitir tráfico HTTP en el puerto `8501`. 

**Resolución de incidencias de conectividad SSH (Troubleshooting)**
Al intentar establecer la conexión SSH con la instancia EC2 mediante el comando `ssh -i labsuser.pem ubuntu@3.80.119.225`, se detectó un error de seguridad relacionado con los permisos del sistema operativo local:

Se establece la conexión SSH con la instancia. (*Nota: Si experimentas errores de permisos con el archivo `.pem` en Windows, consulta la sección de Resolución de Problemas al final de este documento*).
  
```bash
ssh -i labsuser.pem ubuntu@<IP_PUBLICA_EC2>
```

Una vez dentro de la máquina EC2:
1. Se configuran las credenciales de AWS en el archivo oculto ~/.aws/credentials.
   ```bash
   mkdir -p ~/.aws
   nano ~/.aws/credentials
   ```
2. Se clona el repositorio.
   ```bash
   sudo apt update
   sudo apt install -y git
   git clone <URL_DE_VUESTRO_FORK>
   ```
3. Se ejecuta el script de instalación de dependencias (bash scripts/ec2_setup.sh).
   ```bash
   cd <repo>
   bash scripts/ec2_setup.sh
   ```

### Paso 3: Configuración de Variables de Entorno
Para cumplir con las buenas prácticas de seguridad y evitar hardcodear secretos en el código, la aplicación lee la ubicación de los datos desde las variables de entorno del sistema. Antes de arrancar la aplicación, se exportan en la terminal:
```bash
export AWS_REGION="us-east-1"
export S3_BUCKET="iabd03-tarea-ra2"
export S3_KEY="data/sensores/iabd03_sensores.json"
```

### Paso 4: Despliegue en segundo plano
Para garantizar que el Dashboard siga activo tras cerrar la conexión SSH, se procede a ejecutar la aplicación en segundo plano mediante el script automatizado que hemos preparado:
```bash
bash scripts/run_streamlit_nohup.sh
```

**Nota técnica sobre este script:** Internamente, este archivo se encarga de inyectar la variable `PYTHONPATH=`. para resolver correctamente las importaciones de los submódulos y lanza el servidor con `nohup`. El comando subyacente que está ejecutando es:
```bash
PYTHONPATH=. nohup streamlit run app/dashboard.py --server.address 0.0.0.0 --server.port 8501 > streamlit.log 2>&1 &
```
*(Consulta la sección de Resolución de Problemas para más detalles sobre por qué fue necesaria la variable PYTHONPATH).*

---

## Características del Dashboard
El Dashboard es accesible a través de la IP Pública de la instancia: `http://<IP_PUBLICA_EC2>:8501`.

Incluye las siguientes funcionalidades operativas:
* **Lectura Segura:** Conexión a S3 sin exponer claves en el frontend.
* **Filtros interactivos:** Filtrado dinámico por estado del sensor (OK, WARN, FAIL) y rango de temperatura mediante sliders.
* **Métricas KPI:** Conteo de registros totales vs filtrados, número de sensores únicos y momento de la última lectura (formateada para mejor legibilidad).
* **Gráficas Plotly:** Línea de evolución temporal de la temperatura y gráfico de barras con el nivel medio de CO2 agregado por sensor.
* **Geolocalización:** Mapa integrado (`st.map`) con las coordenadas (lat/lon) exactas de las lecturas.

---

## Resolución de Problemas (Troubleshooting)

### 1. Error de conexión SSH: "UNPROTECTED PRIVATE KEY FILE!"
* **Problema:** Al intentar conectar por SSH desde Windows, el cliente rechaza la conexión con el error `Permission denied (publickey)` y advierte que los permisos de `labsuser.pem` están demasiado abiertos. El protocolo SSH exige estrictamente que el archivo de la clave privada solo sea accesible por el propietario.
* **Solución:** Se utilizaron comandos de `icacls` en PowerShell para restringir los permisos y cumplir con la política de seguridad:
  
  Primero, se eliminó la herencia de permisos:
  ```bash
  icacls labsuser.pem /inheritance:r
  ```
  Segundo, se concedió permiso de solo lectura (R) al usuario actual:
  ```bash
  icacls labsuser.pem /grant "Usuario:(R)"
  ``` 

### 2. Error de importación en Streamlit (ModuleNotFoundError)
* **Problema:** Al intentar ejecutar directamente el dashboard, la aplicación fallaba con `ModuleNotFoundError: No module named 'app'`, ya que Python no localizaba la ruta al hacer importaciones absolutas (`from app.services...`).
* **Solución:** Se inyectó la variable de entorno `PYTHONPATH=.` en el script de ejecución (`run_streamlit_nohup.sh`). Al anteponer esto, se indica explícitamente al intérprete que añada el directorio actual (la raíz del proyecto) a su lista de rutas conocidas. De esta forma, resuelve las dependencias sin modificar el código fuente.