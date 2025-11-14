---
sidebar_position: 5
---

# Funciones Globales 🌍

Existen métodos de utilidad importables mediante la inyección de servicios que brindan funcionalidades comunes y estandarizadas en toda la aplicación.

Estos métodos provienen en su mayoría del módulo `Common` y se acceden a través de su service principal, `CommonService`.

---

## `CommonService`

### Manejo de Errores

| Método | Retorno | Descripción |
| :--- | :--- | :--- |
| **`GenHandleError(error: any)`** | `[statuscode: number, message: any]` | Devuelve un formato de error genérico (`[código de estado, mensaje]`) para constructos de manejo de errores. Es usado principalmente por clases internas. |
| **`handleError(error: any)`** | `void` | El manejador de errores por defecto y original. Aunque existió, actualmente su funcionalidad está abstraída dentro del decorador **`@CatchError`**. |
| **`handleErrorManual(error: any, res: Express.Response)`** | `void` | Manejador específico para situaciones donde **no se puede usar** el decorador `@CatchError` (típicamente en funciones que requieren el objeto `Response` de **Express**). |

#### Ejemplo de `handleErrorManual`

```typescript title="Uso con Response de Express"
import { Response } from 'express';
import { CommonService } from './common.service';

class ZService(){

    constructor(
        private readonly common: CommonService,
    ){}

    metodoQueUsaResponse(res: Response){
        try{
            // lógica del método
        }
        catch (error){
            this.common.handleErrorManual(error, res) // Maneja y envía la respuesta de error
        }
    }
}
````

-----

### Métodos Generales para el Tiempo ⏱️

| Método | Retorno | Descripción |
| :--- | :--- | :--- |
| **`formatMIlliesToHHMM(ms: number)`** | `string` | Convierte una cantidad de tiempo en milisegundos (`ms`) al formato `HH:MM` para una expresión más legible del tiempo. |
| **`currentDayMMDD()`** | `string` | Devuelve la fecha actual sin el año con el formato `MM-DD` (Mes-Día). |

-----

### Métodos para Archivos e Imágenes 🖼️

| Método | Retorno | Descripción |
| :--- | :--- | :--- |
| **`GenPathImage(file: Express.Multer.File)`** | `string` | Devuelve la dirección real en disco de un archivo subido, generalmente con el decorador **`@SaveImage`**. Pensado para ser una utilidad universal. |
| **`ServeImage(filePath: string, res: Express.Response, fileActions?: FileActions)`** | `void` | Funciona en complemento con `GenPathImage`. Sirve para **devolver** al cliente un archivo particular. Aunque tienen "Image" en el nombre, funcionan para **cualquier tipo de archivo**. |

El parámetro `fileActions` es un `enum` con los valores `download` y `load`. Por defecto, `ServeImage` usa `load` (mostrar en el navegador).

-----

### Utilidades Varias

| Método | Retorno | Descripción |
| :--- | :--- | :--- |
| **`Logger(message: string)`** | `void` | Simplemente un *logger* que imprime el mensaje que se le envía. |
| **`validateRole(user: User, ...validRoles: Permisos[])`** | `boolean` | Valida si el `user` tiene alguno de los permisos (`Permisos[]`) indicados o si es **Admin**. Devuelve `true` si cumple, `false` en caso contrario. |
| **`normalizarTexto(str: string)`** | `string` | Formatea el *string* de entrada para usar el alfabeto latino estándar (elimina diacríticos, acentos y caracteres no romanos). |
| **`validatorType(type: "string" \| "int" \| "float" \| "date" \| "boolean", value: any)`** | `boolean` | Devuelve si el `value` proporcionado es del `type` especificado. |
| **`normalizarTexto(type: "string" \| "int" \| "float" \| "date" \| "boolean", value: any)`** | `any` | **Convierte** el `value` al `type` indicado. |
| **`async sleep(seconds: number)`** | `Promise<void>` | "Duerme" o pausa la ejecución de una función asíncrona durante la cantidad de `seconds` especificada. |

-----

### Métodos Obsoletos (Deprecated)

| Método | Descripción |
| :--- | :--- |
| **`catchError(promise: Promise)`** | Función antigua que abstraía el bloque `try/catch` dentro de una promesa. **No se recomienda su uso.** a favor de usar `@CatchError` |

## `Color Helper`

A través del servicio **`ColorHelperSErvice`**, se accede a la función para generar colores personalizados.

### createColor(text: string): string

Devuelve un color *custom* (generalmente un *hash* de color) basado en el contenido del `text` de entrada, asegurando que el color generado sea consistente para el mismo texto.

---

## `Gemini Service`

Este es un servicio interno para la comunicación con la plataforma **GEMINI** (Google AI).

### Métodos de Generación

| Método | Argumentos | Descripción |
| :--- | :--- | :--- |
| **`async generateFromData(promptType: TextPromptType, data: any)`** | `promptType`, `data` | A partir de un tipo de *prompt* precargado y los datos que le llegan, consulta a Gemini sobre el tema y devuelve una respuesta en formato *string*. |
| **`async generateFromImage(promptType: TextPromptType, image)`** | `promptType`, `image` | Se le envía una imagen a Gemini, usando un *prompt* específico, y este devuelve el resultado de su análisis de la imagen. |


