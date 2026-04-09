# ¿Cómo funciona Kubernetes por dentro?

Comprender Kubernetes implica entender **cómo está diseñado internamente**, qué componentes lo forman y cómo colaboran para mantener el clúster en funcionamiento.

## etcd: la base de datos del estado

`etcd` es una **base de datos distribuida de tipo clave-valor** donde Kubernetes guarda absolutamente todo el estado del clúster: Pods, Services, Deployments, Secrets, ConfigMaps…

Es el componente más crítico del Control Plane: si `etcd` falla, el clúster pierde toda capacidad de operar. En producción siempre se ejecuta en alta disponibilidad y con backups periódicos.

> Documentación completa en [021-etcd.md](021-etcd.md).

## Componentes del Control Plane

El **Control Plane** es el cerebro del clúster. Orquesta, decide, valida y asegura que el estado deseado se cumpla.

### Componentes principales del plano de control

| Componente                   | Descripción clara y simple                                                                                  |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **kube-apiserver**           | Punto central de comunicación. Todo pasa por su API REST. Valida peticiones y expone el estado del clúster. |
| **etcd**                     | Base de datos distribuida donde vive el estado del clúster.                                                 |
| **kube-scheduler**           | Observa Pods pendientes y decide en qué nodo ejecutarlos (según recursos, afinidades, taints…).             |
| **kube-controller-manager**  | Ejecuta controladores que ajustan el estado real al deseado (ReplicaSet, Node, Endpoint, Job…).             |
| **cloud-controller-manager** | Integra Kubernetes con proveedores cloud para gestionar volúmenes, balanceadores, nodos, etc.               |

Idea clave:

**El plano de control no ejecuta aplicaciones. Solo dirige y decide.**

## Componentes de los Worker Nodes

Los **nodos trabajadores** son las máquinas que ejecutan tus aplicaciones.

### Sus componentes principales

| Componente            | Función                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------- |
| **kubelet**           | Agente que se comunica con la API y asegura que los Pods se ejecuten según lo ordenado. |
| **kube-proxy**        | Gestiona las reglas de red internas y el forwarding del tráfico entre Pods y Services.  |
| **container runtime** | Responsable de ejecutar contenedores (containerd, CRI-O…).                              |

Cómo funciona el proceso:

1. El scheduler asigna un Pod a un nodo.
2. El `kubelet` del nodo recibe la orden.
3. El runtime ejecuta los contenedores.
4. El `kubelet` informa del estado al apiserver.
5. Etcd se actualiza con el nuevo estado.

## Pods del sistema (`kube-system`)

El namespace `kube-system` contiene los **servicios esenciales** de Kubernetes.

| Componente                    | Función                                                          |
| ----------------------------- | ---------------------------------------------------------------- |
| **coredns**                   | Sistema DNS interno para resolución de nombres entre Pods.       |
| **kube-proxy**                | Gestiona la red de los Services.                                 |
| **metrics-server**            | Proporciona métricas de CPU/RAM al HPA para escalado automático. |
| Componentes del control plane | En instalaciones como kubeadm se ejecutan como Pods.             |

Estos Pods garantizan que Kubernetes tenga capacidad de red, DNS, métricas y control.

## Seguridad en Kubernetes

El `kube-apiserver` es también el responsable de toda la seguridad.

Incluye:

* **Autenticación** (tokens, certificados, OIDC…)
* **Autorización (RBAC)** para controlar qué puede hacer cada usuario
* **Admission controllers** que validan o mutan peticiones

Idea clave:
Cada petición a la API **pasa por autenticación, autorización y validación**.

## Cómo se coordinan todos los componentes

Kubernetes funciona como un sistema **reactivo** y **de estado deseado**.

Flujo simplificado:

1. El usuario define un estado deseado (por ejemplo, "quiero 3 réplicas").
2. La API lo guarda en `etcd`.
3. El controller-manager detecta diferencias entre **estado deseado** y **estado real**.
4. Ajusta el clúster: crea Pods, elimina Pods, programa trabajo, etc.
5. Los nodos ejecutan y reportan su estado.

Kubernetes trabaja continuamente para que el estado real coincida con el declarado, **sin intervención humana**.