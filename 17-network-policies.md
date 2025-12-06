# 17. Network Policies en Kubernetes

Las **Network Policies** permiten controlar el tráfico de red entre Pods.
Son el firewall interno de Kubernetes y se usan para definir **quién puede hablar con quién** dentro del clúster.

Este capítulo explica cómo funcionan, cómo aplicarlas, cómo depurarlas y sus mejores prácticas.

---

# 17.1. ¿Qué es una NetworkPolicy?

Una **NetworkPolicy** define reglas de red a nivel de Pod.

Permite restringir el tráfico **ingress** (entrada) y **egress** (salida):

* entre Pods
* hacia/desde namespaces
* hacia/desde CIDRs concretos

> Por defecto, Kubernetes **permite todo el tráfico** entre Pods.
> Con una NetworkPolicy puedes bloquearlo o filtrarlo.

---

# 17.2. Requisito: CNI compatible

Las Network Policies **solo funcionan** si tu CNI las soporta.

Ejemplos compatibles:

* Calico (el más completo)
* Cilium
* Weave Net
* Kube-router

Ejemplos **NO compatibles**:

* Flannel (solo networking básico)

Comprobar tu CNI:

```bash
kubectl get pods -n kube-system
```

---

# 17.3. Conceptos básicos

Una NetworkPolicy define:

### ✔️ `podSelector` → A qué Pods se aplica

### ✔️ `ingress` → Qué tráfico de ENTRADA permitimos

### ✔️ `egress` → Qué tráfico de SALIDA permitimos

### ✔️ `namespaceSelector` → Reglas entre namespaces

### ✔️ `ipBlock` → Filtrado por IP/CIDR

Importantísimo:

> Las Network Policies **siempre son listas blancas (whitelists)**.
> Solo permiten lo que tú defines. Todo lo demás queda bloqueado.

---

# 17.4. Primera regla fundamental

## 🚫 Si defines *cualquier* regla ingress o egress, todo lo que NO permitas queda bloqueado.

Ejemplo:

* Si creas una NetworkPolicy con 1 regla ingress
* Solo ese tráfico podrá entrar
* Todo lo demás se bloquea

Esto es lo que implementa un **default deny**.

---

# 17.5. Ejemplo 1: Restringir todo el tráfico (default deny)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: desarrollo
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

Esto bloquea **todo** el tráfico entrante a **todos** los Pods del namespace.

Estrategia recomendada:

* Primero aplicar default deny
* Luego crear reglas específicas de acceso

---

# 17.6. Ejemplo 2: Permitir solo tráfico de Pods con label específica

Permitir que solo los Pods `app=backend` hablen con los Pods `app=database`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-backend
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
```

### Significado:

* Esta regla aplica a Pods `app=database`
* Solo permiten tráfico de Pods `app=backend`
* Todo lo demás queda bloqueado

---

# 17.7. Ejemplo 3: Permitir acceso desde un namespace concreto

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-monitoring
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
```

Permite que **solo los Pods del namespace `monitoring`** accedan.

---

# 17.8. Ejemplo 4: Permitir tráfico externo desde un rango de IP (CIDR)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cidr
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - ipBlock:
            cidr: 192.168.10.0/24
```

Permite que solo esa red acceda a la API.

---

# 17.9. Ejemplo 5: Controlar tráfico de salida (egress)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/16
```

Permite que el backend solo se comunique con la red 10.0.0.0/16.

---

# 17.10. Ejemplo avanzado: Múltiples reglas combinadas

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: complex-policy
spec:
  podSelector:
    matchLabels:
      app: app1
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: app2
        - namespaceSelector:
            matchLabels:
              env: monitoring
        - ipBlock:
            cidr: 192.168.1.0/24
  egress:
    - to:
        - ipBlock:
            cidr: 8.8.8.8/32
```

---

# 17.11. Troubleshooting de Network Policies

## ❌ El tráfico se bloquea aunque hay una regla

* Revisa el `podSelector`
* Revisa el `namespaceSelector`
* Revisa si hay **otra NetworkPolicy** bloqueando

## ❌ No funciona en absoluto

* CNI no soporta políticas (ej. Flannel)

## ❌ Reglas egress no aplican

* Muchos CNIs requieren habilitar egress explícitamente

## ❌ Servicios dejan de responder

* Readiness probes fallan por políticas restrictivas

---

# 17.12. Buenas prácticas

* Usar **default deny** en entornos de producción
* Organizar namespaces por entorno o función
* Etiquetar Pods claramente para políticas
* Testear cada política con Pods "netshoot" o "curl"
* Documentar reglas críticas
* Separar políticas de ingress y egress

---

# 17.13. Resumen

Las Network Policies permiten implementar un firewall interno a nivel de Pod, controlando ingreso y salida de tráfico dentro del clúster.

Son esenciales para entornos seguros, segmentación por microservicios y cumplimiento de estándares corporativos.

Con esto completamos toda la base de networking interno.
El siguiente módulo será **Ingress**, la puerta de entrada HTTP/HTTPS hacia los servicios del cluster.
