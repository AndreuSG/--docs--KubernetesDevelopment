# Volúmenes en Kubernetes

Los **volúmenes** permiten a los Pods almacenar datos más allá del ciclo de vida de un contenedor. También permiten **inyectar ficheros de configuración**, certificados, directorios completos o datos persistentes.

Esta sección explica los volúmenes nativos de Kubernetes y, sobre todo, cómo montar **ConfigMaps** y **Secrets** como archivos dentro de un Pod.

> Más adelante veremos PV/PVC y almacenamiento persistente. Aquí nos centraremos en los volúmenes que dependen únicamente del Pod.

## ¿Qué es un volumen en Kubernetes?

Un volumen es una **carpeta** accesible desde uno o varios contenedores del Pod. A diferencia del filesystem interno del contenedor:

* **no desaparece** cuando el contenedor se reinicia
* puede ser **compartido** entre contenedores
* permite montar **archivos externos** (configuraciones, certificados…)

## Tipos de volúmenes nativos (sin PV/PVC)

### `hostPath`

Monta un directorio del **nodo físico** dentro del Pod.

* Muy potente, pero riesgoso.
* No recomendado excepto para debugging.

### `configMap`

Monta claves o archivos almacenados en un ConfigMap.

* Ideal para **ficheros de configuración**.

### `secret`

Monta valores sensibles como archivos.

* Ideal para **certificados, claves privadas y credenciales**.

### `downwardAPI`

Permite exponer información del Pod al contenedor.

### `projected`

Permite combinar varios volúmenes (configMap + secret + downwardAPI) en un solo punto de montaje.

## Estructura básica de un volumen en un Pod

Un Pod con volúmenes siempre tiene dos partes:

### 1) Suponemos que tenemos un ConfigMap llamado `app-config` y queremos definirlo.

```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### 2) Montarlo en un contenedor

```yaml
containers:
  - name: app
    image: nginx
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
```

El directorio `/etc/config` contendrá los archivos del ConfigMap.

# Montar volúmenes desde ConfigMaps

## Ejemplo práctico: Inyectar configuración de Nginx

Un caso muy común es **inyectar un fichero de configuración** dentro de un Pod. En lugar de crear una imagen personalizada con la configuración, podemos usar un **ConfigMap** y montarlo como volumen.

### 1. Crear el fichero de configuración `nginx.conf`

Primero, tenemos el fichero de configuración local:

```nginx
# nginx.conf
server {
    listen 80;
    server_name miapp.local;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /api {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 2. Crear un ConfigMap desde el fichero

Podemos crear el ConfigMap directamente desde el archivo:

```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

O definirlo en YAML:

```bash
kubectl create configmap nginx-config --from-file=nginx.conf > nginx-config.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
        listen 80;
        server_name miapp.local;

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }

        location /api {
            proxy_pass http://backend:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
```

### 3. Montar el ConfigMap como volumen en el Pod

Ahora montamos el fichero `nginx.conf` en la ruta donde Nginx lo espera:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: nginx-config-volume
          mountPath: /etc/nginx/conf.d/nginx.conf
          subPath: nginx.conf
          readOnly: true
  volumes:
    - name: nginx-config-volume
      configMap:
        name: nginx-config
```

### 📌 Puntos clave

* **`subPath: nginx.conf`**: Monta solo ese fichero, no todo el directorio.
* **`mountPath: /etc/nginx/conf.d/nginx.conf`**: Sobrescribe el fichero de configuración.
* **`readOnly: true`**: El contenedor no puede modificar la configuración.

### 4. Verificar que funciona

```bash
kubectl exec nginx-pod -- cat /etc/nginx/conf.d/nginx.conf
```

Deberías ver exactamente el contenido del ConfigMap.

### 5. Actualizar configuración sin recrear la imagen

Si cambias el ConfigMap:

```bash
kubectl edit configmap nginx-config
```

El Pod **puede tardar hasta 60 segundos** en ver el cambio (según el kubelet sync period). Para forzar el cambio inmediatamente:

```bash
kubectl rollout restart deployment nginx-deployment
```

# Montar volúmenes desde Secrets

## Ejemplo de Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

## Montar Secret como archivos en un Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/creds
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: db-secret
```

### 📌 Resultado dentro del contenedor

```
/etc/creds/username
/etc/creds/password
```

Los archivos contienen **los valores decodificados**.

## Montar una clave concreta del Secret

```yaml
volumes:
  - name: certs
    secret:
      secretName: tls-secret
      items:
        - key: "tls.crt"
          path: "server.crt"
```

Esto permite elegir qué archivo del Secret quieres montar.

## 11.6. Volúmenes proyectados (mezcla de ConfigMap + Secret)

```yaml
volumes:
  - name: projected
    projected:
      sources:
        - configMap:
            name: app-config
        - secret:
            name: db-secret
```

Esto crea un único directorio con **todo combinado**.

## 11.7. Ciclo de vida de los volúmenes

| Tipo        | Se borra cuando…                                 |
| ----------- | ------------------------------------------------ |
| `emptyDir`  | Muere el Pod                                     |
| `configMap` | Muere el Pod (los datos del ConfigMap persisten) |
| `secret`    | Muere el Pod (el Secret persiste)                |
| `hostPath`  | Nunca (está en el nodo)                          |
| `projected` | Muere el Pod                                     |

Volúmenes persistentes como **PV/PVC** se verán más adelante.

## 11.8. Casos de uso reales

* Pasar `/etc/app/config.json` desde un ConfigMap
* Montar certificados TLS desde un Secret
* Compartir datos temporales con sidecars usando `emptyDir`
* Inyectar ficheros de configuración complejos como directorios
* Separar configuración por entornos

## 11.9. Buenas prácticas

* Evitar `hostPath` en producción
* No montar Secrets en rutas visibles en logs
* Para ficheros grandes, usar PV/PVC
* Rotar Secrets y recrear Pods
* Usar volúmenes proyectados para unificar config
* No meter datos sensibles en ConfigMaps