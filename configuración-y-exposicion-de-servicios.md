# 4. Configuración y exposición de servicios

Además de ejecutar contenedores, Kubernetes permite gestionar cómo se configuran las aplicaciones y cómo se exponen al exterior o a otros servicios dentro del clúster.

Este bloque reúne los recursos fundamentales para **inyectar configuración**, **proteger secretos** y **exponer servicios de forma controlada**.

---

## 4.1. ConfigMap

Un **ConfigMap** permite inyectar valores de configuración no sensibles en tus Pods.

- Guarda datos en forma de pares clave-valor.
- Se puede montar como archivo, variable de entorno o argumento de línea de comandos.

> 🧠 Ejemplo: guardar una URL de API o el nombre de entorno (`ENV=staging`).

---

## 4.2. Secret

Un **Secret** se usa para almacenar información sensible como contraseñas, tokens o claves API.

- Se codifica en Base64 (no cifrado real).
- Se recomienda integrarlo con herramientas externas como HashiCorp Vault o Sealed Secrets para mayor seguridad.

> 🔐 Ejemplo: claves de conexión a una base de datos o credenciales de un servicio cloud.

---

## 4.3. Service

Un **Service** define una política de acceso a uno o más Pods.

Tipos más comunes:

| Tipo         | Uso principal                                             |
|--------------|-----------------------------------------------------------|
| `ClusterIP`  | Comunicación interna entre Pods.                          |
| `NodePort`   | Expone el servicio en un puerto de cada nodo.             |
| `LoadBalancer` | Usa un balanceador externo del proveedor cloud.         |
| `ExternalName` | Redirige a un servicio DNS externo (poco común).        |

> Cada Service selecciona los Pods a los que apunta mediante **label selectors**.

---

## 4.4. Ingress

Un **Ingress** gestiona el acceso HTTP/HTTPS a servicios del clúster mediante reglas:

- Permite enrutar por **dominio o ruta** (`/api`, `/admin`, etc.).
- Soporta HTTPS, redirecciones, cabeceras, etc.
- Es más flexible que exponer servicios directamente por IP o puerto.

> 🧠 Es como un reverse proxy dentro del clúster, pero configurado desde Kubernetes.

---

## 4.5. Ingress Controller

Para que los recursos Ingress funcionen, necesitas desplegar un **Ingress Controller**, que es quien realmente interpreta las reglas.

Opciones comunes:

- `nginx` Ingress Controller (el más popular y fácil de usar)
- `traefik`
- `HAProxy`
- `Istio` (más complejo, usado como service mesh)

> Puedes tener varios Ingress Controllers operando sobre distintos namespaces o dominios.

---

## 4.6. LoadBalancer (nivel cloud o on-premise)

Cuando creas un Service con tipo `LoadBalancer`, Kubernetes solicita al proveedor de nube un balanceador de carga externo (AWS ELB, OVH LBaaS, etc.).

- Funciona automáticamente si tu clúster está en la nube.
- En entornos locales, puedes usar herramientas como **MetalLB** para simularlo.

> 🌐 El LoadBalancer asigna una IP pública accesible desde fuera del clúster.

---

Estos recursos permiten a Kubernetes integrarse perfectamente con el exterior, manteniendo control sobre seguridad, accesibilidad y configuración.
