# Introducción a Kubernetes

## ¿Qué es Kubernetes?

**Kubernetes** (a menudo abreviado como **K8s**) es una plataforma de orquestación de contenedores **open‑source** diseñada para automatizar el despliegue, la gestión, la escalabilidad y la operación de aplicaciones contenerizadas.

Nació en Google a partir de su experiencia gestionando grandes infraestructuras, y hoy es mantenida por la **Cloud Native Computing Foundation (CNCF)**. Kubernetes es independiente del proveedor y puede ejecutarse en:

* Máquinas locales o on‑premise
* Nubes públicas (AWS, Azure, GCP…)
* Entornos híbridos o multi‑cloud

Su objetivo principal es garantizar que las aplicaciones se ejecuten de forma **fiable, escalable y automatizada**.

## ¿Por qué necesitamos orquestadores de contenedores?

Usar contenedores (Docker, Podman…) permite empaquetar aplicaciones de forma ligera y portable. Sin embargo, cuando pasamos a entornos de producción aparecen necesidades que superan la simple ejecución de contenedores:

* **Alta disponibilidad**: si un contenedor falla, debe recrearse automáticamente.
* **Escalabilidad**: poder aumentar o reducir réplicas según la demanda.
* **Distribución inteligente de carga** entre múltiples nodos.
* **Actualizaciones controladas** sin interrumpir el servicio.
* **Monitorización y autosanación**.
* **Aislamiento por entornos** (dev, pre, prod) y control de recursos.

Sin un orquestador, gestionar contenedores de forma manual sería complejo, propenso a errores, difícil de escalar y no tendríamos alta disponibilidad. Aquí es donde Kubernetes destaca.

## ¿Por qué utilizar Kubernetes?

Kubernetes automatiza la operación de aplicaciones contenerizadas y resuelve problemas que aparecen cuando trabajamos con múltiples contenedores en producción.

### Escalado automático

Kubernetes puede ajustar la capacidad de tu aplicación sin que tú intervengas. Esto puede ser de dos formas:

* **Escalado horizontal (HPA)**: añade o elimina Pods según métricas como CPU, RAM o métricas personalizadas (peticiones por segundo, tamaño de cola, latencia…). Por ejemplo: si la CPU supera el 70%, Kubernetes crea más réplicas.
* **Escalado vertical (VPA)**: ajusta los recursos asignados a cada Pod (más CPU o memoria) cuando la carga lo requiere.

### Alta disponibilidad

Si un contenedor deja de funcionar o un nodo falla, Kubernetes detecta el problema y:

* Replica automáticamente los Pods caídos.
* Reubica cargas en otros nodos disponibles.
* Mantiene el servicio operativo sin intervención manual.

### Rollouts y rollbacks controlados

Cuando actualizas una aplicación, Kubernetes realiza un despliegue progresivo:

* Lanza versiones nuevas poco a poco.
* Supervisa su salud.
* Si algo falla, revierte automáticamente a la versión anterior (rollback) sin downtime.

### Recuperación automática (self‑healing)

Kubernetes revisa constantemente el estado de la aplicación. Si un Pod se bloquea o incumple las condiciones de salud:

* Lo reinicia.
* Lo sustituye.
* Asegura que siempre exista el número correcto de réplicas.

### Separación lógica por entornos

Mediante **namespaces**, Kubernetes permite dividir recursos para simular entornos separados:

* desarrollo (dev)
* preproducción (pre)
* producción (prod)
  Esto facilita control de acceso, límites de recursos y organización.

### Independencia del proveedor

Kubernetes no depende de un proveedor específico. Puedes ejecutar el mismo despliegue en:

* tu máquina local
* on‑premise
* AWS, Azure, GCP…
* entornos híbridos o multi‑cloud

Esto evita bloqueos con proveedores y facilita migraciones.

## ¿Cómo funciona un clúster de Kubernetes? (Explicación básica)

Un clúster de Kubernetes está formado por dos partes principales:

### Control Plane (Plano de control)

Es la "capa de cerebro" de Kubernetes. Entre sus componentes clave están:

* **kube‑apiserver**: punto central de comunicación.
* **scheduler**: decide en qué nodo debe ejecutarse cada Pod.
* **controller‑manager**: garantiza que el estado real coincida con el estado deseado.

### Nodos (Workers)

Son las máquinas donde se ejecutan los contenedores.
Cada nodo contiene:

* **kubelet**: agente que gestiona los Pods del nodo.
* **container runtime**: motor que ejecuta los contenedores.

Kubernetes opera mediante un modelo declarativo: el usuario indica *"qué quiere"* y Kubernetes determina *"cómo hacerlo"* a través de su API.