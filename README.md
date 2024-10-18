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

1. Crear un ficheiro `docker-compose.yml` básico:

   ```yaml
   version: '3'
   services:
     web:
       image: nginx
       ports:
         - "8080:80"
     db:
       image: mysql
       environment:
         MYSQL_ROOT_PASSWORD: example
   ```

2. Iniciar os contenedores con Docker Compose:
   

   ```bash
   docker-compose up
   ```

3. Ver o estado dos servizos:

   ```bash
   docker-compose ps
   ```

4. Apagar os servizos:

   ```bash
   docker-compose down
   ```

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

