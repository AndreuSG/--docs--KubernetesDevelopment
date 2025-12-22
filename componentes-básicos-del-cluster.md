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