# ReplicaSets en Kubernetes

Los **ReplicaSets** son un recurso fundamental en Kubernetes, pero rara vez se gestionan directamente, normalmente se manejan a través de Deployments. Su función es asegurar que siempre exista el **número deseado de Pods** ejecutándose.

Este documento explica qué son, cómo funcionan y por qué se utilizan principalmente como parte interna de un Deployment.

---

## ¿Qué es un ReplicaSet?

Un **ReplicaSet** es un controlador cuyo objetivo es mantener siempre una cantidad exacta de Pods en ejecución.

Su misión principal es:

* Crear Pods cuando faltan
* Eliminar Pods sobrantes
* Reemplazar Pods caídos

En resumen:

> **Un ReplicaSet garantiza que existan “N” Pods sanos en todo momento.**

---

## ¿Para qué sirve un ReplicaSet?

Los ReplicaSets se encargan de:

* Asegurar alta disponibilidad mínima
* Mantener un número constante de réplicas
* Auto-recuperar Pods cuando fallan

Pero ojo:

> Aunque un ReplicaSet puede usarse solo, **no es recomendable**. Los Deployments lo gestionan automáticamente.

---

## ¿Por qué normalmente no usamos ReplicaSets directamente?

Porque los **Deployments** son una capa superior que añade funciones esenciales:

* Rollouts progresivos (actualizaciones sin downtime) llamados *rolling updates*
* Rollbacks automáticos en caso de fallos
* Estrategias de despliegue
* Historial de versiones
* Gestión segura de nueva imagen

Un ReplicaSet *no puede*:

❌ Hacer rollouts
❌ Hacer rollbacks
❌ Gestionar versiones
❌ Actualizar contenedores sin borrar Pods manualmente

Por eso, la práctica profesional es:

> **Utilizar Deployments, no ReplicaSets directamente.**

---

## ¿Cómo funciona internamente un ReplicaSet?

Un ReplicaSet funciona comparando:

* **Estado deseado:** cuántos Pods quieres (`spec.replicas`)
* **Estado real:** cuántos Pods existen realmente en el clúster

Si el estado real es menor:

* Crea Pods

Si el estado real es mayor:

* Elimina Pods

Si un Pod muere:

* Crea uno nuevo

---

## Ejemplo básico de ReplicaSet

Archivo `nginx-rs.yaml`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
          image: nginx:latest
```

Aplicar:

```bash
kubectl apply -f nginx-rs.yaml
```

Ver ReplicaSets:

```bash
kubectl get rs

## Resultado esperado:
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   3         3         3       32s
```

Ver Pods:

```bash
kubectl get pods -l app=nginx

## Resultado esperado:
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-49lwv   1/1     Running   0          14s
nginx-rs-4js9t   1/1     Running   0          14s
nginx-rs-l5rzd   1/1     Running   0          14s
```

---

## Relación entre Deployment, ReplicaSet y Pods
Cuando creas un Deployment, Kubernetes hace automáticamente:

1. Crear un ReplicaSet
2. Ese ReplicaSet crea los Pods
3. Si actualizas la imagen, el Deployment:

   * Crea un *nuevo* ReplicaSet
   * Reduce el anterior (rolling update)

Cadena completa:

> **Deployment → ReplicaSet → Pods**

El Deployment es el que manda. El ReplicaSet solo ejecuta.

---

## Comandos útiles

### Ver ReplicaSets

```bash
kubectl get rs
```

### Describir un ReplicaSet

```bash
kubectl describe rs nombre-del-rs
```

### Escalar manualmente un ReplicaSet

(⚠️ No recomendado si existe un Deployment asociado)

```bash
kubectl scale rs nombre --replicas=N
```

### Eliminar un ReplicaSet

```bash
kubectl delete rs nombre-del-rs
```

---

## Buenas prácticas

* No crear ReplicaSets directamente (usar Deployments)
* Solo manipularlos para fines de aprendizaje o debugging
* Nunca escalar un ReplicaSet que depende de un Deployment
* Usar labels claras y selectores bien definidos

---

Los ReplicaSets son el mecanismo que garantiza que tus Pods se mantengan vivos y en número suficiente. En producción, el siguiente paso natural es utilizar **Deployments**, que amplían las capacidades de los ReplicaSets con actualizaciones controladas, rollbacks y estrategias avanzadas.
