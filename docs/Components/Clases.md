---
sidebar_position: 2
---

# Clases Internas

Existe una serie de clases e interfaces diseñadas para funcionalidades particulares dentro del proyecto. A continuación, se presenta una breve descripción de cada clase junto con sus métodos principales.

---

## `ModuleCounter`

Esta clase permite crear y gestionar un **conjunto ordenado y cíclico de números binarios** (o elementos de un arreglo), facilitando la navegación secuencial y controlada. Es ideal para la paginación interna o el manejo de estados que deben ciclar.

### Métodos

| Método | Descripción |
| :--- | :--- |
| `current(): number` | **Retorna el valor actual** utilizado dentro del conjunto. |
| `next(): void` | **Avanza al siguiente elemento.** Si el valor actual es el último elemento del conjunto, el contador vuelve al primer elemento (comportamiento cíclico). |
| `prev(): void` | **Retrocede al elemento anterior.** Si el valor actual es el primer elemento, el contador se mueve al último elemento. |
| `add(n: number): void` | Permite **avanzar `n` posiciones** en el conjunto (si `n` es positivo) o **retroceder `n` posiciones** (si `n` es negativo). |
| `set(n: number): void` | Permite establecer directamente un nuevo valor como **`current()`**, siempre que el valor `n` **exista** dentro del conjunto. De lo contrario, lanzará una excepción. |

### 💡 Ejemplo de Uso

Asumiendo que el `ModuleCounter` fue inicializado con el conjunto `[0, 1, 2, 3]`:

#### 1. Navegación Secuencial

```typescript title="Navegación Cíclica"
const counter = new ModuleCounter([0, 1, 2, 3]);

console.log(counter.current()); // 0

counter.next();
console.log(counter.current()); // 1

counter.prev();
console.log(counter.current()); // 0 (Vuelve)

// Comportamiento cíclico
counter.prev();
console.log(counter.current()); // 3 (Se mueve del 0 al último elemento)
````

#### 2\. Uso del método `add()`

```typescript title="Avance y Retroceso Rápido"
const counter = new ModuleCounter([0, 1, 2, 3]); // current() es 0

// Avanzar 2 posiciones
counter.add(2);
console.log(counter.current()); // 2

// Retroceder 3 posiciones
counter.add(-3);
console.log(counter.current()); // 3 (De 2 -> 1 -> 0 -> 3)
```

#### 3\. Uso del método `set()`

```typescript title="Establecer un Valor Específico"
const counter = new ModuleCounter([0, 1, 2, 3]);

counter.set(2);
console.log(counter.current()); // 2

// Intentar establecer un valor fuera del conjunto [0, 1, 2, 3]
try {
  counter.set(5);
} catch (error) {
  console.error(error.message); // Lanza excepción: "El valor 5 no existe en el conjunto."
}
```

## `Queue<T>`

Esta clase implementa una estructura de datos de **cola (Queue)**, donde los elementos se procesan en un orden **FIFO** (*First In, First Out*). Permite gestionar una lista de datos que se van procesando uno a uno.

### Métodos

| Método | Tipo de Retorno | Descripción |
| :--- | :--- | :--- |
| `enqueue(payload: T)` | `void` | **Añade** un nuevo elemento (`payload`) al final de la cola. |
| `dequeue()` | `T \| undefined` | **Elimina** el elemento que se encuentra al principio de la cola y lo devuelve. Si la cola está vacía, retorna `undefined`. |
| `peek()` | `T \| undefined` | **Devuelve** el elemento que se encuentra al principio de la cola, pero **sin eliminarlo**. Si la cola está vacía, retorna `undefined`. |
| `size()` | `number` | Retorna la **cantidad total de elementos** que contiene la cola actualmente. |
| `isEmply()` | `boolean` | Devuelve `true` si la cola no contiene elementos, o `false` en caso contrario. |
| `clear()` | `void` | **Elimina todos los elementos** de la cola, dejándola vacía. |

### 💡 Ejemplo de Uso

```typescript title="Gestión de Tareas"
// La Queue se inicializa vacía
const tareaQueue = new Queue<string>();

// 1. Agregar elementos (enqueue)
tareaQueue.enqueue("Procesar imagen 1");
tareaQueue.enqueue("Enviar notificación");
tareaQueue.enqueue("Actualizar base de datos");

console.log(tareaQueue.size());    // 3
console.log(tareaQueue.isEmply()); // false

// 2. Ver el elemento siguiente (peek)
console.log(tareaQueue.peek());    // "Procesar imagen 1"
console.log(tareaQueue.size());    // 3 (No se elimina)

// 3. Procesar y eliminar (dequeue)
let tarea = tareaQueue.dequeue();
console.log(`Tarea completada: ${tarea}`); // Tarea completada: Procesar imagen 1

tarea = tareaQueue.dequeue();
console.log(`Tarea completada: ${tarea}`); // Tarea completada: Enviar notificación

// 4. Limpiar la cola
tareaQueue.clear();
console.log(tareaQueue.isEmply()); // true
```

## `QueueExecutor`

Esta clase implementa una cola de tareas (`task[]`) con capacidad de **ejecución automática y secuencial**. Está diseñada para procesar tareas asíncronas o síncronas una tras otra, asegurando que solo se procese una tarea a la vez.

### Tipo de Tarea

El tipo de tarea aceptado es:

```typescript
type task = (payload?: any) => (void | Promise<void>)
````

### Constructor

```typescript
constructor(...init: task[])
```

El constructor recibe tareas iniciales opcionales. Si se inicializa con tareas, el `QueueExecutor` **comenzará a procesar inmediatamente**.

### Métodos de Control y Estado

| Método | Tipo de Retorno | Descripción |
| :--- | :--- | :--- |
| `start()` | `void` | Inicia el procesamiento de las tareas pendientes en la cola. Si la cola está vacía, no hace nada hasta que se añada una tarea. |
| `pause()` | `void` | Detiene inmediatamente la ejecución de la cola. El procesamiento se reanudará en la siguiente tarea al llamar a `start()`. |
| `IsProcessing()` | `boolean` | Devuelve `true` si la cola se está ejecutando activamente, o `false` si está en pausa o terminada. |

### Métodos de Gestión de Cola

| Método | Tipo de Retorno | Descripción |
| :--- | :--- | :--- |
| `enqueue(payload: task)` | `void` | **Añade** una nueva tarea al final de la cola para ser ejecutada. |
| `dequeue()` | `task \| undefined` | **Elimina** la tarea al principio de la cola y la devuelve (usado internamente). Retorna `undefined` si está vacía. |
| `peek()` | `task \| undefined` | **Devuelve** la tarea al principio de la cola, **sin eliminarla**. Retorna `undefined` si está vacía. |
| `size()` | `number` | Retorna la **cantidad total de tareas** que esperan en la cola. |
| `isEmpty()` | `boolean` | Devuelve `true` si la cola de tareas está vacía. |
| `clear()` | `void` | **Elimina todas las tareas** pendientes de la cola. |

### 💡 Ejemplo de Uso

En este ejemplo, se crean dos tareas asíncronas y se gestiona su ejecución secuencial.

```typescript title="Ejecución Secuencial de Tareas"
// Definición de las tareas
const tarea1 = async () => {
  console.log("-> Tarea 1: Iniciada");
  await new Promise(resolve => setTimeout(resolve, 1000));
  console.log("-> Tarea 1: Finalizada");
}

const tarea2 = () => {
  console.log("-> Tarea 2: Síncrona ejecutada");
}

// 1. Inicializar y agregar tareas
const executor = new QueueExecutor();

executor.enqueue(tarea1);
executor.enqueue(tarea2);
executor.enqueue(tarea1);

console.log(`Cola inicializada con ${executor.size()} tareas.`); // 3
console.log(`¿Procesando? ${executor.IsProcessing()}`); // false

// 2. Iniciar el procesamiento
executor.start();
console.log(`Cola en ejecución: ${executor.IsProcessing()}`); // true

// (Tras 1 segundo)
// -> Tarea 1: Iniciada
// (Tras 2 segundos)
// -> Tarea 1: Finalizada
// -> Tarea 2: Síncrona ejecutada
// (Tras 3 segundos)
// -> Tarea 1: Iniciada
// (Tras 4 segundos)
// -> Tarea 1: Finalizada

// (Al finalizar)
// console.log(`¿Procesando? ${executor.IsProcessing()}`); // false
```

## `Mutex`

El **Mutex** (del inglés *Mutual Exclusion*) es un elemento típico de lenguajes modernos pensados en concurrencia como `Golang` y `Rust`. Dado que JavaScript (y TypeScript) no lo incluyen de manera nativa, esta implementación es crucial para proteger bloques de código susceptibles a **condiciones de carrera** debido a la naturaleza asíncrona del entorno.

Se utiliza para asegurar que una sección crítica de código sea ejecutada por una sola llamada a la vez.

### Inicialización

El `Mutex` debe ser inicializado como una propiedad de la clase donde se necesita proteger la concurrencia, preferiblemente como `private`.

```typescript title="Inicialización del Mutex"
export class YService{

  private readonly mu: Mutex = new Mutex()

  // ...
}
````

### `Lock(): Promise<Locked>`

Este método **bloquea** la ejecución de una función hasta que la llamada anterior haya terminado. Si la función es llamada concurrentemente varias veces, todas las llamadas subsecuentes esperarán a que la primera libere el bloqueo.

```typescript title="Aplicando el Bloqueo"
export class YService{

  private readonly mu: Mutex = new Mutex()

  async functionConcurrente(){
    await this.mu.Lock()
    // Lógica que DEBE ser ejecutada en un solo hilo de forma NO asíncrona entre distintas llamadas.
  }

}
```

### `Unlock(): void`

El método `Lock()` por sí mismo no tiene conocimiento de cuándo termina la lógica interna. Por lo tanto, el método **siempre requiere** un llamado explícito a `Unlock()` al finalizar para liberar el Mutex y permitir que la siguiente llamada en espera pueda avanzar.

```typescript title="Liberando el Bloqueo"
export class YService{

  private readonly mu: Mutex = new Mutex()

  async functionConcurrente(){
    await this.mu.Lock();
    // Lógica crítica
    this.mu.Unlock(); // Destraba la función para la siguiente llamada.
    return;
  }

}
```

### Manejo de Errores con Mutex

**Es fundamental** que el `Unlock()` se ejecute incluso si ocurre un error, ya que de lo contrario, el Mutex quedará bloqueado permanentemente (*deadlock*).

**No usar** los decoradores `@CatchError` o `@CatchErrorListener` directamente sobre un método que use `Mutex`, ya que estos manejan el error antes de que se pueda liberar el Mutex, causando un bloqueo permanente.

```typescript title="🚫 Uso Incorrecto (Causa Deadlock)"
export class YService{

  private readonly mu: Mutex = new Mutex()

  @CatchError // 👈 Si el método fallase, este bloqueará permanentemente la función porque el Mutex no se libera.
  async functionConcurrente(){
    await this.mu.Lock()
    // Lógica que podría fallar
  }

}
```

La manera **correcta** de usarlo es mediante un bloque `try...catch...finally` o encapsulando la lógica interna dentro del `try...catch` si se desea utilizar un decorador de manejo de errores, asegurando que `Unlock()` se llame antes de propagar el error.

```typescript title="✅ Uso Correcto (Liberación y Manejo de Errores)"
export class YService{

  private readonly mu: Mutex = new Mutex()

  @CatchError
  async functionConcurrente(){
    await this.mu.Lock()
    try {
      // Lógica del componente
      // ...
    } catch (error) {
      this.mu.Unlock() // 👈 Primero libera el Mutex
      throw error // 👈 Luego propaga el error
    }
    this.mu.Unlock() // 👈 Liberación normal
  }

}
```