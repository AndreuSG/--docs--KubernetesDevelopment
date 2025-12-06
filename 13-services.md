# 13. Services en Kubernetes

Los **Services** son el mecanismo fundamental de Kubernetes para exponer aplicaciones y permitir que los Pods reciban tráfico de forma estable, incluso cuando se recrean, escalan o cambian de IP.

Los Pods son efímeros, cambian de IP constantemente y pueden morir en cualquier momento. Un **Service** proporciona una **IP virtual estable** y se encarga de distribuir tráfico entre los Pods que coinciden con un selector de labels.

Este capítulo explica todos los tipos de Services, cómo funcionan internamente, qué son los Endpoints y cómo se integran con los Deployments y ReplicaSets.

---

# 13.1. ¿Qué es un Service?

Un **Service** es un recurso de Kubernetes que proporciona:

* Una **IP fija** (ClusterIP)
* Un punto de acceso estable para tu aplicación
* **Balanceo de carga interno** entre Pods
* Control inteligente basado en el estado de los Pods (Ready/NotReady)

Un Service "apunta" a Pods mediante **labels**, y automáticamente envía tráfico solo a los Pods sanos.

---

# 13.2. ¿Por qué los Services son necesarios?

Porque los Pods:

* Cambian de IP constantemente
* Pueden morir y ser reemplazados
* Pueden escalar hacia arriba o abajo
* No son accesibles directamente desde fuera del clúster

El Service proporciona:

### ✔️ **Estabilidad**

Tu aplicación tiene siempre el mismo punto de acceso.

### ✔️ **Descubrimiento automático**

Los Pods se añaden/eliminan del Service sin intervención manual.

### ✔️ **Balanceo de carga**

Distribuye tráfico entre réplicas.

### ✔️ **Integración con readiness probes**

Si un Pod no está listo, el Service no le envía tráfico.

---

# 13.3. Tipos de Services

Kubernetes ofrece tres tipos principales:

## ✔️ 1) **ClusterIP** (por defecto)

Expone la aplicación **dentro del clúster**.
Ideal para comunicación entre microservicios.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

### ¿Cuándo usar ClusterIP?

* Comunicaciones internas
* Backends
* Bases de datos internas

---

## ✔️ 2) **NodePort**

Abre un puerto en **todos los nodos** del clúster.
Permite acceder a la app desde fuera, pero no crea balanceo externo.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### ¿Cuándo usar NodePort?

* Pruebas locales
* Acceder desde fuera en entornos simples

⚠️ No recomendado para producción.

---

## ✔️ 3) **LoadBalancer**

Usado en entornos cloud (AWS/Azure/GCP) o on-prem (datacenter propio).
Crea un balanceador externo que apunta al NodePort.

En on-prem, Kubernetes no trae un LoadBalancer por defecto, así que debes instalar tú un proveedor de LoadBalancer.

El estándar en el mundo on-prem es:

MetalLB → el LoadBalancer para datacenters propios

MetalLB implementa la API de LoadBalancer dentro de tu red local.

Ejemplo de Service LoadBalancer:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

### ¿Cuándo usar LoadBalancer?

* Microservicios expuestos públicamente
* Frontends
* APIs públicas

En Minikube, puedes usar:

```bash
minikube service web-lb
```

para obtener una URL accesible.

---

# 13.4. ¿Cómo funciona un Service internamente?

Cuando creas un Service, Kubernetes crea automáticamente un objeto llamado:

### ✔️ **Endpoints** (o EndpointSlice)

Este objeto contiene **las IPs de los Pods** que coinciden con el selector.

Ejemplo:

```bash
kubectl get endpoints web-service
```

Salida:

```
10.244.0.5:80
10.244.0.8:80
10.244.0.9:80
```

El Service usa estos endpoints para distribuir tráfico.

Si un Pod muere → el endpoint desaparece.
Si un Pod nuevo nace → aparece en endpoints.

Todo esto ocurre automáticamente.

---

# 13.5. Selector de labels (clave del Service)

Tu Service "apunta" a Pods que tengan ciertos labels.

```yaml
selector:
  app: web
```

Solo los Pods que tengan:

```yaml
labels:
  app: web
```

serán incluidos como endpoints.

❗ Si el selector no coincide → el Service queda sin Pods.

```bash
kubectl get endpoints web-service
# EMPTY
```

---

# 13.6. `port`, `targetPort` y `nodePort`

### ✔️ `port`

El puerto del **Service** (IP virtual).

### ✔️ `targetPort`

El puerto del **contenedor**.

### ✔️ `nodePort`

El puerto del **nodo** (solo en NodePort).

Ejemplo completo:

```
Navegador → Node (nodePort=30080) → Service (port=80) → Pod (targetPort=80)
```

---

# 13.7. DNS interno de Kubernetes

Cada Service genera automáticamente un nombre DNS:

```
<service-name>.<namespace>.svc.cluster.local
```

Ejemplo:

```
web-service.desarrollo.svc.cluster.local
```

Los Pods pueden comunicarse entre sí sin usar IPs.

---

# 13.8. Services y Readiness Probes

Los Services **solo envían tráfico** a Pods que estén:

* `Running`
* y `Ready: True`

Si el readiness falla:

* El Pod **sigue vivo**
* Pero el Service **no le envía tráfico**

Esto es clave para rollouts sin downtime.

(Leer Probes en el siguiente capítulo.)

---

# 13.9. Ver Services y Endpoints

```bash
kubectl get svc
kubectl get endpoints
kubectl describe svc web-service
```

---

# 13.10. Common Troubleshooting

### ❌ El Service no funciona

* Selector incorrecto
* El Pod no expone el puerto
* Readiness probe fallando

### ❌ Service sin endpoints

```bash
kubectl get endpoints web-service
```

Si está vacío → problema de labels.

### ❌ NodePort inaccesible

* El firewall del nodo lo bloquea
* Intentas acceder a la IP equivocada

---

# 13.11. Buenas prácticas

* Siempre usar **labels coherentes**
* Exponer servicios internos como ClusterIP
* Evitar NodePort en producción
* Un solo Service por funcionalidad
* Usar readiness probes para tráfico fiable
* Mantener Deployments y Services bien separados

---

Los Services son el puente entre tus Pods y el tráfico interno/externo del clúster.
En el siguiente capítulo veremos los **Probes**, que permiten a Kubernetes saber cuándo un Pod está listo o necesita reiniciarse.
