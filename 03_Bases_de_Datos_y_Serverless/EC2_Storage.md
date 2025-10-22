# Almacenamiento en Instancias EC2

---

## 1. Visión General de EBS (Elastic Block Store)

Amazon **EBS (Elastic Block Store)** proporciona volúmenes de almacenamiento en bloque persistentes para las instancias EC2. Se utiliza cuando una aplicación requiere almacenamiento de datos que persista más allá del ciclo de vida de la instancia.

### Características Principales

| Característica | Descripción |
|----------------|--------------|
| Persistente | Los datos se conservan después de detener o terminar la instancia |
| Nivel de Bloque | Proporciona almacenamiento a nivel de bloque, como un disco duro |
| Escalable | Se puede aumentar la capacidad y rendimiento sin reiniciar la instancia |
| Alta Disponibilidad | Replicado automáticamente dentro de la misma zona de disponibilidad |
| Cifrado | Opción de cifrado con KMS al crear el volumen |

### Tipos de Volúmenes EBS

- **gp3 / gp2**: Volúmenes SSD de propósito general.
- **io1 / io2**: SSD con rendimiento optimizado para IOPS.
- **st1**: HDD optimizado para throughput.
- **sc1**: HDD de bajo costo para acceso poco frecuente.

### Uso Típico
- Bases de datos, sistemas de archivos y aplicaciones de misión crítica.
- Almacenamiento persistente de logs o archivos del sistema.

---

## 2. Acerca de EBS Multi-Attach

**EBS Multi-Attach** permite conectar un mismo volumen EBS a múltiples instancias EC2 dentro de la misma zona de disponibilidad.

### Requisitos
- Solo disponible para volúmenes **io1** e **io2**.
- Todas las instancias deben estar en la **misma AZ**.
- Cada instancia debe tener un sistema de archivos compatible con acceso concurrente (por ejemplo, **Cluster File Systems** como Lustre o GFS2).

> ⚠️ Esta característica **no es parte del examen Cloud Practitioner**, pero es importante conocerla para roles más avanzados.

---

## 3. EBS - Práctica

### Objetivo
Aprender a crear y adjuntar volúmenes EBS a una instancia EC2.

### Pasos

1. Ir a **EC2 > Volúmenes (EBS)**  
2. Click en **Crear Volumen**.
3. Configurar:
   - Tamaño: `2 GiB`
   - Zona de disponibilidad: misma que tu instancia (por ejemplo `us-east-1a`)
   - Tipo: `gp3`
4. Click en **Crear Volumen**.
5. Una vez creado, selecciona el volumen > **Acciones > Asociar Volumen**.
6. Selecciona la instancia EC2 activa.
7. Verifica que el volumen esté adjunto desde **Instancia > Almacenamiento**.

#### Nota Importante
El volumen raíz (root) se elimina al terminar la instancia, a menos que se desmarque la opción **"Delete on termination"**.

---

## 4. Visión General de EBS Snapshots

Un **Snapshot** es una copia de seguridad incremental de un volumen EBS en un momento específico. Se almacenan en Amazon S3 (de forma administrada por AWS) y se pueden usar para restaurar o mover volúmenes entre zonas o regiones.

### Beneficios
- Copia de seguridad incremental (solo se guardan los bloques modificados).
- Permite replicar datos entre zonas y regiones.
- Puede archivarse a bajo costo (**Archive Snapshots**).
- Papelera de reciclaje integrada (Trash Bin) para recuperación de snapshots eliminados accidentalmente.

### Tiempo de Restauración
Entre **24 a 72 horas** dependiendo del tamaño del snapshot.

---

## 5. EBS Snapshots - Práctica

### Crear Snapshot desde un Volumen
1. Navegar a **EC2 > Snapshots**.
2. Click en **Crear Snapshot**.
3. Selecciona el volumen de origen.
4. Añade una descripción (ej. `demo-snapshot`).
5. Click en **Crear Snapshot**.

### Restaurar un Snapshot
1. Selecciona el snapshot creado.
2. Click en **Acciones > Crear Volumen desde Snapshot**.
3. Selecciona zona de disponibilidad.
4. Adjunta el nuevo volumen a una instancia EC2.

> 💡 Usa snapshots para mover volúmenes entre zonas o regiones.

---

## 6. Visión General de AMI (Amazon Machine Image)

Una **AMI** es una imagen preconfigurada que contiene el sistema operativo, configuraciones y software necesarios para lanzar instancias EC2.

### Tipos de AMIs
- **Públicas:** proporcionadas por AWS o comunidad.
- **Privadas:** creadas por el usuario.
- **Marketplace:** ofrecidas por terceros (con costo adicional).

### Componentes de una AMI
- Plantilla para el volumen raíz (sistema operativo).
- Permisos de lanzamiento.
- Mapeo de dispositivos de bloque.

---

## 7. AMI - Práctica

### Crear una AMI desde una Instancia Existente
1. Selecciona una instancia EC2 activa.
2. Click en **Acciones > Imagen y Plantilla > Crear Imagen**.
3. Ingresa nombre y descripción.
4. Selecciona si deseas **reiniciar la instancia** durante la creación.
5. Click en **Crear Imagen**.

### Lanzar una Instancia desde AMI
1. Ir a **EC2 > AMIs**.
2. Selecciona la AMI creada.
3. Click en **Lanzar Instancia**.
4. Configura tipo de instancia y almacenamiento.
5. Click en **Launch**.

---

## 8. Visión General de EC2 Image Builder

**EC2 Image Builder** automatiza la creación, mantenimiento y despliegue de imágenes de servidor seguras y actualizadas.

### Beneficios
- Pipelines automatizados para mantener AMIs actualizadas.
- Integración con Amazon Inspector y Systems Manager.
- Reduce errores humanos en configuración de imágenes.

---

## 9. EC2 Instance Store

**Instance Store** proporciona almacenamiento temporal para instancias EC2.

### Características
| Propiedad | Descripción |
|------------|-------------|
| Temporal | Los datos se pierden al detener o terminar la instancia |
| Alto rendimiento | Muy rápido, ideal para almacenamiento en caché o datos efímeros |
| Sin costo adicional | Incluido con la instancia |
| Sin persistencia | No se puede usar para datos críticos |

### Usos Comunes
- Archivos temporales.
- Datos de sesión.
- Cachés de aplicaciones.

> ⚠️ No usar Instance Store para almacenamiento persistente.

---

## 10. Visión General de EFS (Elastic File System)

Amazon **EFS** es un sistema de archivos NFS (Network File System) escalable, completamente administrado y compartido entre múltiples instancias EC2 Linux.

### Características
- Multi-AZ: Alta disponibilidad entre zonas.
- Escalado automático: No requiere planificación de capacidad.
- Pago por uso.
- Sin necesidad de snapshots manuales.

### Clases de Almacenamiento
| Clase | Descripción |
|--------|--------------|
| EFS Standard | Acceso frecuente a archivos. |
| EFS Infrequent Access (IA) | Archivos accedidos ocasionalmente (hasta 92% más barato). |

### Ejemplo Visual
```
Instancia EC2 (us-east-1a) ─┐
                           │
Instancia EC2 (us-east-1b) ├──> EFS Mount Target ──> EFS
                           │
Instancia EC2 (us-east-1c) ┘
```

### Políticas de Ciclo de Vida
Automáticamente mueve archivos no accedidos (por ejemplo, 60 días) a la clase IA.

---

## 11. Modelo de Responsabilidad Compartida para el Almacenamiento EC2

En AWS, la **seguridad** se basa en un modelo de **responsabilidad compartida**:

| AWS | Cliente |
|------|----------|
| Seguridad del Cloud | Seguridad dentro del Cloud |
| Infraestructura física (hardware, red, centros de datos) | Configuración de EBS, AMI, EFS, FSx |
| Redundancia, disponibilidad y cifrado base | Control de acceso, políticas IAM, snapshots y backups |

> AWS protege la infraestructura. Tú proteges tus datos y configuraciones.

---

## 12. Visión General de Amazon FSx

**Amazon FSx** ofrece sistemas de archivos totalmente administrados y optimizados para aplicaciones específicas.

### Tipos de FSx
- **FSx for Windows File Server**: Compatible con SMB, ideal para entornos Microsoft.
- **FSx for Lustre**: Diseñado para cargas de trabajo de alto rendimiento (HPC, IA, análisis de datos).

### Características
- Integración nativa con Active Directory.
- Cifrado en reposo y en tránsito.
- Escalabilidad automática.

---

## 13. Resumen - Almacenamiento de Instancias EC2

| Servicio | Tipo | Persistencia | Uso principal |
|-----------|------|---------------|----------------|
| **EBS** | Bloques | Persistente | Almacenamiento de datos del sistema o aplicaciones |
| **Snapshots** | Backup | Persistente | Copias de seguridad de EBS |
| **AMI** | Imagen | Persistente | Lanzamiento de instancias configuradas |
| **Instance Store** | Bloques | Temporal | Datos efímeros o caché |
| **EFS** | Archivos | Persistente | Almacenamiento compartido Linux |
| **FSx** | Archivos | Persistente | Almacenamiento administrado especializado |

---

## 14. Limpieza de Secciones

Para evitar costos innecesarios:
- Elimina volúmenes EBS no asociados.
- Elimina snapshots no requeridos.
- Detén o termina instancias de prueba.
- Desmonta EFS y FSx si no se usan.

---

## Cuestionario: Almacenamiento de Instancias EC2

**Objetivo:** Reforzar el aprendizaje de los conceptos clave sobre almacenamiento en EC2.

### Preguntas de opción múltiple

1. ¿Qué servicio de AWS proporciona almacenamiento en bloque persistente?
   - A) EFS
   - B) EBS ✅
   - C) FSx
   - D) Instance Store

2. ¿Qué tipo de almacenamiento se pierde al detener la instancia?
   - A) EBS
   - B) EFS
   - C) Instance Store ✅
   - D) FSx

3. ¿Qué característica distingue a EFS frente a EBS?
   - A) EFS solo funciona con Windows
   - B) EFS permite acceso simultáneo desde múltiples instancias ✅
   - C) EFS no puede escalar
   - D) EFS no se replica entre zonas

4. ¿Qué tipos de volumen soportan EBS Multi-Attach?
   - A) gp3 y gp2
   - B) io1 e io2 ✅
   - C) st1 y sc1
   - D) Todos los tipos de EBS

5. ¿Dónde se almacenan los snapshots de EBS?
   - A) En S3 administrado por AWS ✅
   - B) En EFS
   - C) En la instancia EC2
   - D) En FSx

6. ¿Qué tipo de AMI puede crear un usuario?
   - A) Solo públicas
   - B) Solo marketplace
   - C) Privadas ✅
   - D) Ninguna de las anteriores

7. ¿Qué servicio automatiza la creación de AMIs seguras y actualizadas?
   - A) EC2 Image Builder ✅
   - B) FSx
   - C) CloudFormation
   - D) EBS

8. ¿Cuál de los siguientes servicios es ideal para almacenamiento compartido en Linux?
   - A) EFS ✅
   - B) FSx for Windows
   - C) EBS
   - D) Instance Store

9. ¿Qué opción es correcta sobre el modelo de responsabilidad compartida?
   - A) AWS gestiona tus backups
   - B) El cliente configura la seguridad de los snapshots ✅
   - C) AWS cifra automáticamente todos los datos del usuario
   - D) El cliente no tiene responsabilidad de seguridad

10. ¿Qué servicio es ideal para cargas de trabajo HPC (High Performance Computing)?
    - A) FSx for Lustre ✅
    - B) EBS gp3
    - C) EFS-IA
    - D) Instance Store

---
***
## Notas Finales

Este material cubre exhaustivamente todo lo relacionado con Storage para intancias EC2 para el examen AWS Certified Cloud Practitioner. Practica los conceptos en tu propia cuenta de AWS (usa la capa gratuita) para reforzar el aprendizaje.

¡Buena suerte en tu examen!

**Última actualización**: 22/10/2025  
**Versión del curso**: AWS Certified Cloud Practitioner CLF-C02
