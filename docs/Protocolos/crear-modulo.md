---
sidebar_position: 1
---

# Crear un Módulo

La creación de un módulo conlleva varios pasos que involucran el uso de decoradores customizados, los cuales son parte del flujo de creación de un módulo y no fueron listados en la sección de Decoradores generales.

---

### Paso 1: Crear el Módulo Base

La parte más sencilla del proceso. Se crea primero un módulo nativo de NestJS utilizando el comando CLI:

```bash
nest g res modulo-name
````

-----

### Paso 2: Setear los Permisos (`@PermissionsRef`)

Es necesario asignar qué **permisos** serán visibles y administrables para el administrador del *profiler* para que el módulo pueda ser utilizado.

Esto se logra usando el decorador **`@PermissionsRef()`**, el cual se aplica a la clase principal del módulo. Dentro de `@PermissionsRef`, se listan todos los permisos asociados. Si se necesitan más permisos, deben agregarse al *enum* `common/interfaces/permisos.ts`.

#### Ejemplo

```typescript
import { Permisos } from 'common/interfaces/permisos.ts';

@PermissionRef(
  Permisos.USUARIO_LISTAR,
  Permisos.USUARIO_ELIMINAR,
  Permisos.USUARIO_EDITAR,
)
@Module({
    // configuración del módulo
})
export class ModuloCreado {}
```

-----

### Paso 3: Definir Contexto de los Controllers (`@ModuleContext`)

Se agrega el decorador **`@ModuleContext()`** al o los *controllers* del módulo.

Este decorador es crucial, ya que define **a qué módulo pertenece** el *controller*.

Si **`@ModuleContext`** está presente, este **bloquea los *endpoints*** según si el usuario tiene o no ese módulo activo en su *profiler*. Internamente, `@Auth()` se encarga de validar este contexto. Si `@ModuleContext` no se encuentra, el módulo se considera activo por defecto para todos los usuarios.

`@ModuleContext()` acepta dos modos de usarse:

| Modo de Uso | Descripción | Recomendación |
| :--- | :--- | :--- |
| **`@ModuleContext(str: string)`** | Un *string* que establece el nombre del módulo. Se recomienda usar una **constante global** para evitar errores tipográficos (ej. `AUTH_MOD_FLAG`). | **Recomendado** |
| **`@ModuleContext(() => Module)`** | Se le carga el módulo padre. Este método puede fallar a veces al no encontrar la referencia directa. | **Evitar** |

#### Ejemplo

```typescript title="users.controller.ts"
import { AUTH_MOD_FLAG } from 'common/constants';

@ModuleContext(AUTH_MOD_FLAG)
@Controller('users')
export class UsersController {
    // ...
}
```

-----

### Paso 4: Cargar a Historial (`@MetadataEntityUse`)

**(Opcional y sujeto a posible eliminación)**

Se puede registrar en un historial los cambios que suceden a una entidad concreta.

Esto se logra agregando el decorador **`@MetadataEntityUse(Entity)`** al *controller*. `Entity` debe ser la entidad principal que se maneja desde ese *controller* en concreto.

#### Ejemplo

```typescript title="products.controller.ts"
import { ProductEntity } from 'products/entities/product.entity';

@MetadataEntityUse(ProductEntity)
@Controller('products')
export class ProductsController {
    // ...
}
```

```
```