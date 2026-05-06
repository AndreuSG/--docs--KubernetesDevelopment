## ¿Cómo funciona el logging en Kubernetes?

Kubernetes no tiene un sistema de logging centralizado por defecto. Los logs de los contenedores se escriben en la salida estándar (`stdout`) y error estándar (`stderr`), y el container runtime (containerd) los captura y almacena en ficheros en el nodo.

El acceso a esos logs se hace principalmente a través de `kubectl logs`.

## Logs de un Pod

```bash
# Ver los logs de un Pod
kubectl logs <pod-name>

# Ver los logs en tiempo real (equivalente a tail -f)
kubectl logs -f <pod-name>

# Ver solo las últimas N líneas
kubectl logs --tail=50 <pod-name>

# Ver logs de los últimos X minutos/horas
kubectl logs --since=1h <pod-name>
```

## Pods con múltiples contenedores

Cuando un Pod tiene más de un contenedor, hay que especificar de cuál se quieren ver los logs con `-c`:

```bash
# Listar los contenedores de un Pod
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].name}'

# Ver los logs de un contenedor concreto
kubectl logs <pod-name> -c <container-name>

# Ver los logs en tiempo real de un contenedor concreto
kubectl logs -f <pod-name> -c <container-name>
```

### Ejemplo

Dado un Pod con dos contenedores (`app` y `sidecar`):

```bash
kubectl logs my-pod -c app
kubectl logs my-pod -c sidecar
```

## Logs de un Pod que ha fallado o reiniciado

Si un contenedor ha fallado y se ha reiniciado, los logs del contenedor anterior se pueden consultar con `-p` (previous):

```bash
kubectl logs <pod-name> -p
kubectl logs <pod-name> -c <container-name> -p
```

## Logs a nivel de nodo

Los logs de los componentes del sistema (kubelet, containerd, etc.) no son accesibles con `kubectl logs` ya que no son Pods gestionados por Kubernetes (en nodos que usan systemd):

```bash
# Logs del kubelet
journalctl -u kubelet -f

# Logs del container runtime (containerd)
journalctl -u containerd -f
```

Para los componentes del plano de control que sí corren como Pods estáticos (apiserver, scheduler, controller-manager, etcd):

```bash
kubectl logs -n kube-system kube-apiserver-<node-name>
kubectl logs -n kube-system kube-scheduler-<node-name>
kubectl logs -n kube-system etcd-<node-name>
```

## Logging centralizado

Por defecto, los logs solo están disponibles mientras el Pod existe y solo en el nodo donde corre. Para tener logs persistentes y centralizados se necesita una solución externa:

| Stack | Descripción |
|---|---|
| **EFK** (Elasticsearch + Fluentd + Kibana) | El stack más extendido en Kubernetes |
| **ELK** (Elasticsearch + Logstash + Kibana) | Similar al EFK, con Logstash en lugar de Fluentd |
| **Loki + Grafana** | Stack ligero de Grafana Labs, optimizado para Kubernetes |
| **Zabbix** | Si ya existe el ecosistema, puede integrarse para centralizar logs junto con métricas |

En estos stacks, un agente (Fluentd, Fluent Bit, Promtail...) corre como **DaemonSet** en cada nodo, recoge los logs de todos los contenedores y los envía al sistema de almacenamiento centralizado.

## Resumen de comandos

| Comando | Descripción |
|---|---|
| `kubectl logs <pod>` | Logs del Pod |
| `kubectl logs -f <pod>` | Logs en tiempo real |
| `kubectl logs --tail=N <pod>` | Últimas N líneas |
| `kubectl logs --since=1h <pod>` | Logs de la última hora |
| `kubectl logs <pod> -c <container>` | Logs de un contenedor específico |
| `kubectl logs -p <pod>` | Logs del contenedor anterior (tras reinicio) |
