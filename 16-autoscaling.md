# 16. Autoscaling en Kubernetes (HPA, VPA, Cluster Autoscaler)

El Autoscaling permite que Kubernetes ajuste automáticamente los recursos de una aplicación según la carga.
Esto evita sobrecostes, mejora el rendimiento y garantiza que las aplicaciones se adapten al tráfico real.

En este capítulo veremos:

* HPA → Horizontal Pod Autoscaler
* VPA → Vertical Pod Autoscaler
* Cluster Autoscaler → escala nodos

Este módulo se centra especialmente en HPA, que es el autoscaling más usado en el mundo real.

---

# 16.1. ¿Por qué necesitamos autoscaling?

Las cargas de trabajo cambian constantemente:

* horas punta
* Black Friday
* procesos batch
* picos inesperados

Sin autoscaling:

* podrías quedarte sin capacidad (y fallar)
* o pagar de más por recursos infrautilizados

Kubernetes ajusta automáticamente **réplicas**, **recursos** o **nodos completos**.

---

# 16.2. Tipos de autoscaling

Kubernetes ofrece tres niveles:

## ✔️ 1) HPA — Horizontal Pod Autoscaler

Escala **réplicas** de Pods.

Ejemplo: pasar de 3 Pods a 10 cuando sube la CPU.

## ✔️ 2) VPA — Vertical Pod Autoscaler

Ajusta **CPU y memoria** de cada Pod.

Ejemplo: un Pod pasa de 500m CPU a 1 vCPU.

## ✔️ 3) Cluster Autoscaler

Escala **nodos completos** del cluster.

Ejemplo: pasar de 5 a 8 nodos.

> En este capítulo veremos HPA, el más esencial y requerido en el CKAD.

---

# 16.3. HPA (Horizontal Pod Autoscaler)

El HPA ajusta el número de réplicas de un Deployment, StatefulSet o ReplicaSet en función de métricas.

### Métricas soportadas:

* CPU (requiere metrics-server)
* Memoria (requiere addons externos)
* Custom Metrics
* External Metrics

### Ejemplo típico:

"Si la CPU supera el 70% → escala a más Pods."

---

# 16.4. Requisitos para usar HPA

Kubernetes necesita **metrics-server**:

```bash
kubectl get deployment metrics-server -n kube-system
```

Si no existe, instalar en Minikube:

```bash
minikube addons enable metrics-server
```

Comprobar métricas:

```bash
kubectl top pods
kubectl top nodes
```

---

# 16.5. Crear un HPA (ejemplo CPU)

Supongamos este Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx
          resources:
            requests:
              cpu: 100m
```

Creamos un HPA:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

Aplicar:

```bash
kubectl apply -f hpa.yaml
```

Ver estado del HPA:

```bash
kubectl get hpa
```

---

# 16.6. ¿Cómo decide HPA si escala?

La fórmula de Kubernetes:

```
réplicas deseadas = réplicas actuales * (utilización actual / utilización objetivo)
```

Ejemplo:

* 3 Pods
* CPU media = 140%
* objetivo = 70%

```
3 * (140 / 70) = 3 * 2 = 6 réplicas
```

---

# 16.7. Escalado hacia abajo (downscaling)

El HPA reduce réplicas lentamente para evitar oscilaciones:

* periodo de estabilización
* thresholds configurables

Por defecto, baja después de 5 minutos de estabilidad.

---

# 16.8. HPA con custom metrics

Ejemplos:

* Número de peticiones por segundo
* Latencia de API
* Cola de RabbitMQ
* Longitud de una cola de Kafka

Requiere:

* Prometheus Adapter
* Custom Metrics API

Ejemplo (esquema):

```yaml
metrics:
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: 100
```

---

# 16.9. Troubleshooting de HPA

### ❌ `kubectl top pods` no funciona

→ metrics-server no instalado

### ❌ El HPA no escala

* El Deployment no tiene `resources.requests`
* CPU no alcanza el threshold
* metrics-server no tiene permisos

### ❌ El HPA escala demasiado lento

* Ajustar `behavior` en autoscaling/v2

Ejemplo:

```yaml
behavior:
  scaleUp:
    stabilizationWindowSeconds: 0
    policies:
      - type: Percent
        value: 100
        periodSeconds: 10
```

---

# 16.10. Buenas prácticas

* Siempre definir **requests y limits** antes de usar HPA
* Comenzar con thresholds razonables (60-80%)
* Ajustar `behavior` para responder a picos rápidos
* Evitar escalado basado solo en CPU para apps I/O
* Usar custom metrics para microservicios complejos

---

# 16.11. Resumen

El HPA es el mecanismo principal de autoscaling en Kubernetes.
Permite adaptar una aplicación a la carga automáticamente y sin downtime.

Próximo capítulo: **Network Policies**, donde aprenderemos a controlar el tráfico entre Pods dentro del cluster.
