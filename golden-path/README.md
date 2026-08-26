# Fase 2: Construcción del Golden Path

El objetivo es investigar la anatomía de los Software Templates de Backstage y construir tu propio Golden Path para crear microservicios NodeJS de forma automatizada.

## ¿Qué es un Golden Path?

En el contexto de Plataforma de Ingeniería, un **Golden Path** es la ruta pre-aprobada que la plataforma ofrece a los equipos de desarrollo para realizar una tarea de forma segura, eficiente y consistente.

Un desarrollador no necesita saber cómo configurar un repositorio, un pipeline de CI/CD, o un manifiesto de Kubernetes: simplemente llena un formulario y obtiene un servicio listo para producción.

En Backstage, los Golden Paths se implementan mediante **Software Templates**, definidas en archivos `template.yaml`.

---

## Investigación

Antes de escribir código, debes investigar la documentación oficial de Backstage. Esta sección contiene **preguntas guía** que debes responder como parte de esta actividad.

### Recursos clave para investigar

| Recurso | URL |
|---|---|
| Documentación de Software Templates | https://backstage.io/docs/features/software-templates/ |
| Referencia de `template.yaml` (fields) | https://backstage.io/docs/features/software-templates/writing-templates |
| Built-in Actions del Scaffolder | https://backstage.io/docs/features/software-templates/builtin-actions |
| Referencia de Nunjucks (motor de plantillas) | https://mozilla.github.io/nunjucks/templating.html |
| Publicar en GitHub con Backstage | https://backstage.io/docs/features/software-templates/builtin-actions#publish-github |

### 1. Anatomía de un `template.yaml`

Un `template.yaml` es un archivo YAML que Backstage interpreta para renderizar un formulario y orquestar acciones.

#### 1.1 Estructura general

Investiga y responde: ¿Cuáles son las **4 secciones principales** de un `template.yaml`? Describe brevemente el propósito de cada una.

| Sección | Propósito |
|---|---|
| `apiVersion` | Define la versión de la API del Scaffolder utilizada por el template. Normalmente se utiliza `scaffolder.backstage.io/v1beta3`. |
| `kind` | Define el tipo de entidad. Para un Software Template el valor utilizado es `Template`. |
| `metadata` | Contiene la información que identifica y describe el template, como `name`, `title`, `description`, `tags` y `annotations`. |
| `spec` | Define el comportamiento del template, incluyendo propietario, tipo, parámetros de entrada, pasos que ejecutará el Scaffolder y outputs. |


#### 1.2 Metadata del Template

Investiga los campos del bloque `metadata`. ¿Qué es el campo `annotations` y para qué sirve en el contexto de los templates?

El campo `annotations` permite agregar metadatos adicionales en formato clave-valor a una entidad de Backstage. Estos metadatos pueden ser utilizados por Backstage, plugins o integraciones externas para asociar información adicional al template.

Por ejemplo:

```yaml
metadata:
  name: nodejs-microservice
  title: NodeJS Microservice
  description: Golden Path para crear microservicios NodeJS
  annotations:
    backstage.io/techdocs-ref: dir:.
```

En este ejemplo, `backstage.io/techdocs-ref` indica a TechDocs dónde encontrar la documentación asociada al componente.


### 2. Parámetros de Entrada (`parameters`)

La sección `parameters` define el formulario que verá el desarrollador. Es el contrato de tu API de Plataforma.

#### 2.1 Tipos de campos (`ui:widget`)

Backstage usa JSON Schema para definir los campos del formulario. Investiga y lista al menos **5 tipos de campos** disponibles y cuándo usarías cada uno.

| Widget (`ui:widget`) | Tipo de dato | Caso de uso |
|---|---|---|
| `text` | `string` | Nombre del servicio |
| `textarea` | `string` | Descripción larga del microservicio |
| `password` | `string` | Ingresar información que no debe mostrarse directamente en pantalla |
| `select` | `string` | Seleccionar una opción de una lista |
| `radio` | `string` | Seleccionar una única opción entre varias alternativas |
| `checkboxes` | `array` | Seleccionar múltiples opciones |

Backstage también proporciona campos especializados mediante `ui:field`, como `OwnerPicker`, `RepoUrlPicker` y `EntityPicker`.


#### 2.2 Validaciones

¿Cómo puedes hacer que un campo de tipo `string` solo acepte valores en minúsculas sin espacios (útil para nombres de servicios)? Menciona las propiedades de JSON Schema relevantes.

Se puede utilizar la propiedad `pattern` de JSON Schema para definir una expresión regular. También se pueden utilizar propiedades como `minLength` y `maxLength` para controlar la longitud del nombre.

```yaml
serviceName:
  title: Nombre del servicio
  type: string
  description: Nombre del microservicio en minúsculas y sin espacios
  minLength: 3
  maxLength: 50
  pattern: '^[a-z0-9-]+$'
```

La expresión regular:

```text
^[a-z0-9-]+$
```

permite letras minúsculas, números y guiones, evitando espacios y letras mayúsculas.

Ejemplos válidos:

```text
payments-api
users
orders-service
service123
```

Ejemplos no válidos:

```text
Payments-api
payments api
Payments API
```

### 3. Pasos de Orquestación (`steps`)

La sección `steps` define la secuencia de acciones que el scaffolder ejecutará automáticamente.

#### 3.1 Actions disponibles

Investiga las **Built-in Actions** del scaffolder de Backstage. Completa la siguiente tabla con las acciones que necesitarás para tu Golden Path:

| Action ID | ¿Qué hace? | Inputs principales |
|---|---|---|
| `fetch:template` | Obtiene el skeleton del proyecto y procesa sus archivos como templates, reemplazando variables con los valores ingresados por el desarrollador. | `url`, `values`, `targetPath` |
| `publish:github` | Publica el contenido generado por el Scaffolder en un repositorio de GitHub. | `repoUrl`, `description`, `defaultBranch` |
| `catalog:register` | Registra el componente generado en el Software Catalog de Backstage utilizando el archivo `catalog-info.yaml`. | `repoContentsUrl`, `catalogInfoPath` |


#### 3.2 Motor de Plantillas (Nunjucks)

El paso `fetch:template` usa **Nunjucks** para reemplazar variables en los archivos del skeleton. ¿Cómo accedes al valor de un parámetro llamado `serviceName` dentro de un archivo del skeleton?

Primero, el parámetro recibido por el Software Template se envía al skeleton mediante `values`:

```yaml
values:
  serviceName: ${{ parameters.serviceName }}
```

Posteriormente, dentro de los archivos del directorio `skeleton/`, se accede al valor mediante:

```text
${{ values.serviceName }}
```

Por ejemplo, el archivo `package.json` podría contener:

```json
{
  "name": "${{ values.serviceName }}",
  "version": "1.0.0"
}
```

Si el desarrollador introduce:

```text
serviceName = payments-api
```

Backstage generará:

```json
{
  "name": "payments-api",
  "version": "1.0.0"
}
```

#### 3.3 Output del paso `publish:github`

La acción `publish:github` retorna un output que puedes usar en pasos siguientes. ¿Qué propiedad del output contiene la URL del repositorio recién creado? ¿Cómo se referencia en el siguiente paso?

La propiedad `remoteUrl` contiene la URL del repositorio generado.

Si el paso tiene el identificador `publish`, se referencia mediante:

```text
${{ steps.publish.output.remoteUrl }}
```

La acción también genera `repoContentsUrl`, que puede ser utilizada por `catalog:register` para localizar el contenido del repositorio.

Ejemplo:

```yaml
- id: register
  name: Register in Catalog
  action: catalog:register
  input:
    repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
    catalogInfoPath: /catalog-info.yaml
```

La sintaxis general para acceder al resultado de un paso anterior es:

```text
${{ steps.<id-del-step>.output.<propiedad> }}
```


### 4. Outputs del Template

La sección `output` del template permite mostrarle al usuario información al final del proceso (links, instrucciones, etc.).

¿Qué tipos de `links` puedes mostrar en el output? Lista al menos 2 ejemplos concretos para nuestro caso de uso (link al repo, link al componente en el catálogo).

Para este Golden Path se pueden mostrar:

1. Un enlace al repositorio creado en GitHub.
2. Un enlace al componente registrado en el Software Catalog de Backstage.

Ejemplo:

```yaml
output:
  links:
    - title: Repository
      url: ${{ steps.publish.output.remoteUrl }}

    - title: Open in Catalog
      icon: catalog
      entityRef: ${{ steps.register.output.entityRef }}
```

El primer enlace utiliza `remoteUrl`, generado por `publish:github`.

El segundo utiliza `entityRef`, generado por `catalog:register`, para abrir directamente el componente registrado en Backstage.


### 5. El Skeleton del Microservicio

El directorio `skeleton/` contiene la estructura base del proyecto que se copiará como punto de partida. Investiga:

#### 5.1 Parametrización del skeleton

El archivo `skeleton/catalog-info.yaml` también debe parametrizarse. Investiga la estructura del `catalog-info.yaml` de Backstage y responde: ¿Qué campos mínimos son obligatorios?

Para definir un componente en el Software Catalog se utilizan los siguientes campos:

- `apiVersion`: versión de la API utilizada por las entidades del catálogo.
- `kind`: tipo de entidad. En este caso será `Component`.
- `metadata.name`: nombre del componente.
- `spec.type`: tipo de componente, en este caso `service`.
- `spec.lifecycle`: etapa del ciclo de vida del componente.
- `spec.owner`: usuario o grupo responsable del componente.

Ejemplo de `skeleton/catalog-info.yaml`:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component

metadata:
  name: ${{ values.serviceName }}

spec:
  type: service
  lifecycle: experimental
  owner: ${{ values.owner }}
```

Los valores:

```text
${{ values.serviceName }}
${{ values.owner }}
```

serán reemplazados por `fetch:template` utilizando la información ingresada por el desarrollador en el formulario.

---

## Objetivo: Construir tu Golden Path

Con la investigación completa, tienes todo lo necesario para construir tu Software Template. Tu template debe cumplir los siguientes requisitos:

### Contrato de la API (Parámetros)

El formulario que verá el desarrollador debe pedir **exactamente 2 campos**:

| Campo | Tipo | Descripción |
|---|---|---|
| `serviceName` | `string` | Nombre del microservicio (minúsculas, sin espacios) |
| `owner` | `string` | Propietario del componente (user o group del catálogo) |

### Acciones Orquestadas (Steps)

El template debe ejecutar **exactamente 4 pasos** en este orden:

```
Paso 1: fetch:template
        └── Descarga el skeleton de este repositorio e inyecta los parámetros

Paso 2: publish:github
        └── Crea un nuevo repositorio en tu cuenta personal de GitHub
            y hace push del código generado

Paso 3: catalog:register
        └── Registra el nuevo componente en el Software Catalog de Backstage

(Opcional) Paso 4: Output
        └── Muestra al usuario el link al repo y al componente en el catálogo
```

---

## Cómo cargar tu template en Backstage

Una vez que tengas tu `template.yaml` completo y subido a GitHub:

1. En Backstage, ve a **Settings → Catalog → Add Existing Component**
2. Introduce la URL raw de tu `template.yaml` en GitHub, por ejemplo:
   ```
   https://github.com/tu-usuario/tu-repo/blob/main/golden-path/template.yaml
   ```
3. Backstage descargará el template y lo mostrará en la sección **Create** del portal.
4. Prueba tu Golden Path creando un nuevo microservicio de prueba.
