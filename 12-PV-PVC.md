# Almacenamiento Persistente en Kubernetes (PV & PVC)

Los Pods son efímeros: pueden morir, reiniciarse o moverse de nodo en cualquier momento.
Por eso, si tu aplicación necesita **persistir datos** (bases de datos, uploads, logs, almacenamiento compartido…), no puedes depender de un volumen ligado al Pod. Necesitas almacenamiento que **no desaparezca**.

Para eso existen los **PersistentVolumes (PV)** y los **PersistentVolumeClaims (PVC)**.

Este capítulo explica cómo funcionan, cómo crear almacenamiento persistente y cómo conectarlo a tus Pods.

## ¿Qué es un PersistentVolume (PV)?

Un **PersistentVolume (PV)** es un recurso del clúster que representa un espacio físico de almacenamiento.

Puede provenir de:

* Disco local del nodo
* NFS
* Ceph
* iSCSI
* EBS (AWS)
* Persistent Disk (GCP)
* Azure Disk
* GlusterFS
* Cualquier storage CSI

El PV existe **independientemente** de los Pods.

## ¿Qué es un PersistentVolumeClaim (PVC)?

Un **PVC** es una “petición de almacenamiento”.

Un Pod **no usa el PV directamente**, sino que solicita almacenamiento mediante un PVC.

Regla mental:

* PV = recurso físico
* PVC = solicitud lógica
* Pod = usa el PVC

Kubernetes se encarga de unir (“bind”) un PVC con un PV compatible.

## Flujo de uso PV → PVC → Pod

1. El administrador crea un **PV**
2. El desarrollador crea un **PVC** pidiendo X espacio y X acceso
3. Kubernetes empareja el PVC con un PV compatible
4. El Pod monta el PVC como un volumen

Cadena completa:

> **PV → PVC → Pod**

## Tipos de acceso (Access Modes)

Los PV/PVC pueden tener distintos modos:

| Modo                  | Descripción                                               |
| --------------------- | --------------------------------------------------------- |
| `ReadWriteOnce (RWO)` | Un solo nodo puede montar el volumen en lectura/escritura |
| `ReadOnlyMany (ROX)`  | Varios nodos pueden montarlo en solo-lectura              |
| `ReadWriteMany (RWX)` | Varios nodos pueden montarlo en lectura/escritura         |

No todos los proveedores soportan todos los modos.

## Crear un PersistentVolume (PV)

Ejemplo sencillo usando *hostPath* (solo válido para Minikube/labs) Es el equivalente a un bindmount en Docker.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Aplicar:

```bash
kubectl apply -f pv.yaml
```

Ver PVs:

```bash
kubectl get pv
```

## Crear un PersistentVolumeClaim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Aplicar:

```bash
kubectl apply -f pvc.yaml
```

Ver PVCs:

```bash
kubectl get pvc
```

Si encuentra un PV compatible → estado **Bound**.

## Usar un PVC en un Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: pvc-demo
```

### Resultado

Todo lo que se escriba en `/usr/share/nginx/html` **persiste** aunque el Pod muera.



## StorageClasses (aprovisionamiento dinámico)

Crear PVs manualmente es poco práctico.

En clústeres reales usamos **StorageClasses**, que permiten crear PVs automáticamente cuando un PVC lo pide.

Ejemplo de PVC con StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-auto
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 5Gi
```

Kubernetes creará un PV automáticamente.

Para ver StorageClasses disponibles:

```bash
kubectl get storageclass
```

Si queremos crear una StorageClass personalizada (ejemplo on-premise):

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

## Ciclo de vida de los volúmenes

Cuando borras un PVC, depende de la política del PV:

| ReclaimPolicy | Qué hace                                           |
| ------------- | -------------------------------------------------- |
| `Retain`      | El PV no se borra (requiere limpieza manual)       |
| `Delete`      | Se borra automáticamente del proveedor (AWS, GCP…) |
| `Recycle`     | Obsoleto                                           |

## Troubleshooting

### ❌ PVC en estado `Pending`

Causas:

* No existe un PV compatible
* Tamaño mayor que el PV
* AccessMode incompatible
* StorageClass incorrecta

### ❌ Un Pod no arranca por volumen

Causas:

* PVC no está en estado `Bound`
* Error tipográfico en `claimName`
* StorageClass no existe

## Buenas prácticas

* Usar StorageClasses siempre que sea posible
* No usar hostPath en producción
* Separar PVCs por entorno (dev/pre/prod)
* Elegir bien el modo de acceso (RWO/RWX)
* Definir tamaños realistas de storage
* Usar Retain para datos críticos