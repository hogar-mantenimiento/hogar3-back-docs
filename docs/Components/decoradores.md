---
sidebar_position: 1
---

# Decoradores

Hay decoradores generales usables en todo el proyecto.

---

### `@CatchError`

Se utiliza en los **Services** del proyecto, envolviendo los métodos de la clase.

Este decorador permite encapsular el método decorado dentro de un bloque **`try...catch`** ya conectado con la función de manejo de errores (`handleError`) de la capa `common`. Esto evita la repetición manual del manejo de errores en cada método del servicio.

#### Uso

| Sin el decorador | Con el decorador |
| :--- | :--- |
| Se debe implementar un bloque `try...catch` en cada método que lo requiera. | El decorador se encarga de manejar el error automáticamente. |

**Ejemplo sin el decorador**

```typescript
export class XService {
  
  constructor(
    private readonly common: CommonService,
  ){}

  async methodFunction(){
    try {
      // Lógica de la función
      // ...
    } catch (error) {
      this.common.handleError(error) // Manejo manual del error
    }
  }
}
````

**Ejemplo con el decorador**

```typescript
export class XService {

  @CatchError
  async methodFunction(){
    // Lógica de la función (el manejo de errores es implícito)
    // ...
  }
}
```

#### Limitaciones

Este decorador **no funciona o no debe usarse** en los siguientes casos:

1.  **Listeners:** Cuando el método es un *listener* (ej. *event handlers* o *message consumers*).
2.  **Manejo de Response:** Cuando se requiere el objeto `Response` de **ExpressJS** para manipular directamente la respuesta HTTP.

Para casos donde se necesita el `Response`, existe una función en el *common* para ello.

-----

### `@CatchErrorListener`

Este decorador funciona igual que `@CatchError`, pero está **diseñado específicamente para los *listeners***.

Es menos común su uso en el proyecto, pero está disponible si se requiere manejar errores en métodos que actúan como *listeners* de eventos o mensajes.

-----

### `@SaveImage(path: string, options...)`

El decorador `@SaveImage` es una utilidad personalizada diseñada para simplificar la configuración de subida de archivos e imágenes en *endpoints* de NestJS. Combina múltiples interceptores y decoradores de Swagger, gestionando la lógica de **`multer`** para el guardado en disco y el manejo de imágenes.

#### 1\. Uso Básico (Una sola imagen)

Para guardar una sola imagen, aplica `@SaveImage` a tu método de controlador, especificando la ruta dentro de la carpeta `static` donde se guardará el archivo.

**Sintaxis:**

```typescript
@SaveImage(path: string)
```

**Ejemplo:**

```typescript title="users.controller.ts"
@Post('avatar')
@SaveImage('users/avatars') // La imagen se guarda en `./static/users/avatars`
async uploadAvatar(
  @UploadedFile() file: Express.Multer.File
) {
  // `file` contendrá la información del archivo guardado
  return { filename: file.filename, path: file.path };
}
```

> **Nota Importante:** El *endpoint* espera que el archivo se envíe en el campo **`file`** del formulario *multipart*.

-----

#### 2\. Uso con Múltiples Imágenes (Plural)

Para permitir la subida de varios archivos a la vez, se debe usar la opción `plural: true`. Opcionalmente, puedes limitar la cantidad con `pluralCount`.

**Sintaxis:**

```typescript
@SaveImage(path: string, { plural: true, pluralCount?: number })
```

**Ejemplo:**

```typescript title="products.controller.ts"
@Post(':id/images')
@SaveImage('products/gallery', { plural: true, pluralCount: 5 }) // Permite hasta 5 imágenes
async uploadImages(
  @UploadedFiles() files: Array<Express.Multer.File>
) {
  // `files` es un array con la información de los archivos guardados
  return files.map(f => f.filename);
}
```

> **Nota Importante:** Cuando `plural: true`, el *endpoint* espera que los archivos se envíen en el campo **`files`** del formulario *multipart*.

-----

#### 3\. Opciones Avanzadas (Combinando con DTOs y Swagger)

El decorador permite integrar un **Data Transfer Object (DTO)** para incluir datos adicionales del cuerpo de la solicitud (como descripciones) junto con el archivo en Swagger (`ApiBody`).

**Sintaxis Completa:**

```typescript
@SaveImage<T>(path: string, options?: {
  filter?: filterType;      // Función personalizada para filtrar archivos.
  doc?: Type<T>;            // DTO para incluir en el body de Swagger.
  plural?: boolean;
  pluralCount?: number;
})
```

**Ejemplo con DTO:**

```typescript title="products.controller.ts"
// DTO para datos adicionales
export class CreateProductImageDto {
  @ApiProperty()
  description: string;
}

@Post(':id/image-with-data')
@SaveImage<CreateProductImageDto>('products/single', { doc: CreateProductImageDto })
async uploadImageWithData(
  @Body() createDto: CreateProductImageDto,
  @UploadedFile() file: Express.Multer.File
) {
  // El DTO (createDto) es automáticamente parseado
  return { ...createDto, filename: file.filename };
}
```

### `@Auth()` y `@GetUser()`

Ambos decoradores se correlacionan, siendo `@GetUser()` dependiente de que el *endpoint* esté protegido por `@Auth()`.

#### `@Auth()`

Este decorador se usa directamente en el **controller** (a nivel de clase o de método), protegiendo los *endpoints* en base a la validación de un token y **permisos** asociados.

El @Auth ademas internamente verifica si el usuario tiene ese modulo en activo, bloqueandolo si no lo tiene activado como parte de su **SuperCliente o Profiler**

#### `@GetUser()`

Este es un decorador de parámetros que solo funciona en *endpoints* protegidos con `@Auth()`. Permite inyectar el objeto `user` (extraído del token validado por `@Auth()`) directamente en el método del controlador.

#### Ejemplo de Uso Combinado

```typescript title="auth.controller.ts"
@Get('me')
@Auth() // 1. Protege el endpoint y valida el token/permisos
getme(@GetUser() user: User){ // 2. Captura el objeto User ya validado
  return this.Service.MethodFunction(user)
}

```

esos serian todos los decoradores importantes, si me acuerdo de alguno mas, lo agregare
