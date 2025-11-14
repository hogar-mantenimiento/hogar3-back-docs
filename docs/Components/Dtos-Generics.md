---
sidebar_position: 3
---

# DTOs Genéricos 🧩

Hay presentes DTOs Genéricos diseñados para ser usados como base (`extends`) o simplemente como tipos para los DTOs específicos del proyecto.

---

### `Day`

Este DTO está diseñado para representar un día dentro de un mes.

| Campo | Tipo | Descripción | Ejemplo |
| :--- | :--- | :--- | :--- |
| **`day`** | `number` | El día del mes. Debe estar en el rango de **1 a 31**. | `15` |
| **`month`** | `number` | El mes al que pertenece. Sigue la convención del objeto `Date` nativo de JavaScript: **0 (Enero) a 11 (Diciembre)**. | `0` (Enero) |

> **Nota:** Aunque el usuario pueda enviar el mes en el rango **1-12**, se aplica un *transform* interno para ajustarlo al rango **0-11** de JavaScript antes de su uso.

---

### `Pagination`

Llamado internamente como `PaginationDto`, este DTO es la **base de todas las consultas de paginación** (`queries`) en la aplicación.

Este trae por defecto los siguientes atributos:

| Atributo | Tipo | Default | Descripción |
| :--- | :--- | :--- | :--- |
| **`limit`** | `number` | `10` | Establece la **cantidad máxima** de elementos que se traerán. Debe ser un número entero positivo. |
| **`offset`** | `number` | `0` | La cantidad de elementos que se **saltarán** (skip). Debe ser un número entero positivo (incluyendo el 0). |

**Ejemplo de uso (Extends):**

```typescript
// Define un DTO de paginación específico extendiendo el genérico
class GetUsersPaginationDto extends Pagination {
  @IsOptional()
  @IsString()
  search?: string;
}
````

-----

### `Coords`

Llamado internamente como `coordsDto`, se utiliza para definir una ubicación geográfica precisa.

| Atributo | Tipo | Descripción |
| :--- | :--- | :--- |
| **`lat`** | `number` | Latitud de la ubicación. |
| **`lng`** | `number` | Longitud de la ubicación. |

-----

### `Coords con dirección`

Llamado internamente como `coordsSucDto`, este DTO extiende a **`Coords`** para añadir información de dirección.

| Atributo | Tipo | Heredado de | Descripción |
| :--- | :--- | :--- | :--- |
| **`lat`** | `number` | `Coords` | Latitud de la ubicación. |
| **`lng`** | `number` | `Coords` | Longitud de la ubicación. |
| **`address`** | `string` | N/A | La dirección física asociada a las coordenadas. |

**Ejemplo de uso (Extends):**

```typescript
// El DTO ya incluye lat y lng
class SucursalLocationDto extends CoordsConDireccion {
  @IsUUID()
  sucursalId: string;
}
```
