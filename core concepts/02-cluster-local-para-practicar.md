# Crear un clúster local para practicar Kubernetes

Para aprender Kubernetes sin coste y sin depender de un proveedor cloud, lo más práctico es levantar un **clúster local**. Existen varias herramientas, pero la más ligera, sencilla y perfecta para empezar es **Minikube**.

En este apartado explicamos qué es Minikube, por qué usarlo y cómo crear tu primer clúster local.

## ¿Qué es Minikube?

**Minikube** es una herramienta que permite ejecutar un clúster de Kubernetes de un solo nodo en tu propio ordenador. Está pensado para:

* Aprender conceptos de Kubernetes
* Probar manifiestos YAML
* Testear Deployments, Services, Ingress, HPA, etc.
* Desarrollar aplicaciones localmente con un entorno real de Kubernetes

## ¿Por qué usar Minikube?

* Es rápido de instalar
* No necesita recursos de nube
* Simula un entorno Kubernetes completo
* Permite activar addons como Ingress, Dashboard, Metrics Server…
* Perfecto para experimentar sin romper nada en producción

## Requisitos previos

Necesitarás Docker instalado en tu máquina, ya que Minikube usará contenedores para crear el clúster.

Y, por supuesto, tener instalado `kubectl`.

## Instalación de Minikube

### Linux

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Crear tu primer clúster local

Usando Docker como driver:

```bash
minikube start --driver=docker
```

Esto crea un clúster Kubernetes completo dentro de tu máquina.

Para verificar que está funcionando:

```bash
# Ver versión de kubectl conectada al clúster
minikube kubectl version

# Ver estado del clúster
minikube status

# Ver nodos del clúster
minikube kubectl get nodes
```

Deberías ver un nodo con estado **Ready**.

Cuando quieras parar el clúster

```bash
minikube stop
```

<br>

---

**A partir de aquí, usaremos `minikube kubectl` en lugar de `kubectl` para interactuar con el clúster local.**

En un entorno real, usarías `kubectl` configurado con el contexto adecuado. Pero Minikube simplifica esto al incluir `kubectl` integrado. Nosotros pondremos `minikube kubectl`, pero recuerda que en un cluster productivo usarías simplemente `kubectl`.

---