# etcd: la base de datos del clúster

## ¿Qué es etcd?

`etcd` es una **base de datos distribuida de tipo clave-valor**, desarrollada por CoreOS (hoy parte de Red Hat) y actualmente un proyecto graduado de la CNCF. Es el componente donde Kubernetes guarda **absolutamente todo el estado del clúster**.

Sin `etcd`, Kubernetes no tiene memoria: no sabe qué Pods existen, qué está corriendo, qué se ha declarado ni qué hay que hacer.

---

## Bases de datos SQL vs clave-valor: ¿por qué no SQL?

Para entender la elección de etcd, conviene compararlo con una base de datos relacional tradicional.

### Base de datos relacional (SQL)

Una base de datos SQL organiza los datos en **tablas con filas y columnas**, relacionadas entre sí mediante claves foráneas. Es el modelo clásico: PostgreSQL, MySQL, SQLite…

```sql
-- Ejemplo de cómo se vería un Pod en SQL
CREATE TABLE pods (
    id         UUID PRIMARY KEY,
    name       VARCHAR,
    namespace  VARCHAR,
    node       VARCHAR,
    status     VARCHAR,
    spec       JSONB
);
```

Ventajas del modelo SQL:
- Consultas complejas con `JOIN`, filtros, agregaciones
- Integridad referencial entre entidades
- Transacciones ACID completas
- Perfecto para datos con estructura estable y relaciones complejas

Desventajas para el caso de Kubernetes:
- Requiere definir un esquema rígido de antemano
- Escalar horizontalmente es complejo (sharding, replicación, conflictos)
- El modelo relacional no encaja bien con objetos anidados y dinámicos
- Las consultas complejas añaden latencia innecesaria para un caso de uso simple

### Almacén clave-valor (etcd)

Un **almacén clave-valor** asocia cada dato a una clave única, como un diccionario. No hay tablas, no hay esquema, no hay relaciones.

```
clave                              → valor
/registry/pods/default/mi-app      → {spec: {...}, status: {...}}
/registry/services/default/nginx   → {spec: {ports: [...]}}
/registry/secrets/default/db-pass  → {data: {password: "..."}}
```

En Kubernetes, todas las claves siguen una jerarquía de rutas bajo `/registry/`, organizadas por tipo de recurso, namespace y nombre. El valor es siempre el objeto serializado en Protocol Buffers (protobuf).

### Comparativa directa

| Característica            | SQL relacional                          | etcd (clave-valor)                             |
| ------------------------- | --------------------------------------- | ---------------------------------------------- |
| **Modelo de datos**       | Tablas, filas, columnas, relaciones     | Pares clave → valor jerárquicos                |
| **Esquema**               | Rígido, definido de antemano            | Flexible, sin esquema                          |
| **Escalado horizontal**   | Complejo (sharding, réplicas, conflictos) | Nativo mediante clúster Raft                 |
| **Consistencia**          | Depende del motor y configuración       | Fuerte consistencia garantizada (Raft)         |
| **Latencia**              | Mayor (parseo SQL, planificación)       | Muy baja (operaciones directas en memoria)     |
| **Consultas complejas**   | Muy potente (JOIN, GROUP BY…)           | No hay. Solo get/put/delete/watch por clave    |
| **Notificación de cambios** | Requiere polling o triggers            | Nativo con `watch` (push en tiempo real)       |
| **Alta disponibilidad**   | Requiere soluciones externas            | Integrada en el propio protocolo               |
| **Adecuado para**         | Datos relacionales, reporting, negocio  | Coordinación distribuida, configuración, estado |

### ¿Por qué etcd gana para Kubernetes?

Kubernetes no necesita hacer `JOIN` entre tablas ni consultas analíticas. Sus necesidades son distintas:

1. **Leer y escribir objetos completos** por su identificador (namespace + nombre)
2. **Ser notificado al instante** cuando algo cambia (el scheduler, los controladores…)
3. **Garantizar consistencia total** en un entorno distribuido de varios nodos
4. **Escalar sin complejidad** añadida de gestión de base de datos

El modelo clave-valor con `watch` y consenso Raft cubre exactamente esos requisitos, con una complejidad operativa mucho menor que cualquier base de datos relacional distribuida.

---

## ¿Por qué Kubernetes usa etcd?

Kubernetes necesitaba una base de datos que fuera:

| Requisito        | Por qué importa en Kubernetes                                                               |
| ---------------- | ------------------------------------------------------------------------------------------- |
| **Simple**       | API mínima (get, put, delete, watch). Fácil de integrar y razonar.                         |
| **Rápida**       | Escrita en Go, operaciones en memoria con persistencia en disco. Latencias de milisegundos. |
| **Segura**       | Comunicación cifrada con TLS. Autenticación mediante certificados de cliente.               |
| **Distribuida**  | Puede funcionar como clúster de nodos para tolerar fallos (alta disponibilidad).            |
| **Consistente**  | Garantiza que todos los nodos del clúster ven exactamente el mismo estado en todo momento.  |
| **Con `watch`**  | Los clientes pueden suscribirse a cambios en una clave y ser notificados en tiempo real.    |

La capacidad de **`watch`** es especialmente clave para Kubernetes: es lo que permite que el `kube-apiserver`, los controladores y el scheduler reaccionen inmediatamente ante cualquier cambio de estado, sin necesidad de hacer polling.

---

## Cómo funciona la distribución: el algoritmo Raft

`etcd` es **distribuido**, lo que significa que puede ejecutarse como clúster de varios nodos (normalmente 3 o 5 en producción). Para garantizar consistencia entre todos ellos, usa el algoritmo de consenso **Raft**:

1. Uno de los nodos es elegido **líder** (_leader_).
2. Todas las escrituras van al líder.
3. El líder replica la entrada en los demás nodos antes de confirmar la escritura.
4. Una escritura solo se confirma si **la mayoría de nodos** la han aceptado (quórum).

Esto garantiza que nunca haya dos versiones del estado distintas en el clúster, incluso si un nodo falla.

> **Quórum**: con 3 nodos, puede fallar 1. Con 5 nodos, pueden fallar 2. La fórmula es: necesitas al menos `(n/2)+1` nodos operativos.

---

## Qué guarda etcd

Todo lo que existe en Kubernetes vive en `etcd`:

* Especificación completa de recursos (Pods, Services, Deployments, ConfigMaps, Secrets…)
* Estado deseado y estado observado del clúster
* Información de control, coordinación y metadatos

**Ningún otro componente de Kubernetes tiene su propio almacenamiento persistente.** Todo pasa por `etcd`.

---

## Por qué es el componente más crítico

* Es el **"punto único de verdad"** (_single source of truth_) del clúster
* Si `etcd` pierde datos, el clúster no sabe qué recursos existen
* Si `etcd` falla completamente, el Control Plane deja de funcionar
* En producción siempre se hace backup de `etcd` y se ejecuta en alta disponibilidad

Ejemplo claro:

> Cuando haces `kubectl apply -f app.yaml`, el `kube-apiserver` valida la petición y la persiste en `etcd`. A partir de ahí, el scheduler, los controladores y los kubelets trabajan para hacer realidad ese estado declarado. Sin `etcd`, no hay nada que recordar ni coordinar.
