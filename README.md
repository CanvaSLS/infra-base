# infra-base

Recursos compartidos: tema **SNS** `sp-eventos-<stage>` (bus de eventos) y bucket **S3** privado `sp-storage-<cuenta>-<stage>` (SSE, CORS para PUT/GET con URL prefirmada).

```bash
npx --yes serverless@3 deploy --stage dev     # SIEMPRE el primero
```
Salidas: `TopicArn`, `BucketName`.

## RDS MySQL (cursos y evaluaciones)
`infra-base` crea UNA instancia `sp-rds-<stage>` (MySQL 8, `db.t3.micro`, 20 GB) en la VPC por defecto y un Security Group para el puerto 3306.
- **Antes de desplegar:** `export DB_PASSWORD='...'` (8+ caracteres, sin `/ @ "` ni espacios). La contraseña no se guarda en el repo.
- La primera vez tarda ~10 minutos (CloudFormation espera a que la RDS esté disponible).
- Salida `DbEndpoint`: la leen `ms-cursos` y `ms-evaluaciones` (`${cf:sp-infra-<stage>.DbEndpoint}`) y el seed (`DB_HOST`).
- **Decisión de laboratorio:** acceso público + contraseña, porque las Lambdas no están en VPC y no tienen IP fija. Diseño de producción (para el informe y el examen): Lambdas y RDS en subredes privadas, Security Group origen→destino, VPC endpoints para SNS/S3 y credenciales en Secrets Manager.
- Esquema por tenant ("Pattern 2"): `cursos_<tenant>` y `evaluaciones_<tenant>`, creados por las propias Lambdas con `CREATE ... IF NOT EXISTS`.
