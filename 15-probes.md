# 15. Probes en Kubernetes (Liveness, Readiness, Startup)

Las **Probes** permiten a Kubernetes detectar si un contenedor está sano, listo para recibir tráfico o si necesita reiniciarse.
Son una parte crítica del funcionamiento interno de un Pod, especialmente cuando se combina con Services y Deployments.

Este capítulo explica cómo funcionan, para qué sirven, cómo configurarlas y cómo afectan al tráfico dentro del clúster.

---

# 15.1. ¿Qué es una Probe?

Una **probe** es un chequeo que Kubernetes realiza periódicamente dentro de un contenedor para determinar su estado.

Kubernetes puede ejecutar estos chequeos mediante:

* `httpGet`: llamar a un endpoint HTTP
* `tcpSocket`: comprobar que un puerto está abierto
* `exec`: ejecutar un comando dentro del contenedor

El resultado es:

* **0** → éxito
* **1** o **timeout** → fallo

---

# 15.2. Tipos de Probes

Kubernetes tiene tres tipos principales:

## ✔️ 1) **Liveness Probe**

Determina si el contenedor está **vivo**.

Si falla:

* Kubernetes **reinicia el contenedor**.

Se usa para:

* Bloqueos
* Deadlocks
* Aplicaciones colgadas que no responden

Ejemplo:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

---

## ✔️ 2) **Readiness Probe**

Determina si el contenedor está **listo para recibir tráfico**.

Si falla:

* El Pod **entra en estado NotReady**
* Los **Services dejan de enviarle tráfico**

Esto evita que tu app reciba tráfico si:

* Está arrancando
* Está sobrecargada
* Está reiniciando procesos internos
* Está en medio de un despliegue

Ejemplo:

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## ✔️ 3) **Startup Probe**

Determina si la aplicación ha conseguido **arrancar correctamente**.

Se usa para:

* Apps lentas al arrancar
* Frameworks pesados
* Microservicios con mucha inicialización

Si la Startup probe **falla repetidamente**, Kubernetes **mata el contenedor**.

Mientras la startup probe está activa:

* Liveness y readiness están **deshabilitadas** temporalmente.

Ejemplo:

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 5
```

---

# 15.3. ¿Cómo funcionan juntas?

Piensa en estas probes como tres niveles de salud:

1. **Startup probe** → "¿Ha arrancado bien?"
2. **Liveness probe** → "¿Sigue vivo?"
3. **Readiness probe** → "¿Puede recibir tráfico?"

Flujo típico:

* La startup probe valida el arranque → si pasa, habilita readiness y liveness
* La readiness probe decide si entra en el balancer
* La liveness probe vigila durante toda la vida del Pod

---

# 15.4. Cómo afectan a los Services

Los Services **solo envían tráfico** a Pods con:

* `Ready = True`

Si la readiness falla:

* El Pod *sigue vivo*
* Pero **el tráfico deja de llegarle** inmediatamente

Esto es esencial para despliegues RollingUpdate sin downtime.

---

# 15.5. Métodos de comprobación

## ✔️ `httpGet`

Llama a un endpoint y espera un 200 OK.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
```

## ✔️ `tcpSocket`

Comprueba conectividad.

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
```

## ✔️ `exec`

Ejecuta un comando dentro del contenedor.

```yaml
readinessProbe:
  exec:
    command: ["/bin/check_ready.sh"]
```

---

# 15.6. Parámetros importantes de Probes

| Parámetro             | Qué hace                             |
| --------------------- | ------------------------------------ |
| `initialDelaySeconds` | Espera antes del primer chequeo      |
| `periodSeconds`       | Frecuencia de chequeos               |
| `timeoutSeconds`      | Cuánto esperar cada chequeo          |
| `successThreshold`    | Éxitos necesarios antes de marcar OK |
| `failureThreshold`    | Fallos necesarios antes de marcar KO |

---

# 15.7. Ejemplo completo con las 3 probes

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
  failureThreshold: 1

startupProbe:
  httpGet:
    path: /startup
    port: 8080
  periodSeconds: 5
  failureThreshold: 30
```

---

# 15.8. Errores típicos al usar Probes

### ❌ 1. Probes demasiado agresivas

Provoca reinicios constantes.

### ❌ 2. Readiness incorrecta

Pod vivo pero sin tráfico.

### ❌ 3. Startup probe ausente

Apps lentas al arrancar entran en CrashLoopBackoff.

### ❌ 4. Health checks que dependen de otros servicios

No deberían fallar porque una dependencia está caída.

---

# 15.9. Buenas prácticas

* Usar **startupProbes** en todas las apps lentas
* Hacer health checks ligeros y rápidos
* Nunca hacer ready checks que dependan del exterior
* Mantener probes separadas en endpoints claros (/health, /ready, /startup)
* Ajustar thresholds para evitar ruido durante despliegues

---

Los Probes son fundamentales para asegurar despliegues estables, tráfico controlado y Pods sanos en un ambiente real de Kubernetes.
Ahora que dominamos Services + Probes, ya podemos avanzar a temas más avanzados como **Autoscaling**, **Network Policies** o **Ingress**.
