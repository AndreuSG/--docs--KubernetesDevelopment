# `kubectl describe pod`: Guía completa

El comando `kubectl describe pod` es una de las herramientas más importantes para **diagnosticar problemas**, entender el estado real de un Pod y ver cómo Kubernetes está gestionando sus contenedores. Mientras que `kubectl get pods` muestra un resumen, **describe** proporciona un análisis profundo y detallado.

En este caso nos centraremos en su uso con Pods, pero `kubectl describe` funciona con muchos otros recursos (Nodes, Deployments, Services, etc...).

## ¿Qué es `kubectl describe pod`?

`kubectl describe pod` muestra información completa sobre:

* Contenedores del Pod y su estado
* Probes (liveness / readiness / startup)
* Volúmenes
* Eventos recientes
* Recursos y límites
* Configuración de red
* QoS
* Nodo asignado

En palabras simples:

> **`describe` te dice "qué está pasando", “cómo está pasando” y “por qué está pasando”.**

Mientras que `kubectl get` solo te dice “qué está pasando”.

## Comando básico

```bash
kubectl describe pod nombre-del-pod
```

Si quieres especificar un namespace:

```bash
kubectl describe pod nombre-del-pod -n desarrollo
```

## Secciones importantes del `describe`

Vamos a revisar todas las secciones relevantes.

### Información general

Aquí se muestra:

* Namespace
* Node
* Labels
* Annotations
* IP del Pod
* IP del nodo
* QoS Class

Ejemplo:

```
Name:         nginx-pod
Namespace:    desarrollo
Node:         minikube/192.168.49.2
Start Time:   Wed, 27 Nov 2025 12:34:56 +0100
Labels:       app=nginx
Status:       Running
IP:           10.244.0.5
QoS Class:    BestEffort
```

**Qué mirar:**

* ¿Está en el namespace que esperabas?
* ¿Está asignado a un nodo?
* ¿Tiene IP? Si no tiene IP → problema de red.

### Contenedores del Pod

Aquí verás cada contenedor listado con:

* Imagen
* Estado
* Reason (muy importante)
* Restarts
* Puertos expuestos

Ejemplo:

```
Containers:
  nginx:
    Container ID:   docker://123abc...
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:...
    Port:           80/TCP
    State:          Running
    Ready:          True
    Restart Count:  0
```

**Qué mirar:**

* `State`: Running / Waiting / Terminated
* `Restart Count`: si sube rápido → CrashLoopBackOff
* `Ready`: indica si pasó los readiness probes---

### Estados clave de contenedores

**Running:**
Todo bien.

**Waiting:**
El contenedor no arrancó. Razones típicas:

* `ImagePullBackOff`
* `ErrImagePull`
* `CrashLoopBackOff`

**Terminated:**
Ejecutó y terminó (normal en Jobs).

Aquí siempre aparece un `Reason` explicando el porqué.

### Probes (liveness, readiness, startup)

Si un probe falla, lo verás aquí.

Ejemplo:

```
Liveness: http-get http://:80/health delay=10s timeout=1s period=10s
Readiness: http-get http://:80/ready delay=5s timeout=1s period=5s
```

**Indicadores de fallo:**

* Eventos con `probe failed`
* `Ready: False`
* Muchos reinicios

### Volúmenes y mounts

Ejemplo:

```
Volumes:
  html-volume:
    emptyDir: {}
```

Qué mirar:

* ¿El volumen existe?
* ¿El mountPath es correcto?

### Eventos (¡la parte más importante!)

Siempre está al final.

Ejemplo:

```
Events:
  Type     Reason     Age    From               Message
  ----     ------     ----   ----               -------
  Normal   Scheduled  12s    default-scheduler  Successfully assigned desarrollo/nginx-pod to minikube
  Normal   Pulling    12s    kubelet            Pulling image "nginx:latest"
  Normal   Pulled     1s     kubelet            Successfully pulled image "nginx:latest"
  Normal   Created    1s     kubelet            Created container nginx
  Normal   Started    1s     kubelet            Started container nginx
```

Si hay problemas, verás avisos como:

* `FailedScheduling`
* `Back-off pulling image`
* `Unhealthy`
* `Probe failed`

## Consejos finales

* Mirar SIEMPRE los eventos al final
* Revisar número de reinicios
* Verificar probes cuando Ready = False
* Confirmar que la imagen existe
* Revisar volúmenes cuando hay errores en mounts
* Confirmar que el Pod está en el namespace correcto

`kubectl describe pod` es la herramienta esencial para entender qué está pasando dentro de un Pod. Dominarla te permitirá hacer troubleshooting rápido y preciso en Kubernetes.