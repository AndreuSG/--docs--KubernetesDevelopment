# 14. LoadBalancer en entornos On‑Premise con MetalLB

En Kubernetes **on‑premise**, los Services tipo `LoadBalancer` NO funcionan de forma nativa porque **no existe un proveedor cloud** que gestione IPs externas (como hace AWS, GCP o Azure).

Para cubrir este hueco, Kubernetes usa proyectos externos. El estándar profesional en entornos propios es:

# ⭐ **MetalLB**

MetalLB implementa un LoadBalancer completamente funcional para redes locales y datacenters corporativos.

Te permite:

* Asignar **IPs reales** de tu red a Services de tipo `LoadBalancer`.
* Crear balanceo de carga **Layer 2 (L2)** o **BGP (L3)**.
* Tener alta disponibilidad y failover.

---

# 14.1. ¿Por qué necesito MetalLB?

Porque:

* Los Pods no deben ser accesibles directamente.
* Los NodePorts no ofrecen IP estable ni balancer profesional.
* Las empresas con hardware propio necesitan exponer servicios internos/externos.

MetalLB te permite usar:

```yaml
spec:
  type: LoadBalancer
```

tal como lo harías en cloud.

---

# 14.2. ¿Cómo funciona MetalLB?

MetalLB asigna IPs a los Services de dos formas distintas:

## ✔️ Modo Layer 2 (L2)

* Usa ARP/NDP para anunciar la IP en la red.
* Simple, no requiere routers configurados.
* Perfecto para redes locales, labs y clusters empresariales sencillos.

## ✔️ Modo BGP (L3)

* MetalLB se comunica directamente con tus routers mediante BGP.
* Alta disponibilidad nativa.
* Ideal para empresas medianas/grandes con routing avanzado.

---

# 14.3. Instalación de MetalLB (manera oficial)

> Se asume que ya tienes un clúster funcionando y acceso mediante kubectl.

### 1) Instalar MetalLB

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

Ver pods:

```bash
kubectl get pods -n metallb-system
```

---

# 14.4. Configurar un Pool de IPs

MetalLB necesita saber qué rango de IPs puede usar.

Ejemplo usando Layer 2 con rango `192.168.50.240‑250`:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.50.240-192.168.50.250
```

Activar L2:

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2
  namespace: metallb-system
spec: {}
```

Aplicar:

```bash
kubectl apply -f pool.yaml
kubectl apply -f l2.yaml
```

---

# 14.5. Crear un Service tipo LoadBalancer (on‑premise)

Ejemplo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

Ver el Service:

```bash
kubectl get svc web-lb
```

Resultado típico:

```
web-lb   LoadBalancer   192.168.50.241   <nodes>   80:30642/TCP   5s
```

La IP `192.168.50.241` ahora es **una IP real de tu red corporativa**.

Puedes abrirla en el navegador.

---

# 14.6. Funcionamiento interno

## ✔️ En modo L2

MetalLB hace anuncios ARP:

* Los switches creen que la IP pertenece al nodo que anuncia.
* Si ese nodo cae, MetalLB mueve el anuncio a otro nodo.

Es muy similar a un **VIP (Virtual IP)** de keepalived.

## ✔️ En modo BGP

* MetalLB anuncia rutas a routers externos.
* Los routers distribuyen tráfico hacia el nodo activo.
* Permite mayor rendimiento y escalabilidad.

---

# 14.7. Troubleshooting

### ❌ El Service no recibe IP externa

* No tienes un IPAddressPool configurado
* Namespace incorrecto
* Versión de MetalLB desactualizada

### ❌ La IP responde desde un solo nodo

* Anuncio L2 funcionando correctamente (esto es normal)

### ❌ Acceso intermitente

* Multicast/ARP filtrado por switches
* VLAN mal configurada

---

# 14.8. Buenas prácticas en producción

* Definir rangos de IP separados para dev/pre/prod
* Documentar qué equipo gestiona BGP o ARP
* Usar L2 para simplicidad, BGP para empresas grandes
* Supervisar los logs de `metallb-controller`
* Usar LoadBalancer en vez de NodePort

---

# 14.9. Resumen

MetalLB permite usar Services tipo `LoadBalancer` en un entorno on‑premise.
Esto aporta:

* IPs estables
* Balanceo de carga
* Integración con redes corporativas
* Alta disponibilidad

Es la forma profesional de exponer servicios Kubernetes sin depender del cloud.

Con esto tu guía cubre de forma completa cómo exponer aplicaciones *on-premise* al mismo nivel que en cualquier proveedor cloud.
