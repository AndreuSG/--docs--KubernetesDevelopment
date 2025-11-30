# ConfigMaps en Kubernetes

Los **ConfigMaps** permiten almacenar y gestionar configuración externa para tus aplicaciones. Son fundamentales para seguir el principio de **“configuración fuera del contenedor”**, evitando valores hardcodeados y permitiendo que la misma imagen funcione en múltiples entornos (dev, pre, prod).

Este recurso es uno de los más utilizados en Kubernetes, ya que te permite pasar:

* Variables de entorno
* Ficheros de configuración
* Bloques de texto
* Claves/valores simples

## ¿Qué es un ConfigMap?

Un **ConfigMap** es un objeto de Kubernetes diseñado para almacenar datos de configuración en formato:

* clave/valor
* ficheros completos
* directorios simulados

Es información **no sensible**.

(Si es sensible → usar Secrets, lo veremos en el siguiente capítulo.)

## Formas de crear un ConfigMap

Kubernetes permite tres formas principales de crear ConfigMaps.

### 1) Desde línea de comando (clave/valor)

```bash
kubectl create configmap app-config \
  --from-literal=APP_MODE=production \
  --from-literal=LOG_LEVEL=debug
```

### 2) Desde un fichero

```bash
kubectl create configmap app-config --from-file=config.json
```

Esto creará una clave llamada `config.json` cuyo valor es el contenido del archivo.

### 3) Desde varios ficheros

```bash
kubectl create configmap app-config --from-file=./config-dir/
```

Cada archivo dentro del directorio se convierte en una clave dentro del ConfigMap.

### 4) Desde un manifiesto YAML (recomendado)

`configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: desarrollo
data:
  APP_MODE: "production"
  LOG_LEVEL: "debug"
  WELCOME_MSG: "Hola desde ConfigMap"
```

Aplicar:

```bash
kubectl apply -f configmap.yaml
```

## Ver ConfigMaps

```bash
kubectl get configmaps
kubectl get cm
kubectl describe configmap app-config
kubectl describe cm app-config
```

## ¿Cómo se usa un ConfigMap dentro de un Pod?

Hay **dos formas** de consumir un ConfigMap desde un manifest.

### 1) Como variables de entorno una por una

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_MODE
          key: LOG_LEVEL
          key: WELCOME_MSG
```

### 2) Como fichero completo (envFrom)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
```

## Actualizar ConfigMaps

⚠️ Importante: **los Pods no se reinician automáticamente** cuando cambia un ConfigMap.

Para que tomen los cambios, debes:

* recrear el Pod
* o hacer rollout del Deployment

Ejemplo:

```bash
kubectl rollout restart deployment nginx-deployment
```

## Cosas que suelen romperse (troubleshooting)

### ❌ Error: ConfigMap no existe

Si creaste el resource en otro namespace, debes usar `-n` o incluir `namespace:` en el YAML.

### ❌ Cambié un ConfigMap y el Pod no ve los nuevos valores

Correcto: **los Pods no se actualizan solos**.
Debes reiniciar o hacer rollout.

## Buenas prácticas

* Usar ConfigMaps solo para datos NO sensibles
* Mantener cada ConfigMap con un propósito claro
* Crear ConfigMaps por manifiesto YAML, no por comandos temporales