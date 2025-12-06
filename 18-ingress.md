# 18. Ingress en Kubernetes

Un **Ingress** es el componente que permite exponer **múltiples aplicaciones HTTP/HTTPS** a través de una única IP externa, con reglas basadas en dominios y rutas.

Es uno de los recursos más importantes para entornos reales y producción, ya que permite:

* Terminar TLS (HTTPS)
* Hacer routing basado en paths
* Hacer routing basado en hostnames
* Redirecciones
* Balanceo de carga L7
* Reglas avanzadas para APIs, microservicios y frontends

---

# 18.1. ¿Qué es un Ingress?

Un Ingress es un conjunto de **reglas HTTP/HTTPS** que se apoyan en un **Ingress Controller** para dirigir tráfico hacia Services dentro del clúster.

Piensa en él como:

* Un **reverse proxy inteligente**
* Que vive dentro del clúster
* Y que sabe cómo enrutar tráfico hacia tus Services

El Ingress NO funciona solo: necesita un **Ingress Controller**.

---

# 18.2. ¿Qué es un Ingress Controller?

El Ingress Controller es el componente que interpreta las reglas Ingress y realiza el routing real.

Controladores más usados:

* **NGINX Ingress Controller (el estándar)**
* Traefik
* HAProxy
* Istio Gateway (en service mesh)
* Kong Ingress Controller

Para Minikube:

```bash
minikube addons enable ingress
```

---

# 18.3. Arquitectura Ingress

```
 Client
   ↓
 LoadBalancer (o MetalLB)
   ↓
 Ingress Controller (NGINX)
   ↓
 Ingress rules
   ↓
 Service (ClusterIP)
   ↓
 Pods
```

El tráfico NO llega directamente al Pod ni al Service sin pasar por el Controller.

---

# 18.4. Crear un Ingress básico

Service previo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
    - host: ejemplo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

**Resultado:**
Todas las peticiones a `ejemplo.com` → `web-service`.

---

# 18.5. Routing basado en paths

```yaml
spec:
  rules:
    - host: ejemplo.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /app
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

Permite que múltiples microservicios compartan una sola IP.

---

# 18.6. Routing basado en dominios

```yaml
spec:
  rules:
    - host: app.empresa.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-service
                port:
                  number: 80
    - host: api.empresa.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
```

---

# 18.7. TLS/HTTPS en Ingress

Para habilitar HTTPS se usa un Secret TLS:

```bash
kubectl create secret tls tls-secret --key clave.key --cert cert.crt
```

Ingress:

```yaml
spec:
  tls:
    - hosts:
        - ejemplo.com
      secretName: tls-secret
```

Así el Ingress Controller termina TLS (SSL offloading).

---

# 18.8. Redirección HTTP → HTTPS

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

---

# 18.9. Reescritura de paths (path rewriting)

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /api/(.*)
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 80
```

Permite mapear rutas internas sin cambiar la URL externa.

---

# 18.10. Health checks

NGINX realiza sus propios health checks internos hacia el Service.

Si los Pods no están en `Ready=True`, el Ingress NO les envía tráfico.

---

# 18.11. Integración con LoadBalancer y MetalLB

Si estás en cloud:

* Un Ingress Controller crea automáticamente un `LoadBalancer` externo

On-prem con MetalLB:

* El Ingress Controller se expone con un Service tipo `LoadBalancer`

Ejemplo:

```bash
kubectl get svc -n ingress-nginx
```

---

# 18.12. Troubleshooting

### ❌ 404 Not Found

* El Ingress Controller no está instalado
* El `host` no coincide
* El path no coincide

### ❌ Conexiones fallando

* TLS mal configurado
* Secret TLS inválido

### ❌ 503 Service Unavailable

* El Service no tiene endpoints
* Los Pods están NotReady

---

# 18.13. Buenas prácticas

* Un solo Ingress Controller por clúster
* Organizar reglas por dominios y paths
* Usar TLS siempre que se exponga tráfico externo
* Comprobar que los Services tienen endpoints antes de depurar Ingress
* Ver logs del Ingress Controller para errores de routing

---

# 18.14. Resumen

Los Ingress permiten exponer múltiples aplicaciones HTTP/HTTPS mediante una única IP y reglas avanzadas basadas en paths y hostnames.
Son esenciales en entornos modernos y completan toda la capa de networking de Kubernetes junto con Services y Network Policies.
