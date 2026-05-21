# innovatech-frontend

Frontend del sistema Innovatech, preparado para contenedorización con Docker, despliegue en AWS EC2 y automatización CI/CD con GitHub Actions.

## Descripción general

Este repositorio contiene el frontend del proyecto Innovatech. La aplicación corresponde a una interfaz web simple desarrollada con HTML, CSS y JavaScript, servida mediante Nginx dentro de un contenedor Docker.

El frontend permite visualizar, crear, editar y eliminar productos mediante consumo de la API backend a través de rutas relativas.

## Estructura del repositorio

```txt
innovatech-frontend/
├── app.js
├── default.conf
├── Dockerfile
├── index.html
├── .gitignore
└── README.md
````

## Tecnologías utilizadas

* HTML
* CSS
* JavaScript
* Nginx
* Docker
* AWS EC2
* Amazon ECR
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

El frontend consume el backend mediante la ruta relativa:

```txt
/api/productos
```

Esto permite que Nginx redirija las solicitudes hacia el backend sin dejar direcciones IP fijas dentro del código JavaScript.

## Configuración de Nginx

El archivo:

```txt
default.conf
```

contiene la configuración personalizada de Nginx.

Este archivo cumple dos funciones principales:

1. Servir los archivos estáticos del frontend.
2. Redirigir las solicitudes `/api/` hacia el backend.

Ejemplo de flujo:

```txt
Navegador
   ↓
Frontend Nginx
   ↓
/api/productos
   ↓
Backend Node.js en puerto 3001
```

En AWS, la configuración de Nginx debe apuntar a la IP privada de la instancia EC2 donde se ejecuta el backend.

## Dockerfile del frontend

El Dockerfile utiliza la imagen:

```txt
nginx:alpine
```

Su función es:

1. Limpiar el contenido por defecto de Nginx.
2. Copiar los archivos `index.html` y `app.js`.
3. Copiar la configuración personalizada `default.conf`.
4. Exponer el puerto `80`.
5. Servir el frontend mediante Nginx.

## Construcción de la imagen Docker

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

## Integración con backend

El frontend se comunica con el backend mediante rutas relativas:

```txt
/api/productos
```

La redirección hacia el backend se realiza mediante Nginx, configurado en el archivo:

```txt
default.conf
```

En el despliegue AWS, el backend debe estar disponible en una instancia EC2 y el frontend debe apuntar a la IP privada de esa instancia.

## Despliegue en AWS

El despliegue proyectado considera:

* Una instancia EC2 pública para el frontend.
* Docker instalado en la instancia EC2.
* Imagen Docker publicada en Amazon ECR.
* Contenedor frontend ejecutándose en el puerto `80`.
* Configuración Nginx para redirigir solicitudes `/api/` hacia el backend.
* Automatización mediante GitHub Actions.

## Registro de imágenes

Se utilizará Amazon ECR como registro de imágenes Docker, ya que el despliegue se realizará dentro del ecosistema AWS.

Esta decisión permite mantener las imágenes del frontend dentro de la misma infraestructura cloud utilizada por el proyecto, facilitando la integración entre GitHub Actions, ECR y EC2.

## Rama de despliegue

La rama utilizada para el despliegue será:

```txt
deploy
```

Los workflows de GitHub Actions serán configurados para ejecutarse al hacer push sobre esta rama.

## Seguridad y red

En AWS, se recomienda que:

* El puerto `80` del frontend esté disponible para acceso desde Internet.
* El backend no sea expuesto directamente al público.
* La comunicación Frontend → Backend se realice usando la IP privada del backend o reglas de Security Group.
* El acceso administrativo a EC2 se realice mediante SSH restringido o AWS Systems Manager Session Manager.

## Estado actual del proyecto

Actualmente el repositorio contiene:

* Código base del frontend.
* Dockerfile para servir el frontend con Nginx.
* Configuración personalizada de Nginx.
* Documentación técnica inicial.