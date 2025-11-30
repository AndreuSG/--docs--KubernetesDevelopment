# Secrets en Kubernetes

Los **Secrets** son el mecanismo de Kubernetes para almacenar información **sensible**: contraseñas, tokens, claves privadas, certificados, credenciales de bases de datos, etc. Funcionan de forma muy similar a los ConfigMaps, pero están pensados específicamente para proteger datos que no deben estar en texto plano dentro de un contenedor o repositorio.

## ¿Qué es un Secret?

Un **Secret** es un objeto de Kubernetes que almacena datos sensibles codificados en **base64** y con mecanismos de protección adicionales.

### ¿Por qué no usar ConfigMaps para datos sensibles?

Porque los ConfigMaps se almacenan en texto plano.

Un Secret proporciona:

* Codificación base64 (evita texto plano directo)
* Mejor integración con mecanismos de seguridad
* Posibilidad de cifrado en reposo (*at rest*) si se configura en el cluster
* Control de acceso mediante RBAC

**Regla básica:**

> ConfigMap = datos NO sensibles

> Secret = datos sensibles

## Crear un Secret

### 1) Desde línea de comando (clave-valor)

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=admin123
```

### 2) Desde fichero

```bash
kubectl create secret generic ssl-secret --from-file=cert.pem
```

### 3) Desde YAML (recomendado)

Para tener manifiestos reproducibles, es mejor definir los Secrets en YAML, codificando los valores en base64.

`secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_USER: YWRtaW4=
  DB_PASSWORD: YWRtaW4xMjM=
```

Aplicar:

```bash
kubectl apply -f secret.yaml
```

## Ver Secrets

```bash
kubectl get secrets
kubectl describe secret db-secret
```

⚠️ Importante:
`kubectl describe` **no mostrará los valores decodificados**, solo las claves.

## Usar Secrets dentro de un Pod

Hay **dos formas principales** de consumir un Secret:

### 1) Como variables de entorno una a una

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
      - name: DB_USER
        valueFrom:
          secretKeyRef:
            name: db-secret
            key: DB_USER
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-secret
            key: DB_PASSWORD
```

### 2) Como variables de entorno completas

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
      - secretRef:
          name: db-secret
```

## Actualizar un Secret

Igual que los ConfigMaps:

* Los Pods **no se reinician automáticamente**.
* Para que tomen los cambios, debes hacer rollout.

```bash
kubectl rollout restart deployment mi-app
```

## Seguridad de los Secrets

Los Secrets **no están cifrados por defecto**, solo se almacenan en base64.

Pero Kubernetes ofrece protección mediante:

* **RBAC** (control de acceso)
* **Encryption at Rest** (si se activa)
* Control de quién puede leer secrets directamente

Opciones avanzadas (no incluidas, pero importantes):

* Integración con **HashiCorp Vault** o OpenBao que es la versión open source de Vault.

## Cosas que suelen romperse (troubleshooting)

### ❌ Error: `secret "db-secret" not found`

Motivo: namespace incorrecto.

### ❌ Error: la app dice que faltan variables

Motivo: clave mal escrita en el YAML.

### ❌ Cambié el Secret pero la app no lo ve

Correcto: necesitas reiniciar Pods.

### ❌ Fichero vacío al montar un Secret

Probable causa:

* La clave existe pero su valor base64 está vacío
* Error al hacer la codificación base64

## Buenas prácticas

* Nunca guardar secretos en repositorios
* Usar YAML sobre CLI para tener manifiestos reproducibles
* Dar permisos **mínimos** mediante RBAC
* Mantener secretos separados por aplicación
* Rotar secretos regularmente
* Evitar decodificar secrets en logs del cluster
