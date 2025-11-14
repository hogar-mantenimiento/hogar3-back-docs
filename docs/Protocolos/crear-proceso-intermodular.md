---
sidebar_position: 2
---

# Crear Proceso Intermodular

El diseño semi-modular del *backend* puede colisionar con la necesidad de implementar funcionalidades (*features*) que requieran la cooperación y datos de **más de un módulo** simultáneamente (más allá de una simple relación de base de datos). Para comunicar una tarea entre módulos de manera segura y **asíncrona**, y poder devolver una respuesta consolidada al cliente, se utiliza el **`ListenerBetweenModules`**, conocido informalmente como **"globito"**.

Este mecanismo tiene la capacidad de viajar entre módulos sin romper el diseño modular del sistema.

---

### Paso 1: Analizar el Objetivo y Crear Listeners

Lo primero es definir todos los módulos y tareas que serán parte de la *feature*.

En los *listeners* de los módulos involucrados (o en archivos *service* que los contengan), se deben crear funciones marcadas con el decorador **`@OnEvent()`** de NestJS.

Cada *listener* debe aceptar un único parámetro tipado como **`ListenerBetweenModules<T>`**, donde `T` es el genérico que representa la estructura de datos que viaja. Este genérico (`T`) debe definirse en el módulo padre que inicia el proceso o en `common`.

Un punto crucial es que, al finalizar su lógica, cada *listener* **debe llamar** a `payload.next()` para avanzar a la siguiente etapa del proceso.

La información que se desea pasar entre las etapas debe guardarse dentro del campo **`.data`** del *payload* (`globito`).

#### Ejemplo de Listener

```typescript
export class XListenerModule{

  @OnEvent("proceso.por.aca.pasa")
  async functionListener<T>(payload: ListenerBetweenModules<T>){
    // Lógica interna que manipula payload.data
    // ...
    payload.next() // Llama a la siguiente acción en la cola
  }

}
````

#### Manejo de Excepciones

Para las excepciones, el `payload` trae un *handler* de errores integrado (`handleError`).

```typescript
export class XListenerModule{

  @OnEvent("proceso.por.aca.pasa")
  async functionListener<T>(payload: ListenerBetweenModules<T>){
    try {
      // lógica interna
      payload.next()
    } catch (error) {
      payload.handleError(error) // Maneja y finaliza el proceso con el error
    }
  }

}
```

> **Nota:** Este `handleError` interno utiliza el método `CommonService->HandleErrorManual`, por lo que sus respuestas son los estándares definidos en el *handleError* de la aplicación.

-----

### Paso 2: Crear el Proceso (`CreateQueueEvent`)

El servicio que dispare el proceso debe considerar:

1.  **Response:** El método debe recibir un objeto **`Express.Response`** (generalmente como parámetro) para manejar manualmente las respuestas del *endpoint*, ya que el "globito" lo requiere para su respuesta final.
2.  **Creación:** El proceso se crea utilizando la función `CreateQueueEvent()` proveniente de `CommonModule`.

Esta función requiere un objeto de configuración con los siguientes campos:

#### `data: T`

El objeto base que contiene la **memoria** de lo que requiere el *globito* en todas sus instancias. También es el tipado final que se espera en la respuesta.

#### `res: Express.Response`

El objeto **`Express.Response`** que será usado para enviar la respuesta HTTP final al cliente.

#### `returnEvent: (payload) => void`

La función final que debe ser la encargada de hacer el `return` utilizando el objeto `res` para enviar la respuesta consolidada del proceso (`payload.data`). Puede ser una **función flecha** o un **listener**, según convenga.

#### `acts: (string | Function)[]`

La parte más importante del *globito*. Es un *array* de **strings** o **funciones flecha** que se ejecutarán en orden (de arriba a abajo):

| Tipo de Acción | Descripción |
| :--- | :--- |
| **`string`** | Se utiliza para llamar a los *listeners* creados en el Paso 1 (mediante el *string* del evento **`@OnEvent()`**). |
| **`Function (flecha)`** | Simplifican el trabajo. Su fin es realizar tareas en el proceso que no requieran estar en otros módulos. **Mantienen el contexto y las referencias** en memoria del método que creó el `QueueEvent`. |

-----

### Paso 3: Ejecutar

Una vez creado el flujo en una constante, se debe usar el método `.next()` para iniciar el procesamiento.

#### Ejemplo de Uso Completo

```typescript title="O.service.ts"
import { Response } from 'express';

export class OService {

  constructor(
    private readonly common: CommonService,
  ){}

  FeatureMultiModule(res: Response){
    const QueueEvent = this.common.CreateQueueEvent({
      data: {
        // información inicial del data (memoria)
      },
      returnEvent: (payload) => {
        // Envía la respuesta final al cliente
        payload.res.status(200).send(payload.data)
      },
      res,
      acts: [
        "listener.primer.paso", // Llama a un listener en otro módulo
        (payload) => { // Lógica local (por ejemplo, validación)
          try {
            // lógica local
            payload.data.resultadoLocal = true;
            payload.next()
          } catch (error) {
            payload.handleError(error)
          }
        },
        "listener.paso.final", // Llama al último listener
      ],
    })
    
    // Inicia el flujo del proceso
    QueueEvent.next()
  }

}
```