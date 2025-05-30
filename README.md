# 🚀 Despliegue de una API con FastAPI en AWS Fargate

Este repositorio contiene el código de una API simple construida con **FastAPI**, junto con la configuración necesaria para empaquetarla en un contenedor Docker y desplegarla en **AWS Fargate** usando **Amazon ECS**.

---

## 🧠 Plan de Estudio y Conceptos Clave

### 🌩️ Introducción a AWS

Amazon Web Services (AWS) es la nube más utilizada del mundo. Ofrece servicios bajo demanda para almacenamiento, computación, bases de datos, redes, análisis, inteligencia artificial, desarrollo y más.

### 🐳 Docker

Docker es una herramienta que permite "empaquetar" aplicaciones y todas sus dependencias en un contenedor. Esto asegura que la app funcione igual sin importar el entorno donde se ejecute.

**Beneficios:**
- Portabilidad
- Escalabilidad
- Aislamiento
- Fácil despliegue

### ⚙️ AWS Fargate + ECS

#### ¿Qué es AWS Fargate?

Fargate es una tecnología serverless para ejecutar contenedores en AWS sin tener que administrar servidores o clústeres. Se integra con ECS (Elastic Container Service) y EKS (Elastic Kubernetes Service).

#### ¿Qué es Amazon ECS?

ECS es un servicio de orquestación de contenedores. Permite ejecutar, detener y administrar contenedores en un clúster de instancias administradas.

#### Principales conceptos:

- **Task Definition**: Documento JSON que describe uno o varios contenedores a ejecutar.
- **Service**: Ejecuta y mantiene una cantidad deseada de tareas.
- **Cluster**: Agrupación lógica de recursos donde se ejecutan las tareas.
- **IAM Role**: Controla los permisos del contenedor para interactuar con otros servicios de AWS.
- **Security Group**: Define reglas de acceso de red.
- **VPC**: Red virtual privada donde viven los servicios.
- **CloudWatch Logs**: Donde se almacenan los registros del contenedor.

---

## 📦 Estructura del Proyecto

```
📁 fastapi-aws
├── app.py                # Aplicación principal FastAPI
├── Dockerfile            # Instrucciones para construir la imagen
├── requirements.txt      # Dependencias del proyecto
└── README.md            # Documentación y guía de despliegue
```

---

## 🚀 Despliegue en AWS ECS con Fargate

### 1️⃣ Crear repositorio en Amazon ECR

1. Ve a [Amazon ECR](https://console.aws.amazon.com/ecr).
2. Crea un repositorio llamado `fastapi-task`.
3. Desde tu terminal:

```bash
# Reemplaza <account_id> y <region> según tu cuenta
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.us-east-1.amazonaws.com

docker build -t fastapi-task .

docker tag fastapi-task:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/fastapi-task:latest

docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/fastapi-task:latest
```

### 2️⃣ Crear definición de tarea (Task Definition)

1. Ir a **Amazon ECS > Task Definitions > Create new Task Definition**.
2. Seleccionar **Fargate**.
3. Configurar lo siguiente:
   - **Task Name**: `fastapi-task`
   - **Task Role**: `ecsTaskExecutionRole`
   - **Container**:
     - **Name**: `fastapi-container`
     - **Image**: URL del ECR (ej. `<account_id>.dkr.ecr.us-east-1.amazonaws.com/fastapi-task:latest`)
     - **Port mapping**: TCP 8000

### 3️⃣ Crear clúster

1. Ir a **Amazon ECS > Clusters > Create Cluster**
2. Seleccionar "Networking only (Powered by AWS Fargate)"
3. **Nombre del clúster**: `fastapi-cluster`

### 4️⃣ Crear servicio ECS

1. Ir a **Clusters > fastapi-cluster > Services > Create**
2. Configurar:
   - **Launch type**: Fargate
   - **Task definition**: `fastapi-task`
   - **Service name**: `fastapi-service`
   - **Number of tasks**: 1
3. Seleccionar:
   - **VPC**: Usar una existente o dejar la predeterminada
   - **Subnets**: al menos 2
   - **Assign Public IP**: ENABLED
   - **Security group**: Crear uno nuevo que permita tráfico entrante en el puerto 8000 desde `0.0.0.0/0`

### 5️⃣ Verificar despliegue

1. Ir al clúster > pestaña **Tasks**
2. Esperar que el estado sea **RUNNING**
3. Seleccionar la tarea > copiar la **Public IP**
4. Probar la API en el navegador o con `curl`:

```bash
curl http://<public-ip>:8000
```

Deberías ver:

```json
{"message": "Hello World"}
```

---

## 🧪 Rutas de prueba

- `GET /` → retorna `{"message": "Hello World"}`
- `GET /health` → retorna `{"status": "healthy"}`

---

## 🛠️ Recursos adicionales

- [Documentación oficial de AWS Fargate](https://docs.aws.amazon.com/fargate/)
- [Documentación de Amazon ECS](https://docs.aws.amazon.com/ecs/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Docker Cheat Sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)
- [Artículo: Scalable applications with AWS Fargate & ECS](https://aws.amazon.com/blogs/containers/)

---

## 📚 Lecturas recomendadas

- [AWS Getting Started - Cloud Essentials](https://aws.amazon.com/es/getting-started/cloud-essentials/)
- [Docker from Beginner to Expert Tutorial](https://medium.com/@techsuneel99/docker-from-beginner-to-expert-a-comprehensive-tutorial-5efec10c82ab)
- [A Comprehensive Guide to Docker](https://medium.com/@moraneus/a-comprehensive-guide-to-docker-286d6f3ad122)
- [Guía Rápida de Docker en Español](https://medium.com/@iamluisgb/gu%C3%ADa-r%C3%A1pida-de-docker-3069065fa629)
