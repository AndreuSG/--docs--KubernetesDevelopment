# Pods en Kubernetes

Los **Pods** son la unidad mínima de ejecución dentro de Kubernetes. Entender qué son, cómo funcionan y cómo se diferencian de los contenedores es esencial antes de avanzar hacia objetos más complejos como Deployments, ReplicaSets o DaemonSets.

## ¿Qué es un Pod?

Un **Pod** es la unidad más pequeña que Kubernetes puede desplegar. Representa una **envoltura lógica** que contiene uno o varios contenedores que deben ejecutarse juntos.

Un Pod incluye:

* Contenedor(es)
* Volúmenes compartidos
* Red compartida
* Configuración (variables de entorno, puertos, probes…)

**Idea clave:**

> Un Pod es un "mini entorno" donde uno o varios contenedores trabajan como un único servicio.

## ¿En qué se diferencia un Pod de un contenedor?

| Característica | Contenedor                                     | Pod                                                     |
| -------------- | ---------------------------------------------- | ------------------------------------------------------- |
| ¿Qué es?       | Una instancia aislada de una app               | Un envoltorio que agrupa uno o varios contenedores      |
| Red            | Tiene su propia red                            | Todos los contenedores del Pod comparten IP y puertos   |
| Almacenamiento | Propio de cada contenedor                      | Volúmenes compartidos entre contenedores                |
| Gestión        | La hace Docker/Containerd                      | Kubernetes gestiona Pods (no contenedores individuales) |
| Fallos         | Si un contenedor muere, el runtime lo reinicia | Kubernetes recrea el Pod completo si es necesario       |

**Regla de oro:**

> Un Pod suele contener *un solo contenedor*. Varios contenedores en un Pod se usan solo en casos muy especiales, por ejemplo si necesitan compartir red/volúmenes de forma íntima.

## Crear un Pod con un manifiesto YAML (forma declarativa — recomendada)

Esta es la forma estándar en Kubernetes.

Es reproducible, versionable y la que se usa SIEMPRE en entornos reales.

### Archivo `nginx-pod.yaml`

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
```

Aplicarlo:

```bash
kubectl apply -f nginx-pod.yaml
```

Verificar:

```bash
kubectl get pods
```

Detalles del Pod:

```bash
kubectl describe pod nginx-pod
```

## Crear un Pod con kubectl run (no recomendado)

Sirve para levantar un Pod rápido sin manifest YAML.

Ideal para pruebas y experimentos, pero NO se usa en producción.

```bash
kubectl run nginx-pod --image=nginx:latest --port=80
```

Desventajas:

* No es reproducible ni versionable.
* No permite configuraciones avanzadas.
* No es un archivo declarativo.

## Operaciones básicas con Pods

### Listar Pods

```bash
kubectl get pods
```

### Ver Pods con más información

```bash
kubectl get pods -o wide
```

### Ver logs del contenedor principal

```bash
kubectl logs nginx-pod
```

### Entrar dentro del contenedor (shell)

```bash
kubectl exec -it nginx-pod -- bash
```

### Eliminar un Pod

```bash
kubectl delete pod nginx-pod
```

## Estados comunes de un Pod

| Estado               | Significado                                                   |
| -------------------- | ------------------------------------------------------------- |
| **Pending**          | Se ha enviado el Pod pero aún no asignado a un nodo           |
| **Running**          | El contenedor está ejecutándose correctamente                 |
| **Succeeded**        | El contenedor terminó correctamente (Jobs)                    |
| **Failed**           | El contenedor terminó con error                               |
| **CrashLoopBackOff** | El contenedor falla repetidamente al arrancar                 |
| **ImagePullBackOff** | No puede descargar la imagen (error en credenciales/registro) |
| **Unknown**          | El estado del Pod no se puede determinar (problema de red)    |
| **Completed**        | Se ejecutó y finalizó correctamente (Jobs)                    |

## ¿Qué pasa cuando un Pod muere?

Un Pod **no es permanente**. Puede morir por:

* Fallos internos
* Reschedule del nodo
* Falta de recursos
* Problemas de red

Cuando eso pasa, Kubernetes:

* No recrea automáticamente el Pod **a menos que exista un controlador** (Deployment, ReplicaSet, Job…)

Por eso **nunca se despliegan aplicaciones en Pods sueltos en producción**.

Los Pods reales se gestionan a través de Deployments.

## Buenas prácticas con Pods

* Usar **un contenedor por Pod** salvo casos de sidecars.
* Nunca desplegar Pods "a mano" en producción.
* Usar health checks (liveness/readiness probes).
* Exponer solo los puertos necesarios.
* Mantener las imágenes pequeñas.

Los Pods son la base de todo Kubernetes. Una vez entiendes su lógica, será mucho más fácil comprender Deployments, Services, Ingress, StatefulSets y toda la capa superior del ecosistema.