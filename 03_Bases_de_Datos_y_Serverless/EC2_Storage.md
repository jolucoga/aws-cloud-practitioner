1. Visión General de EBS (Elastic Block Store)
Amazon EBS (Elastic Block Store) es un servicio de almacenamiento en bloque de alto rendimiento diseñado para su uso con Amazon EC2. Proporciona volúmenes de almacenamiento persistentes que pueden adjuntarse a instancias EC2.
¿Qué es un Volumen EBS?
Un volumen EBS es una unidad de red (no un disco físico) que puedes adjuntar a tus instancias mientras se ejecutan. Esto permite que tus datos persistan incluso después de que la instancia se detenga o termine.
Características Principales
CaracterísticaDescripciónPersistenteLos datos se conservan después de detener o terminar la instanciaNivel de BloqueProporciona almacenamiento a nivel de bloque, similar a un disco duro tradicionalEscalablePuedes aumentar la capacidad y rendimiento sin reiniciar la instanciaVinculado a una AZCada volumen está bloqueado en una Zona de Disponibilidad específicaAlta DisponibilidadReplicado automáticamente dentro de la misma zona de disponibilidadCifradoOpción de cifrado con AWS KMS al crear el volumenSnapshotsPermite crear copias de seguridad (almacenadas en S3)
Características de Red

Latencia: Al ser una unidad de red, puede haber algo de latencia en comparación con el almacenamiento físico
Separable: Se puede desconectar de una instancia EC2 y reconectarse a otra rápidamente
Una instancia a la vez: Por defecto, un volumen EBS solo puede montarse en una instancia a la vez (excepto con Multi-Attach)

Tipos de Volúmenes EBS
SSD (Solid State Drives)
TipoNombreCaracterísticasUso Idealgp3SSD de propósito general (última generación)• 3,000-16,000 IOPS<br>• 125-1,000 MB/s throughput<br>• Precio más económico que gp2Aplicaciones de propósito general, entornos de desarrollogp2SSD de propósito general• 3-16,000 IOPS<br>• Relación entre IOPS y tamaño<br>• 3 IOPS por GBVolúmenes de arranque, aplicaciones pequeñas y medianasio2 Block ExpressSSD de alto rendimiento (última generación)• Hasta 256,000 IOPS<br>• 99.999% durabilidad<br>• Relación 1000:1 IOPS:GBAplicaciones críticas de misiónio1SSD de alto rendimiento• 64,000 IOPS máximo<br>• 50 IOPS por GB<br>• 99.9% durabilidadBases de datos críticas, aplicaciones que requieren IOPS sostenidas
HDD (Hard Disk Drives)
TipoNombreCaracterísticasUso Idealst1HDD optimizado para throughput• 500 IOPS máximo<br>• 500 MB/s throughput máximo<br>• No puede ser volumen de arranqueBig Data, data warehouses, procesamiento de logssc1HDD de bajo costo• 250 IOPS máximo<br>• 250 MB/s throughput máximo<br>• No puede ser volumen de arranqueDatos de acceso poco frecuente, escenarios donde el menor costo es importante
Capacidad Provisionada

Debes provisionar la capacidad de antemano (tamaño en GBs e IOPS)
Se te facturará por toda la capacidad provisionada
Puedes aumentar la capacidad del volumen con el tiempo

Atributo "Delete on Termination"
┌─────────────────────────────────────────────────┐
│ Instancia EC2                                   │
│                                                 │
│  ┌──────────────┐         ┌──────────────┐    │
│  │ Volumen Root │         │ Volumen EBS  │    │
│  │   (Delete:   │         │  adicional   │    │
│  │   ✓ Enabled) │         │   (Delete:   │    │
│  │              │         │   ✗ Disabled)│    │
│  └──────────────┘         └──────────────┘    │
│         │                        │             │
│         ▼                        ▼             │
│   Se elimina al               Persiste         │
│   terminar instancia          después          │
└─────────────────────────────────────────────────┘
```

- **Por defecto**: 
  - El volumen raíz (root) se **elimina** cuando se termina la instancia (✓ habilitado)
  - Cualquier otro volumen EBS adjunto **NO se elimina** (✗ deshabilitado)
- Este comportamiento puede controlarse desde la consola de AWS o AWS CLI
- **Caso de uso**: Preservar el volumen root cuando se termina la instancia (desmarcar "Delete on termination")

### Casos de Uso Típicos

- Bases de datos relacionales y NoSQL
- Sistemas de archivos empresariales
- Aplicaciones de misión crítica
- Almacenamiento persistente de logs
- Volúmenes de arranque para instancias EC2

### Diagrama de Arquitectura
```
┌─────────────────────────────────────────────────────────────┐
│  Región: us-east-1                                          │
│                                                             │
│  ┌────────────────────────────────┐  ┌──────────────────┐ │
│  │ us-east-1a                     │  │ us-east-1b       │ │
│  │                                │  │                  │ │
│  │  ┌──────────┐   ┌──────────┐  │  │  ┌──────────┐   │ │
│  │  │ Instancia│───│   EBS    │  │  │  │ Instancia│   │ │
│  │  │   EC2    │   │ 100 GB   │  │  │  │   EC2    │   │ │
│  │  └──────────┘   └──────────┘  │  │  └──────────┘   │ │
│  │                                │  │       │         │ │
│  │  ┌──────────┐                 │  │  ┌────▼─────┐   │ │
│  │  │   EBS    │ (No adjunto)    │  │  │   EBS    │   │ │
│  │  │  10 GB   │                 │  │  │  50 GB   │   │ │
│  │  └──────────┘                 │  │  └──────────┘   │ │
│  └────────────────────────────────┘  └──────────────────┘ │
│                                                             │
│  ⚠️  Un volumen EBS en us-east-1a NO puede adjuntarse      │
│      a una instancia en us-east-1b                         │
└─────────────────────────────────────────────────────────────┘
```

### Facturación

- Se factura por:
  - GB provisionados por mes
  - IOPS provisionadas (para volúmenes io1/io2)
  - Throughput provisionado (para volúmenes gp3)
  - Snapshots almacenados en S3

---

## 2. Acerca de EBS Multi-Attach

**EBS Multi-Attach** es una característica que permite conectar un mismo volumen EBS a múltiples instancias EC2 **dentro de la misma zona de disponibilidad**.

### Características Principales
```
┌──────────────────────────────────────────────────────┐
│  Zona de Disponibilidad: us-east-1a                  │
│                                                       │
│  ┌─────────────┐                                     │
│  │ Instancia 1 │──┐                                  │
│  └─────────────┘  │                                  │
│                   │    ┌─────────────────┐           │
│  ┌─────────────┐  ├────│  Volumen EBS    │           │
│  │ Instancia 2 │──┤    │  Multi-Attach   │           │
│  └─────────────┘  │    │    (io1/io2)    │           │
│                   │    └─────────────────┘           │
│  ┌─────────────┐  │                                  │
│  │ Instancia 3 │──┘                                  │
│  └─────────────┘                                     │
│                                                       │
│  Hasta 16 instancias EC2 conectadas simultáneamente  │
└──────────────────────────────────────────────────────┘
```

### Requisitos

| Requisito | Detalle |
|-----------|---------|
| **Tipos de volumen** | Solo disponible para volúmenes **io1** e **io2** (Provisioned IOPS SSD) |
| **Ubicación** | Todas las instancias deben estar en la **misma Zona de Disponibilidad** |
| **Sistema de archivos** | Debe usar un sistema de archivos compatible con clústeres |
| **Límite de instancias** | Hasta **16 instancias EC2** por volumen |

### Sistemas de Archivos Compatibles

Para acceso concurrente, necesitas sistemas de archivos de clúster como:

- **XFS** (con gestión externa de acceso concurrente)
- **ext4** (con gestión externa de acceso concurrente)
- **GFS2** (Global File System 2)
- **OCFS2** (Oracle Cluster File System 2)

> ⚠️ **Advertencia**: Los sistemas de archivos estándar como ext4 o XFS sin configuración especial NO son seguros para acceso concurrente de escritura.

### Casos de Uso

1. **Aplicaciones de clúster** que requieren acceso concurrente a datos
2. **Bases de datos en clúster** (con software que soporte escrituras concurrentes)
3. **Aplicaciones que requieren alta disponibilidad** de aplicaciones

### Limitaciones

- No funciona con todos los tipos de volúmenes (solo io1/io2)
- Limitado a una zona de disponibilidad
- Requiere que la aplicación gestione la escritura concurrente
- No está disponible en todas las regiones

> ⚠️ **Nota para el examen**: Esta característica **no es parte del examen Cloud Practitioner**, pero es importante conocerla para roles más avanzados y certificaciones de nivel Associate o Professional.

### Comparación: EBS Estándar vs Multi-Attach

| Característica | EBS Estándar | EBS Multi-Attach |
|----------------|--------------|------------------|
| Instancias conectadas | 1 | Hasta 16 |
| Tipos de volumen | Todos | Solo io1/io2 |
| Zona de disponibilidad | Una AZ | Mismo requisito (una AZ) |
| Sistema de archivos | Cualquiera | Cluster-aware necesario |
| Caso de uso | General | Aplicaciones en clúster |

---

## 3. EBS - Práctica

En esta sección aprenderás a crear y gestionar volúmenes EBS a través de la consola de AWS.

### Objetivos de la Práctica

- Crear un volumen EBS
- Adjuntar un volumen a una instancia EC2
- Verificar el volumen desde la instancia
- Entender el atributo "Delete on Termination"

### Pasos Detallados

#### Paso 1: Acceder a la Consola de EBS
```
1. Inicia sesión en la Consola de AWS
2. Navega a: EC2 > Elastic Block Store > Volumes
   
   O busca "EBS" en la barra de búsqueda superior
```

#### Paso 2: Crear un Nuevo Volumen
```
1. Click en "Create Volume" (Crear volumen)

2. Configurar las propiedades:
   
   ┌────────────────────────────────────────────┐
   │ Volume Settings                            │
   ├────────────────────────────────────────────┤
   │ Volume Type:        [gp3 ▼]               │
   │ Size (GiB):         [2]                    │
   │ IOPS:               [3000] (automático)    │
   │ Throughput (MB/s):  [125]  (automático)    │
   │ Availability Zone:  [us-east-1a ▼]        │
   │ Snapshot ID:        (Opcional)             │
   │                                            │
   │ Encryption:         [☐] Encrypt this      │
   │                          volume            │
   │                                            │
   │ Tags:               Key: Name              │
   │                     Value: Mi-Volumen-EBS  │
   └────────────────────────────────────────────┘

3. Click en "Create Volume"
```

#### Paso 3: Detalles de Configuración

| Campo | Descripción | Recomendación |
|-------|-------------|---------------|
| **Volume Type** | Tipo de volumen EBS | `gp3` para propósito general (mejor precio-rendimiento) |
| **Size** | Tamaño en GiB | `2 GiB` para pruebas (dentro de free tier) |
| **Availability Zone** | Zona de disponibilidad | Debe ser la **misma AZ** que tu instancia EC2 |
| **Snapshot ID** | Para crear desde snapshot | Dejar vacío para volumen nuevo |
| **Encryption** | Cifrar el volumen | Recomendado para datos sensibles |

#### Paso 4: Adjuntar el Volumen a una Instancia EC2
```
1. Una vez creado el volumen, selecciónalo en la lista

2. Click en "Actions" > "Attach Volume"

3. Configurar el adjunto:
   
   ┌────────────────────────────────────────────┐
   │ Attach Volume                              │
   ├────────────────────────────────────────────┤
   │ Instance:     [i-1234567890abcdef0 ▼]     │
   │               (my-ec2-instance)            │
   │                                            │
   │ Device name:  [/dev/sdf ▼]                │
   │               (sugerido por AWS)           │
   │                                            │
   │ ⓘ El nombre del dispositivo puede variar  │
   │   dentro del sistema operativo             │
   └────────────────────────────────────────────┘

4. Click en "Attach Volume"

5. Verifica el estado: debe cambiar a "in-use"
```

#### Paso 5: Verificar desde la Instancia EC2

Una vez adjunto el volumen, verifica desde la consola de EC2:
```
1. Navega a: EC2 > Instances
2. Selecciona tu instancia
3. Tab: "Storage" o "Almacenamiento"
4. Deberías ver:
   
   ┌────────────────────────────────────────────┐
   │ Block devices                              │
   ├─────────────────┬──────────────────────────┤
   │ Device name     │ Volume ID                │
   ├─────────────────┼──────────────────────────┤
   │ /dev/xvda       │ vol-0abc123def (Root)   │
   │ /dev/sdf        │ vol-0xyz789ghi (EBS)    │
   └─────────────────┴──────────────────────────┘
Paso 6: Formatear y Montar el Volumen (Opcional - Avanzado)
Si deseas usar el volumen, conéctate por SSH a la instancia:
bash# 1. Verificar los dispositivos disponibles
lsblk

# Salida esperada:
# NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
# xvda    202:0    0   8G  0 disk 
# └─xvda1 202:1    0   8G  0 part /
# xvdf    202:80   0   2G  0 disk    ← Nuestro nuevo volumen

# 2. Verificar si tiene sistema de archivos
sudo file -s /dev/xvdf

# Si devuelve "data", está vacío y necesita formato

# 3. Crear sistema de archivos (⚠️ SOLO para volúmenes nuevos)
sudo mkfs -t ext4 /dev/xvdf

# 4. Crear punto de montaje
sudo mkdir /data

# 5. Montar el volumen
sudo mount /dev/xvdf /data

# 6. Verificar que está montado
df -h

# 7. (Opcional) Montaje automático al reiniciar
# Editar /etc/fstab
sudo cp /etc/fstab /etc/fstab.backup
echo '/dev/xvdf /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
```

#### Paso 7: Gestionar el Atributo "Delete on Termination"
```
1. Selecciona tu instancia EC2
2. Tab: "Storage"
3. Click en el ID del volumen
4. Actions > Modify attribute

   O desde el lanzamiento de la instancia:
   
   ┌────────────────────────────────────────────┐
   │ Configure storage                          │
   ├────────────────────────────────────────────┤
   │ Volume 1 (Root)                            │
   │ Size:                  8 GiB               │
   │ Volume Type:           gp3                 │
   │ [☑] Delete on Termination  ← AQUÍ        │
   │                                            │
   │ Volume 2 (Additional)                      │
   │ Size:                  2 GiB               │
   │ Volume Type:           gp3                 │
   │ [☐] Delete on Termination  ← AQUÍ        │
   └────────────────────────────────────────────┘
```

### Notas Importantes

| ⚠️ Advertencia | Descripción |
|---------------|-------------|
| **Zona de Disponibilidad** | El volumen debe estar en la **misma AZ** que la instancia |
| **Volumen Root** | Por defecto se elimina al terminar la instancia (a menos que se deshabilite) |
| **Facturación** | Se te cobra por el almacenamiento provisionado, incluso si no está adjunto a ninguna instancia |
| **Snapshots antes de cambios** | Siempre crea un snapshot antes de modificar volúmenes en producción |

### Mejores Prácticas

✅ **Hacer**:
- Etiquetar volúmenes con nombres descriptivos
- Crear snapshots regulares de volúmenes importantes
- Monitorear el uso del volumen con CloudWatch
- Cifrar volúmenes que contengan datos sensibles

❌ **Evitar**:
- Dejar volúmenes no adjuntos (siguen costando dinero)
- Eliminar volúmenes sin hacer snapshot de los datos importantes
- Usar volúmenes más grandes de lo necesario

### Limpieza de Recursos

Para evitar cargos innecesarios:
```
1. Desmontar el volumen desde la instancia (si está montado)
   sudo umount /data

2. Desadjuntar el volumen:
   EC2 > Volumes > Seleccionar volumen
   Actions > Detach Volume

3. Eliminar el volumen:
   Actions > Delete Volume
   
   ⚠️ Esta acción es irreversible. Crea un snapshot si necesitas
      conservar los datos.
```

---

## 4. Visión General de EBS Snapshots

Un **EBS Snapshot** es una copia de seguridad incremental de un volumen EBS en un momento específico en el tiempo. Los snapshots se almacenan en Amazon S3 (administrado automáticamente por AWS).

### ¿Qué es un Snapshot?
```
┌─────────────────────────────────────────────────────────┐
│  Línea de Tiempo del Volumen EBS                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Volumen EBS (100 GB)                                   │
│  ┌──────────┬──────────┬──────────┬──────────┐         │
│  │ Día 1    │ Día 2    │ Día 3    │ Día 4    │         │
│  │ 10GB     │ +5GB     │ +3GB     │ +2GB     │         │
│  └────┬─────┴──────────┴────┬─────┴────┬─────┘         │
│       │                     │          │                │
│       ▼                     ▼          ▼                │
│  Snapshot 1            Snapshot 2  Snapshot 3           │
│  (10 GB)               (+5 GB)     (+2 GB)              │
│  Completo              Incremental Incremental          │
│                                                          │
│  Almacenado en Amazon S3 (gestionado por AWS)          │
└─────────────────────────────────────────────────────────┘
```

### Características Principales

| Característica | Descripción |
|----------------|-------------|
| **Incremental** | Solo se guardan los bloques que han cambiado desde el último snapshot |
| **Point-in-time** | Captura el estado del volumen en un momento específico |
| **Almacenamiento** | Se almacenan en Amazon S3 (transparente para el usuario) |
| **Regional** | Los snapshots son regionales, pero pueden copiarse entre regiones |
| **Restauración** | Puedes crear nuevos volúmenes EBS desde snapshots |
| **No requiere detener** | No necesitas detener la instancia, pero se recomienda para consistencia |

### Beneficios de los Snapshots

#### 1. Copia de Seguridad y Recuperación
```
Volumen EBS Original
       │
       │ snapshot
       ▼
   Snapshot
       │
       │ restore
       ▼
 Nuevo Volumen EBS
 (en la misma AZ o diferente)
```

#### 2. Migración entre Zonas de Disponibilidad
```
┌──────────────────────────────────────────────────────┐
│  Región: us-east-1                                   │
│                                                       │
│  ┌─────────────────┐         ┌─────────────────┐   │
│  │  us-east-1a     │         │  us-east-1b     │   │
│  │                 │         │                 │   │
│  │  ┌───────────┐  │         │  ┌───────────┐  │   │
│  │  │ Volumen   │  │         │  │  Nuevo    │  │   │
│  │  │ EBS       │  │         │  │  Volumen  │  │   │
│  │  │ Original  │  │         │  │  EBS      │  │   │
│  │  └─────┬─────┘  │         │  └─────▲─────┘  │   │
│  └────────┼────────┘         └────────┼────────┘   │
│           │                           │             │
│           │  ┌─────────────────┐     │             │
│           └──│   Snapshot      │─────┘             │
│              │ (S3 - Regional) │                   │
│              └─────────────────┘                   │
└──────────────────────────────────────────────────────┘
```

#### 3. Migración entre Regiones
```
┌────────────────┐         ┌────────────────┐
│  us-east-1     │         │  eu-west-1     │
│                │         │                │
│  ┌──────────┐  │         │  ┌──────────┐  │
│  │ Volumen  │  │         │  │  Nuevo   │  │
│  │ EBS      │  │         │  │  Volumen │  │
│  └────┬─────┘  │         │  └────▲─────┘  │
│       │        │         │       │        │
│  ┌────▼─────┐  │  Copy   │  ┌────┴─────┐  │
│  │Snapshot  │──┼────────▶│  │Snapshot  │  │
│  │ (S3)     │  │         │  │ (S3)     │  │
│  └──────────┘  │         │  └──────────┘  │
└────────────────┘         └────────────────┘
```

### Tipos de Snapshots y Características Especiales

#### EBS Snapshot Archive
```
┌─────────────────────────────────────────────────────┐
│  Ciclo de Vida del Snapshot                         │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Snapshot Estándar                                  │
│  (Almacenamiento Normal)                            │
│  Costo: $$                                          │
│  Restauración: Minutos                              │
│           │                                          │
│           │ Archive (Archivar)                      │
│           ▼                                          │
│  Snapshot Archivado                                 │
│  (Archive Tier - 75% más barato)                   │
│  Costo: $                                           │
│  Restauración: 24-72 horas                         │
│           │                                          │
│           │ Restore (Restaurar)                     │
│           ▼                                          │
│  Snapshot Estándar                                  │
│  (Listo para crear volúmenes)                      │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**Características del Archive**:
- Hasta **75% más barato** que el almacenamiento estándar
- Tiempo de restauración: **24 a 72 horas**
- Ideal para snapshots de retención a largo plazo
- Snapshots de cumplimiento normativo

#### Recycle Bin para Snapshots EBS
```
┌─────────────────────────────────────────────────────┐
│  Papelera de Reciclaje de EBS                       │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Usuario elimina       ┌──────────────────┐         │
│  snapshot          ────▶│  Recycle Bin     │         │
│  (accidental)           │                  │         │
│                         │  Retención:      │         │
│                         │  1 día - 1 año   │         │
│                         │                  │         │
│                         └────────┬─────────┘         │
│                                  │                   │
│                    ┌─────────────┼─────────────┐    │
│                    │             │             │    │
│              Recuperar      Período de    Eliminación│
│              snapshot       retención     permanente │
│              (Restore)      expira                   │
│                                                      │
└─────────────────────────────────────────────────────┘
Características de Recycle Bin:

Protección contra eliminaciones accidentales
Período de retención configurable: 1 día a 1 año
Se aplica mediante reglas de retención
Disponible para snapshots de EBS y AMIs
Sin costo adicional (solo pagas por el almacenamiento)

Mejores Prácticas con Snapshots
1. Frecuencia de Snapshots
Tipo de DatosFrecuencia RecomendadaDatos críticos de producciónCada 1-4 horasBases de datosCada 6-12 horasDatos de desarrolloDiariaDatos de archivoSemanal o mensual
2. Estrategia de Retención
┌─────────────────────────────────────────────────────┐
│  Estrategia de Retención 3-2-1                      │
├─────────────────────────────────────────────────────┤
│                                                      │
│  3 copias de los datos                              │
│  ├─ Original                                        │
│  ├─ Snapshot diario                                 │
│  └─ Snapshot semanal                                │RetryClaude does not have the ability to run the code it generates yet.LContinue│                                                      │
│  2 tipos de almacenamiento                          │
│  ├─ Snapshots estándar (acceso rápido)             │
│  └─ Snapshots archivados (retención largo plazo)   │
│                                                      │
│  1 copia fuera de sitio                             │
│  └─ Snapshot copiado a otra región                 │
│                                                      │
└─────────────────────────────────────────────────────┘

#### 3. Automatización de Snapshots

Puedes automatizar la creación de snapshots usando:

- **Amazon Data Lifecycle Manager (DLM)**
- **AWS Backup**
- **Scripts con AWS CLI**
- **AWS Lambda con EventBridge**

### Proceso de Creación de Snapshots
```
┌─────────────────────────────────────────────────────┐
│  Flujo de Creación de Snapshot                      │
├─────────────────────────────────────────────────────┤
│                                                      │
│  1. Volumen EBS en uso                              │
│     │                                                │
│     ▼                                                │
│  2. Iniciar snapshot                                │
│     (puede continuar usándose)                      │
│     │                                                │
│     ▼                                                │
│  3. Snapshot en progreso                            │
│     (pending → completed)                           │
│     │                                                │
│     ▼                                                │
│  4. Snapshot completado                             │
│     (almacenado en S3)                              │
│     │                                                │
│     ├─── Crear volumen                              │
│     ├─── Copiar a otra región                       │
│     ├─── Archivar                                   │
│     └─── Compartir con otras cuentas               │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Consideraciones de Rendimiento

| Aspecto | Detalle |
|---------|---------|
| **Durante el snapshot** | El volumen sigue siendo completamente utilizable |
| **Primer snapshot** | Es una copia completa del volumen (puede tardar más) |
| **Snapshots posteriores** | Solo copian los bloques modificados (más rápidos) |
| **Restauración** | Los volúmenes creados desde snapshots se "hidratan" en segundo plano |
| **Rendimiento inicial** | Los bloques se cargan bajo demanda (primera lectura puede ser más lenta) |

### Cifrado de Snapshots
```
┌─────────────────────────────────────────────────────┐
│  Reglas de Cifrado                                  │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Volumen NO cifrado                                 │
│         │                                            │
│         ├─ Snapshot NO cifrado                      │
│         └─ Puede crear snapshot cifrado (copia)     │
│                                                      │
│  Volumen cifrado                                    │
│         │                                            │
│         └─ Snapshot SIEMPRE cifrado                 │
│            (con la misma clave KMS)                 │
│                                                      │
│  Snapshot cifrado                                   │
│         │                                            │
│         └─ Volumen SIEMPRE cifrado                  │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Compartir Snapshots

Los snapshots pueden compartirse de varias formas:

1. **Privado**: Solo tu cuenta (por defecto)
2. **Específico**: Compartir con cuentas AWS específicas
3. **Público**: Hacer el snapshot público (⚠️ no recomendado para datos sensibles)
```
⚠️ IMPORTANTE: No puedes compartir snapshots cifrados públicamente.
   Para compartir snapshots cifrados, debes compartirlos con 
   cuentas específicas y dar permisos a la clave KMS.
```

### Costos de Snapshots

Los snapshots tienen un modelo de precios específico:
```
┌─────────────────────────────────────────────────────┐
│  Modelo de Costos                                   │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Almacenamiento estándar:  ~$0.05 por GB/mes       │
│  Archive tier:             ~$0.0125 por GB/mes      │
│                            (75% descuento)          │
│                                                      │
│  Ejemplo: Volumen de 100 GB con cambios del 10%    │
│                                                      │
│  Snapshot 1:  100 GB  →  $5.00/mes                 │
│  Snapshot 2:  +10 GB  →  $5.50/mes (total)         │
│  Snapshot 3:  +10 GB  →  $6.00/mes (total)         │
│                                                      │
│  Si se archiva Snapshot 1:                          │
│  Archive:     100 GB  →  $1.25/mes                 │
│  Estándar:    20 GB   →  $1.00/mes                 │
│  Total:                  $2.25/mes                  │
│                          (62.5% ahorro)             │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Tiempo de Restauración

| Tipo de Snapshot | Tiempo de Restauración | Caso de Uso |
|------------------|------------------------|-------------|
| **Estándar** | Minutos | Recuperación rápida, volúmenes activos |
| **Fast Snapshot Restore (FSR)** | Instantáneo (sin hidratación) | Aplicaciones críticas que requieren rendimiento completo inmediato |
| **Archivado** | 24-72 horas | Cumplimiento normativo, retención a largo plazo |

### Límites y Cuotas

| Recurso | Límite Predeterminado |
|---------|----------------------|
| Snapshots concurrentes | 5 por volumen |
| Snapshots por región | 100,000 |
| Copias de snapshots concurrentes | 5 |

> 💡 **Tip**: Estos límites pueden aumentarse contactando al soporte de AWS.

---

## 5. EBS Snapshots - Práctica

Aprende a crear, gestionar y restaurar snapshots de EBS a través de la consola de AWS.

### Objetivos de la Práctica

- Crear un snapshot manual de un volumen EBS
- Copiar un snapshot a otra región
- Restaurar un volumen desde un snapshot
- Configurar el Archive tier
- Usar Recycle Bin para recuperación

### Parte 1: Crear un Snapshot

#### Paso 1: Crear un Snapshot Manual
```
1. Navega a: EC2 > Elastic Block Store > Volumes

2. Selecciona el volumen del que quieres hacer snapshot

3. Click en "Actions" > "Create Snapshot"

   ┌────────────────────────────────────────────┐
   │ Create Snapshot                            │
   ├────────────────────────────────────────────┤
   │ Description:                               │
   │ [Snapshot de mi volumen de datos]         │
   │                                            │
   │ Tags (opcional):                           │
   │ Key: Name                                  │
   │ Value: [backup-datos-2024-01]             │
   │                                            │
   │ Key: Environment                           │
   │ Value: [Production]                        │
   │                                            │
   │ ⓘ Estimación de tamaño y costo:           │
   │   Tamaño: ~50 GB                           │
   │   Costo aproximado: $2.50/mes              │
   └────────────────────────────────────────────┘

4. Click en "Create Snapshot"

5. El estado inicial será "pending"
   pending → completed (puede tardar minutos u horas)
```

#### Paso 2: Monitorear el Progreso
```
1. Navega a: EC2 > Elastic Block Store > Snapshots

2. Busca tu snapshot en la lista
   
   ┌─────────────────────────────────────────────────────┐
   │ Snapshot ID         │ Status    │ Progress          │
   ├─────────────────────┼───────────┼───────────────────┤
   │ snap-0abc123def     │ pending   │ 45%               │
   │ snap-0xyz789ghi     │ completed │ 100%              │
   └─────────────────────────────────────────────────────┘

3. Refresca la página periódicamente hasta ver "completed"
```

### Parte 2: Restaurar un Volumen desde Snapshot

#### Paso 1: Crear Volumen desde Snapshot
```
1. En la lista de Snapshots, selecciona tu snapshot

2. Click en "Actions" > "Create Volume from Snapshot"

3. Configurar el nuevo volumen:
   
   ┌────────────────────────────────────────────┐
   │ Create Volume from Snapshot                │
   ├────────────────────────────────────────────┤
   │ Snapshot ID:     snap-0abc123def           │
   │                  (automático)              │
   │                                            │
   │ Volume Type:     [gp3 ▼]                  │
   │                  (puede cambiar)           │
   │                                            │
   │ Size (GiB):      [50]                      │
   │                  (puede aumentar, no       │
   │                   disminuir)               │
   │                                            │
   │ IOPS:            [3000]                    │
   │ Throughput:      [125] MB/s                │
   │                                            │
   │ Availability     [us-east-1a ▼]           │
   │ Zone:            ⚠️ IMPORTANTE: Elegir AZ │
   │                                            │
   │ Encryption:      [☑] Encrypt this volume  │
   │                  (si el snapshot estaba    │
   │                   cifrado)                 │
   │                                            │
   │ Tags:            Key: Name                 │
   │                  Value: restored-volume    │
   └────────────────────────────────────────────┘

4. Click en "Create Volume"

5. Una vez creado, puedes adjuntarlo a una instancia EC2
```

#### Paso 2: Adjuntar el Volumen Restaurado
```
1. Navega a: EC2 > Volumes

2. Selecciona el nuevo volumen creado

3. Actions > Attach Volume

4. Selecciona la instancia de destino

5. El volumen estará disponible para montar
```

### Parte 3: Copiar Snapshot a Otra Región

#### Paso 1: Copiar entre Regiones
```
1. En la lista de Snapshots, selecciona el snapshot

2. Actions > "Copy Snapshot"

3. Configurar la copia:
   
   ┌────────────────────────────────────────────┐
   │ Copy Snapshot                              │
   ├────────────────────────────────────────────┤
   │ Destination Region:                        │
   │ [eu-west-1 (Ireland) ▼]                   │
   │                                            │
   │ Description:                               │
   │ [Copia DR del snapshot de producción]     │
   │                                            │
   │ Encryption:                                │
   │ ○ Encrypt this snapshot                    │
   │   KMS Key: [Default ▼]                    │
   │                                            │
   │ ⓘ La transferencia de datos entre          │
   │   regiones tiene costo adicional           │
   │   (~$0.02 por GB)                          │
   └────────────────────────────────────────────┘

4. Click en "Copy Snapshot"

5. Cambia a la región de destino para ver el progreso
   (esquina superior derecha)
```

### Parte 4: Archivar Snapshots

#### Paso 1: Mover Snapshot a Archive Tier
```
1. Selecciona un snapshot que no necesites acceso rápido

2. Actions > "Archive Snapshot"

3. Confirmar:
   
   ┌────────────────────────────────────────────┐
   │ Archive Snapshot                           │
   ├────────────────────────────────────────────┤
   │ Snapshot ID:  snap-0abc123def              │
   │                                            │
   │ Current Cost: $5.00/mes                    │
   │ Archive Cost: $1.25/mes (75% ahorro)       │
   │                                            │
   │ ⚠️ Tiempo de restauración: 24-72 horas    │
   │                                            │
   │ ☑ Entiendo que el snapshot no estará      │
   │   disponible para restauración inmediata   │
   └────────────────────────────────────────────┘

4. Click en "Archive"

5. El estado cambiará a "archiving" → "archived"
```

#### Paso 2: Restaurar desde Archive
```
1. Selecciona un snapshot archivado

2. Actions > "Restore from Archive"

3. Configurar restauración:
   
   ┌────────────────────────────────────────────┐
   │ Restore Snapshot from Archive              │
   ├────────────────────────────────────────────┤
   │ Snapshot ID:  snap-0abc123def              │
   │                                            │
   │ Restore Type:                              │
   │ ○ Temporary restore (tempor restoration)  │
   │   Duration: [1-14 days]                    │
   │                                            │
   │ ○ Permanent restore                        │
   │   (vuelve a tier estándar)                 │
   │                                            │
   │ Estimated time: 24-72 hours                │
   │ Cost: $0.03 per GB restaurado              │
   └────────────────────────────────────────────┘

4. Click en "Restore"

5. Espera 24-72 horas para que complete
```

### Parte 5: Configurar Recycle Bin

#### Paso 1: Crear Regla de Retención
```
1. Navega a: EC2 > Elastic Block Store > Recycle Bin

2. Click en "Create retention rule"

3. Configurar la regla:
   
   ┌────────────────────────────────────────────┐
   │ Create Retention Rule                      │
   ├────────────────────────────────────────────┤
   │ Retention rule name:                       │
   │ [snapshot-protection-rule]                 │
   │                                            │
   │ Description:                               │
   │ [Protección contra eliminación accidental] │
   │                                            │
   │ Resource type:                             │
   │ ○ EBS snapshots                            │
   │ ○ EBS-backed AMIs                          │
   │                                            │
   │ Retention period:                          │
   │ [30] days                                  │
   │                                            │
   │ Resource tags (opcional):                  │
   │ Tag key: [Environment]                     │
   │ Tag value: [Production]                    │
   │                                            │
   │ ⓘ Solo snapshots con este tag serán        │
   │   protegidos por esta regla                │
   └────────────────────────────────────────────┘

4. Click en "Create Retention Rule"
```

#### Paso 2: Recuperar un Snapshot Eliminado
```
1. Navega a: EC2 > Recycle Bin

2. Busca el snapshot eliminado:
   
   ┌─────────────────────────────────────────────────┐
   │ Resources in Recycle Bin                        │
   ├──────────────┬──────────┬─────────────────────┤
   │ Resource ID  │ Type     │ Deletion date       │
   ├──────────────┼──────────┼─────────────────────┤
   │ snap-0abc... │ Snapshot │ 2024-01-15 10:30   │
   │ snap-0xyz... │ Snapshot │ 2024-01-14 15:45   │
   └──────────────┴──────────┴─────────────────────┘

3. Selecciona el snapshot

4. Actions > "Recover"

5. Confirmar la recuperación:
   
   ┌────────────────────────────────────────────┐
   │ Recover Resource                           │
   ├────────────────────────────────────────────┤
   │ Resource ID:  snap-0abc123def              │
   │                                            │
   │ El snapshot será restaurado a su estado   │
   │ original y estará disponible              │
   │ inmediatamente.                            │
   │                                            │
   │ ⓘ Retention period remaining: 25 days     │
   └────────────────────────────────────────────┘

6. Click en "Recover"

7. El snapshot reaparecerá en la lista de snapshots
```

### Parte 6: Automatización con Data Lifecycle Manager

#### Crear Política de Lifecycle
```
1. Navega a: EC2 > Elastic Block Store > Lifecycle Manager

2. Click en "Create lifecycle policy"

3. Configurar política:
   
   ┌────────────────────────────────────────────┐
   │ Create Lifecycle Policy                    │
   ├────────────────────────────────────────────┤
   │ Policy type:                               │
   │ ○ EBS snapshot policy                      │
   │                                            │
   │ Target resources:                          │
   │ ○ Volume                                   │
   │   Target with tags:                        │
   │   Tag: [Backup]                            │
   │   Value: [Daily]                           │
   │                                            │
   │ Schedule:                                  │
   │ Schedule name: [daily-backup]              │
   │ Frequency: [Daily]                         │
   │ Starting at: [02:00 UTC]                   │
   │                                            │
   │ Retention:                                 │
   │ Count: [7] (mantener últimos 7)           │
   │                                            │
   │ Cross-region copy (opcional):              │
   │ ☑ Enable                                   │
   │   Target region: [eu-west-1]              │
   │   Retention: [30] days                     │
   │                                            │
   │ Tags to add to snapshots:                  │
   │ Key: [AutomatedBackup]                     │
   │ Value: [True]                              │
   └────────────────────────────────────────────┘

4. Click en "Create Policy"
```

### Parte 7: Monitoreo y Verificación

#### Verificar Snapshots Creados
```bash
# Usando AWS CLI (opcional)

# Listar todos tus snapshots
aws ec2 describe-snapshots --owner-ids self

# Listar snapshots con un tag específico
aws ec2 describe-snapshots \
  --owner-ids self \
  --filters "Name=tag:Name,Values=backup-*"

# Ver el estado de un snapshot específico
aws ec2 describe-snapshots \
  --snapshot-ids snap-0abc123def

# Crear un snapshot desde CLI
aws ec2 create-snapshot \
  --volume-id vol-0abc123def \
  --description "Backup manual desde CLI" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=cli-backup}]'
```

### Mejores Prácticas Implementadas

| Práctica | Implementación |
|----------|----------------|
| **Etiquetado consistente** | Usa tags descriptivos (Name, Environment, BackupType) |
| **Automatización** | Configura DLM para backups automáticos |
| **Retención adecuada** | 7 días para diarios, 30 días para semanales |
| **Copia entre regiones** | Para recuperación ante desastres |
| **Archive tier** | Para snapshots de retención a largo plazo |
| **Recycle Bin** | Configurado con 30 días de retención |
| **Monitoreo** | Usa CloudWatch para alertas de fallos |

### Tabla de Costos Estimados (Ejemplo)
```
┌─────────────────────────────────────────────────────┐
│  Ejemplo: Volumen de 100 GB con cambio diario 5%   │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Día 1:  Snapshot completo (100 GB)  → $5.00/mes   │
│  Día 2:  +5 GB incremental           → $5.25/mes   │
│  Día 3:  +5 GB incremental           → $5.50/mes   │
│  ...                                                 │
│  Día 7:  +5 GB incremental           → $6.50/mes   │
│                                                      │
│  Con política de retención de 7 días:               │
│  Costo estable: ~$6.50/mes                          │
│                                                      │
│  Archivando snapshots > 30 días:                    │
│  Estándar (7 días):  $6.50/mes                      │
│  Archivado (30+ días): $3.00/mes                    │
│  Total: $9.50/mes                                   │
│  vs. Todo estándar: $22.50/mes                      │
│  Ahorro: 58%                                         │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Troubleshooting Común

| Problema | Solución |
|----------|----------|
| **Snapshot se queda en "pending"** | Espera más tiempo; snapshots grandes pueden tardar horas |
| **No puedo crear volumen en otra AZ** | Los snapshots son regionales; copia el snapshot si necesitas cambiar de región |
| **Error al restaurar snapshot archivado** | Primero debes restaurar del archive tier (24-72 horas) |
| **Snapshot eliminado no aparece en Recycle Bin** | Verifica que existe una regla de retención activa |
| **Costo inesperado de snapshots** | Revisa snapshots antiguos y archívalos o elimínalos |

### Limpieza de Recursos

Para evitar costos innecesarios:
```
1. Listar todos tus snapshots

2. Identificar snapshots innecesarios:
   - Snapshots de prueba
   - Snapshots muy antiguos sin tag de retención
   - Snapshots duplicados

3. Archivar snapshots de retención a largo plazo

4. Eliminar snapshots que ya no necesitas:
   Seleccionar snapshot > Actions > Delete Snapshot
   
   ⚠️ Asegúrate de que no lo necesitas; una vez eliminado
      (y pasado el período de Recycle Bin), no se puede recuperar

5. Verificar políticas de DLM están funcionando correctamente
```

---

## 6. Visión General de AMI (Amazon Machine Image)

Una **AMI (Amazon Machine Image)** es una plantilla preconfigurada que contiene el software, sistema operativo, configuraciones y datos necesarios para lanzar una instancia EC2.

### ¿Qué es una AMI?
```
┌─────────────────────────────────────────────────────┐
│  Componentes de una AMI                             │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────────────────────────────────┐         │
│  │  Sistema Operativo                     │         │
│  │  (Amazon Linux, Ubuntu, Windows, etc.) │         │
│  └────────────────────────────────────────┘         │
│                    │                                 │
│  ┌────────────────▼────────────────────┐            │
│  │  Software Preinstalado               │            │
│  │  (Apache, MySQL, Node.js, etc.)     │            │
│  └────────────────────────────────────┘            │
│                    │                                 │
│  ┌────────────────▼────────────────────┐            │
│  │  Configuraciones                     │            │
│  │  (Variables de entorno, configs)    │            │
│  └────────────────────────────────────┘            │
│                    │                                 │
│  ┌────────────────▼────────────────────┐            │
│  │  Datos (Opcional)                    │            │
│  │  (Archivos de aplicación)           │            │
│  └────────────────────────────────────┘            │
│                    │                                 │
│                    ▼                                 │
│          Plantilla lista para usar                   │
│          Lanzar instancias idénticas                │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Características Principales

| Característica | Descripción |
|----------------|-------------|
| **Regional** | Una AMI está asociada a una región específica de AWS |
| **Reutilizable** | Puedes lanzar múltiples instancias desde una misma AMI |
| **Personalizable** | Crea tus propias AMIs con tu configuración exacta |
| **Compartible** | Comparte AMIs entre cuentas de AWS |
| **Versionable** | Mantén diferentes versiones de tus configuraciones |

### Tipos de AMIs

#### 1. AMIs Públicas (Public AMIs)
```
┌─────────────────────────────────────────────────────┐
│  AMIs Públicas                                      │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Proporcionadas por AWS:                            │
│  • Amazon Linux 2023                                │
│  • Amazon Linux 2                                   │
│  • Ubuntu Server                                    │
│  • Red Hat Enterprise Linux                        │
│  • Windows Server                                   │
│  • macOS (para instancias Mac)                     │
│                                                      │
│  Proporcionadas por la Comunidad:                   │
│  • AMIs verificadas por AWS                         │
│  • AMIs de la comunidad (usar con precaución)      │
│                                                      │
│  ✅ Ventajas:                                       │
│  • Gratis (solo pagas por la instancia)            │
│  • Mantenidas y actualizadas                        │
│  • Ampliamente probadas                             │
│                                                      │
│  ⚠️ Consideraciones:                                │
│  • Configuración genérica                           │
│  • Requiere personalización posterior              │
│                                                      │
└─────────────────────────────────────────────────────┘
```

#### 2. AMIs Privadas (My AMIs)
```
┌─────────────────────────────────────────────────────┐
│  AMIs Privadas (Propias)                            │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Creadas por ti:                                    │
│  • Basadas en tus configuraciones específicas      │
│  • Incluyen tu software y datos                     │
│  • Totalmente personalizadas                        │
│                                                      │
│  ✅ Ventajas:                                       │
│  • Control total sobre la configuración            │
│  • Despliegue rápido de entornos idénticos        │
│  • Consistencia entre instancias                    │
│  • Reducción del tiempo de configuración          │
│                                                      │
│  Casos de uso:                                      │
│  • Servidores web preconfigurados                  │
│  • Entornos de desarrollo estandarizados          │
│  • Aplicaciones empaquetadas                        │
│  • Disaster Recovery                                │
│                                                      │
└─────────────────────────────────────────────────────┘
```

#### 3. AWS Marketplace AMIs
```
┌─────────────────────────────────────────────────────┐
│  AWS Marketplace AMIs                               │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Vendidas por terceros:                             │
│  • Software comercial preinstalado                  │
│  • Soluciones empresariales                         │
│  • Aplicaciones optimizadas                         │
│                                                      │
│  Ejemplos:                                          │
│  • WordPress preconfigurado                         │
│  • Servidores de bases de datos                    │
│  • Firewalls y seguridad                            │
│  • Herramientas de desarrollo                       │
│  • Software de backup                               │
│                                                      │
│  💰 Modelo de pricing:                              │
│  • Costo de la instancia EC2                        │
│  • + Costo adicional del software                   │
│  • Facturado por hora o por año                     │
│                                                      │
│  ✅ Ventajas:                                       │
│  • Solución completa lista para usar               │
│  • Soporte del vendedor                             │
│  • Actualizaciones incluidas                        │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Componentes de una AMI

Una AMI contiene los siguientes elementos:
┌─────────────────────────────────────────────────────┐
│  Estructura de una AMI                              │
├─────────────────────────────────────────────────────┤
│                                                      │
│  1. Plantilla del Volumen Raíz                      │
│     └─ Snapshot del volumen raíz (/)                │
│        • Sistema operativo                          │
│        • Software instalado                         │
│        • Configuraciones del sistema                │
│                                                      │
│  2. Permisos de Lanzamiento (Launch Permissions)    │
│     └─ Quién puede usar la AMI                      │
│        • Privada (solo tu cuenta)                   │
│        • Específica (cuentas seleccionadas)        │
│        • Pública (cualquiera)                       │
│                                                      │
│  3. Mapeo de Dispositivos de Bloque                 │
│     └─ Configuración de volúmenes al lanzar        │
│        • Volumen raíz (/dev/xvda)                   │
│        • Volúmenes adicionales (opcional)           │
│        • Instance store volumes (si aplica)         │
│                                                      │

---
***
## Notas Finales

Este material cubre exhaustivamente todo lo relacionado con Storage para intancias EC2 para el examen AWS Certified Cloud Practitioner. Practica los conceptos en tu propia cuenta de AWS (usa la capa gratuita) para reforzar el aprendizaje.

¡Buena suerte en tu examen!

**Última actualización**: 22/10/2025  
**Versión del curso**: AWS Certified Cloud Practitioner CLF-C02
