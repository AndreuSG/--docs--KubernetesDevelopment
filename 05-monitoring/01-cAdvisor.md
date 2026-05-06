## ¿Cómo monitoriza Kubernetes los nodos y Pods?

Kubernetes no tiene un sistema de métricas propio integrado en su núcleo. En su lugar, delega la recogida de métricas en componentes que corren en cada nodo. El flujo es el siguiente:

```
contenedores
     ↓
  cAdvisor  (integrado en el kubelet)
     ↓
   kubelet
     ↓
Metrics Server / herramientas externas
```

## El kubelet como punto central de monitorización

El **kubelet** no solo gestiona el ciclo de vida de los Pods: también actúa como **agente de monitorización** del nodo. Para ello expone una API de métricas en el puerto `10250` que otros componentes pueden consultar.

El kubelet recoge métricas de dos fuentes:

- **Del propio nodo:** uso de CPU, memoria, disco, red.
- **De los contenedores:** gracias a **cAdvisor**, que está embebido dentro del propio proceso del kubelet.

## cAdvisor (Container Advisor)

**cAdvisor** es una herramienta open source de Google integrada directamente en el kubelet. Se encarga de inspeccionar cada contenedor que corre en el nodo y recoger métricas en tiempo real. cAdvisor expone estas métricas a través del kubelet, que las hace disponibles en el endpoint `/metrics/cadvisor`.

```bash
# Ver las métricas expuestas por el kubelet (desde dentro del nodo)
curl https://localhost:10250/metrics/cadvisor --insecure
```

## Metrics Server

El **Metrics Server** es el componente que agrega las métricas de todos los kubelets del clúster y las expone a través de la **Metrics API** de Kubernetes. Es necesario para que funcionen comandos como `kubectl top`.

```
kubelet (nodo 1) ──┐
kubelet (nodo 2) ──┼──→  Metrics Server  →  Metrics API  →  kubectl top
kubelet (nodo N) ──┘
```

> El Metrics Server almacena las métricas **solo en memoria**, no persiste datos históricos. Para métricas históricas se necesitan soluciones como Prometheus + Grafana o Zabbix + Grafana.

### Instalación del Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Comandos útiles

```bash
# Ver uso de CPU y memoria de los nodos
kubectl top nodes

# Ver uso de CPU y memoria de los Pods
kubectl top pods

# Ver métricas de Pods en un namespace concreto
kubectl top pods -n kube-system
```

### Ejemplo práctico

**Identifica el Pod que consume menos CPU en el namespace `default`:**

```bash
kubectl top pod -n default
```

```
NAME       CPU(cores)   MEMORY(bytes)
elephant   13m          30Mi
lion       1m           16Mi
rabbit     91m          250Mi
```

En este caso el Pod que menos CPU consume es **`lion`** con `1m` (1 milicpu).