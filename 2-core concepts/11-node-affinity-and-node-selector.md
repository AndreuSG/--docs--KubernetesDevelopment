# Node Selectors y Node Affinity

## Node Selectors

**Node Selector** es la forma más sencilla de restringir en qué nodos puede schedularse un Pod. Se basa en asignar labels a los nodos y luego referenciarlas en el Pod con `nodeSelector`.

### Añadir una label a un nodo

```bash
kubectl label nodes <node-name> <label-key>=<label-value>

# Ejemplo: etiquetar node-1 como "Large"
kubectl label nodes node-1 size=Large
```

### Usar nodeSelector en un Pod

```yaml
spec:
  nodeSelector:
    size: Large
```

El scheduler solo colocará ese Pod en nodos que tengan la label `size=Large`.

### Limitaciones de Node Selector

Node Selector es simple y directo, pero tiene restricciones importantes:

- Solo permite condiciones de igualdad (`key=value`). No hay operadores como `NotIn`, `Gt`, etc.
- No se pueden expresar condiciones compuestas: por ejemplo, "nodos Large **o** Medium", o "cualquier nodo que **no** sea Small".
- No permite preferencias: o cumple la condición o el Pod queda en `Pending`.

> Cuando la arquitectura crece o las necesidades de scheduling se vuelven más complejas, Node Selector se queda corto. La solución es **Node Affinity**.

## ¿Qué es Node Affinity?

**Node Affinity** es un mecanismo que permite **atraer Pods hacia nodos concretos** basándose en las labels de los nodos. A diferencia de los taints y tolerations (que repelen), Node Affinity actúa como un imán: le dice al scheduler en qué nodos *quieres* o *necesitas* que corra un Pod.

> Si los taints/tolerations controlan quién **puede entrar** en un nodo, Node Affinity controla a qué nodo **quiere ir** un Pod.

## Tipos de Node Affinity

| Tipo | Comportamiento |
|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | El Pod **solo** se schedulea en nodos que cumplan la regla. Si no hay ninguno, el Pod queda en `Pending` |
| `preferredDuringSchedulingIgnoredDuringExecution` | El scheduler **intenta** cumplir la regla, pero si no hay nodo que la cumpla, scheduleará el Pod igualmente en otro nodo |

> El sufijo `IgnoredDuringExecution` significa que si las labels del nodo cambian después de que el Pod esté corriendo, el Pod **no es expulsado**.

---

## Definición en YAML

### Required (obligatorio)

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: color
                operator: In
                values:
                  - blue
```

Este Pod solo será schedulado en nodos que tengan la label `color=blue`.

El campo `values` acepta **múltiples valores**. El operador `In` actúa como un OR: el Pod se schedulará en cualquier nodo cuya label coincida con alguno de ellos:

```yaml
              - key: size
                operator: In
                values:
                  - Large
                  - Medium
```

Esto scheduleará el Pod en nodos con `size=Large` **o** `size=Medium`.

### Preferred (preferido)

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: color
                operator: In
                values:
                  - blue
```

El Pod preferirá nodos con `color=blue`, pero si no hay ninguno disponible, irá a cualquier otro nodo.

---

## Operadores disponibles

| Operador | Descripción |
|---|---|
| `In` | La label del nodo tiene alguno de los valores indicados |
| `NotIn` | La label del nodo NO tiene ninguno de los valores indicados |
| `Exists` | La key existe en el nodo (sin importar el valor) |
| `DoesNotExist` | La key no existe en el nodo |
| `Gt` | El valor de la label es mayor que el indicado |
| `Lt` | El valor de la label es menor que el indicado |

---

## Añadir labels a los nodos

Para que Node Affinity funcione, los nodos deben tener labels. Se pueden añadir así:

```bash
# Añadir una label a un nodo
kubectl label nodes <node-name> <key>=<value>

# Ejemplo
kubectl label nodes node1 color=blue

# Ver las labels de un nodo
kubectl get node node1 --show-labels
```

---

## Ejemplo completo: Pod D solo en Nodo 1

Continuando con el ejemplo de taints y tolerations: para garantizar que el **Pod D** vaya **siempre** al **Nodo 1** (y no a cualquier otro), se combina un taint en el nodo con Node Affinity en el Pod:

```bash
# 1. Etiquetar el nodo
kubectl label nodes node1 color=blue

# 2. Aplicar el taint
kubectl taint nodes node1 color=blue:NoSchedule
```

```yaml
# 3. Pod con toleration + nodeAffinity
spec:
  tolerations:
    - key: "color"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: color
                operator: In
                values:
                  - blue
```

Con esta combinación:
- El **taint** impide que otros Pods vayan al Nodo 1.
- La **Node Affinity** garantiza que el Pod D va al Nodo 1 y no a otro.

---

## Por qué se necesitan ambos mecanismos juntos

Cada mecanismo por separado resuelve solo la mitad del problema. La combinación es la única solución completa.

### El problema con solo Taints + Tolerations

Un taint en el Nodo 1 impide que Pods sin toleration entren. Pero si el Pod D tiene la toleration, puede ir al Nodo 1, **pero también puede ir a los Nodos 2 y 3** (que no tienen taint y no lo rechazan).

```
Nodo 1 [taint: color=blue:NoSchedule]
Nodo 2 [sin taint]  ← Pod D puede acabar aquí
Nodo 3 [sin taint]  ← Pod D puede acabar aquí
```

> La toleration es un **permiso**, no una **obligación**. Solo dice "puedo entrar en ese nodo", no "debo ir a ese nodo".

### El problema con solo Node Affinity

Node Affinity dirige al Pod D hacia el Nodo 1. Pero si el Nodo 1 no tiene taint, **otros Pods que no deberían estar ahí también pueden schedularse en él**.

```
Nodo 1 [label: color=blue, sin taint]
        ← Pod D va aquí ✓
        ← Pod X (cualquier otro Pod) también puede ir aquí ✗
```

> Node Affinity atrae Pods al nodo, pero no repele a los demás.

### La solución: combinar ambos

| Mecanismo | Qué resuelve |
|---|---|
| Taint en Nodo 1 | Bloquea la entrada de Pods no autorizados |
| Toleration en Pod D | Le da permiso para entrar al Nodo 1 |
| Node Affinity en Pod D | Lo obliga a ir al Nodo 1 y no a otro |

Con los tres juntos se consigue un **aislamiento bidireccional**:
- El Nodo 1 solo acepta Pods que toleren su taint.
- El Pod D solo va al Nodo 1, aunque haya otros nodos disponibles.

---

## Casos en los que los Pods acaban en nodos no deseados

### Caso 1: Solo toleration, sin Node Affinity

El Pod tiene la toleration para el taint del Nodo 1, pero el scheduler lo puede colocar en cualquier nodo disponible, incluidos Nodo 2 y Nodo 3.

**Solución:** añadir `nodeAffinity` con `required` para forzar el destino.

### Caso 2: Solo Node Affinity, sin taint

El Pod D va al Nodo 1, pero otros Pods también pueden ir ahí. El nodo no está protegido.

**Solución:** añadir un taint al Nodo 1 para que solo entren Pods con la toleration correspondiente.

### Caso 3: Node Affinity con `preferred` en lugar de `required`

Con `preferredDuringSchedulingIgnoredDuringExecution`, si el Nodo 1 está lleno o no disponible, el scheduler enviará el Pod a otro nodo sin error.

**Solución:** usar `requiredDuringSchedulingIgnoredDuringExecution` cuando el destino sea estrictamente obligatorio. El Pod quedará en `Pending` antes de ir a un nodo incorrecto.

### Caso 4: Labels eliminadas o cambiadas en el nodo

Si la label del Nodo 1 se elimina después de que el Pod ya está corriendo, el Pod **no se expulsa** (por el sufijo `IgnoredDuringExecution`), pero nuevos Pods con la misma Node Affinity quedarán en `Pending`.

**Solución:** gestionar las labels de los nodos con cuidado y monitorizar cambios. En el futuro, el tipo `requiredDuringSchedulingRequiredDuringExecution` (aún no implementado en Kubernetes) forzaría la expulsión en este caso.

---

## Resumen

| Mecanismo | Dirección | Garantía |
|---|---|---|
| Taints | El nodo repele Pods | Evita que Pods no deseados entren |
| Tolerations | El Pod tolera el taint | Da permiso para entrar, no garantiza destino |
| Node Affinity | El Pod se atrae al nodo | Garantiza (o prefiere) el nodo de destino |
| **Taint + Toleration + Node Affinity** | **Bidireccional** | **Aislamiento completo: el Pod va donde debe y el nodo solo acepta a quien debe** |