# Kubernetes con kubeadm - Introducción

## Objetivo

Aprender a instalar los componentes base de Kubernetes usando kubeadm en un entorno Linux (Ubuntu recomendado), preparándonos para crear un cluster real.

## ¿Qué vamos a instalar?

En esta fase NO estamos creando el cluster todavía.
Solo instalamos:

* kubeadm → herramienta de bootstrap
* kubelet → agente que corre en cada nodo
* kubectl → cliente para interactuar con el cluster

## Requisitos previos

### Sistema

* Linux (Ubuntu 24.04 recomendado)
* 2 CPU mínimo
* 4 GB RAM mínimo

### Red

* Conectividad entre nodos
* IP fija

### Identidad única

Cada nodo debe tener:

* hostname único
* MAC única
* product_uuid único

Comprobar:

```
ip link
sudo cat /sys/class/dmi/id/product_uuid
```

## Paso 1 — Preparar el sistema

Desactivar swap:

```
sudo swapoff -a
```

(Importante: Kubernetes no funciona con swap activo)

---

## Paso 2 — Instalar dependencias

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

---

## Paso 3 — Añadir repo de Kubernetes

Crear keyring:

```
sudo mkdir -p -m 755 /etc/apt/keyrings
```

Añadir clave:

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Añadir repositorio:

```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
```

---

## Paso 4 — Instalar kubelet, kubeadm y kubectl

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

Bloquear versiones:

```
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## Paso 5 — Activar kubelet

```
sudo systemctl enable --now kubelet
```

⚠️ Es normal que falle o reinicie continuamente.
Está esperando a que kubeadm le diga qué hacer.

---

## ⚠️ Configuración importante: cgroup driver

El container runtime y kubelet deben usar el mismo driver.

Si no:
→ kubelet falla

Esto lo configuraremos junto con containerd en la siguiente parte.

---

## 🚨 Problemas típicos

* Swap activo → kubeadm falla
* MAC duplicada (en VMs clonadas)
* Kernel no compatible
* Falta container runtime

---

## ⏭️ Siguiente paso

👉 Crear el cluster con kubeadm
👉 Instalar container runtime (containerd)
👉 Añadir red (CNI)

---

## 🧠 Resumen

Esta fase deja los nodos preparados.

Aún NO hay cluster.

El cluster empieza con:

```
kubeadm init
```

---

(Continuará...)
