# 19. `kubectl explain`: entender cualquier recurso en tiempo real

El comando **`kubectl explain`** es una de las herramientas más potentes y menos aprovechadas de Kubernetes.
En el examen **CKAD** (y también en el CKA) es un **salvavidas**, porque te permite consultar la **documentación oficial del API de Kubernetes directamente desde el clúster**, sin salir de la terminal.

Este capítulo explica cómo usarlo correctamente y cómo sacarle el máximo partido en un examen.

---

## 19.1. ¿Qué es `kubectl explain`?

`kubectl explain` muestra la **estructura exacta de los recursos Kubernetes** tal y como están definidos en el API Server:

* Campos disponibles
* Tipos de datos
* Campos obligatorios
* Jerarquía correcta del YAML

Es, literalmente:

> 📘 **La documentación oficial de Kubernetes en formato CLI**

---

## 19.2. ¿Por qué es clave para el CKAD?

En el CKAD:

* No se espera que memorices todos los campos
* Sí se espera que sepas **encontrarlos rápido**

`kubectl explain` te permite:

* Construir YAMLs sin memorizar
* Ver rápidamente dónde va cada campo
* Evitar errores de indentación y estructura
* Confirmar si un campo existe o no

👉 En el examen, **es totalmente válido y recomendado usarlo**.

---

## 19.3. Uso básico

```bash
kubectl explain pod
```

Salida típica:

* Descripción del recurso
* Campos principales (`metadata`, `spec`, `status`)

---

## 19.4. Navegar por la jerarquía del YAML

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

---

## 19.5. Ver campos obligatorios

Usa `--recursive` para ver toda la estructura:

```bash
kubectl explain deployment.spec --recursive
```

Muy útil para:

* Deployments
* Services
* Ingress
* HPA

---

## 19.6. Ejemplo práctico (CKAD real)

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

---

## 19.7. `kubectl explain` vs documentación web

| Web                      | kubectl explain         |
| ------------------------ | ----------------------- |
| Lento                    | Instantáneo             |
| Requiere navegador       | Solo terminal           |
| Puede variar por versión | Coincide con tu clúster |
| Distracciones            | 100% enfocado           |

En examen, **kubectl explain es superior**.

---

## 19.8. Errores comunes

### ❌ Pensar que hay que memorizar YAML

No. Usa `kubectl explain`.

### ❌ No profundizar en los campos

`kubectl explain pod` no es suficiente.

### ❌ Usar documentación de otra versión

`kubectl explain` siempre muestra la versión correcta.

---

## 19.9. Tips específicos para CKAD

* Usa `kubectl explain` **antes de escribir YAML**
* Navega por el recurso hasta el campo exacto
* Copia mentalmente la estructura
* Combínalo con `kubectl apply -f -`
* No pierdas tiempo dudando si un campo existe

Ejemplo rápido:

```bash
kubectl explain service.spec.ports
```

---

## 19.10. Resumen

* `kubectl explain` es la documentación oficial del API
* Es una herramienta clave para CKAD
* Permite escribir YAMLs correctos sin memorizar
* Reduce errores y ahorra tiempo
* Siempre refleja la versión real del clúster

Dominar `kubectl explain` es una ventaja competitiva clara en el examen CKAD y en el día a día profesional.
