# Diccionario de Datos  
## Sistema de Gestión de Conciertos
**Entorno:** PostgreSQL 18  
**Versión:** 1.0    
**Año:** 2026    
**Mes:** Abril  
**Responsable:** Grupo 2 (Mateo Salcedo, Tobias Centurion, Camila Tenorio)

# 1. Descripción del modelo

La base de datos...

# 2. Información General

# 3. Dominios Definidos

## 3.1. dom_cuit
- Tipo base: `VARCHAR(13)`
- Permite valores que cumplan la expresión regular: `^[0-9]{2}-[0-9]{8}-[0-9]$`
```sql
CONSTRAINT chk_dom_cuit CHECK (VALUE ~ '^[0-9]{2}-[0-9]{8}-[0-9]$');
```

## 3.2. dom_email
- Tipo base: `VARCHAR(255)`
- Permite valores que cumplan la expresión regular: `^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$`
```sql
CONSTRAINT chk_dom_email CHECK (VALUE ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

## 3.3. rol_dom
- Tipo base: `VARCHAR(10)`
- Valores permitidos:
  - PRINCIPAL
  - INVITADO
```sql
CHECK (VALUE IN ('PRINCIPAL', 'INVITADO'));
```

# 4. Definición de Tablas

---

## Tabla: artistas

**Tipo:** Dato maestro

| Campo          | Tipo         | Nulo |PK |UK |FK | Descripción |
|----------------|--------------|------|---|---|---|-------------|          
| id_artista     | SERIAL       | No   | ✔ | - | - |Identificador único|    
| nombre_artista | VARCHAR(255) | No   | - | ✔ | - |Nombre único|
| email_contacto |dom_email     | No   | - | ✔ | - |Email de contacto único|
| cuit           |dom_cuit      | No   | - | ✔ | - |CUIT único|
| genero         |INTEGER       | No   | - | - | ✔ |FK a generos|

```sql
CONSTRAINT fk_genero_artista FOREIGN KEY (genero) REFERENCES generos(id_genero) ON UPDATE NO ACTION ON DELETE NO ACTION;
```

### Reglas de Validación — Artistas

#### Campo: `nombre_artista`
**Descripción:**  
Nombre del artista (solista o banda).

**Tipo de dato:** `VARCHAR(255)`

**Obligatoriedad:**
Requerido (`NOT NULL`)

**Unicidad:** 
Debe ser único dentro de la tabla (`UNIQUE`)
```sql
CONSTRAINT ck_artistas_nombre UNIQUE (nombre_artista)
```

**Normalización:** 
El valor debe almacenarse:
- Respetando la forma original del nombre artístico.
- Sin espacios al inicio ni al final.
```sql
TRIM(nombre_artista)
``` 


#### Campo: `email_contacto`
**Descripción:**  
Dirección de correo electrónico de contacto del artista.

**Tipo de dato:** `dom_email`

**Obligatoriedad:**
Requerido (`NOT NULL`)

**Unicidad:** 
Debe ser único dentro de la tabla (`UNIQUE`)
```sql
CONSTRAINT ck_artistas_email UNIQUE (email_contacto)
```

**Normalización:**  
El valor debe almacenarse:
- Sin espacios al inicio ni al final.
- Sin espacios internos.
- Convertido a minúsculas.
```sql
LOWER(TRIM(email_contacto))
```
**Validación:**
El valor se valida mediante el dominio `dom_email`.
Debe corresponder a una dirección de correo electrónico válida según el formato general:
- Debe contener exactamente un carácter @.
- Debe existir una parte local antes de @.
- Debe existir un dominio válido después de @.
- La extensión del dominio debe tener al menos 2 caracteres.
- No debe permitir espacios.

**Formato requerido:**
```
local_part@dominio.ext
```

Expresión regular:
```regex
^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$
```

Ejemplos válidos:
| Valor ingresado             | Valor almacenado            |
| --------------------------- | --------------------------- |
| `juan.perez@example.com`    | `juan.perez@example.com`    |
| `Maria_Gomez99@empresa.com` | `maria_gomez99@empresa.com` |
| `USER+TEST@GMAIL.COM`       | `user+test@gmail.com`       |

Ejemplos inválidos:
| Valor ingresado       | Motivo                        |
| ----------------------| ------------------------------|
| `juan.perez`          |Falta `@` y dominio            |
| `juan@`               | Falta dominio                 |
| `@empresa.com`        | Falta parte local             |
| `juan @empresa.com`   | Contiene espacios             |
| `juan@empresa`        | Falta extensión del dominio   |
| `juan@empresa.c`      | Extensión demasiado corta     |
| `juan@@empresa.com`   | Contiene más de un `@`        |


#### Campo: `cuit`

**Descripción:**  
CUIT (Clave Única de Identificación Tributaria) del artista.

**Tipo de dato:** `dom_cuit`

**Obligatoriedad:**
Requerido (`NOT NULL`)

**Unicidad:** 
Debe ser único dentro de la tabla (`UNIQUE`)
```sql
CONSTRAINT ck_artistas_cuit UNIQUE (cuit)
```

**Validación:**
El valor se valida mediante el dominio `dom_cuit`.
Debe corresponder a un CUIT válido según el formato oficial definido por ARCA:
- Debe contener 11 dígitos numéricos, separados por `-` en tres partes:
  - Un prefijo de 2 dígitos.
  - 8 dígitos correspondientes al DNI o número de sociedad.
  - 1 dígito verificador.
- No debe permitir espacios ni caracteres adicionales.

**Formato requerido:**
```
XX-XXXXXXXX-X
```

Expresión regular:
```regex
^[0-9]{2}-[0-9]{8}-[0-9]$
```

Ejemplos válidos:
| Valor ingresado |
| ----------------|
| `20-12345678-3` |
| `30-82416434-2` | 

Ejemplos inválidos:
| Valor ingresado       | Motivo                                |
| ----------------------| --------------------------------------|
| `20123456783`         | Faltan separadores (`-`)              |
| `20-1234567-3`        | Cantidad incorrecta de dígitos        |
| `AA-12345678-3`       | Contiene caracteres no numéricos      |
| `20_12345678_3`       | Separadores incorrectos (`_`)         |
| `20-12345678`         | Falta separador (`-`) y último dígito |
| `20-12345678-3 `      | Contiene espacios                     |

---
## Tabla: participan

**Tipo:** ??
**Descripción:** Tabla intermedia que representa la relación muchos a muchos entre artistas y conciertos, indicando el rol del artista en el concierto.

| Campo        | Tipo    | Nulo |PK |UK   |FK | Descripción |
|--------------|---------|------|---|-----|---|-------------|          
| id_participa | SERIAL  | No   | ✔ | -  | - | Identificador único|    
| artista      | INTEGER | No   | - | ✔* | ✔ | FK a artistas     |
| concierto    | INTEGER | No   | - | ✔* | ✔ | FK a conciertos   |   
| rol          | rol_dom | No   | - | -  | - | Rol del artista en el concierto|

```sql
CONSTRAINT ck_participan UNIQUE (artista,concierto); --CK compuesta
CONSTRAINT fk_participan_artista FOREIGN KEY (artista) REFERENCES artistas(id_artista) ON UPDATE NO ACTION ON DELETE NO ACTION;
CONSTRAINT fk_participan_concierto FOREIGN KEY (concierto) REFERENCES conciertos(id_concierto) ON UPDATE NO ACTION ON DELETE NO ACTION;
```
### Reglas de Validación — Participan

#### Campo: `rol`

**Descripción:**  
Rol del artista referenciado en el concierto referenciado.

**Tipo de dato:** `rol_dom`

**Obligatoriedad:**
Requerido (`NOT NULL`)

**Validación:**  
El valor se valida mediante el dominio `rol_dom`.

Valores permitidos:
- `PRINCIPAL`
- `INVITADO`

---

# 5. Reglas de Calidad de Datos

# 6. Clasificación de Datos

| Tabla        | Clasificación sugerida     | Comentario    |
| -------------| ---------------------------| --------------|
| `conciertos` |                            |               |
| `artistas`   |                            |               |
| `lugares`    |                            |               |
| `participan` |                            |               |


# 7. Diagrama Entidad–Relación (Mermaid)

```mermaid
erDiagram

CONCIERTOS {
    serial id_concierto PK
    text nombre_concierto 
    text descripcion
    date fecha
    time hora_inicio
    time hora_fin
    integer capacidad_vendida
    integer lugar FK
}

ARTISTAS {
    serial id_artista PK
    varchar(255) nombre_artista
    dom_email[varchar(255)] email_contacto
    dom_cuit[varchar(13)] cuit
    integer genero FK
}

PARTICIPAN {
    serial id_participa PK
    integer artista FK
    integer concierto FK
    rol_dom[varchar(10)] rol
}

LUGARES {
    serial id_lugar PK
    varchar(255) nombre_lugar
    dom_email[varchar(255)] email_contacto
    varchar(255) direccion
    integer capacidad_total
    integer ciudad FK 
}

CONCIERTOS }o--|| LUGARES : se_realiza_en
ARTISTAS ||--o{ PARTICIPAN : participa
CONCIERTOS ||--|{ PARTICIPAN : tiene_la_participacion

```
