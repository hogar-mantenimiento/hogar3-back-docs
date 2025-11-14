---
sidebar_position: 4
---

# Entidades Genéricas 💾

Existe una serie de clases base que pueden utilizarse como entidades genéricas para facilitar la gestión de datos comunes en las entidades reales del proyecto.

---

## `Audit`

Esta clase permite integrar de forma cómoda y directa las columnas de auditoría estándar en cualquier entidad.

| Columna | Descripción | Comportamiento |
| :--- | :--- | :--- |
| **`createdAt`** | Marca de tiempo de la creación de la entidad. | Se genera automáticamente al guardar. |
| **`updatedAt`** | Marca de tiempo de la última modificación. | Se actualiza automáticamente en cada cambio. |
| **`deletedAt`** | Marca de tiempo de la eliminación. | Requiere que se use la opción **SoftDelete** al eliminar una entidad. |

### Uso

La entidad puede heredar de `Audit` mediante `extends` o incrustar las propiedades mediante el decorador `@Column`.

```typescript title="Incrustación de Columnas de Auditoría"
import { Audit } from './audit.entity';
import { Column, Entity } from 'typeorm';

// Opción 1: Extends
@Entity()
export class MiEntidad extends Audit {
  // ... otras columnas
}

// Opción 2: Incrustación (Recomendado si ya hay herencia)
@Entity()
export class MiOtraEntidad {
  @Column(() => Audit)
  audit: Audit;
  // ...
}
````

-----

## `Coords` y `CoordsSuc`

Estos son *classes* base diseñados para incrustar propiedades de coordenadas geográficas en las entidades.

| Entidad Base | Propiedades | Uso |
| :--- | :--- | :--- |
| **`Coords`** | `lat` (Latitud) y `lng` (Longitud). | Igual que el DTO, pero configurado para persistencia. |
| **`CoordsSuc`** | `lat`, `lng`, y `address` (Dirección). | Extiende a `Coords`, añadiendo la columna de dirección. |

### Uso

Ambos se usan con el decorador `@Column()` para incrustar sus propiedades dentro de la entidad:

```typescript title="Incrustación de Coordenadas"
import { Coords } from './coords.entity';
import { Column, Entity } from 'typeorm';

@Entity()
export class Ubicacion {
  // Las columnas lat y lng se crean directamente en esta entidad
  @Column(() => Coords)
  coordenadas: Coords;
  // ...
}
```

-----

## `TimestampFromEnum(enum, options?)`

Esta es una **función factoría** que genera dinámicamente propiedades de tipo `timestamp` en una entidad, basándose en los valores de un `enum` proporcionado.

### 1\. Uso Básico

Cada valor del `enum` se convierte en una columna de tipo `timestamp` en la entidad.

```typescript title="Definición del Enum"
export enum EstadosDeProceso {
  INICIADO = 'timestamp_inicio',
  EN_REVISION = 'timestamp_revision',
  FINALIZADO = 'timestamp_final',
}
```

```typescript title="Aplicación en la Entidad"
import { Entity } from 'typeorm';
import { TimestampFromEnum } from './timestamp-from-enum.util';

@Entity()
export class Proceso extends TimestampFromEnum(EstadosDeProceso) {}
// La entidad 'Proceso' ahora tendrá las columnas:
// timestamp_inicio, timestamp_revision, timestamp_final (todas como timestamps)
```

### 2\. Uso con Valor por Defecto (`default`)

El segundo argumento de la función permite especificar un valor por defecto para una de las columnas generadas.

```typescript title="Definiendo un Default"
@Entity()
export class ProcesoConDefault extends TimestampFromEnum(EstadosDeProceso, {
  // El nombre debe coincidir con un valor del enum
  value: 'timestamp_inicio', 
  // La función SQL que establece el valor por defecto
  valueToSet: () => "CURRENT_TIMESTAMP",
}) {}
// En este ejemplo, la columna `timestamp_inicio` tendrá por default la hora en que fue creado el registro.
```

```
```