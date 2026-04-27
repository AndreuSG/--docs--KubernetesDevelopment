# Tips para CKAD

## `kubectl explain`: entender cualquier recurso en tiempo real

El comando **`kubectl explain`** es una de las herramientas más potentes y menos aprovechadas de Kubernetes.
En el examen **CKAD** (y también en el CKA) es un **salvavidas**, porque te permite consultar la **documentación oficial del API de Kubernetes directamente desde el clúster**, sin salir de la terminal.

Este capítulo explica cómo usarlo correctamente y cómo sacarle el máximo partido en un examen.



### ¿Qué es `kubectl explain`?

`kubectl explain` muestra la **estructura exacta de los recursos Kubernetes** tal y como están definidos en el API Server:

* Campos disponibles
* Tipos de datos
* Campos obligatorios
* Jerarquía correcta del YAML

Es, literalmente:

> 📘 **La documentación oficial de Kubernetes en formato CLI**

### ¿Por qué es clave para el CKAD?

En el CKAD:

* No se espera que memorices todos los campos
* Sí se espera que sepas **encontrarlos rápido**

`kubectl explain` te permite:

* Construir YAMLs sin memorizar
* Ver rápidamente dónde va cada campo
* Evitar errores de indentación y estructura
* Confirmar si un campo existe o no

👉 En el examen, **es totalmente válido y recomendado usarlo**.

### Uso básico

```bash
kubectl explain pod
```

Salida típica:

* Descripción del recurso
* Campos principales (`metadata`, `spec`, `status`)

### Navegar por la jerarquía del YAML

La verdadera potencia está en navegar por los campos.

Ejemplo:

```bash
kubectl explain pod.spec
```

Luego:

```bash
kubectl explain pod.spec.containers
```

Y más profundo:

```bash
kubectl explain pod.spec.containers.resources
```

Esto te muestra exactamente:

```yaml
resources:
  limits:
    cpu: string
    memory: string
  requests:
    cpu: string
    memory: string
```

### Ver campos obligatorios

Usa `--recursive` para ver toda la estructura:

```bash
kubectl explain deployment.spec --recursive
```

Muy útil para:

* Deployments
* Services
* Ingress
* HPA

### Ejemplo práctico (CKAD real)

### Objetivo:

Crear un Deployment con límites de CPU y memoria.

### Pasos mentales en examen:

1️⃣ Ver estructura del Deployment:

```bash
kubectl explain deployment
```

2️⃣ Bajar al template del Pod:

```bash
kubectl explain deployment.spec.template.spec.containers
```

3️⃣ Ver recursos:

```bash
kubectl explain deployment.spec.template.spec.containers.resources
```

4️⃣ Escribir el YAML con confianza.


## Redireccionar la salida a un archivo

En un entorno de examen, puede ser útil guardar la salida de un comando con dry-run:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Luego, puedes usar `kubectl explain` para completar los detalles faltantes en el archivo YAML generado.


### Todos los dry-run que necesitas:

Genera rápidamente los manifiestos YAML de los recursos más comunes y guárdalos en archivos para editarlos:

#### Pod
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

#### Deployment
```bash
kubectl create deployment nginx --image=nginx --replicas=2 --dry-run=client -o yaml > deployment.yaml
```

#### Service (ClusterIP)
```bash
kubectl expose deployment nginx --port=80 --target-port=80 --dry-run=client -o yaml > svc.yaml
```

#### ConfigMap
```bash
kubectl create configmap mi-config --from-literal=clave=valor --dry-run=client -o yaml > configmap.yaml
```

#### Secret
```bash
kubectl create secret generic mi-secret --from-literal=clave=valor --dry-run=client -o yaml > secret.yaml
```

#### PersistentVolumeClaim
```bash
kubectl create pvc mi-pvc --storage=1Gi --access-modes=ReadWriteOnce --dry-run=client -o yaml > pvc.yaml
```

#### Job
```bash
kubectl create job mi-job --image=busybox --dry-run=client -o yaml > job.yaml
```

#### CronJob
```bash
kubectl create cronjob mi-cron --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml > cronjob.yaml
```

#### Namespace
```bash
kubectl create namespace mi-namespace --dry-run=client -o yaml > ns.yaml
```

#### Ingress
```bash
kubectl create ingress mi-ingress --rule="host.com/path=svc:80" --dry-run=client -o yaml > ingress.yaml
```

#### NetworkPolicy
```bash
kubectl create networkpolicy mi-np --pod-selector=app=nginx --policy-types=Ingress --dry-run=client -o yaml > np.yaml
```

#### StatefulSet
```bash
kubectl create statefulset mi-sts --image=nginx --replicas=2 --dry-run=client -o yaml > sts.yaml
```

#### Horizontal Pod Autoscaler
```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=1 --max=5 --dry-run=client -o yaml > hpa.yaml
```

> Edita los archivos generados según lo que te pidan en el examen. ¡Ahorra tiempo y evita errores!

