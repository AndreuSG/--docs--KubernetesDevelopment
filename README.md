# Guía de Kubernetes

## Índice

1. [Introducción a Kubernetes](./01-intro.md)
   
   - ¿Qué es Kubernetes? 
   - ¿Por qué necesitamos orquestadores de contenedores?
   - ¿Por qué utilizar Kubernetes?
   - ¿Cómo funciona un clúster de Kubernetes? (Explicación básica)

2. [Funcionamiento Interno](./02-funcionamiento-interno.md)
    - Arquitectura de Kubernetes
    - El papel de `etcd` en Kubernetes
    - Componentes del Control Plane
    - Componentes de los Worker Nodes
    - Pods del sistema (`kube-system`)
    - Seguridad en Kubernetes
    - Cómo se coordinan todos los componentes
  
3. [Cluster Local para Practicar](./03-cluster-local-para-practicar.md)

    - ¿Qué es Minikube?
    - ¿Por qué usar Minikube?
    - Requisitos previos
    - Instalación de Minikube
    - Crear tu primer clúster local

4. [Namespaces en Kubernetes](./04-namespaces-en-kubernetes.md)

    - ¿Qué son los Namespaces?
    - ¿Por qué usar Namespaces?
    - Creación y gestión de Namespaces
    - Ejemplos básicos con Namespaces
    - Buenas prácticas con Namespaces

5. [Pods en Kubernetes](./05-pods-en-k8s.md)

    - ¿Qué es un Pod?
    - ¿En qué se diferencia un Pod de un contenedor?
    - Creación y gestión de Pods
    - Estados comunes de un Pod
    - ¿Qué pasa cuando un Pod muere?
    - Ejemplos básicos con Pods
    - Buenas prácticas con Pods

6. [Describe pods](./06-describe-pods.md)

    - ¿Qué es `kubectl describe`?
    - Uso de `kubectl describe` con Pods
    - Interpretación de la salida
    - Ejemplos prácticos
    - Buenas prácticas al usar `kubectl describe`