# Decisiones de diseño (opcional)

Este documento recoge las justificaciones técnicas y decisiones arquitectónicas adoptadas durante el desarrollo de la práctica, especialmente aquellas condicionadas por el entorno de despliegue.

## Justificación de Seguridad y Permisos IAM (Cambio de variante)

Para cumplir estrictamente con el **principio de mínimos privilegios** (Variante A), la instancia EC2 (o las credenciales utilizadas por Streamlit) solo debería tener permisos de lectura sobre el bucket específico del proyecto, y no acceso global a otros recursos de AWS. 

La política en formato JSON ideal y correcta para este escenario de producción sería la siguiente:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PermitirListarBucketS3",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::iabd03-tarea-ra2"
            ]
        },
        {
            "Sid": "PermitirLecturaObjetosS3",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::iabd03-tarea-ra2/data/sensores/*"
            ]
        }
    ]
}
```

Sin embargo, debido a las restricciones del entorno de **AWS Academy Learner Lab** (donde los permisos de administración de IAM están fuertemente limitados y la opción de "Crear política" se encuentra deshabilitada para los alumnos), no ha sido posible adjuntar esta política personalizada.

Para sortear esta limitación técnica de la plataforma académica y permitir el correcto funcionamiento de la instancia EC2 y la lectura del archivo JSON, se ha tomado la decisión de utilizar el rol/credenciales por defecto proporcionados por el laboratorio (configurados localmente en `~/.aws/credentials` con su respectivo `aws_session_token`). Esta decisión prioriza la viabilidad del despliegue asumiendo las restricciones del entorno de pruebas.

