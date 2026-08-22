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

```
| Sección       | Propósito                                                                             |
|---------------|---------------------------------------------------------------------------------------|
| apiVersion    | Indica qué versión del esquema/API se está usando                                     |
| kind          | Indica qué tipo de objeto se está definiendo (Template, API, Component, etc)          |
| metadata      | Identifica y describe la plantilla (título, nombre, descripción)                      |
| spec          | Contiene el comportamiento/configuración de la plantilla (parameters, steps, outputs) |
```

#### 1.2 Metadata del Template

Investiga los campos del bloque `metadata`. ¿Qué es el campo `annotations` y para qué sirve en el contexto de los templates?

```
name: identificador único del template.
title: nombre que verá el usuario.
description: explicación de qué hace.
tags: categorías para facilitar búsquedas.
annotations: metadatos adicionales.
```

Adicionalmente, el campo `annotations` permite agregar metadatos técnicos que otros plugins o componentes de Backstage pueden interpretar. Por ejemplo:

```
metadata:
  name: payment-service
  description: Payment processing service
  annotations:
    github.com/project-slug: my-org/payment-service
```
Esencialmente, las annotations son pares llave-valor para descubrir información adicional. En el ejemplo anterior, con la anotación de github Backstage puede utilizar esa información para mostrar información relacionada como pull request, documentación, etc, por medio de la generación de un `catalog-info.yaml`.

### 2. Parámetros de Entrada (`parameters`)

La sección `parameters` define el formulario que verá el desarrollador. Es el contrato de tu API de Plataforma.

#### 2.1 Tipos de campos (`ui:widget`)

Backstage usa JSON Schema para definir los campos del formulario. Investiga y lista al menos **5 tipos de campos** disponibles y cuándo usarías cada uno.

```
| Widget (`ui:widget`)  | Tipo de dato  | Caso de uso                                                                   |
| --------------------- | ------------- | ----------------------------------------------------------------------------- |
| `text`                | `string`      | Nombre del servicio, nombre de un repositorio o descripción corta             |
| `textarea`            | `string`      | Descripciones largas, instrucciones, configuración YAML o texto multilínea    |
| `password`            | `string`      | Contraseñas o valores sensibles que no deberían mostrarse en texto plano      |
| `radio`               | `string/bool` | Elegir una única opción entre pocas alternativas Yes/No Dev/Prod/Stg`         |
| `select`              | `string`      | Elegir una opción de una lista, por ejemplo el lenguaje: Java, Python o Node  |
| `checkbox`            | `bool`        | Activar o desactivar una opción, por ejemplo `enableMonitoring: true/false`   |

```

#### 2.2 Validaciones

¿Cómo puedes hacer que un campo de tipo `string` solo acepte valores en minúsculas sin espacios (útil para nombres de servicios)? Menciona las propiedades de JSON Schema relevantes.

Para esta validación en particular se podría usar la propiedad pattern → aplica una expresión regular al valor.

```
  pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
```

Otras propiedades de validación pueden ser:
type: string → garantiza que el valor sea texto.
minLength: 1 → evita que el usuario deje el campo vacío.
maxLength → opcionalmente limita la longitud máxima.

### 3. Pasos de Orquestación (`steps`)

La sección `steps` define la secuencia de acciones que el scaffolder ejecutará automáticamente.

#### 3.1 Actions disponibles

Investiga las **Built-in Actions** del scaffolder de Backstage. Completa la siguiente tabla con las acciones que necesitarás para tu Golden Path:

```

| Action ID          | ¿Qué hace?                                                   | Inputs principales                              |
| ------------------ | ------------------------------------------------------------ |------------------------------------------------ |
| `fetch:template`   | Toma un **skeleton/template de archivos**, procesa las variables y genera el proyecto resultante en el workspace del Scaffolder. | `url`: ubicación del template; `values`: variables que se pasan al template; opcionalmente `targetPath`                  |
| `publish:github`   | Publica el contenido generado en un **repositorio de GitHub**. Puede crear un nuevo repositorio o trabajar con uno existente según la configuración.| `repoUrl`: URL/identificador del repositorio; `description`; `defaultBranch`; `repoVisibility`; opcionalmente `token` |
| `catalog:register` | Registra en el **Backstage Catalog** el `catalog-info.yaml` que acaba de ser creado/publicado, convirtiendo el proyecto en una entidad conocida por Backstage. | `repoContentsUrl`: ubicación del contenido del repositorio; `catalogInfoPath`: ruta al `catalog-info.yaml`            |

```

#### 3.2 Motor de Plantillas (Nunjucks)

El paso `fetch:template` usa **Nunjucks** para reemplazar variables en los archivos del skeleton. ¿Cómo accedes al valor de un parámetro llamado `serviceName` dentro de un archivo del skeleton?

Si tenemos un parámetro definido de la siguiente forma:
```
parameters:
  - title: Información del servicio
    properties:
      serviceName:
        type: string
```
La manera de acceder en los steps sería:

```
${{ values.serviceName }}
```

#### 3.3 Output del paso `publish:github`

La acción `publish:github` retorna un output que puedes usar en pasos siguientes. ¿Qué propiedad del output contiene la URL del repositorio recién creado? ¿Cómo se referencia en el siguiente paso?

Si tenemos una estructura de pasos como la siguiente:

```
- id: publish
  name: Publish to GitHub
  action: publish:github
  input:
    repoUrl: github.com?repo=${{ values.serviceName }}&owner=my-org

- id: output
  name: Show repository URL
  action: debug:log
  input:
    message: ${{ steps.publish.output.remoteUrl }}

```

La manera de acceder el output del repositorio sería:
```
${{ steps.publish.output.remoteUrl }}
```

### 4. Outputs del Template

La sección `output` del template permite mostrarle al usuario información al final del proceso (links, instrucciones, etc.).

¿Qué tipos de `links` puedes mostrar en el output? Lista al menos 2 ejemplos concretos para nuestro caso de uso (link al repo, link al componente en el catálogo).

En la sección de output se podrían declarar los siguientes links que serían las salidas del repositorio y del link a backstage.

```
output:
  links:
    - title: Repository
      icon: github
      url: ${{ steps.publish.output.remoteUrl }}

    - title: Open in Backstage
      icon: catalog
      url: ${{ steps.register.output.entityRef }}
```

### 5. El Skeleton del Microservicio

El directorio `skeleton/` contiene la estructura base del proyecto que se copiará como punto de partida. Investiga:

#### 5.1 Parametrización del skeleton

El archivo `skeleton/catalog-info.yaml` también debe parametrizarse. Investiga la estructura del `catalog-info.yaml` de Backstage y responde: ¿Qué campos mínimos son obligatorios?

Esta sería la estructura del archivo catalog-info.yaml con los campos mínimos:

```
apiVersion: backstage.io/v1alpha1
kind: Component

metadata:
  name: ${{ values.serviceName }}

spec:
  type: service
  lifecycle: production
  owner: ${{ values.owner }}
```

En general, los primeros tres campos son parecidos a los de un Template, pero con la salvedad que en este caso este es un Component.
Para spec spec.type, spec.lifecycle y spec.owner son obligatorios al igual que para metadata metadata.name es necesario de igual forma. 

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
