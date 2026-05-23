# innovatech-frontend

Frontend del sistema Innovatech, implementado con HTML, CSS, JavaScript, Nginx, Docker, Amazon EC2, Amazon ECR y GitHub Actions.

Este repositorio contiene la interfaz web del sistema, la configuración de Nginx para servir la aplicación y redirigir solicitudes hacia el backend, además del workflow necesario para automatizar el despliegue mediante CI/CD.

## Descripción general

El proyecto forma parte de una arquitectura de tres capas:

```txt
Internet → Frontend → Backend → Base de datos
````

En esta arquitectura, el frontend representa la capa pública del sistema. Es el único componente accesible desde Internet y permite a los usuarios interactuar con la aplicación mediante una interfaz web.

El frontend se comunica con el backend a través de rutas relativas, las cuales son redirigidas por Nginx hacia la IP privada de la instancia EC2 donde se ejecuta la API backend.

## Estructura del repositorio

```txt
innovatech-frontend/
├── .github/
│   └── workflows/
│       └── deploy-frontend.yml
│
├── app.js
├── default.conf
├── Dockerfile
├── index.html
├── .gitignore
└── README.md
```

## Tecnologías utilizadas

* HTML
* CSS
* JavaScript
* Nginx
* Docker
* Amazon EC2
* Amazon ECR
* AWS Systems Manager
* GitHub Actions

## Funcionamiento del frontend

El archivo principal de la interfaz es:

```txt
index.html
```

La lógica de consumo de la API se encuentra en:

```txt
app.js
```

El frontend permite realizar operaciones CRUD sobre productos:

```txt
Crear producto
Listar productos
Editar producto
Eliminar producto
```

Las solicitudes hacia el backend se realizan mediante la ruta relativa:

```txt
/api/productos
```

Esto permite evitar direcciones IP fijas dentro del código JavaScript, dejando que Nginx se encargue de redirigir las solicitudes hacia el backend.

## Configuración de Nginx

El archivo:

```txt
default.conf
```

contiene la configuración personalizada de Nginx.

Este archivo cumple dos funciones principales:

1. Servir los archivos estáticos del frontend.
2. Redirigir las solicitudes `/api/` hacia el backend.

Flujo de comunicación:

```txt
Navegador
   ↓
EC2 Frontend / Nginx
   ↓
/api/productos
   ↓
EC2 Backend / API Node.js
   ↓
EC2 DB / MySQL
```

Durante el despliegue automatizado, el workflow de GitHub Actions actualiza la configuración de Nginx utilizando la IP privada del backend almacenada en GitHub Secrets.

## Dockerfile del frontend

El Dockerfile utiliza la imagen:

```txt
nginx:alpine
```

Su función principal es:

1. Limpiar el contenido por defecto de Nginx.
2. Copiar los archivos `index.html` y `app.js`.
3. Copiar la configuración personalizada `default.conf`.
4. Exponer el puerto `80`.
5. Servir el frontend mediante Nginx.

## Construcción de imagen Docker

Desde la raíz del repositorio:

```bash
docker build -t innovatech-frontend .
```

## Ejecución del contenedor

```bash
docker run -d --name innovatech-frontend -p 80:80 innovatech-frontend
```

Ver contenedores activos:

```bash
docker ps
```

Ver logs del contenedor:

```bash
docker logs innovatech-frontend
```

Detener el contenedor:

```bash
docker stop innovatech-frontend
```

Eliminar el contenedor:

```bash
docker rm innovatech-frontend
```

## Puerto utilizado

```txt
Frontend: 80
```

El puerto `80` permite acceder a la aplicación desde el navegador mediante HTTP.

## Despliegue manual en AWS

El frontend fue desplegado manualmente en una instancia EC2 pública llamada:

```txt
ec2-frontend
```

Esta instancia se ubicó en la subred pública de la VPC del proyecto, permitiendo el acceso desde Internet mediante una IP pública.

El proceso realizado fue:

1. Conexión a la instancia mediante AWS Systems Manager Session Manager.
2. Verificación de Docker y Git.
3. Clonación del repositorio `innovatech-frontend`.
4. Cambio a la rama `deploy`.
5. Edición del archivo `default.conf` para apuntar al backend privado.
6. Construcción de la imagen Docker.
7. Ejecución del contenedor en el puerto `80`.
8. Validación desde navegador usando la IP pública de la instancia.

Comandos principales utilizados:

```bash
git clone https://github.com/dvnxeee/innovatech-frontend.git
cd innovatech-frontend
git checkout deploy
docker build -t innovatech-frontend .
docker run -d --name innovatech-frontend -p 80:80 innovatech-frontend
```

## Integración con backend

El frontend se conecta al backend mediante la configuración de Nginx.

La ruta:

```txt
/api/
```

es redirigida hacia el backend privado:

```txt
http://<IP_PRIVADA_BACKEND>:3001
```

En el proyecto desplegado, esta IP se configuró mediante el secret:

```txt
BACKEND_HOST
```

De esta forma, el navegador accede únicamente al frontend, mientras que la comunicación con el backend ocurre internamente dentro de la infraestructura AWS.

## Amazon ECR

Se creó un repositorio privado en Amazon ECR para almacenar la imagen Docker del frontend:

```txt
innovatech-frontend
```

La imagen fue publicada con la etiqueta:

```txt
latest
```

Esto permite que la instancia EC2 frontend pueda descargar la imagen actualizada desde ECR durante el despliegue automatizado.

## CI/CD con GitHub Actions

Este repositorio cuenta con un workflow de GitHub Actions ubicado en:

```txt
.github/workflows/deploy-frontend.yml
```

El workflow se ejecuta al realizar cambios sobre la rama:

```txt
deploy
```

El flujo automatizado realiza las siguientes acciones:

1. Obtiene el código del repositorio.
2. Configura credenciales temporales de AWS.
3. Actualiza la IP privada del backend en la configuración de Nginx.
4. Inicia sesión en Amazon ECR.
5. Construye la imagen Docker del frontend.
6. Etiqueta la imagen con la URI de ECR.
7. Publica la imagen en Amazon ECR.
8. Ejecuta comandos en EC2 mediante AWS Systems Manager.
9. Detiene el contenedor anterior.
10. Descarga la nueva imagen desde ECR.
11. Levanta el contenedor actualizado en el puerto `80`.

## Secrets utilizados en GitHub Actions

Para el funcionamiento del workflow se configuraron los siguientes secrets en GitHub:

```txt
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
AWS_REGION
AWS_ACCOUNT_ID
ECR_FRONTEND_REPOSITORY
EC2_FRONTEND_INSTANCE_ID
BACKEND_HOST
```

Estos valores permiten que GitHub Actions pueda autenticarse en AWS, publicar la imagen en ECR y desplegar el contenedor actualizado en la instancia EC2 frontend mediante SSM.

## Seguridad

La arquitectura implementada permite que solo el frontend sea accesible desde Internet.

Reglas principales:

* El frontend acepta tráfico HTTP en el puerto `80`.
* El backend no se expone directamente al público.
* El frontend se comunica con el backend mediante la IP privada del backend.
* El acceso administrativo se realiza mediante AWS Systems Manager Session Manager.
* La base de datos no recibe tráfico directo desde el frontend ni desde Internet.

## Estado final del proyecto

El frontend fue desplegado correctamente en AWS y quedó disponible desde la IP pública de la instancia `ec2-frontend`.

La aplicación permitió cargar productos desde la base de datos y ejecutar correctamente las operaciones CRUD desde la interfaz web.

Además, la imagen Docker del frontend fue publicada en Amazon ECR y el workflow de GitHub Actions quedó configurado para automatizar el proceso de construcción, publicación y despliegue.

## Rama de despliegue

La rama utilizada para despliegue es:

```txt
deploy
```

Los cambios realizados sobre esta rama activan el workflow de CI/CD del frontend.