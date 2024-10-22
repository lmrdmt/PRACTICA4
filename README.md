# **Guía de Comandos Docker e Docker Compose**

## **1. Crear unha Rede en Docker**

Para crear unha rede en Docker:

```bash
docker network create mi_red
```

Comprobar que se creou con éxito:

```bash
docker network inspect mi_red
```

---

## **2. Crear Dous Contenedores Conectados a esa Rede**

Crear dous contenedores e conectalos á rede `mi_red`:

```bash
docker run -dit --name contenedor1 --network mi_red busybox
docker run -dit --name contenedor2 --network mi_red busybox
```

---

## **3. Comprobar que os Contenedores están na Rede**

Para ver que os contenedores están conectados á rede `mi_red`:

```bash
docker network inspect mi_red
```

---

## **4. Comprobar que os Contenedores poden verse entre eles**

1. Accede ao shell do contedor:

   ```bash
   docker exec -it contenedor1 sh
   ```

2. Fai ping ao outro contenedor:

   ```bash
   ping contenedor2
   ```
3. Se dentro do contenedor ping command not found.
   ```bash
   apt-get install -y iputils-ping
   ```
Acordarse de lanzar antes un apt update e un apt upgrade.

---

## **5. Listar os Contenedores Conectados á Rede**

Ver todos os contenedores conectados a unha rede específica:

```bash
docker network inspect mi_red
```

---

## **6. Listar as Propiedades da Rede**

Para ver as propiedades dunha rede:

```bash
docker network inspect mi_red
```

---

## **7. Crear Outra Rede**

Crear unha segunda rede:

```bash
docker network create mi_red2
```

---

## **8. Lanza Dous Novos Contenedores Conectados á Nova Rede**

Crear dous novos contenedores conectados a `mi_red2`:

```bash
docker run -dit --name contenedor3 --network mi_red2 busybox
docker run -dit --name contenedor4 --network mi_red2 busybox
```

---

## **9. Comprobar as Conexións Entre os 4 Contenedores**

1. Desde `contenedor1`, intenta facer ping a `contenedor3`:

   ```bash
   docker exec -it contenedor1 sh
   ping contenedor3  # Non debería funcionar porque están en redes distintas
   ```

2. Desde `contenedor1`, proba facer ping a `contenedor2`:

   ```bash
   ping contenedor2  # Isto debería funcionar porque están na mesma rede
   ```

---

# **Docker Compose**

## **10. Guía de Iniciación de Docker Compose**
Deberemos asegurarnos de tener:
   - Instalada la última versión de Docker Compose
   - Un básico entendimiento de los conceptos de docker y como trabaja.

1.Crear un directorio para el proyecto:
```bash
mkdir Práctica4
cd Práctica4
```
2.Crear un archivo llamado app.py.Dentro del mismo pegamos el siguiente código:
```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
En el ejemplo redis is el hostname y el puerto por defecto es el 6379.
**función get_hit_count**. Este bucle de reintento básico intenta la petición varias veces si el servicio Redis no está disponible. Esto es útil en el inicio mientras la aplicación está en línea, pero también hace que la aplicación sea más resistente si el servicio Redis necesita ser reiniciado en cualquier momento durante la vida de la aplicación. En un cluster, esto también ayuda a manejar caídas momentáneas de conexión entre nodos.

3.Creamos otro fichero llamado requirements.txt en nuestro directorio y pegamos:
```
flask
redis
```
4.Creamos un Dockerfile y pegamos dentro:
```# syntax=docker/dockerfile:1
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
```
Deberemos tener en cuenta que este archivo no debe tener ningún tipo de extensión.

   Comprexión del archivo anterior:
   - Construye una imagen usando Python3.10 image.
   - Elige el directorio de trabajo - /code
   - instala gcc y otras dependencias
   - Copia el documento _requirements.txt_ e instala las python dependencias
   - Añade metadatos a la imagen que describen que el contenedor está escuchando por el puerto 5000.
   - Copia el actual directorio . en el proyecto al directorio de trabajo . en la imagen
   - Establece el comando por defecto para el contenedor a _flask run_ --debug

5. Definimos los servicios en un Compose file.

---

## **11. Crear un Ficheiro de Configuración con Tres Contenedores**

Exemplo de ficheiro `docker-compose.yml` con tres contenedores conectados a unha rede:

```yaml
version: '3'
services:
  app1:
    image: busybox
    command: sleep 1000
    networks:
      - mi_red_compose

  app2:
    image: busybox
    command: sleep 1000
    networks:
      - mi_red_compose

  app3:
    image: busybox
    command: sleep 1000
    networks:
      - mi_red_compose

networks:
  mi_red_compose:
    driver: bridge
```

---

## **12. Parámetros e Configuracións Avanzadas de Docker Compose**
### 0. **NOTAS**
Para lanzar un Docker compose, crearemos unha carpeta propia e dentro un archivo ***.yml***.
Debemos ter en conta que para lanzar un compose teremos que estar dentro da propia carpeta.

### 1. **`depends_on`**

Define a orde de inicio dos servizos. Exemplo:

```yaml
services:
  app:
    image: myapp
    depends_on:
      - db
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
```

### 2. **`volumes`**

Montar volumes para persistir datos. Exemplo:

```yaml
services:
  web:
    image: nginx
    volumes:
      - ./html:/usr/share/nginx/html
```

### 3. **`environment`**

Definir variables de contorno para o contenedor. Exemplo:

```yaml
services:
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: mydb
```

### 4. **`restart`**

Especificar a política de reinicio dos contenedores. Exemplo:

```yaml
services:
  app:
    image: busybox
    restart: always
```

---

