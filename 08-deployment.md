# Deployments en Kubernetes

Los **Deployments** son el recurso más importante y utilizado en Kubernetes para desplegar aplicaciones de manera declarativa, segura y escalable. Representan la forma profesional de gestionar Pods en producción y proporcionan funcionalidades críticas como actualizaciones progresivas, rollbacks y control total sobre el ciclo de vida de la aplicación.



## ¿Qué es un Deployment?

Un **Deployment** es un controlador de Kubernetes que se encarga de:

* Crear y gestionar **ReplicaSets**
* Definir cuántas réplicas de Pods queremos
* Controlar actualizaciones (rolling updates)
* Volver a versiones anteriores (rollbacks)
* Mantener el estado deseado

En otras palabras:

> **El Deployment es el cerebro que gestiona la aplicación; los ReplicaSets y Pods son sus piezas.**



## Relación Deployment → ReplicaSet → Pod

Cuando creas un Deployment, Kubernetes hace automáticamente:

1. Crear un **ReplicaSet**
2. Ese ReplicaSet crea los **Pods**
3. Si actualizas la app (por ejemplo, cambias la imagen), el Deployment:

   * Crea un **nuevo ReplicaSet**
   * Escala el nuevo hacia arriba
   * Escala el antiguo hacia abajo

Cadena completa:

> **Deployment → ReplicaSet → Pods**

Por eso nunca trabajamos con ReplicaSets directamente: el Deployment los gestiona por nosotros.



## Crear un Deployment

### Manifiesto básico: `nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
```

Aplicar:

```bash
kubectl apply -f nginx-deployment.yaml
```

Ver Deployments:

```bash
kubectl get deployments

## Resultado esperado:
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           41s
```

Ver ReplicaSets generados:

```bash
kubectl get rs

## Resultado esperado:
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-77bf8679f9   3         3         3       73s
```

Ver Pods creados:

```bash
kubectl get pods

## Resultado esperado:
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-77bf8679f9-6smql   1/1     Running   0          101s
nginx-deployment-77bf8679f9-7c8b9   1/1     Running   0          101s
nginx-deployment-77bf8679f9-snqtg   1/1     Running   0          101s
```



## Actualizaciones (Rollouts)

El Deployment permite actualizar la aplicación sin downtime.

Ejemplo: cambiar la imagen

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.26
```

Ver el progreso del rollout:

```bash
kubectl rollout status deployment/nginx-deployment
```

Historial de versiones:

```bash
kubectl rollout history deployment/nginx-deployment
```

Ver pods mientras se actualiza:

```bash
kubectl get pods

## Resultado esperado:
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-57489d7c8d-zl6sz   0/1     ContainerCreating   0          3s
nginx-deployment-77bf8679f9-6smql   1/1     Running             0          2m34s
nginx-deployment-77bf8679f9-7c8b9   1/1     Running             0          2m34s
nginx-deployment-77bf8679f9-snqtg   1/1     Running             0          2m34s
```

Lo que sucede es que Kubernetes crea un **nuevo ReplicaSet** con la nueva imagen y va escalándolo mientras reduce el antiguo, asegurando que siempre haya Pods disponibles.

Tambíen puedes ver los replicasets:

```bash
kubectl get rs

## Resultado esperado:
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-57489d7c8d   3         3         3       45s
nginx-deployment-77bf8679f9   0         0         0       5m
```

Si os fijaís, el antiguo ReplicaSet tiene el id `77bf8679f9` y el nuevo `57489d7c8d`. Es exactamente el id que tiene el pod entre medio.

Sintaxis del nombre del pod:

```bash
<deployment-name>-<replicaset-hash>-<pod-unique-id>
nginx-deployment-57489d7c8d-zl6sz
```



## Rollbacks (volver atrás)

Si una actualización falla, puedes volver a la versión anterior:

```bash
kubectl rollout undo deployment/nginx-deployment
```

O a una versión específica:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```



## Estrategias de despliegue

Los Deployments soportan dos estrategias:

### **RollingUpdate (por defecto)**

Actualiza poco a poco.

* Crea nuevos Pods
* Va eliminando los antiguos
* No hay downtime

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

### **Recreate**

Borra todos los Pods antiguos antes de crear los nuevos.

* Tiempo de inactividad
* Solo recomendado en casos donde no puede haber 2 versiones a la vez

```yaml
strategy:
  type: Recreate
```



## Escalado de un Deployment

### Escalado manual

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

### Escalado automático (HPA, lo veremos más adelante)

```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=70 --min=1 --max=10
```



## Describir un Deployment

```bash
kubectl describe deployment nginx-deployment
```

Secciones importantes:

* ReplicaSets asociados
* Estado del rollout
* Imágenes actuales
* Eventos
* Especificación del Pod template



## Actualización mediante editar YAML

```bash
kubectl edit deployment nginx-deployment
```

Esto abrirá el YAML en tu editor.
Kubernetes reconciliará automáticamente los cambios.



## Eliminar un Deployment

```bash
kubectl delete deployment nginx-deployment
```

Esto elimina también los ReplicaSets y los Pods asociados.



## Buenas prácticas

* No usar `latest` como etiqueta de imagen
* Usar probes (readiness y liveness) que los veremos más adelante
* Definir requests y limits de CPU/RAM
* Versionar imágenes correctamente
* Mantener labels coherentes
* No escalar ReplicaSets directamente
* No crear Pods sueltos en producción

---

Los Deployments son el método estándar para ejecutar aplicaciones en Kubernetes.
Controlan Pods, actualizaciones, historial y escalado, permitiendo un despliegue seguro, estable y profesional.
