# AWS-Iac-150

Infraestructura como código para desplegar una arquitectura AWS mediante **AWS CloudFormation** y **GitHub Actions**.

## Arquitectura

El despliegue se realiza de forma modular y ordenada en la región `us-east-1`:

1. VPC con dos subredes públicas.
2. Security Group y reglas de acceso.
3. Dos instancias EC2 estáticas.
4. Application Load Balancer (ALB), listener y target group.
5. Auto Scaling Group (ASG), Launch Template y Warm Pool.

Los stacks principales son:

- `MinticVPCStack`
- `MinticSecurityStack`
- `MinticEC2Stack`
- `MinticALBStack`
- `MinticASGStack`

## Estructura del repositorio

```text
.
├── .github/workflows/
│   ├── deploy.yml       # Validación y despliegue por etapas
│   └── detele.yml       # Eliminación manual de los stacks
└── templates/
    ├── alb.yml
    ├── asg.yml
    ├── ec2-static.yml
    ├── ec2.yml
    ├── mintic-architecture.yml
    ├── s3.yml
    ├── security.yml
    └── vpc.yml
```

Los templates modulares (`vpc`, `security`, `ec2-static`, `alb` y `asg`) son los utilizados por el workflow principal. Los demás templates se conservan como alternativas o componentes adicionales.

## Requisitos

- Una cuenta de AWS.
- Un repositorio de GitHub con permisos para ejecutar GitHub Actions.
- AWS CLI, si se desea ejecutar los templates manualmente.
- Un key pair de EC2 llamado `KeyNew`, o actualizar el parámetro `KeyName` en `.github/workflows/deploy.yml`.
- Una AMI válida para `us-east-1`, o actualizar el valor `AmiId` en el workflow.

## Configuración de secretos

En **Settings → Secrets and variables → Actions**, configura estos secretos:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

La identidad de AWS debe tener permisos suficientes para validar templates, crear y eliminar stacks de CloudFormation y administrar los recursos EC2, VPC, seguridad y balanceo utilizados por la arquitectura.

## Despliegue

El workflow `deploy.yml` se ejecuta automáticamente al hacer push a `main` o manualmente desde **Actions → MinTic Deploy to AWS CloudFormation → Run workflow**.

Antes de crear recursos, valida los templates y después despliega cada stack en orden. Al finalizar, muestra en los logs las salidas de CloudFormation de los stacks principales.

## Eliminación de recursos

Para eliminar la infraestructura, ejecuta manualmente el workflow **MinTic Delete AWS CloudFormation Stacks** desde la pestaña **Actions**. La eliminación respeta el orden inverso de dependencias y también intenta eliminar el stack monolítico antiguo `MinticFullArchitecture`, si existe.

> Ejecutar este workflow elimina los recursos creados por los stacks indicados. Verifica siempre la cuenta y la región antes de ejecutarlo.

## Ejecución local

Para validar un template:

```bash
aws cloudformation validate-template \
  --template-body file://templates/vpc.yml \
  --region us-east-1
```

Para desplegar un stack manualmente:

```bash
aws cloudformation deploy \
  --template-file templates/vpc.yml \
  --stack-name MinticVPCStack \
  --region us-east-1 \
  --no-fail-on-empty-changeset
```

## Seguridad y costos

Este proyecto crea recursos que pueden generar costos en AWS, especialmente instancias EC2, un Application Load Balancer y otros componentes asociados. Elimina los stacks cuando ya no sean necesarios y evita guardar credenciales de AWS dentro del repositorio.
