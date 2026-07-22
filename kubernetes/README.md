# Fase 1: Creación del clúster de Kubernetes con Kind

Primero, crearemos un clúster de Kubernetes funcional directamente en tu máquina local. Para ello, utilizaremos **Kind** (Kubernetes in Docker), una herramienta que simplifica este proceso.

## ¿Qué es Kind y cómo funciona?

**Kind** es una herramienta diseñada para ejecutar clústeres locales de Kubernetes utilizando contenedores Docker como "nodos". En lugar de necesitar máquinas virtuales, Kind empaqueta cada nodo del clúster (tanto el `control-plane` como los `workers`) dentro de un contenedor Docker.

Esto lo convierte en la opción ideal para:
*   **Desarrollo local:** Probar aplicaciones en un entorno realista de Kubernetes.
*   **Pruebas de CI/CD:** Ejecutar pruebas de integración en un clúster efímero.
*   **Aprendizaje y Talleres:** Como este, donde necesitamos un clúster rápido, ligero y desechable.

## Entendiendo la configuración `kind.yaml`

El archivo `kind.yaml` le dice a Kind exactamente cómo queremos que construya nuestro clúster. Analicemos su contenido:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: backstage-platform
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30000
        hostPort: 80
        protocol: TCP
      - containerPort: 30001
        hostPort: 443
        protocol: TCP
  - role: worker
```

*   **`nodes:`**: Aquí definimos la topología de nuestro clúster.
    *   **`role: control-plane`**: Este es el cerebro del clúster. Ejecuta los componentes principales de Kubernetes como el API Server, scheduler, etc.
    *   **`role: worker`**: Este es el nodo de trabajo. Aquí es donde se ejecutarán nuestras aplicaciones, como los pods de Backstage.
*   **`extraPortMappings:`**: Permite redirigir el tráfico desde un puerto en tu máquina (`hostPort`) a un puerto dentro del contenedor del nodo de Kind (`containerPort`).
    *   **`hostPort: 80` -> `containerPort: 30000`**: Mapea el puerto 80 de tu máquina al puerto 30000 del nodo. Más adelante, expondremos el servicio de Backstage en este puerto `30000`, permitiéndonos acceder a Backstage desde el navegador simplemente con `http://localhost`.
    *   **`hostPort: 443` -> `containerPort: 30001`**: Hace lo mismo para el tráfico HTTPS.

## Comandos para Gestionar el Clúster

### 1. Crear el Clúster

Abre tu terminal, asegúrate de estar en la raíz del proyecto y ejecuta el siguiente comando.

```bash
kind create cluster --config kubernetes/kind.yaml
```

### 2. Verificar la Creación

Una vez que el comando anterior termine, puedes verificar que el clúster está en funcionamiento y que `kubectl` está configurado para apuntar a él.

```bash
kubectl cluster-info --context kind-backstage-platform
```

Deberías ver una salida que te indica la dirección del "Kubernetes control plane".

### 3. (Opcional) Eliminar el Clúster

Cuando termines el taller o si necesitas empezar de cero, puedes eliminar el clúster por completo con este comando:

```bash
kind delete cluster --name backstage-platform
```

---

# Paso 2: Instalación de Backstage con Helm

Con el clúster en funcionamiento, vamos a desplegar Backstage. Usaremos **Helm**, el gestor de paquetes de Kubernetes, para instalar el chart de Bitnami, que tiene una configuración de Backstage pre-empaquetada y lista para usar.

## ¿Qué es Helm?

Helm funciona como un gestor de paquetes (como `apt` en Ubuntu o `brew` en macOS) para Kubernetes. Permite definir, instalar y actualizar aplicaciones complejas a través de "charts". Un chart es una colección de archivos que describen un conjunto de recursos de Kubernetes.

### 1. Añadir el Repositorio de Helm de Bitnami

Primero, debemos decirle a Helm dónde encontrar el chart de Backstage. Bitnami mantiene un repositorio público con cientos de charts populares.

```bash
# Añade el repositorio de Bitnami a tu configuración local de Helm
helm repo add bitnami https://charts.bitnami.com/bitnami

# Actualiza la información de los repositorios para obtener las últimas versiones
helm repo update
```

Puedes verificar que el chart de Backstage está disponible con:

```bash
helm search repo bitnami/backstage
```

---

### 2. Entender el archivo `backstage-values.yaml`

En lugar de pasar decenas de parámetros por línea de comandos, usamos un archivo `values.yaml` que sobreescribe los valores por defecto del chart. El archivo `backstage-values.yaml` de este repositorio ya está pre-configurado para nuestro entorno local.

Los puntos clave que configura son:

| Sección | Qué hace |
|---|---|
| `backstage.appConfig` | El corazón de Backstage: configura URL base, integraciones con GitHub y las reglas del catálogo |
| `service.type: NodePort` | Expone Backstage en un puerto accesible desde fuera del clúster |
| `service.nodePorts.backend: "30000"` | El puerto `30000` que mapeamos al puerto `80` del host en `kind.yaml` |
| `postgresql.enabled: false` | Usa una base de datos en memoria para simplificar el entorno local |
| `auth.providers.guest` | Permite acceder sin autenticación (modo desarrollo) |

#### Configurar el Token de GitHub

Backstage necesita un **Personal Access Token (PAT)** de GitHub para poder leer repositorios y publicar código. Debes crearlo antes de continuar.

1. Ve a **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**
2. Crea un token con los siguientes scopes:
   - `repo` (acceso completo a repositorios)
   - `workflow` (para poder hacer push a `.github/workflows`)
   - `read:org` (para leer info de organizaciones)
3. Copia el token generado.

Ahora, **exporta tu token como variable de entorno** en tu terminal. Helm la inyectará al instalar el chart:

```bash
# En macOS/Linux
export GITHUB_TOKEN="ghp_tuTokenAqui"

# Verifica que está disponible
echo $GITHUB_TOKEN
```

> **Importante:** Nunca guardes tu token directamente en el archivo `backstage-values.yaml` ni lo subas a un repositorio público.

---

### 3. Instalar Backstage con Helm

Con el repositorio añadido y el token configurado, ejecuta el siguiente comando para instalar Backstage. Usamos el flag `--set` para pasar el token de forma segura desde la variable de entorno:

```bash
helm install backstage bitnami/backstage \
  --namespace backstage \
  --create-namespace \
  --values kubernetes/backstage-values.yaml \
  --set backstage.extraEnvVars[0].name=GITHUB_TOKEN \
  --set backstage.extraEnvVars[0].value="$GITHUB_TOKEN"
```

Este comando:
- `--namespace backstage --create-namespace`: Crea el namespace `backstage` si no existe y despliega ahí.
- `--values kubernetes/backstage-values.yaml`: Aplica nuestra configuración personalizada.
- `--set`: Inyecta el token de GitHub de forma segura desde tu variable de entorno local.

---

### 4. Verificar la instalación

#### Ver el estado del despliegue

```bash
# Listar todos los recursos en el namespace 'backstage'
kubectl get all -n backstage
```

#### Esperar a que el Pod esté listo

El pod de Backstage tardará unos minutos en iniciar. Puedes monitorear su progreso con:

```bash
# Observa el estado de los pods en tiempo real (Ctrl+C para salir)
kubectl get pods -n backstage -w
```

Espera hasta que el pod muestre el estado `Running` y la columna `READY` muestre `1/1`.

#### Ver los logs del Pod

Si algo no funciona como esperabas, los logs son tu mejor herramienta de diagnóstico:

```bash
# Reemplaza <nombre-del-pod> con el nombre real del pod (obtenido con el comando anterior)
kubectl logs -n backstage <nombre-del-pod> -f
```

---

### 5. Acceder a Backstage

Una vez que el pod esté en estado `Running`, abre tu navegador y visita:

```
http://localhost
```

Gracias al `extraPortMappings` en `kind.yaml` (puerto 80 del host → puerto 30000 del nodo → servicio NodePort de Backstage), el tráfico llega directamente al portal de Backstage.

Deberías ver la pantalla de bienvenida de Backstage con la opción **"Enter as guest"** (modo invitado activado para desarrollo local).

