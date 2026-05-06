# Resource Requirements

Kubernetes permite controlar cuántos recursos (CPU y memoria) consume cada contenedor. Esto es fundamental para garantizar la estabilidad del clúster, evitar que un Pod monopolice recursos y permitir que el scheduler tome decisiones de placement correctas.

## Resource Requests

Un **request** es la cantidad mínima de recursos que Kubernetes **reserva** para un contenedor. El scheduler usa este valor para decidir en qué nodo colocar el Pod: solo lo colocará en un nodo que tenga al menos esa cantidad disponible.

> Si ningún nodo tiene suficientes recursos para satisfacer el request, el Pod queda en estado `Pending` con el error `Insufficient cpu` o `Insufficient memory`.

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
```

- `250m` equivale a 0.25 vCPU (250 millicores).
- `64Mi` son 64 mebibytes de memoria.

El contenedor **puede usar más** recursos de los solicitados si el nodo tiene capacidad libre, pero el scheduler garantiza que siempre habrá al menos esa cantidad disponible.

## Resource Limits

Un **limit** es el máximo de recursos que un contenedor puede consumir. Kubernetes no permite que el contenedor supere este valor.

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

## Comportamiento cuando se superan los límites

### CPU: throttling

La CPU es un recurso **compresible**. Si un contenedor intenta usar más CPU de su límite, el kernel simplemente **le recorta el tiempo de CPU** (throttling). El contenedor no muere, solo se ralentiza.

```
Límite: 500m
Uso intentado: 800m
Resultado: el proceso sigue corriendo pero limitado a 500m → mayor latencia
```

### Memoria: OOM Kill

La memoria es un recurso **no compresible**. Si un contenedor supera su límite de memoria, el kernel lanza un **OOM Kill** (Out Of Memory) y **el contenedor es terminado**.

```
Límite: 128Mi
Uso: 200Mi
Resultado: OOM Kill → el contenedor se reinicia (según la restartPolicy)
```

Kubernetes registra el evento como `OOMKilled`:

```bash
kubectl describe pod <pod-name>
# State: Terminated
# Reason: OOMKilled
```

## Combinaciones posibles de requests y limits

Como se muestra en el diagrama, hay cuatro combinaciones. La recomendación **varía según el recurso** (CPU vs memoria):

| Configuración | Comportamiento | Recomendación |
|---|---|---|
| Sin requests, sin limits | El Pod puede consumir todos los recursos del nodo. Otros Pods pueden quedar sin recursos | ✗ Evitar en producción |
| Sin requests, con limits | Kubernetes asigna el mismo valor del limit como request implícito. El Pod tiene techo pero puede desalojar a otros | ✗ Poco predecible |
| Con requests y limits | El scheduler sabe exactamente qué necesita y el Pod tiene techo. Control total | ~ Válido pero puede desperdiciar recursos si el limit es muy alto |
| Con requests, sin limits | El scheduler puede colocar el Pod correctamente y el contenedor puede aprovechar recursos libres del nodo | ✓ Recomendado para CPU |

### ¿Y para memoria?

La recomendación cambia porque CPU y memoria se comportan de forma muy distinta al superar los límites:

- **CPU** es compresible: si un contenedor intenta usar más de su límite, simplemente se le hace throttling. No muere, no afecta a otros. Por eso no poner limit de CPU es seguro: el contenedor aprovecha recursos libres sin riesgo.

- **Memoria** es no compresible: si un contenedor sin limit de memoria tiene un memory leak, puede consumir toda la RAM del nodo. Cuando el nodo se queda sin memoria, el kernel empieza a hacer OOMKill de forma indiscriminada, pudiendo matar Pods de otros equipos o servicios críticos.

Por eso la recomendación para memoria es diferente:

| Recurso | Recomendación | Motivo |
|---|---|---|
| CPU | Request sin limit | El throttling es inofensivo; aprovechar recursos libres es beneficioso |
| Memoria | Request + limit | Sin limit, un leak puede provocar OOMKill en cascada en todo el nodo |

## LimitRange: valores por defecto

Si no se especifican requests ni limits en un Pod, el namespace puede tener un **LimitRange** que los aplique automáticamente como valores por defecto.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: my-namespace
spec:
  limits:
    - type: Container
      default:          # limit por defecto
        cpu: "500m"
        memory: "128Mi"
      defaultRequest:   # request por defecto
        cpu: "250m"
        memory: "64Mi"
      max:              # límite máximo permitido
        cpu: "1"
        memory: "512Mi"
      min:              # mínimo obligatorio
        cpu: "100m"
        memory: "32Mi"
```

Con este LimitRange, cualquier contenedor en `my-namespace` que no declare resources recibirá automáticamente `cpu: 250m` de request y `cpu: 500m` de limit.

```bash
# Ver los LimitRanges de un namespace
kubectl get limitrange -n my-namespace

# Ver el detalle
kubectl describe limitrange default-limits -n my-namespace
```

> Los LimitRange solo afectan a Pods creados **después** de su aplicación. Los Pods existentes no se ven afectados.

## Resource Quota

Mientras que LimitRange controla los límites **por contenedor**, **ResourceQuota** controla el consumo total de recursos **a nivel de namespace**. Es el mecanismo para garantizar que un equipo o aplicación no acapare todos los recursos del clúster.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
  namespace: my-namespace
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    pods: "10"
```

Con esta quota, el namespace `my-namespace` no puede tener en total más de:
- 4 vCPU en requests, 8 vCPU en limits
- 8Gi de memoria en requests, 16Gi en limits
- 10 Pods simultáneos

Si se intenta crear un Pod que haría superar alguno de estos valores, la creación es **rechazada**:

```
Error from server (Forbidden): pods "nginx" is forbidden:
exceeded quota: my-quota, requested: requests.cpu=1,
used: requests.cpu=4, limited: requests.cpu=4
```

### Consultar el estado de una quota

```bash
kubectl get resourcequota -n my-namespace
kubectl describe resourcequota my-quota -n my-namespace
```

La salida de `describe` muestra el consumo actual frente al límite definido:

```
Name:             my-quota
Namespace:        my-namespace
Resource          Used   Hard
--------          ----   ----
limits.cpu        3      8
limits.memory     6Gi    16Gi
pods              3      10
requests.cpu      1500m  4
requests.memory   3Gi    8Gi
```