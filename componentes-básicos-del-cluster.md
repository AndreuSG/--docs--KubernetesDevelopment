# 3. Componentes básicos del clúster

En Kubernetes, todo gira en torno a unos recursos fundamentales que se combinan para definir cómo se ejecutan y gestionan las aplicaciones dentro del clúster.

---

## 3.1. Pod

El **Pod** es la **unidad mínima de ejecución** en Kubernetes. Un pod puede contener uno o varios contenedores que comparten:

- Red
- Sistema de ficheros temporal
- Variables de entorno

> 🔹 Todos los contenedores dentro de un mismo Pod están diseñados para ejecutarse juntos y comunicarse localmente.

---

## 3.2. ReplicaSet

Un **ReplicaSet** garantiza que haya un número deseado de réplicas de un Pod ejecutándose en todo momento.

- Si un Pod falla, el ReplicaSet lo recrea automáticamente.
- Si hay más de los necesarios, elimina los extra.

> ❗ No se usa directamente casi nunca; suele ir dentro de un Deployment.

---

## 3.3. Deployment

Un **Deployment** gestiona el ciclo de vida de las aplicaciones:

- Permite crear y actualizar Pods de forma declarativa.
- Realiza **rollouts** y **rollbacks** de nuevas versiones.
- Escala automáticamente (si se configura con HPA).

> ✅ Es el recurso más usado para desplegar aplicaciones stateless.

---

## 3.4. StatefulSet

Se usa para aplicaciones **con estado** como bases de datos, donde el orden, la identidad y el almacenamiento persistente son importantes.

- Cada Pod tiene un nombre estable y su propio volumen.
- No se recrean de forma intercambiable como en los Deployments.

> 🧠 Ejemplo: PostgreSQL, MongoDB, Kafka...

---

## 3.5. DaemonSet

Un **DaemonSet** asegura que un Pod se ejecute en **todos** los nodos del clúster (o en un subconjunto).

- Ideal para tareas de monitorización, logging, o servicios como `fluentbit`, `node-exporter`, etc.

---

## 3.6. Job y CronJob

- **Job**: ejecuta tareas **una vez** hasta que terminan correctamente.
- **CronJob**: lanza Jobs de forma **periódica** (como un cron de Linux).

> 🛠️ Útiles para tareas batch, backups, migraciones de base de datos, etc.

---

## 3.7. Node

Un **Node** es una máquina física o virtual dentro del clúster de Kubernetes. Cada nodo contiene:

- El agente `kubelet`
- El proxy de red (`kube-proxy`)
- Un runtime de contenedores (Docker, containerd, etc.)

> Los Pods se ejecutan en estos nodos. El scheduler del clúster decide dónde se colocan.

---

## 3.8. Namespace

Los **Namespaces** permiten dividir un clúster en entornos lógicos separados:

- Facilitan la organización por equipo, proyecto o entorno (dev, stage, prod).
- Permiten aplicar políticas de acceso diferentes.

> Kubernetes incluye algunos namespaces por defecto: `default`, `kube-system`, `kube-public`, etc.

---

## 3.9. Label y Selector

- Las **labels** son pares clave-valor que puedes asignar a cualquier recurso.
- Los **selectors** permiten agrupar o filtrar recursos según sus labels.

> 🔍 Por ejemplo, un Service usa un selector para conectar con los Pods adecuados.

---

Estos componentes son los **bloques de construcción** con los que se define casi todo lo que ocurre en un clúster de Kubernetes.
