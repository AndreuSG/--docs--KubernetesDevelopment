# Namespaces en Kubernetes

Los **namespaces** son una forma de organizar, aislar y gestionar recursos dentro de un clúster de Kubernetes. 

En otras palabras, permiten dividir un clúster en entornos lógicos separados

Piensa en él como si fueran "carpetas" donde guardas tus recursos de Kubernetes.

Cada namespace contiene sus propios Pods, Services, Deployments, ConfigMaps, Secrets, etc.

## ¿Para qué sirven los namespaces?

Los namespaces permiten:

* **Organizar recursos** por proyecto, equipo o entorno.
* **Evitar colisiones de nombres** (dos Pods pueden llamarse igual si están en diferentes namespaces).
* **Aplicar límites de recursos** por equipo o entorno (ResourceQuota y LimitRange).
* **Controlar accesos** mediante RBAC.
* **Separar entornos** dentro del mismo clúster (dev, pre, prod).

Ejemplo real:

> Un mismo clúster puede tener los namespaces: `empresa1`, `empresa2`, `empresa3` cada uno con sus propias apps.

## Namespaces por defecto
Kubernetes crea algunos namespaces automáticamente:

| Namespace           | Uso                                                      |
| ------------------- | -------------------------------------------------------- |
| **default**         | Donde van los recursos si no especificas ninguno.        |
| **kube-system**     | Componentes internos del clúster (CoreDNS, kube-proxy…). |
| **kube-public**     | Acceso público (rara vez usado).                         |
| **kube-node-lease** | Heartbeats de los nodos para detectar disponibilidad.    |



## Crear un namespace

**Si estás usando minkube para entorno local, recuerda poner siempre delante `minikube kubectl --`**

### Forma 1: comando directo

```bash
kubectl create namespace desarrollo
```

### Forma 2: manifiesto YAML

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: desarrollo
```

Aplicar el manifiesto:

```bash
kubectl apply -f namespace-desarrollo.yaml
```

## Ver y gestionar namespaces

### Listar todos los namespaces

```bash
kubectl get namespaces
```

### Ver recursos dentro de un namespace

```bash
kubectl get pods -n desarrollo
```

### Listar *todos* los recursos de un namespace

```bash
kubectl get all -n desarrollo
```

### Eliminar un namespace

```bash
kubectl delete namespace desarrollo
```

⚠️ **Ojo:**
Eliminar un namespace elimina *todos* sus recursos. Pods, Services, Deployments, etc.

## Cambiar el namespace por defecto

Para trabajar más cómodamente sin tener que usar `-n` todo el rato:

```bash
kubectl config set-context --current --namespace=desarrollo
```

Ver el namespace activo:

```bash
kubectl config view --minify | grep namespace
```

Si queremos volver al namespace `default`:

```bash
kubectl config set-context --current --namespace=default
```

El namespace `default` es el que se usa si no especificas ninguno. Pero cuidado, el namespace `default` no es ningun entorno especial, es solo un namespace más.

## Buenas prácticas con namespaces

* Un namespace para cada entorno: `dev`, `pre`, `prod`.
* O crear un namespace para cada proyecto u organización.
* Usar ResourceQuota para limitar recursos por equipo.
* Usar RBAC para separar accesos entre equipos.
* No desplegar apps en `default`.
* Mantener `kube-system` solo para componentes internos.

Los namespaces son el punto de partida para estructurar un clúster real. Ahora que conoces cómo aislar recursos, el siguiente paso natural es profundizar en los Pods y, después, avanzar hacia Deployments, Services e Ingress.
