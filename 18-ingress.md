# Ingress en Kubernetes

Un **Ingress** es el componente que permite exponer **múltiples aplicaciones HTTP/HTTPS** a través de una única IP externa, con reglas basadas en dominios y rutas.

Es uno de los recursos más importantes para entornos reales y producción, ya que permite:

* Terminar TLS (HTTPS)
* Hacer routing basado en paths
* Hacer routing basado en hostnames
* Redirecciones
* Balanceo de carga L7
* Reglas avanzadas para APIs, microservicios y frontends

## ¿Qué es un Ingress?

Un Ingress es un conjunto de **reglas HTTP/HTTPS** que se apoyan en un **Ingress Controller** para dirigir tráfico hacia Services dentro del clúster.

Piensa en él como:

* Un **reverse proxy inteligente**
* Que vive dentro del clúster
* Y que sabe cómo enrutar tráfico hacia tus Services

El Ingress NO funciona solo: necesita un **Ingress Controller**.

## ¿Qué es un Ingress Controller?

El Ingress Controller es el componente que interpreta las reglas Ingress y realiza el routing real.

Controladores más usados:

* **NGINX Ingress Controller (el estándar)**
* Traefik
* HAProxy

Para Minikube:

```bash
minikube addons enable ingress
```

## Arquitectura Ingress

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

## Crear un Ingress básico

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

## Routing basado en paths

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

## Routing basado en dominios

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

## TLS/HTTPS en Ingress

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

## Redirección HTTP → HTTPS

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

## Reescritura de paths (path rewriting)

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

## Health checks

NGINX realiza sus propios health checks internos hacia el Service.

Si los Pods no están en `Ready=True`, el Ingress NO les envía tráfico.


## Troubleshooting

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

## Buenas prácticas

* Un solo Ingress Controller por clúster
* Organizar reglas por dominios y paths
* Usar TLS siempre que se exponga tráfico externo
* Comprobar que los Services tienen endpoints antes de depurar Ingress
* Ver logs del Ingress Controller para errores de routing

## Resumen

Los Ingress permiten exponer múltiples aplicaciones HTTP/HTTPS mediante una única IP y reglas avanzadas basadas en paths y hostnames.
Son esenciales en entornos modernos y completan toda la capa de networking de Kubernetes junto con Services y Network Policies.
