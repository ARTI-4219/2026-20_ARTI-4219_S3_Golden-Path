# Taller: Construcción de un Developer Control Plane y Golden Paths en Kubernetes

El objetivo de este ejercicio es desplegar una **"Plataforma Viable Más Fina" (Thinnest Viable Platform — TVP)**. Para ello, desplegaremos un portal de desarrollador, **Backstage**, directamente en un clúster de Kubernetes que se ejecutará en la máquina local usando **Kind** (Kubernetes in Docker).

El taller está dividido en dos fases:

| Fase | Actividad | Tipo |
|---|---|---|
| **Fase 1** | [Despliegue del Developer Control Plane](#-fase-1-despliegue-del-developer-control-plane) | Guiada |
| **Fase 2** | [Construcción del Golden Path](#-fase-2-construcción-del-golden-path) | Semi-autónoma |

---

## Requisitos Previos

Para poder seguir este taller, debes tener todas las herramientas listadas a continuación instaladas y funcionando en tu computador.

### 1. Docker

Docker es necesario para que Kind pueda crear los nodos del clúster como contenedores.

*   **Verificación:**
    ```bash
    docker --version
    ```
*   **Instalación:**
    *   [Docker Desktop en macOS](https://docs.docker.com/desktop/install/mac-install/)
    *   [Docker Desktop en Windows](https://docs.docker.com/desktop/install/windows-install/)
    *   [Docker Desktop en Linux](https://docs.docker.com/desktop/install/linux-install/)

### 2. kubectl

`kubectl` es la herramienta de línea de comandos para interactuar con clústeres de Kubernetes.

*   **Verificación:**
    ```bash
    kubectl version --client
    ```
*   **Instalación:** [Guía oficial de Kubernetes](https://kubernetes.io/docs/tasks/tools/)

### 3. Helm

Helm es el gestor de paquetes para Kubernetes. Lo usaremos para desplegar Backstage de forma declarativa.

*   **Verificación:**
    ```bash
    helm version
    ```
*   **Instalación:** [Guía oficial de Helm](https://helm.sh/docs/intro/install/)

### 4. Kind

Kind (Kubernetes IN Docker) permite ejecutar un clúster local de Kubernetes usando contenedores Docker como nodos.

*   **Verificación:**
    ```bash
    kind --version
    ```
*   **Instalación:** [Guía oficial de Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

---

## 🗺️ Estructura del Repositorio

```
S3 - Developer Control Plane/
│
├── README.md                          ← Índice del taller (este archivo)
│
├── kubernetes/                        ← Fase 1: Configuración de la plataforma
│   ├── README.md                      ← Guía paso a paso de la Fase 1
│   ├── kind.yaml                      ← Configuración del clúster Kind
│   └── backstage-values.yaml          ← Values de Helm para Backstage
│
└── golden-path/                       ← Fase 2: Construcción del Golden Path
    ├── README.md                      ← Guía de investigación (debes completarlo)
    ├── template.yaml                  ← Software Template (debes completarlo)
    └── skeleton/                      ← Microservicio NodeJS base
        ├── catalog-info.yaml          ← Descriptor de Backstage
        ├── package.json               ← Proyecto NodeJS
        └── src/
            └── index.js               ← Servidor Express seguro por defecto
```

---

## Fase 1: Despliegue del Developer Control Plane

En esta fase **guiada** desplegarás Backstage sobre un clúster local de Kubernetes.

➡️ **[Ir a la guía de la Fase 1](kubernetes/README.md)**

La guía cubre:
1. Creación del clúster Kind con `kind.yaml`
2. Entendimiento del archivo `backstage-values.yaml`
3. Configuración del Personal Access Token de GitHub
4. Instalación de Backstage con `helm install`
5. Verificación del despliegue con `kubectl`
6. Acceso al portal en `http://localhost`

---

## Fase 2: Construcción del Golden Path

En esta fase **semi-autónoma** investigarás la anatomía de los Software Templates de Backstage y construirás tu propio Golden Path para crear microservicios de forma automatizada.

➡️ **[Ir a la guía de la Fase 2](golden-path/README.md)**

La guía incluye:
- Preguntas de investigación sobre Backstage que **debes responder** como parte de tu entrega
- Un `template.yaml` parcialmente completo con **TODOs** que debes implementar
- Un skeleton de microservicio NodeJS "Seguro por Defecto" como punto de partida

### Objetivo del Golden Path

Tu template debe permitir que un desarrollador ingrese **2 parámetros** (nombre del servicio y propietario) y automáticamente:

1. Descargue el skeleton del microservicio e inyecte los parámetros
2. Publique el código en un repositorio de GitHub personal
3. Registre el componente en el Software Catalog de Backstage
