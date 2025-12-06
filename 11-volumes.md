# 11. Volúmenes en Kubernetes

Los **volúmenes** permiten a los Pods almacenar datos más allá del ciclo de vida de un contenedor. También permiten **inyectar ficheros de configuración**, certificados, directorios completos o datos persistentes.

Esta sección explica los volúmenes nativos de Kubernetes y, sobre todo, cómo montar **ConfigMaps** y **Secrets** como archivos dentro de un Pod.

> Más adelante veremos PV/PVC y almacenamiento persistente. Aquí nos centraremos en los volúmenes que dependen únicamente del Pod.

---

## 11.1. ¿Qué es un volumen en Kubernetes?

Un volumen es una **carpeta** accesible desde uno o varios contenedores del Pod. A diferencia del filesystem interno del contenedor:

* **no desaparece** cuando el contenedor se reinicia
* puede ser **compartido** entre contenedores
* permite montar **archivos externos** (configuraciones, certificados…)

---

## 11.2. Tipos de volúmenes nativos (sin PV/PVC)

### ✔️ `emptyDir`

Un directorio vacío que se crea cuando el Pod arranca.

* Se borra cuando el Pod muere.
* Útil para almacenamiento temporal.

### ✔️ `hostPath`

Monta un directorio del **nodo físico** dentro del Pod.

* Muy potente, pero riesgoso.
* No recomendado excepto para debugging.

### ✔️ `configMap`

Monta claves o archivos almacenados en un ConfigMap.

* Ideal para **ficheros de configuración**.

### ✔️ `secret`

Monta valores sensibles como archivos.

* Ideal para **certificados, claves privadas y credenciales**.

### ✔️ `downwardAPI`

Permite exponer información del Pod al contenedor.

### ✔️ `projected`

Permite combinar varios volúmenes (configMap + secret + downwardAPI) en un solo punto de montaje.

---

## 11.3. Estructura básica de un volumen en un Pod

Un Pod con volúmenes siempre tiene dos partes:

### 1) Definir el volumen

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

---

# 11.4. Montar volúmenes desde ConfigMaps

## ✔️ Ejemplo de ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  app.conf: |
    port=8080
    log=debug
```

## ✔️ Montar ConfigMap como archivos en un Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/app
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

### 📌 Resultado dentro del contenedor

```
/etc/app/APP_MODE
/etc/app/app.conf
```

Los valores del ConfigMap se convierten en ficheros.

---

## ✔️ Montar una sola clave del ConfigMap

```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
        - key: "app.conf"
          path: "configuracion.conf"
```

Esto creará exactamente:

```
/etc/app/configuracion.conf
```

---

# 11.5. Montar volúmenes desde Secrets

## ✔️ Ejemplo de Secret

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

## ✔️ Montar Secret como archivos en un Pod

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

---

## ✔️ Montar una clave concreta del Secret

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

---

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

---

## 11.7. Ciclo de vida de los volúmenes

| Tipo        | Se borra cuando…                                 |
| ----------- | ------------------------------------------------ |
| `emptyDir`  | Muere el Pod                                     |
| `configMap` | Muere el Pod (los datos del ConfigMap persisten) |
| `secret`    | Muere el Pod (el Secret persiste)                |
| `hostPath`  | Nunca (está en el nodo)                          |
| `projected` | Muere el Pod                                     |

Volúmenes persistentes como **PV/PVC** se verán más adelante.

---

## 11.8. Casos de uso reales

* Pasar `/etc/app/config.json` desde un ConfigMap
* Montar certificados TLS desde un Secret
* Compartir datos temporales con sidecars usando `emptyDir`
* Inyectar ficheros de configuración complejos como directorios
* Separar configuración por entornos

---

## 11.9. Buenas prácticas

* Evitar `hostPath` en producción
* No montar Secrets en rutas visibles en logs
* Para ficheros grandes, usar PV/PVC
* Rotar Secrets y recrear Pods
* Usar volúmenes proyectados para unificar config
* No meter datos sensibles en ConfigMaps

---

Este capítulo cubre todo lo necesario para montar ConfigMaps y Secrets como archivos dentro de un Pod.

El siguiente paso será aprender sobre **PV/PVC** o pasar directamente a **Probes**, según el flujo que quieras seguir.
