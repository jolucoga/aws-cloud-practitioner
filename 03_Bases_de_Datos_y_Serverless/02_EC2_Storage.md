# Almacenamiento de Instancias EC2

## 1. Visión General de EBS (Elastic Block Store)

Amazon EBS (Elastic Block Store) es un servicio de almacenamiento en bloque de alto rendimiento diseñado para usarse con Amazon EC2. Proporciona volúmenes de almacenamiento persistentes que se comportan como discos duros virtuales.

### ¿Qué es un Volumen EBS?

Un volumen EBS es una unidad de red (no es una unidad física) que puedes adjuntar a tus instancias mientras se ejecutan. Esto te permite:

- ✅ Persistir datos incluso después de terminar la instancia
- ✅ Montar en una instancia a la vez (a nivel CCP - Cloud Practitioner)
- ✅ Vincularlo a una zona de disponibilidad específica

**Analogía:** Piensa en EBS como una "memoria USB de red" que puedes conectar y desconectar de tus servidores virtuales.

### Características Principales

| Característica | Descripción |
|---|---|
| **Persistente** | Los datos se conservan después de detener o terminar la instancia |
| **Nivel de Bloque** | Proporciona almacenamiento a nivel de bloque, como un disco duro tradicional |
| **Escalable** | Se puede aumentar la capacidad y rendimiento sin reiniciar la instancia |
| **Alta Disponibilidad** | Replicado automáticamente dentro de la misma zona de disponibilidad |
| **Cifrado** | Opción de cifrado con AWS KMS al crear el volumen |
| **Facturación** | Pagas por toda la capacidad aprovisionada (no solo por lo que usas) |

### ¿Cómo Funciona?

```
┌─────────────────────────────┐
│   Zona Disponibilidad       │
│        us-east-1a           │
│                             │
│  ┌────────────────┐         │
│  │  EC2 Instance  │ ──┐     │
│  │                │  │     │
│  └────────────────┘  │ Adjunto
│                      │     │
│        ┌──────────┐  │     │
│        │EBS 10 GB │←─┘     │
│        └──────────┘  │     │
│                      │     │
│  ┌────────────────┐  │     │
│  │  EC2 Instance  │  │     │
│  │                │  │     │
│  └────────────────┘  │     │
│                      │No adjunto
│        ┌──────────┐  │     │
│        │EBS 50 GB │  │     │
│        └──────────┘  │     │
│                      │     │
└─────────────────────────────┘
```

### Tipos de Volúmenes EBS

AWS ofrece diferentes tipos de volúmenes EBS optimizados para distintos casos de uso:

#### SSD de Propósito General (gp3/gp2)

- **Uso:** Equilibrio entre precio y rendimiento
- **IOPS:** hasta 16,000
- **Casos de uso:** Volúmenes de arranque, desarrollo/pruebas, aplicaciones de bajo rendimiento

#### SSD de IOPS Provisionadas (io2/io1)

- **Uso:** Alto rendimiento para cargas críticas
- **IOPS:** hasta 64,000 (io2) o 64,000 (io1)
- **Casos de uso:** Bases de datos críticas, aplicaciones que requieren IOPS sostenidas

#### HDD Optimizado para Throughput (st1)

- **Uso:** Alto throughput a bajo costo
- **Throughput:** hasta 500 MB/s
- **Casos de uso:** Big Data, data warehouses, procesamiento de logs

#### HDD Cold (sc1)

- **Uso:** Almacenamiento de bajo costo
- **Throughput:** hasta 250 MB/s
- **Casos de uso:** Datos a los que se accede con poca frecuencia

### Atributo "Delete on Termination" (Borrar al Terminar)

Por defecto:

- ✅ El volumen **root** (raíz) se **elimina** cuando terminas la instancia
- ❌ Los volúmenes **adicionales** **NO se eliminan** cuando terminas la instancia

Puedes cambiar este comportamiento en la configuración de la instancia.

### Nivel Gratuito (Free Tier)

AWS ofrece **30 GB** de almacenamiento EBS gratuito al mes (tipo gp2 o Magnetic) durante 12 meses.

### Casos de Uso Típicos

- 📁 Bases de datos (MySQL, PostgreSQL, Oracle)
- 📂 Sistemas de archivos persistentes
- 📊 Aplicaciones de misión crítica
- 📝 Almacenamiento de logs del sistema
- 💾 Volúmenes de arranque (boot) para instancias EC2

---

## 2. Acerca de EBS Multi-Attach

**EBS Multi-Attach** es una característica que permite adjuntar un mismo volumen EBS a **múltiples instancias EC2** al mismo tiempo dentro de la **misma zona de disponibilidad**.

### Requisitos y Limitaciones

| Requisito | Detalle |
|---|---|
| **Tipo de Volumen** | Solo disponible para volúmenes **io1** e **io2** (IOPS provisionadas) |
| **Zona de Disponibilidad** | Todas las instancias deben estar en la **misma AZ** |
| **Sistema de Archivos** | Requiere un sistema de archivos compatible con acceso concurrente |
| **Número de Instancias** | Hasta 16 instancias Linux EC2 por volumen |

### Arquitectura de EBS Multi-Attach

```
┌──────────────────────────────────────┐
│  Zona de Disponibilidad us-east-1a   │
│                                      │
│  ┌──────────────────┐                │
│  │ Instance 1       │                │
│  └────────┬─────────┘                │
│           │                          │
│  ┌────────┴──────────┐   ┌────────┐ │
│  │ Instance 2       │───┤EBS Vol │ │
│  └────────┬──────────┘   │ io2    │ │
│           │              │Multi   │ │
│  ┌────────┴──────────┐   │Attach  │ │
│  │ Instance 3       │───┤        │ │
│  └──────────────────┘   └────────┘ │
│                                      │
└──────────────────────────────────────┘
```

### Sistemas de Archivos Compatibles

Para usar EBS Multi-Attach necesitas un **sistema de archivos cluster** como:

- **GFS2** (Global File System 2)
- **Lustre**
- **OCFS2** (Oracle Cluster File System)

⚠️ **Nota importante:** Los sistemas de archivos estándar como ext4, XFS o NTFS **NO** soportan escrituras concurrentes y causarán corrupción de datos.

### Casos de Uso

- 🏢 **Aplicaciones cluster** que requieren acceso compartido
- 📊 **Bases de datos cluster** como Oracle RAC
- 📄 **Alta disponibilidad** de aplicaciones que requieren failover rápido
- 📈 **Aplicaciones que requieren mayor disponibilidad** en una única AZ

### ⚠️ Importante para el Examen

Esta característica **NO es parte del examen Cloud Practitioner**, pero es útil conocerla para roles más avanzados (Solutions Architect, SysOps Administrator).

---

## 3. EBS - Práctica

En esta sección aprenderás a crear y administrar volúmenes EBS desde la consola de AWS.

### Paso 1: Acceder a la Consola de EBS

1. Inicia sesión en la **Consola de AWS**
2. Navega a: **EC2 > Elastic Block Store > Volúmenes**
3. Observa los volúmenes existentes (incluido el volumen root de tus instancias)

### Paso 2: Crear un Nuevo Volumen EBS

1. Click en "Crear Volumen" (Create Volume)
2. Configurar los parámetros:

```
Tipo de Volumen: gp3
Tamaño: 2 GiB
IOPS: 3000 (predeterminado)
Throughput: 125 MB/s
Zona de Disponibilidad: us-east-1a (misma que EC2)
Cifrado: Deshabilitado
Tags:
  Nombre: demo-volume
```

3. Click en "Crear Volumen" (Create Volume)

### Paso 3: Adjuntar el Volumen a una Instancia EC2

Una vez creado el volumen:

1. Selecciona el volumen creado
2. Click en "Acciones" > "Asociar Volumen" (Attach Volume)
3. Configurar:

```
Instancia: Selecciona tu EC2
Dispositivo: /dev/sdf
(AWS lo ajustará automáticamente si es necesario)
```

4. Click en "Asociar" (Attach)

### Paso 4: Verificar desde la Instancia EC2

Conéctate a tu instancia y verifica:

```bash
# Listar todos los dispositivos de bloque
lsblk

# Salida esperada:
# NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
# xvda    202:0    0   8G  0 disk
# └─xvda1 202:1    0   8G  0 part /
# xvdf    202:80   0   2G  0 disk  # <- Tu nuevo volumen
```

### Paso 5: Formatear y Montar el Volumen (Opcional)

```bash
# 1. Verificar que el volumen no tiene sistema de archivos
sudo file -s /dev/xvdf

# 2. Crear sistema de archivos ext4
sudo mkfs -t ext4 /dev/xvdf

# 3. Crear punto de montaje
sudo mkdir /data

# 4. Montar el volumen
sudo mount /dev/xvdf /data

# 5. Verificar que está montado
df -h

# 6. Para montaje permanente, editar /etc/fstab
sudo nano /etc/fstab
# Agregar: /dev/xvdf  /data  ext4  defaults,nofail  0  2
```

### Paso 6: Desadjuntar un Volumen

Para desadjuntar un volumen:

1. Desde la instancia EC2, desmontar primero:
   ```bash
   sudo umount /data
   ```

2. En la Consola de AWS:
   - Selecciona el volumen
   - Click en "Acciones" > "Desasociar Volumen"
   - Confirmar la operación

⚠️ **IMPORTANTE:** Siempre desmonta primero desde el SO antes de desadjuntar desde la consola

### Paso 7: Modificar un Volumen Existente

Puedes modificar un volumen sin detener la instancia:

1. Selecciona el volumen
2. Click en "Acciones" > "Modificar Volumen"
3. Puedes cambiar:
   - ✓ Tipo de volumen (gp2 → gp3)
   - ✓ Tamaño (2 GB → 10 GB)
   - ✓ IOPS (solo io1/io2/gp3)
   - ✓ Throughput (solo gp3)

4. Click en "Modificar"

Después de modificar el tamaño, necesitas extender el sistema de archivos:

```bash
# Para ext4
sudo resize2fs /dev/xvdf

# Para XFS
sudo xfs_growfs -d /data
```

### 🎯 Ejercicio Práctico

Completa los siguientes pasos:

1. ✅ Crea un volumen EBS de 5 GB tipo gp3
2. ✅ Adjúntalo a una instancia EC2 activa
3. ✅ Conéctate por SSH y verifica con `lsblk`
4. ✅ Formatea el volumen con ext4
5. ✅ Móntalo en `/datos`
6. ✅ Crea un archivo de prueba: `echo "Hola EBS" > /datos/prueba.txt`
7. ✅ Verifica que el archivo persiste después de reiniciar la instancia

### ⚠️ Notas Importantes

| Concepto | Detalle |
|---|---|
| **Volumen Root** | Se elimina por defecto al terminar la instancia (puedes cambiar esto) |
| **Volúmenes Adicionales** | NO se eliminan por defecto al terminar la instancia |
| **Zona de Disponibilidad** | Un volumen solo puede adjuntarse a instancias en la misma AZ |
| **Snapshots** | Usa snapshots para mover volúmenes entre AZs o regiones |

---

## 4. Visión General de EBS Snapshots

Un **Snapshot de EBS** es una copia de seguridad puntual (point-in-time) de un volumen EBS. Los snapshots son **incrementales**, lo que significa que solo se guardan los bloques que han cambiado desde el último snapshot.

### ¿Qué son los Snapshots?

```
┌─────────────────────────────────┐
│         Línea de Tiempo          │
├─────────────────────────────────┤
│                                 │
│  Día 1: Volumen EBS (10 GB)     │
│         ↓                        │
│    [Snapshot 1: 10 GB]          │
│                                 │
│  Día 2: Cambios (+ 2 GB nuevos) │
│         ↓                        │
│    [Snapshot 2: +2 GB] ← Inc.   │
│                                 │
│  Día 3: Cambios (+ 1 GB nuevos) │
│         ↓                        │
│    [Snapshot 3: +1 GB] ← Inc.   │
│                                 │
└─────────────────────────────────┘

Total almacenado: 13 GB (no 30 GB)
```

### Características Principales

| Característica | Descripción |
|---|---|
| **Incremental** | Solo guarda los bloques modificados desde el último snapshot |
| **Almacenamiento** | Se almacenan en Amazon S3 (gestionado automáticamente por AWS) |
| **Región** | Puede copiarse a otras regiones |
| **Zona de Disponibilidad** | Puede restaurarse en cualquier AZ dentro de la región |
| **Cifrado** | Los snapshots de volúmenes cifrados están automáticamente cifrados |

### Beneficios de los Snapshots

✅ **Backup y Recuperación**
- Protección contra pérdida de datos
- Restauración rápida de volúmenes

✅ **Migración**
- Mover datos entre zonas de disponibilidad
- Copiar datos a otras regiones

✅ **Creación de AMIs**
- Base para crear Amazon Machine Images

✅ **Ahorro de Costos**
- Almacenamiento incremental reduce costos
- Solo pagas por los datos únicos

### Características Avanzadas de Snapshots

#### EBS Snapshot Archive (Archivo)

Mueve snapshots antiguos a un nivel de almacenamiento de menor costo:

| Aspecto | Standard | Archive |
|---|---|---|
| **Costo** | Standard | 75% más barato |
| **Restauración** | Instantánea | 24-72 horas |
| **Caso de uso** | Snapshots frecuentes | Snapshots antiguos raramente accedidos |

#### Recycle Bin (Papelera de Reciclaje)

Protege contra eliminación accidental de snapshots:

```
┌──────────────────────────────────┐
│ Snapshot eliminado accidentalmente │
│              ↓                     │
│ Movido a Recycle Bin automáticamente
│              ↓                     │
│ Retención: 1 día - 1 año          │
│              ↓                     │
│ Puede recuperarse durante ese tiempo
└──────────────────────────────────┘
```

Configuración de Recycle Bin:
- Retención mínima: 1 día
- Retención máxima: 1 año
- Se aplica a nivel de región
- Puede configurarse por etiquetas (tags)

#### Fast Snapshot Restore (FSR)

Acelera la restauración de snapshots:

| Sin FSR | Con FSR |
|---|---|
| Restauración gradual | Restauración instantánea |
| Puede tardar horas para volúmenes grandes | Rendimiento completo desde el inicio |
| Sin costo adicional | $0.75/hora por snapshot por AZ |

### Copiar Snapshots Entre Regiones

Puedes copiar snapshots a otras regiones para:

- 🌍 Recuperación ante desastres
- 🚀 Expansión global de tu aplicación
- 📋 Cumplimiento normativo (datos en múltiples regiones)

```
┌──────────────────┐         ┌──────────────────┐
│  us-east-1       │ Copiar  │  eu-west-1       │
│                  │────────→│                  │
│  Snapshot        │         │  Snapshot        │
│  snap-abc123     │         │  snap-def456     │
└──────────────────┘         └──────────────────┘
```

### ðŸ'¡ Mejores Prácticas

1. ✅ **No necesitas desadjuntar el volumen** para crear un snapshot, pero es recomendado para garantizar consistencia
2. ✅ **Programa snapshots regulares** para protección de datos críticos
3. ✅ **Usa etiquetas** para organizar y gestionar snapshots
4. ✅ **Elimina snapshots antiguos** que ya no necesites (ahorra costos)
5. ✅ **Copia snapshots críticos** a otra región para disaster recovery
6. ✅ **Habilita Recycle Bin** para protección contra eliminación accidental

---

## 5. EFS (Elastic File System)

**Amazon EFS** es un sistema de archivos NFS (Network File System) totalmente administrado, escalable y elástico para usar con servicios de AWS Cloud. A diferencia de EBS, EFS puede montarse en **múltiples instancias EC2 simultáneamente**.

### ¿Qué es EFS?

```
EBS (Almacenamiento de Bloques):
┌────────────────────────┐
│ Volumen EBS (10 GB)    │
│        │                │
│        ↓                │
│   EC2 Instance          │
│   (1 a 1)              │
└────────────────────────┘

EFS (Sistema de Archivos Compartido):
┌────────────────────────────────┐
│      EFS File System           │
│            │                    │
│     ┌──────┴────┬────┬────┐   │
│     ↓          ↓    ↓    ↓    │
│   EC2-1    EC2-2 EC2-3 EC2-4  │
│   (Many to Many)               │
└────────────────────────────────┘
```

### Características Principales

| Característica | Descripción |
|---|---|
| **Sistema de archivos compartido** | Múltiples instancias EC2 pueden acceder simultáneamente |
| **Totalmente administrado** | AWS gestiona hardware, parches, backups |
| **Altamente disponible** | Datos replicados en múltiples AZs |
| **Escalable** | Crece y se reduce automáticamente (hasta petabytes) |
| **Rendimiento** | Puede escalar a miles de conexiones concurrentes |
| **Compatible con NFS** | Protocolo estándar NFSv4.1 |
| **Solo para Linux** | Compatible con instancias EC2 Linux, no Windows |
| **Pago por uso** | Solo pagas por el almacenamiento que uses |

### Clases de Almacenamiento en EFS

EFS ofrece dos clases de almacenamiento:

| Clase | Descripción | Costo | Uso |
|---|---|---|---|
| **EFS Standard** | Acceso frecuente | ~$0.30/GB/mes | Archivos accedidos regularmente |
| **EFS Infrequent Access (IA)** | Acceso poco frecuente | ~$0.025/GB/mes | Archivos accedidos raramente |

**Ahorro con EFS-IA:** Hasta 92% comparado con Standard

### Modos de Rendimiento

EFS ofrece dos modos de rendimiento:

| Modo | Descripción | Latencia | Throughput | Uso |
|---|---|---|---|---|
| **General Purpose** | Balanceado | Baja | Hasta 7,000 ops/seg por FS | Mayoría de casos de uso |
| **Max I/O** | Alto throughput | Mayor | +500,000 ops/seg | Big data, procesamiento masivo |

### Casos de Uso de EFS

#### Aplicaciones Web Escalables

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Web-1    │  │ Web-2    │  │ Web-3    │
└─────┬────┘  └─────┬────┘  └─────┬────┘
      │             │             │
      └─────────────┴─────────────┘
                    │
              ┌─────┴─────┐
              │    EFS    │
              │ /var/www  │
              └───────────┘

Beneficio: Contenido consistente en todos los servidores
```

#### Directorios Home Compartidos

Múltiples usuarios acceden a sus directorios home desde diferentes servidores

#### Content Management Systems (CMS)

WordPress, Drupal, Joomla en múltiples instancias

#### Desarrollo y Testing

Equipos de desarrollo comparten código y recursos

#### Machine Learning

Múltiples instancias de entrenamiento acceden a datasets compartidos

#### Big Data y Analytics

Cluster de procesamiento con datos compartidos

### ðŸ'¡ Mejores Prácticas para EFS

1. ✅ **Usa General Purpose mode** para la mayoría de casos
2. ✅ **Habilita Lifecycle Management** para ahorrar costos
3. ✅ **Usa Elastic Throughput** para workloads impredecibles
4. ✅ **Monta desde la misma AZ** para menor latencia
5. ✅ **Cifra datos sensibles** (en reposo y tránsito)
6. ✅ **Usa Access Points** para aislar aplicaciones
7. ✅ **Implementa Security Groups restrictivos** (solo puerto 2049)
8. ✅ **Monitorea métricas** en CloudWatch

---

## 6. Comparativa: EBS vs EFS vs Instance Store vs S3

| Característica | EBS | EFS | Instance Store | S3 |
|---|---|---|---|---|
| **Tipo** | Bloques | Archivos NFS | Bloques | Objeto |
| **Protocolo** | N/A | NFS | N/A | HTTP(S) |
| **Acceso** | Instancia única | Múltiples instancias | Múltiples instancias | Global |
| **Persistencia** | ✅ Sí | ✅ Sí | ❌ No | ✅ Sí |
| **Disponibilidad** | 1 AZ | Multi-AZ | 1 AZ | Global |
| **Escalamiento** | Manual | Automático | Fijo | Ilimitado |
| **Precio** | $0.08/GB/mes | $0.30/GB/mes | Incluido | $0.023/GB/mes |
| **Latencia** | ~ms | ~ms | ~μs | ~100ms |
| **Casos de uso** | BD, boot | Compartido | Cache temp | Backups |

---

## 7. Modelo de Responsabilidad Compartida para el Almacenamiento EC2

El **Modelo de Responsabilidad Compartida** de AWS define claramente qué aspectos de seguridad y operación gestiona AWS y cuáles son responsabilidad del cliente.

### Concepto General

```
┌──────────────────────────────────────────┐
│ MODELO DE RESPONSABILIDAD COMPARTIDA      │
├──────────────────────────────────────────┤
│                                          │
│  AWS               CLIENTE               │
│  ████              ████████             │
│  Seguridad         Seguridad             │
│  DEL Cloud         EN el Cloud           │
│                                          │
└──────────────────────────────────────────┘
```

### Responsabilidades de AWS (DEL Cloud)

AWS es responsable de la **infraestructura** que ejecuta todos los servicios de AWS.

### Responsabilidades del Cliente (EN el Cloud)

El cliente es responsable de la **seguridad y configuración** de lo que construye en AWS.

### Matriz de Responsabilidades por CategorÍa

#### Seguridad

| Aspecto | AWS | Cliente |
|---|---|---|
| Seguridad física | ✅ | ❌ |
| Seguridad de red (infraestructura) | ✅ | ❌ |
| Seguridad de red (configuración) | ❌ | ✅ |
| Cifrado en reposo (capacidad) | ✅ | ❌ |
| Cifrado en reposo (activación) | ❌ | ✅ |
| Cifrado en tránsito (capacidad) | ✅ | ❌ |
| Cifrado en tránsito (configuración) | ❌ | ✅ |
| Gestión de claves | ✅ (infra) | ✅ (uso) |

### Checklist de Responsabilidades del Cliente

**Configuración Inicial:**
- ☑ Cifrar volúmenes EBS con KMS
- ☑ Configurar Security Groups apropiados
- ☑ Establecer políticas IAM de acceso
- ☑ Etiquetar todos los recursos

**Operaciones Continuas:**
- ☑ Crear snapshots regularmente (EBS)
- ☑ Copiar snapshots a otra región (DR)
- ☑ Configurar Lifecycle policies (EFS)
- ☑ Eliminar recursos no usados

**Monitoreo y Alertas:**
- ☑ Configurar alarmas CloudWatch
- ☑ Revisar métricas de uso/rendimiento
- ☑ Auditar accesos con CloudTrail
- ☑ Verificar cumplimiento con Config

**Disaster Recovery:**
- ☑ Documentar procedimientos de restauración
- ☑ Probar restauraciones periódicamente
- ☑ Mantener AMIs actualizadas
- ☑ Plan de failover a otra región

**Seguridad:**
- ☑ Rotar claves de cifrado
- ☑ Revisar permisos IAM regularmente
- ☑ Actualizar Security Groups según sea necesario
- ☑ Eliminar snapshots/AMIs públicos no intencionales

---

## Resumen - Almacenamiento de Instancias EC2

### EBS (Elastic Block Store)

- **Descripción:** Almacenamiento persistente a nivel de bloque
- **Persistencia:** Datos se conservan después de detener/terminar la instancia
- **Disponibilidad:** Replicado automáticamente dentro de la AZ
- **Escalabilidad:** Puede aumentarse dinámicamente
- **Casos de uso:** Bases de datos, aplicaciones críticas, sistemas de archivos
- **Tipos:** gp3/gp2, io1/io2, st1/sc1

### EBS Snapshots

- **Descripción:** Copias de seguridad incrementales de volúmenes EBS
- **Almacenamiento:** Se guardan en S3
- **Casos de uso:** Backup, migración, DR
- **Características:** Papelera de reciclaje, archivo a bajo costo

### Instance Store

- **Descripción:** Almacenamiento temporal incluido con la instancia
- **Persistencia:** Datos se pierden al detener/terminar
- **Rendimiento:** Muy rápido (acceso directo)
- **Casos de uso:** Cachés, datos de sesión, archivos temporales

### EFS (Elastic File System)

- **Descripción:** Sistema de archivos NFS compartido
- **Acceso:** Multi-AZ automático
- **Escalabilidad:** Crecimiento automático
- **Clases:** Standard e Infrequent Access (IA)
- **Casos de uso:** Almacenamiento compartido, colaborativo

---

## Herramientas Recomendadas

- [Documentación oficial de EBS](https://docs.aws.amazon.com/ebs/)
- [Documentación oficial de EFS](https://docs.aws.amazon.com/efs/)
- [AWS Pricing Calculator](https://calculator.aws/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---
***
## Notas Finales

Este material cubre la Introducción a Intancias EC2 para el examen AWS Certified Cloud Practitioner. Practica los conceptos en tu propia cuenta de AWS (usa la capa gratuita) para reforzar el aprendizaje.

¡Buena suerte en tu examen!

**Última actualización**: 22/10/2025  
**Versión del curso**: AWS Certified Cloud Practitioner CLF-C02