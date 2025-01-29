### Prueba 1 - Diagrama de Red

**Descripción de la arquitectura:**

Para esta prueba, vamos a diseñar una arquitectura que cumpla con los requisitos mencionados: alta disponibilidad (HA), escalabilidad, y el soporte de un frontend en React.js, un backend en Django, bases de datos relacionales y no relacionales, y el consumo de dos microservicios externos. La arquitectura se diseñará para funcionar en **Google Cloud Platform (GCP)**, aunque también sería aplicable en **AWS**.

**Componentes:**

1. **Frontend (React.js)**: 
   - Se encuentra en un **bucket de almacenamiento estático** (por ejemplo, en GCP, un Bucket de Cloud Storage o en AWS, un S3 bucket) para asegurar una alta disponibilidad. Este bucket estará configurado con un CDN (por ejemplo, **Cloud CDN en GCP** o **CloudFront en AWS**) para mejorar la velocidad de acceso y reducir la latencia global.

2. **Backend (Django)**:
   - El backend estará alojado en **Google Kubernetes Engine (GKE)** (en GCP) o **Elastic Kubernetes Service (EKS)** (en AWS) para garantizar escalabilidad y alta disponibilidad.
   - Los pods de Kubernetes estarán desplegados en múltiples zonas de disponibilidad para soportar fallos en una sola zona y para distribuir la carga de manera eficiente.

3. **Bases de datos**:
   - **Relacional**: Utilizaremos **Cloud SQL en GCP** o **RDS en AWS**. Esta base de datos será de alta disponibilidad, con un clúster multi-región o con réplicas de lectura para distribuir la carga.
   - **No Relacional**: Usaremos **Firestore (GCP)** o **DynamoDB (AWS)**, que es completamente gestionada, escalable y permite manejar cargas variables.

4. **Microservicios Externos**:
   - El backend en Django se conecta a dos microservicios externos. Estos servicios se pueden alojar en contenedores dentro de la infraestructura, o pueden ser aplicaciones SaaS externas a las que se haga peticiones HTTP/REST desde el backend.

5. **Balanceo de Carga**:
   - Utilizaremos un **Load Balancer** global (como **Google Cloud Load Balancer** o **AWS ELB**) para distribuir el tráfico entre los servicios frontend, backend y bases de datos, asegurando que el tráfico se dirija a las instancias más cercanas geográficamente para reducir la latencia.

6. **Monitorización y Logging**:
   - Integración de **Google Operations (anteriormente Stackdriver)** o **AWS CloudWatch** para monitorear la infraestructura y generar alertas de salud del sistema.
   - **Logging centralizado** con **Google Cloud Logging** o **AWS CloudTrail** para auditar y analizar el rendimiento y los fallos.

**Flujo de tráfico:**
1. Los usuarios acceden al frontend (React.js) a través de un CDN.
2. El frontend realiza peticiones a la API RESTful del backend (Django) a través del balanceador de carga.
3. El backend maneja las solicitudes y consulta las bases de datos (relacional o no relacional) según el tipo de datos que se necesiten.
4. El backend también interactúa con los microservicios externos, enviando y recibiendo datos según lo requerido.
5. Los datos procesados se devuelven al frontend para su visualización.

**Alta disponibilidad y escalabilidad**:
- Utilizamos Kubernetes para gestionar la infraestructura del backend, lo que permite que el número de instancias de Django escale según la carga.
- Las bases de datos están configuradas para replicación multi-región y las aplicaciones están distribuidas en múltiples zonas de disponibilidad.

**Diagramas de Red**:
- Puedes realizar un diagrama usando herramientas como **Lucidchart**, representando las conexiones entre los diferentes servicios, balanceadores de carga, bases de datos y microservicios externos.

---

### Prueba 2 - Despliegue de una aplicación Django y React.js

Para esta prueba, vamos a crear un **docker-compose** que contenga tanto la aplicación backend en Django como el frontend en React.js.

**Pasos para crear el despliegue:**

1. **Dockerfiles**:
   - **Dockerfile para Django**:
     ```Dockerfile
     FROM python:3.9-slim

     WORKDIR /app

     COPY requirements.txt /app/
     RUN pip install -r requirements.txt

     COPY . /app/

     CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
     ```

   - **Dockerfile para React**:
     ```Dockerfile
     FROM node:16

     WORKDIR /app

     COPY package.json /app/
     RUN npm install

     COPY . /app/

     RUN npm run build

     CMD ["npm", "start"]
     ```

2. **docker-compose.yml**:
   ```yaml
   version: '3'
   services:
     django-backend:
       build:
         context: ./backend
       ports:
         - "8000:8000"
       networks:
         - app-network
     
     react-frontend:
       build:
         context: ./frontend
       ports:
         - "3000:3000"
       networks:
         - app-network

   networks:
     app-network:
       driver: bridge
   ```

3. **Instrucciones**:
   - Clona el repositorio y navega al directorio de la aplicación.
   - Ejecuta `docker-compose up --build` para construir y levantar los contenedores.
   - La aplicación backend estará disponible en `http://localhost:8000` y el frontend en `http://localhost:3000`.

4. **Despliegue en la nube**:
   - Puedes desplegar esta solución en la nube usando **Google Cloud Run**, **AWS ECS** o un **Cluster de Kubernetes**.
   - Asegúrate de exponer los puertos y configurar el dominio, la base de datos y otros servicios necesarios en la nube.

---

### Prueba 3 - CI/CD

Para esta prueba, vamos a crear un **pipeline de CI/CD** usando **GitHub Actions** para automatizar la creación de la imagen Docker de un contenedor **Nginx** con un archivo `index.html` por defecto. Cada vez que se haga un cambio en el archivo `index.html`, el pipeline debe crear una nueva imagen y desplegarla.

**Configuración de GitHub Actions**:

1. **Dockerfile para Nginx**:
   ```Dockerfile
   FROM nginx:latest
   COPY index.html /usr/share/nginx/html/index.html
   ```

2. **Archivo de workflow (`.github/workflows/cicd.yml`)**:
   ```yaml
   name: CI/CD for Nginx

   on:
     push:
       paths:
         - index.html
     branches:
       - main

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Build Docker image
           run: |
             docker build -t nginx-image .
             docker tag nginx-image myrepo/nginx:latest

         - name: Push to Docker Hub
           run: |
             docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
             docker push myrepo/nginx:latest
         
         - name: Deploy to cloud
           run: |
             # Here you can use tools like AWS CLI, GCP SDK or other deployment tools to update the image in the cloud platform.
   ```

3. **Configurar los secretos** en GitHub para almacenar las credenciales de Docker Hub.

Este pipeline detectará cambios en el archivo `index.html`, construirá la nueva imagen y la subirá a Docker Hub o el repositorio que elijas, desplegándola automáticamente en tu infraestructura en la nube.

---

### Conclusión:

Este enfoque proporciona una solución distribuida y escalable utilizando tecnologías de contenedores, Kubernetes, bases de datos gestionadas y herramientas de CI/CD para automatizar el proceso de despliegue. Todo el proceso se puede escalar fácilmente y se adapta a los requisitos de alta disponibilidad y carga variable.