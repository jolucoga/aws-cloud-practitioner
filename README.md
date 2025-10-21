# ☁️ AWS Certified Cloud Practitioner (CCP) - Base de Conocimiento Técnica

## 🎯 Propósito del Repositorio

Este proyecto es mi **Fuente Única de Verdad (Single Source of Truth)** para la certificación AWS Certified Cloud Practitioner. Está diseñado como un recurso de referencia personal, técnico y escalable.

El objetivo principal es:

1.  **Consolidar el Conocimiento:** Documentar de forma estructurada los conceptos fundamentales y los servicios clave de AWS para la certificación.
2.  **Referencia Continua:** Servir como un manual técnico de consulta rápida para futuras implementaciones o preparación para certificaciones de mayor nivel (Ej: Solutions Architect Associate).
3.  **Refuerzo Metodológico:** Aplicar y perfeccionar el flujo de trabajo de **Git y GitHub**, manteniendo una disciplina de versionamiento profesional (DevOps).

---

## 📚 Estructura y Mapeo del Contenido

La documentación está organizada por módulos temáticos, siguiendo la estructura del curso de estudio, lo que facilita la revisión y la búsqueda de conceptos.

| Módulo | Enfoque y Servicios Clave | Estado |
| :--- | :--- | :--- |
| **`01_Cloud_Computing_e_IAM`** | Fundamentos de la Nube, Infraestructura Global, Modelo de Responsabilidad Compartida, IAM. | ⬜ |
| **`02_Computo_y_Almacenamiento`** | EC2, EBS, S3 (Clases, Seguridad), ELB, Auto Scaling Groups (ASG). | ⬜ |
| **`03_Bases_de_Datos_y_Serverless`**| RDS (Multi-AZ), DynamoDB, Arquitecturas sin Servidor (Lambda, ECS, Fargate). | ⬜ |
| **`04_Redes_y_Despliegue`** | VPC, Route 53, CloudFront (CDN), CloudFormation (Infraestructura como Código). | ⬜ |
| **`05_Seguridad_Monitorizacion`**| CloudWatch, CloudTrail, KMS, GuardDuty, SQS/SNS, Pruebas de Penetración. | ⬜ |
| **`06_Facturacion_y_Arquitectura`**| Facturación Consolidada, Modelos de Precios, Trusted Advisor, Well-Architected Framework. | ⬜ |
| **`07_Otros_Servicios_y_Examen`**| Servicios Avanzados (ML, IoT), Identity Center, Estrategia del Examen. | ⬜ |
| **`codigo-ejemplo`** | **Código Versionado:** Scripts y plantillas de las prácticas de despliegue para replicación rápida de entornos. | ⬜ |

---

## 🛠️ Disciplina Técnica y Flujo de Trabajo

El mantenimiento de este repositorio sigue estrictos estándares de calidad y gestión de código:

### 1. Versionamiento y Trazabilidad (Git)

* **Historial de Cambios:** Se utiliza el *commit* history para registrar de forma granular la evolución de las notas y las correcciones.
* **Gestión de Tareas (*Issues*):** La sección de **Issues** se emplea para planificar y llevar un registro del progreso de cada módulo de estudio y documentar errores encontrados, asegurando la trazabilidad entre la planificación y el contenido.
* **Convención de Commits:** Se sigue una convención clara (Ej: `feat:` para nuevo contenido, `fix:` para correcciones), mejorando la legibilidad del historial.

### 2. Seguridad del Repositorio

* **Prevención de Fugas de Credenciales:** Un archivo `.gitignore` robusto está implementado para asegurar que **ningún dato sensible** (claves `.pem`, *access keys* de IAM, archivos de configuración local) sea accidentalmente subido o expuesto.
* **Seguridad en la Nube:** Las prácticas de AWS están diseñadas para utilizar **Roles de IAM** en lugar de credenciales estáticas, adhiriéndose al principio de menor privilegio.

### 3. Gestión de Recursos y Costos

* **Optimización de Costos:** Se mantiene una estricta disciplina en el uso de la **Capa Gratuita de AWS (Free Tier)**.
* **Protocolos de Limpieza:** Cada código de práctica en la carpeta `codigo-ejemplo` incluye la documentación necesaria para la **terminación y eliminación completa** de los recursos (como EC2 y RDS) para evitar costos residuales.

---
***
## Notas Finales

Este material cubre exhaustivamente los fundamentos para el examen de certificación AWS Certified Cloud Practitioner CLF-C02. Practica los conceptos en tu propia cuenta de AWS (usa la capa gratuita) para reforzar el aprendizaje.

¡Buena suerte en tu examen!

**Última actualización**: 21/10/2025  
**Versión del curso**: AWS Certified Cloud Practitioner CLF-C02