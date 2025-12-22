# StatefulSets en Kubernetes

Los **StatefulSets** están diseñados para ejecutar aplicaciones **con estado** (*stateful applications*), es decir, aplicaciones que necesitan **identidad estable**, **almacenamiento persistente asociado** y **orden en el arranque y apagado**.

Son el recurso adecuado para bases de datos, sistemas distribuidos y servicios que **no pueden tratar a sus Pods como intercambiables**.

## ¿Qué problema resuelven los StatefulSets?

Los Deployments asumen que:

* Todos los Pods son iguales
* Los Pods pueden destruirse y recrearse sin consecuencias
* El orden no importa

Pero muchas aplicaciones **NO funcionan así**:

* Bases de datos (MySQL, PostgreSQL, MongoDB)
* Sistemas distribuidos (Kafka, ZooKeeper, etcd)
* Aplicaciones con replicación primaria/secundaria

Estas aplicaciones necesitan:

* Un **nombre estable** por Pod
* Un **volumen persistente fijo** por Pod
* Un **orden controlado** de arranque y apagado

👉 Ahí es donde entra **StatefulSet**.

## Diferencias clave: Deployment vs StatefulSet

| Característica    | Deployment            | StatefulSet              |
| ----------------- | --------------------- | ------------------------ |
| Identidad del Pod | Aleatoria             | Estable                  |
| Nombre del Pod    | Cambia                | Fijo (`app-0`, `app-1`…) |
| Almacenamiento    | Compartido / opcional | Volumen dedicado por Pod |
| Escalado          | Paralelo              | Ordenado                 |
| Arranque          | Sin orden             | Secuencial               |
| Uso típico        | Apps stateless        | Apps stateful            |

## Identidad estable de los Pods

Un StatefulSet crea Pods con nombres predecibles:

```
mi-app-0
mi-app-1
mi-app-2
```

Si un Pod muere:

* Se recrea con **el mismo nombre**
* Vuelve a montar **el mismo volumen**

Esto permite que:

* Cada instancia tenga su propio estado
* La app reconozca a sus nodos

## El Service Headless (obligatorio)

Los StatefulSets **requieren** un **Service headless** (`clusterIP: None`).

¿Por qué?

* Para resolver DNS individuales por Pod
* Para mantener identidad de red estable

Ejemplo de Service headless:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  clusterIP: None
  selector:
    app: db
  ports:
    - port: 5432
```

DNS generado:

```
db-0.db.namespace.svc.cluster.local
db-1.db.namespace.svc.cluster.local
```

---

## 20.5. StatefulSet básico (ejemplo)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

---

## 20.6. `volumeClaimTemplates`

Esta sección es **clave** en los StatefulSets.

* Kubernetes crea **un PVC por Pod**
* Cada Pod tiene su propio volumen

Ejemplo:

```
data-db-0
data-db-1
data-db-2
```

Los volúmenes:

* NO se borran si el Pod muere
* Persisten incluso si escalas a 0

---

## 20.7. Escalado en StatefulSets

### Escalado hacia arriba

* Se crean Pods **uno a uno**
* Primero `app-0`, luego `app-1`, etc.

### Escalado hacia abajo

* Se eliminan Pods **en orden inverso**
* Primero `app-2`, luego `app-1`

Esto protege la integridad del estado.

---

## 20.8. Actualizaciones (rolling updates)

StatefulSets también soportan rolling updates, pero:

* Se aplican **de uno en uno**
* En orden
* Esperando a que cada Pod esté Ready

Configuración:

```yaml
updateStrategy:
  type: RollingUpdate
```

---

## 20.9. Borrado de StatefulSets

⚠️ IMPORTANTE:

```bash
kubectl delete statefulset db
```

* El StatefulSet se elimina
* **Los Pods se borran**
* **Los PVCs NO se borran**

Esto evita pérdida de datos accidental.

---

## 20.10. Casos de uso reales

Usa StatefulSet para:

* PostgreSQL
* MySQL
* MongoDB
* Kafka
* Redis (con persistencia)
* Elasticsearch
* ZooKeeper

NO usar StatefulSet para:

* Frontends
* APIs stateless
* Workers sin estado

---

## 20.11. Errores comunes

### ❌ No crear Service headless

→ DNS no funciona correctamente

### ❌ Usar RWX sin soporte del storage

→ PVC no se enlaza

### ❌ Intentar tratar Pods como intercambiables

→ Rompe la lógica de la app

---

## 20.12. Buenas prácticas

* Usar siempre Service headless
* Usar StorageClasses fiables
* Proteger bases de datos con Network Policies
* Ajustar probes cuidadosamente
* Documentar claramente la topología

---

## 20.13. Resumen

Los StatefulSets permiten ejecutar aplicaciones con estado de forma segura y controlada en Kubernetes.

Son esenciales cuando:

* Cada Pod importa
* El almacenamiento debe persistir
* El orden y la identidad son críticos

Con esto completas el bloque de **workloads avanzados** en Kubernetes.
