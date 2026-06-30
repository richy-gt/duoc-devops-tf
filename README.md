## Link para video completo de presentación

https://youtu.be/inD8l64yt4U

# Innovatech Chile - Orquestación y Automatización en AWS EKS

Proyecto de la Evaluación Parcial N°3 (Introducción a Herramientas DevOps, Duoc UC). Continúa el trabajo de EP1 (infraestructura base en AWS) y EP2 (contenedorización), avanzando hacia la orquestación productiva de los servicios Frontend y Backend mediante AWS EKS y un pipeline CI/CD automatizado con GitHub Actions.

## Arquitectura

- Clúster Kubernetes desplegado en AWS EKS.
- VPC con subredes públicas y privadas, Security Groups dedicados para el clúster y los nodos.
- Roles IAM: EKS Cluster Role, EKS Node Role y Service Account roles para los pods.
- Imágenes de Frontend y Backend almacenadas en Amazon ECR.
- Exposición del Frontend mediante un Load Balancer (ALB / Service type LoadBalancer).
- Comunicación interna Frontend → Backend a través de DNS interno de Kubernetes (Service ClusterIP).
- Infraestructura como código gestionada con Terraform (carpeta `terraform/`).
- Manifiestos de despliegue de la aplicación en Kubernetes (carpeta `app-k8s/`).
- Pipeline CI/CD definido en `.github/workflows/`.

## Estructura del repositorio

```
.
├── terraform/         # IaC: VPC, EKS, IAM, ECR, networking
├── app-k8s/            # Manifiestos Kubernetes (Deployments, Services, HPA)
├── .github/workflows/  # Pipeline CI/CD (build, push, deploy)
└── README.md
```

## Roles, redes y seguridad

- **VPC y subredes:** subredes públicas para el Load Balancer y subredes privadas para los nodos de trabajo.
- **Security Groups:** reglas restringidas entre el control plane, los nodos y el tráfico externo necesario.
- **Roles IAM:**
  - EKS Cluster Role: permisos para que el control plane administre recursos de AWS.
  - EKS Node Role: permisos para que los nodos se unan al clúster y accedan a ECR.
  - Execution Role / Service Account: permisos mínimos necesarios para cada pod.
- **Secrets y credenciales:** gestionados mediante Kubernetes Secrets y/o GitHub Secrets, sin exposición de credenciales en el código ni en los manifiestos versionados.

## Autoscaling

- Implementado mediante Horizontal Pod Autoscaler (HPA), basado en uso de CPU.
- Umbral configurado en 50% de uso de CPU como gatillo de escalamiento.
- El umbral se definió para mantener un margen de respuesta ante incrementos de carga sin generar escalamientos innecesarios en cargas normales.
- Validado mediante simulación de carga y observación de la creación de nuevos pods.

## Pipeline CI/CD

El pipeline se ejecuta en GitHub Actions ante cada commit a la rama principal y realiza:

1. **Build:** construcción de las imágenes Docker de Frontend y Backend.
2. **Push:** publicación de las imágenes en Amazon ECR.
3. **Deploy:** actualización de los Deployments en el clúster EKS mediante `kubectl apply` / `kubectl set image`.

El estado de cada ejecución, tiempos y logs quedan disponibles en la pestaña Actions del repositorio.

## Validación funcional

- Frontend accesible públicamente a través de la URL del Load Balancer.
- Backend respondiendo correctamente dentro del clúster.
- Comunicación Frontend → Backend verificada vía DNS interno.
- Logs revisados mediante `kubectl logs` y CloudWatch.
- Recuperación automática de pods verificada tras fallos y tras nuevos despliegues (redeploy).

## Problemas encontrados y soluciones

- **Conectividad entre Frontend y Backend:** se resolvió ajustando las reglas de Security Groups y verificando el nombre de Service usado como variable de entorno.
- **Fallos en el pipeline durante el push a ECR:** se corrigieron los permisos del rol IAM utilizado por GitHub Actions.
- **Pods en estado pendiente:** se solucionó ajustando los recursos solicitados (requests/limits) y la capacidad de los nodos.

## Requisitos para ejecutar el proyecto

- Cuenta AWS Academy (Learner Lab).
- AWS CLI configurado.
- kubectl configurado contra el clúster EKS.
- Terraform instalado.
- Docker instalado.

## Despliegue

```bash
# Provisionar infraestructura
cd terraform
terraform init
terraform apply

# Configurar acceso al clúster
aws eks update-kubeconfig --name <nombre-clúster> --region <región>

# Desplegar la aplicación
cd ../app-k8s
kubectl apply -f .
```

## Equipo

Proyecto desarrollado en dupla para la asignatura ISY1101 - Introducción a Herramientas DevOps, Duoc UC.
