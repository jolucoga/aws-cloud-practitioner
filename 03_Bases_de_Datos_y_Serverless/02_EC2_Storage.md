
1. Visión General de EBS (Elastic Block Store)
Amazon EBS (Elastic Block Store) es un servicio de almacenamiento en bloque de alto rendimiento diseñado para usarse con Amazon EC2. Proporciona volúmenes de almacenamiento persistentes que se comportan como discos duros virtuales.
¿Qué es un Volumen EBS?
Un volumen EBS es una unidad de red (no es una unidad física) que puedes adjuntar a tus instancias mientras se ejecutan. Esto te permite:

✅ Persistir datos incluso después de terminar la instancia
✅ Montar en una instancia a la vez (a nivel CCP - Cloud Practitioner)
✅ Vincularlo a una zona de disponibilidad específica

Analogía: Piensa en EBS como una "memoria USB de red" que puedes conectar y desconectar de tus servidores virtuales.
Características Principales
CaracterísticaDescripciónPersistenteLos datos se conservan después de detener o terminar la instanciaNivel de BloqueProporciona almacenamiento a nivel de bloque, como un disco duro tradicionalEscalableSe puede aumentar la capacidad y rendimiento sin reiniciar la instanciaAlta DisponibilidadReplicado automáticamente dentro de la misma zona de disponibilidadCifradoOpción de cifrado con AWS KMS al crear el volumenFacturaciónPagas por toda la capacidad aprovisionada (no solo por lo que usas)
¿Cómo Funciona?
┌─────────────────────────┐
│   Zona Disponibilidad   │
│        us-east-1a       │
│                         │
│  ┌─────────┐  ┌──────┐ │
│  │  EC2    │──│ EBS  │ │  Adjunto
│  │Instance │  │10 GB │ │
│  └─────────┘  └──────┘ │
│                         │
│  ┌─────────┐  ┌──────┐ │
│  │  EC2    │  │ EBS  │ │  No adjunto
│  │Instance │  │50 GB │ │
│  └─────────┘  └──────┘ │
└─────────────────────────┘
```

### Tipos de Volúmenes EBS

AWS ofrece diferentes tipos de volúmenes EBS optimizados para distintos casos de uso:

#### 1. **SSD de Propósito General (gp3/gp2)**
- **Uso:** Equilibrio entre precio y rendimiento
- **IOPS:** hasta 16,000
- **Casos de uso:** Volúmenes de arranque, desarrollo/pruebas, aplicaciones de bajo rendimiento

#### 2. **SSD de IOPS Provisionadas (io2/io1)**
- **Uso:** Alto rendimiento para cargas críticas
- **IOPS:** hasta 64,000 (io2) o 64,000 (io1)
- **Casos de uso:** Bases de datos críticas, aplicaciones que requieren IOPS sostenidas

#### 3. **HDD Optimizado para Throughput (st1)**
- **Uso:** Alto throughput a bajo costo
- **Throughput:** hasta 500 MB/s
- **Casos de uso:** Big Data, data warehouses, procesamiento de logs

#### 4. **HDD Cold (sc1)**
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

- 🗄️ Bases de datos (MySQL, PostgreSQL, Oracle)
- 📁 Sistemas de archivos persistentes
- 📊 Aplicaciones de misión crítica
- 📝 Almacenamiento de logs del sistema
- 💾 Volúmenes de arranque (boot) para instancias EC2

---

## 2. Acerca de EBS Multi-Attach

**EBS Multi-Attach** es una característica que permite adjuntar un mismo volumen EBS a **múltiples instancias EC2** al mismo tiempo dentro de la **misma zona de disponibilidad**.

### Requisitos y Limitaciones

| Requisito | Detalle |
|-----------|---------|
| **Tipo de Volumen** | Solo disponible para volúmenes **io1** e **io2** (IOPS provisionadas) |
| **Zona de Disponibilidad** | Todas las instancias deben estar en la **misma AZ** |
| **Sistema de Archivos** | Requiere un sistema de archivos compatible con acceso concurrente |
| **Número de Instancias** | Hasta 16 instancias Linux EC2 por volumen |

### Arquitectura de EBS Multi-Attach
```
┌──────────────────────────────────┐
│  Zona de Disponibilidad us-east-1a│
│                                   │
│  ┌──────────┐                    │
│  │Instance 1│──┐                 │
│  └──────────┘  │                 │
│                 │   ┌──────────┐ │
│  ┌──────────┐  ├───│EBS Volume│ │
│  │Instance 2│──┤   │  io2     │ │
│  └──────────┘  │   │Multi-Att│ │
│                 │   └──────────┘ │
│  ┌──────────┐  │                 │
│  │Instance 3│──┘                 │
│  └──────────┘                    │
└──────────────────────────────────┘
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
- 🔄 **Alta disponibilidad** de aplicaciones que requieren failover rápido
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
```
1. Click en "Crear Volumen" (Create Volume)

2. Configurar los parámetros:
   ┌──────────────────────────────────┐
   │ Tipo de Volumen: gp3            │
   │ Tamaño: 2 GiB                   │
   │ IOPS: 3000 (predeterminado)     │
   │ Throughput: 125 MB/s            │
   │ Zona de Disponibilidad:         │
   │    us-east-1a (misma que EC2)   │
   │ Cifrado: Deshabilitado          │
   │ Tags:                           │
   │    Nombre: demo-volume          │
   └──────────────────────────────────┘

3. Click en "Crear Volumen" (Create Volume)
```

### Paso 3: Adjuntar el Volumen a una Instancia EC2

Una vez creado el volumen:
```
1. Selecciona el volumen creado
2. Click en "Acciones" > "Asociar Volumen" (Attach Volume)
3. Configurar:
   ┌──────────────────────────────────┐
   │ Instancia: Selecciona tu EC2    │
   │ Dispositivo: /dev/sdf           │
   │ (AWS lo ajustará automáticamente│
   │  si es necesario)               │
   └──────────────────────────────────┘
4. Click en "Asociar" (Attach)
Paso 4: Verificar desde la Instancia EC2
Conéctate a tu instancia y verifica:
bash# Listar todos los dispositivos de bloque
lsblk

# Salida esperada:
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0   2G  0 disk  # <- Tu nuevo volumen
Paso 5: Formatear y Montar el Volumen (Opcional)
bash# 1. Verificar que el volumen no tiene sistema de archivos
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
```
1. Desde la instancia EC2, desmontar primero:
   sudo umount /data

2. En la Consola de AWS:
   - Selecciona el volumen
   - Click en "Acciones" > "Desasociar Volumen"
   - Confirmar la operación

⚠️ IMPORTANTE: Siempre desmonta primero desde el SO
   antes de desadjuntar desde la consola
```

### Paso 7: Modificar un Volumen Existente

Puedes modificar un volumen sin detener la instancia:
```
1. Selecciona el volumen
2. Click en "Acciones" > "Modificar Volumen"
3. Puedes cambiar:
   ┌──────────────────────────────────┐
   │ ✓ Tipo de volumen (gp2 → gp3)   │
   │ ✓ Tamaño (2 GB → 10 GB)         │
   │ ✓ IOPS (solo io1/io2/gp3)       │
   │ ✓ Throughput (solo gp3)         │
   └──────────────────────────────────┘
4. Click en "Modificar"
Después de modificar el tamaño, necesitas extender el sistema de archivos:
bash# Para ext4
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
|----------|---------|
| **Volumen Root** | Se elimina por defecto al terminar la instancia (puedes cambiar esto) |
| **Volúmenes Adicionales** | NO se eliminan por defecto al terminar la instancia |
| **Zona de Disponibilidad** | Un volumen solo puede adjuntarse a instancias en la misma AZ |
| **Snapshots** | Usa snapshots para mover volúmenes entre AZs o regiones |

---

## 4. Visión General de EBS Snapshots

Un **Snapshot de EBS** es una copia de seguridad puntual (point-in-time) de un volumen EBS. Los snapshots son **incrementales**, lo que significa que solo se guardan los bloques que han cambiado desde el último snapshot.

### ¿Qué son los Snapshots?
```
┌─────────────────────────────────────────┐
│         Línea de Tiempo                 │
├─────────────────────────────────────────┤
│                                         │
│  Día 1: Volumen EBS (10 GB usados)     │
│         ↓                               │
│    [Snapshot 1: 10 GB]                 │
│                                         │
│  Día 2: Cambios (+ 2 GB nuevos)       │
│         ↓                               │
│    [Snapshot 2: +2 GB] ← Incremental   │
│                                         │
│  Día 3: Cambios (+ 1 GB nuevos)       │
│         ↓                               │
│    [Snapshot 3: +1 GB] ← Incremental   │
└─────────────────────────────────────────┘

Total almacenado: 13 GB (no 30 GB)
```

### Características Principales

| Característica | Descripción |
|----------------|--------------|
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

### Proceso de Creación de Snapshots
```
┌─────────────────────────────────────────────┐
│  PASO 1: Volumen EBS en uso                │
│  ┌──────────┐                              │
│  │ EBS Vol  │ Datos: A B C D E            │
│  │ 10 GB    │                              │
│  └──────────┘                              │
└─────────────────────────────────────────────┘
            ↓ Crear Snapshot
┌─────────────────────────────────────────────┐
│  PASO 2: Snapshot creado                   │
│  ┌──────────────┐                          │
│  │  Snapshot 1  │ Datos: A B C D E        │
│  │  (en S3)     │ Tamaño: 10 GB           │
│  └──────────────┘                          │
└─────────────────────────────────────────────┘
            ↓ Modificar datos en volumen
┌─────────────────────────────────────────────┐
│  PASO 3: Volumen modificado                │
│  ┌──────────┐                              │
│  │ EBS Vol  │ Datos: A B C X Y (cambios)  │
│  │ 10 GB    │                              │
│  └──────────┘                              │
└─────────────────────────────────────────────┘
            ↓ Crear Snapshot
┌─────────────────────────────────────────────┐
│  PASO 4: Segundo snapshot (incremental)    │
│  ┌──────────────┐                          │
│  │  Snapshot 2  │ Solo cambios: X Y       │
│  │  (en S3)     │ Tamaño: +2 GB           │
│  └──────────────┘                          │
└─────────────────────────────────────────────┘
```

### Características Avanzadas de Snapshots

#### 1. **EBS Snapshot Archive (Archivo)**

Mueve snapshots antiguos a un nivel de almacenamiento de menor costo:

| Aspecto | Standard | Archive |
|---------|----------|---------|
| **Costo** | Standard | 75% más barato |
| **Restauración** | Instantánea | 24-72 horas |
| **Caso de uso** | Snapshots frecuentes | Snapshots antiguos raramente accedidos |

#### 2. **Recycle Bin (Papelera de Reciclaje)**

Protege contra eliminación accidental de snapshots:
```
┌────────────────────────────────────────┐
│  Snapshot eliminado accidentalmente    │
│              ↓                         │
│  Movido a Recycle Bin automáticamente │
│              ↓                         │
│  Retención: 1 día - 1 año             │
│              ↓                         │
│  Puede recuperarse durante ese tiempo  │
└────────────────────────────────────────┘
```

Configuración de Recycle Bin:
- Retención mínima: 1 día
- Retención máxima: 1 año
- Se aplica a nivel de región
- Puede configurarse por etiquetas (tags)

#### 3. **Fast Snapshot Restore (FSR)**

Acelera la restauración de snapshots:

| Sin FSR | Con FSR |
|---------|---------|
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
│  us-east-1       │  Copiar │  eu-west-1       │
│                  │  ──────>│                  │
│  Snapshot        │         │  Snapshot        │
│  snap-abc123     │         │  snap-def456     │
└──────────────────┘         └──────────────────┘
```

### Tiempo de Restauración

El tiempo de restauración depende del tamaño y estado del snapshot:

| Escenario | Tiempo |
|-----------|--------|
| Snapshot Standard | Inmediato (con inicialización gradual) |
| Snapshot Archive | 24-72 horas |
| Con FSR habilitado | Rendimiento completo inmediato |

### 💡 Mejores Prácticas

1. ✅ **No necesitas desadjuntar el volumen** para crear un snapshot, pero es recomendado para garantizar consistencia
2. ✅ **Programa snapshots regulares** para protección de datos críticos
3. ✅ **Usa etiquetas** para organizar y gestionar snapshots
4. ✅ **Elimina snapshots antiguos** que ya no necesites (ahorra costos)
5. ✅ **Copia snapshots críticos** a otra región para disaster recovery
6. ✅ **Habilita Recycle Bin** para protección contra eliminación accidental

---

## 5. EBS Snapshots - Práctica

Aprende a crear, gestionar y restaurar snapshots de EBS desde la consola de AWS.

### Paso 1: Crear un Snapshot

Desde la **Consola de AWS**:
```
1. Navega a: EC2 > Elastic Block Store > Snapshots

2. Click en "Crear Snapshot" (Create Snapshot)

3. Configurar:
   ┌────────────────────────────────────┐
   │ Tipo de recurso: Volumen          │
   │ ID del volumen: vol-abc123        │
   │ Descripción: "Backup mensual Ene" │
   │ Tags:                             │
   │   Nombre: backup-produccion-ene   │
   │   Ambiente: produccion            │
   └────────────────────────────────────┘

4. Click en "Crear Snapshot"
```

El snapshot comenzará a crearse en segundo plano. Puedes continuar usando el volumen.

### Paso 2: Monitorear el Progreso del Snapshot
```
Estado del Snapshot:
┌──────────────────────────────────────┐
│ pending    → El snapshot se está     │
│              creando                 │
│              ↓                       │
│ completed  → Snapshot listo para usar│
└──────────────────────────────────────┘
```

Verifica el progreso:
- En la columna "Status" verás "pending" o "completed"
- En la columna "Progress" verás el porcentaje (ej: 67%)

### Paso 3: Crear un Volumen desde un Snapshot

Una vez que el snapshot está completo:
```
1. Selecciona el snapshot
2. Click en "Acciones" > "Crear Volumen desde Snapshot"

3. Configurar el nuevo volumen:
   ┌────────────────────────────────────┐
   │ Tipo de volumen: gp3              │
   │ Tamaño: 10 GiB (o mayor)          │
   │ Zona de Disponibilidad: us-east-1b│
   │ Cifrado: (mismo que snapshot)     │
   │ Tags:                             │
   │   Nombre: volumen-restaurado      │
   └────────────────────────────────────┘

4. Click en "Crear Volumen"
```

⚠️ **Nota:** Puedes crear el volumen en una **zona de disponibilidad diferente** al volumen original.

### Paso 4: Copiar un Snapshot a Otra Región

Para recuperación ante desastres:
```
1. Selecciona el snapshot
2. Click en "Acciones" > "Copiar Snapshot"

3. Configurar:
   ┌────────────────────────────────────┐
   │ Región de destino: eu-west-1      │
   │ Descripción: "Copia para DR"      │
   │ Cifrado: Habilitar (opcional)     │
   │   Clave KMS: (seleccionar)        │
   └────────────────────────────────────┘

4. Click en "Copiar Snapshot"
```

### Paso 5: Compartir un Snapshot

Puedes compartir snapshots con otras cuentas de AWS:
```
1. Selecciona el snapshot
2. Click en "Acciones" > "Modificar Permisos"

3. Opciones:
   ┌────────────────────────────────────┐
   │ ○ Privado (solo tu cuenta)        │
   │ ○ Público (cualquiera - ⚠️)       │
   │ ○ Compartir con cuentas específicas│
   │   Agregar ID de cuenta AWS:       │
   │   123456789012                    │
   └────────────────────────────────────┘

4. Click en "Guardar"
```

⚠️ **Advertencia:** Ten mucho cuidado al hacer snapshots públicos. Nunca hagas público un snapshot con datos sensibles.

### Paso 6: Configurar Snapshot Archive

Para mover snapshots antiguos a almacenamiento más económico:
```
1. Selecciona el snapshot
2. Click en "Acciones" > "Archivar Snapshot"

3. Confirmar:
   ┌────────────────────────────────────┐
   │ ⚠️ El snapshot archivado tardará   │
   │    24-72 horas en restaurarse     │
   │                                   │
   │ 💰 Ahorro: ~75% en costos        │
   └────────────────────────────────────┘

4. Click en "Archivar"
```

Para restaurar un snapshot archivado:
```
1. Selecciona el snapshot archivado
2. Click en "Acciones" > "Restaurar desde Archivo"
3. Espera 24-72 horas
4. Una vez restaurado, créa un volumen normalmente
```

### Paso 7: Configurar Recycle Bin

Protege contra eliminación accidental:
```
1. Navega a: EC2 > Elastic Block Store > Recycle Bin

2. Click en "Crear Regla de Retención"

3. Configurar:
   ┌────────────────────────────────────┐
   │ Nombre: regla-snapshots-prod      │
   │ Tipo de recurso: Snapshot EBS     │
   │ Retención: 7 días                 │
   │ Tags para aplicar (opcional):     │
   │   Ambiente = produccion           │
   └────────────────────────────────────┘

4. Click en "Crear Regla de Retención"
```

Ahora, si eliminas un snapshot que coincida con las etiquetas, irá a la papelera de reciclaje durante 7 días.

### Paso 8: Recuperar un Snapshot de Recycle Bin

Si eliminaste un snapshot accidentalmente:
```
1. Navega a: EC2 > Recycle Bin > Resources

2. Busca el snapshot eliminado

3. Selecciónalo y click en "Recover"

4. El snapshot volverá a estar disponible
```

### Paso 9: Automatizar Snapshots con Data Lifecycle Manager

Crea políticas automáticas de backup:
```
1. Navega a: EC2 > Elastic Block Store > Lifecycle Manager

2. Click en "Crear Política de Ciclo de Vida"

3. Configurar:
   ┌────────────────────────────────────┐
   │ Tipo: Política de snapshot de EBS │
   │ Volúmenes objetivo:               │
   │   Etiqueta: backup=daily          │
   │ Programación:                     │
   │   Frecuencia: Diaria              │
   │   Hora de inicio: 03:00 UTC       │
   │   Retención: 7 snapshots          │
   │ Copiar a región: (opcional)       │
   │   eu-west-1                       │
   └────────────────────────────────────┘

4. Click en "Crear Política"
🎯 Ejercicio Práctico Completo
Completa este ejercicio de principio a fin:
PARTE 1: Crear y Preparar
1. ✅ Crea un volumen EBS de 8 GB
2. ✅ Adjúntalo a una instancia EC2
3. ✅ Formatea y monta el volumen en /backup
4. ✅ Crea algunos archivos de prueba:
   echo "Datos importantes" > /backup/datos.txt
   echo "Configuración" > /backup/config.txt

PARTE 2: Snapshot y Restauración
5. ✅ Crea un snapshot del volumen
6. ✅ Etiquétalo: Nombre=backup-inicial, Tipo=manual
7. ✅ Espera que el snapshot se complete
8. ✅ Crea un nuevo volumen desde el snapshot
9. ✅ Adjunta el nuevo volumen a otra instancia
10. ✅ Monta y verifica que los archivos están ahí

PARTE 3: Gestión Avanzada
11. ✅RetryClaude does not have the ability to run the code it generates yet.LContinue✅ Copia el snapshot a otra región (ej: us-west-2)
12. ✅ Configura una regla de Recycle Bin para snapshots
13. ✅ Crea una política de Data Lifecycle Manager
14. ✅ Elimina el snapshot original (irá a Recycle Bin)
15. ✅ Recupéralo desde Recycle Bin
PARTE 4: Limpieza
16. ✅ Elimina los volúmenes de prueba
17. ✅ Elimina los snapshots que no necesites
18. ✅ Verifica que no quedaron recursos huérfanos

### 💡 Mejores Prácticas de Snapshots

| Práctica | Razón |
|----------|-------|
| **Etiqueta todo** | Facilita la organización y búsqueda |
| **Nombra descriptivamente** | "backup-prod-db-20250123" es mejor que "snapshot-1" |
| **Programa backups automáticos** | Usa Data Lifecycle Manager |
| **Mantén snapshots cross-region** | Para disaster recovery |
| **Elimina snapshots antiguos** | Ahorra costos, pero mantén según política de retención |
| **Usa Recycle Bin** | Protección contra eliminación accidental |
| **Documenta qué contiene cada snapshot** | En la descripción o en etiquetas |
| **Verifica restauraciones periódicamente** | Asegúrate que los backups funcionan |

### 📊 Costos de Snapshots

Los snapshots tienen costo de almacenamiento en S3:

| Tipo | Costo (us-east-1) |
|------|-------------------|
| **Standard** | ~$0.05 por GB/mes |
| **Archive** | ~$0.0125 por GB/mes (75% ahorro) |
| **Copia entre regiones** | Cargo por transferencia de datos |
| **Restauración desde Archive** | $0.03 por GB restaurado |

**Ejemplo de cálculo:**
```
Snapshot de 100 GB durante 30 días:
- Standard: 100 GB × $0.05 = $5.00/mes
- Archive: 100 GB × $0.0125 = $1.25/mes
- Ahorro: $3.75/mes (75%)
```

### ⚠️ Errores Comunes a Evitar

❌ **Error 1:** Eliminar el primer snapshot de una cadena
- **Problema:** Aunque AWS mantiene los datos necesarios, puede causar confusión
- **Solución:** Documenta claramente las dependencias entre snapshots

❌ **Error 2:** No verificar que el snapshot se completó
- **Problema:** Intentar usar un snapshot "pending" fallará
- **Solución:** Espera a que el estado sea "completed"

❌ **Error 3:** Crear volumen en AZ incorrecta
- **Problema:** No podrás adjuntarlo a tu instancia
- **Solución:** Verifica la AZ de la instancia antes de crear el volumen

❌ **Error 4:** No probar las restauraciones
- **Problema:** Descubres en un desastre que el backup no funciona
- **Solución:** Haz pruebas de restauración periódicas

❌ **Error 5:** Hacer snapshots públicos sin querer
- **Problema:** Exposición de datos sensibles
- **Solución:** Siempre revisa los permisos antes de guardar

---

## 6. Visión General de AMI (Amazon Machine Image)

Una **AMI (Amazon Machine Image)** es una plantilla preconfigurada que contiene el sistema operativo, aplicaciones y configuraciones necesarias para lanzar una instancia EC2. Es como una "fotografía completa" de un servidor que puedes usar para crear copias idénticas.

### ¿Qué es una AMI?
```
┌─────────────────────────────────────────┐
│            AMI Completa                 │
├─────────────────────────────────────────┤
│                                         │
│  📦 Sistema Operativo (Linux/Windows)  │
│  ⚙️  Configuraciones del Sistema        │
│  📝 Software Instalado                  │
│  🔧 Configuraciones de Aplicaciones     │
│  💾 Volumen(es) Root y Datos           │
│  🏷️  Metadata y Permisos               │
│                                         │
└─────────────────────────────────────────┘
              ↓
    Lanzar Instancia EC2
              ↓
┌─────────────────────────────────────────┐
│   Instancia EC2 lista para usar        │
│   (configurada exactamente igual)      │
└─────────────────────────────────────────┘
```

### Tipos de AMIs

#### 1. **AMIs Públicas** 
Proporcionadas por AWS o la comunidad
```
Ejemplos de AMIs públicas de AWS:
- Amazon Linux 2023
- Ubuntu Server 22.04 LTS
- Red Hat Enterprise Linux 9
- Windows Server 2022
- SUSE Linux Enterprise Server
- Amazon Linux 2 (AMI antiguo)
```

#### 2. **AMIs Privadas**
Creadas por ti y solo visibles en tu cuenta
```
Tu organización puede crear AMIs privadas para:
✅ Servidores web preconfigurados
✅ Aplicaciones corporativas
✅ Configuraciones de seguridad específicas
✅ Entornos de desarrollo estandarizados
```

#### 3. **AWS Marketplace AMIs**
Ofrecidas por terceros (pueden tener costo adicional)
```
Ejemplos del Marketplace:
💰 Software comercial preinstalado
💰 Soluciones de seguridad (firewalls, IDS)
💰 Bases de datos optimizadas
💰 Stacks de aplicaciones completas
```

### Componentes de una AMI

Una AMI contiene:

| Componente | Descripción |
|------------|-------------|
| **Plantilla del volumen root** | El sistema operativo y aplicaciones base |
| **Permisos de lanzamiento** | Qué cuentas de AWS pueden usar la AMI |
| **Mapeo de dispositivos de bloque** | Especifica los volúmenes a adjuntar al lanzar |
| **Region** | Las AMIs son específicas de región |
| **Arquitectura** | x86_64 o ARM64 (Graviton) |
| **Tipo de virtualización** | HVM (Hardware Virtual Machine) o PV (obsoleto) |

### Proceso de Creación de una AMI
```
┌─────────────────────────────────────────┐
│  PASO 1: Instancia EC2 en ejecución    │
│                                         │
│  ┌──────────────────────────────┐     │
│  │  Sistema Operativo: Linux    │     │
│  │  Software: Apache, MySQL     │     │
│  │  Configuración: Custom       │     │
│  └──────────────────────────────┘     │
└─────────────────────────────────────────┘
              ↓
        Crear Imagen (AMI)
              ↓
┌─────────────────────────────────────────┐
│  PASO 2: AWS crea snapshots            │
│                                         │
│  Snapshot del volumen root: snap-abc123│
│  Snapshot de datos: snap-def456        │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│  PASO 3: AMI lista (ami-xyz789)        │
│                                         │
│  ✅ Puede usarse para lanzar instancias│
│  ✅ Incluye todos los snapshots        │
│  ✅ Preserva configuraciones           │
└─────────────────────────────────────────┘
```

### Ventajas de Usar AMIs

| Ventaja | Beneficio |
|---------|-----------|
| **Tiempo de arranque rápido** | Todo el software está preinstalado |
| **Configuración consistente** | Todas las instancias son idénticas |
| **Escalabilidad** | Lanza múltiples instancias rápidamente |
| **Backup completo del sistema** | Incluye SO, apps y configuraciones |
| **Portabilidad** | Copia AMIs entre regiones |
| **Recuperación ante desastres** | Restaura sistemas completos rápidamente |

### Casos de Uso Comunes

#### 1. **Entornos de Desarrollo/Testing**
```
Problema: Los desarrolladores necesitan entornos idénticos
Solución: Crea una AMI con todas las herramientas
Beneficio: Setup de nuevos entornos en minutos
```

#### 2. **Despliegue de Aplicaciones**
```
Problema: Necesitas desplegar tu app en múltiples servidores
Solución: AMI con la aplicación preconfigurada
Beneficio: Despliegues rápidos y consistentes
```

#### 3. **Auto Scaling**
```
Problema: Necesitas escalar automáticamente según demanda
Solución: Auto Scaling Group usa tu AMI
Beneficio: Nuevas instancias listas instantáneamente
```

#### 4. **Disaster Recovery**
```
Problema: Servidor de producción falla
Solución: Lanzar nueva instancia desde AMI
Beneficio: Recuperación en minutos, no horas
```

#### 5. **Migración a AWS**
```
Problema: Migrar servidor on-premises a AWS
Solución: Crear AMI del servidor (con AWS Application Migration Service)
Beneficio: Migración simplificada
```

### AMI Lifecycle
```
┌────────────┐
│   CREAR    │ ← Desde instancia EC2 en ejecución
└─────┬──────┘
      │
      ↓
┌────────────┐
│ REGISTRADA │ ← AMI disponible en tu cuenta
└─────┬──────┘
      │
      ↓
┌────────────┐
│   COPIAR   │ ← A otra región si es necesario
└─────┬──────┘
      │
      ↓
┌────────────┐
│   LANZAR   │ ← Crear instancias EC2 desde AMI
└─────┬──────┘
      │
      ↓
┌────────────┐
│ DEPRECAR   │ ← Marcar como obsoleta (opcional)
└─────┬──────┘
      │
      ↓
┌────────────┐
│ ELIMINAR   │ ← Deregistrar AMI cuando ya no se necesita
└────────────┘
```

### AMIs y Regiones

⚠️ **Importante:** Las AMIs son **específicas de región**
```
┌────────────────────────────────────────┐
│           us-east-1                    │
│                                        │
│  AMI: ami-abc123                       │
│  ✅ Disponible en us-east-1           │
│  ❌ NO disponible en otras regiones   │
└────────────────────────────────────────┘

Para usar en otra región:
1. Copiar AMI a la región destino
2. Se creará nueva AMI con ID diferente
3. Ambas AMIs existen independientemente
```

### Permisos de AMI

Puedes controlar quién puede usar tu AMI:

| Nivel de Permiso | Descripción |
|------------------|-------------|
| **Privada** | Solo tú puedes usarla (predeterminado) |
| **Pública** | Cualquier usuario de AWS puede usarla |
| **Compartida** | Específicas cuentas de AWS pueden usarla |
| **AWS Marketplace** | Vendida/compartida en el Marketplace |

### 💡 Mejores Prácticas para AMIs

1. ✅ **Usa AMIs propias para producción** - Mayor control y seguridad
2. ✅ **Documenta qué contiene cada AMI** - Usa descripciones claras
3. ✅ **Etiqueta tus AMIs** - Nombre, Versión, Fecha, Ambiente
4. ✅ **Mantén AMIs actualizadas** - Crea nuevas versiones regularmente
5. ✅ **Elimina AMIs antiguas** - Ahorra costos de almacenamiento
6. ✅ **Prueba antes de usar en producción** - Valida que funciona correctamente
7. ✅ **Copia AMIs críticas a otras regiones** - Para disaster recovery
8. ✅ **Cifra AMIs con datos sensibles** - Usa KMS para cifrado

### 📊 Costos de AMIs

Las AMIs en sí no tienen costo, pero sí los snapshots subyacentes:
```
Costo = Suma de costos de los snapshots EBS

Ejemplo:
AMI con:
- Snapshot root (10 GB): $0.50/mes
- Snapshot datos (50 GB): $2.50/mes
- Total: $3.00/mes

Si copias a otra región: costo duplicado ($6.00/mes)
```

### AMI vs Snapshot

Es importante entender la diferencia:

| AMI | Snapshot |
|-----|----------|
| Plantilla completa de instancia | Copia de volumen individual |
| Incluye metadatos de configuración | Solo datos de volumen |
| Se usa para lanzar instancias | Se usa para restaurar volúmenes |
| Contiene uno o más snapshots | Es un componente de AMI |
| Específica de región | Específico de región |

---

## 7. AMI - Práctica

Aprende a crear, gestionar y usar AMIs personalizadas en AWS.

### Paso 1: Preparar una Instancia para la AMI

Primero, configura una instancia con todo lo que necesitas:
```bash
# Conéctate a tu instancia EC2
ssh -i mi-clave.pem ec2-user@<ip-publica>

# Actualiza el sistema
sudo yum update -y  # Amazon Linux
# o
sudo apt update && sudo apt upgrade -y  # Ubuntu

# Instala software necesario
sudo yum install -y httpd git  # Amazon Linux
# o
sudo apt install -y apache2 git  # Ubuntu

# Configura la aplicación
sudo systemctl start httpd
sudo systemctl enable httpd

# Crea contenido personalizado
echo "<h1>Mi servidor web custom</h1>" | sudo tee /var/www/html/index.html

# Limpia logs y datos temporales (buena práctica)
sudo rm -rf /tmp/*
sudo rm -rf /var/log/*.log
history -c
```

### Paso 2: Crear una AMI desde la Instancia

Desde la **Consola de AWS**:
```
1. Navega a: EC2 > Instancias

2. Selecciona tu instancia configurada

3. Click en "Acciones" > "Imagen y plantillas" > "Crear imagen"

4. Configurar:
   ┌────────────────────────────────────────┐
   │ Nombre de la imagen:                   │
   │   webserver-apache-v1.0                │
   │                                        │
   │ Descripción:                           │
   │   Servidor web Apache con config       │
   │   personalizada - Creado 2025-01-23    │
   │                                        │
   │ Sin reiniciar: ☐                       │
   │   (☑ para no reiniciar la instancia)  │
   │                                        │
   │ Volúmenes de instancia:                │
   │   /dev/xvda - 8 GiB - gp3             │
   │   ☑ Eliminar al terminar              │
   │                                        │
   │ Tags:                                  │
   │   Nombre: webserver-apache-v1          │
   │   Versión: 1.0                         │
   │   Ambiente: desarrollo                 │
   │   Proyecto: webapp-demo                │
   └────────────────────────────────────────┘

5. Click en "Crear imagen"
```

⚠️ **Nota sobre "Sin reiniciar":**
- ☐ **Desmarcado (predeterminado):** AWS reinicia la instancia para garantizar integridad del sistema de archivos
- ☑ **Marcado:** AWS crea la AMI sin reiniciar (más rápido, pero puede haber inconsistencias)

### Paso 3: Monitorear la Creación de la AMI
```
1. Navega a: EC2 > Imágenes > AMIs

2. Observa el estado:
   ┌────────────────────────────────────────┐
   │ Estado: pending                        │
   │ Progreso: Creando snapshots...         │
   │              ↓                         │
   │ Estado: available                      │
   │ ✅ AMI lista para usar                 │
   └────────────────────────────────────────┘

3. Verifica los snapshots:
   - Navega a: EC2 > Snapshots
   - Encontrarás snapshots asociados a la AMI
```

Tiempo estimado: 5-15 minutos dependiendo del tamaño del volumen.

### Paso 4: Lanzar una Instancia desde tu AMI

Ahora usa tu AMI personalizada:
```
1. Navega a: EC2 > AMIs

2. Selecciona tu AMI

3. Click en "Lanzar instancia desde AMI"

4. Configurar la instancia:
   ┌────────────────────────────────────────┐
   │ Nombre: webserver-from-ami             │
   │ AMI: webserver-apache-v1.0 ✓           │
   │ Tipo de instancia: t2.micro            │
   │ Par de claves: tu-clave                │
   │ Grupo de seguridad:                    │
   │   ☑ HTTP (puerto 80)                  │
   │   ☑ SSH (puerto 22)                   │
   └────────────────────────────────────────┘

5. Click en "Lanzar instancia"

6. Espera que la instancia esté "running"

7. Accede a http://<ip-publica>
   Deberías ver: "Mi servidor web custom"
```

✅ **¡La instancia ya tiene todo configurado!** No necesitas instalar nada.

### Paso 5: Copiar AMI a Otra Región

Para disaster recovery o expansión global:
```
1. Navega a: EC2 > AMIs

2. Selecciona tu AMI

3. Click en "Acciones" > "Copiar AMI"

4. Configurar:
   ┌────────────────────────────────────────┐
   │ Región de destino: us-west-2           │
   │                                        │
   │ Nombre: webserver-apache-v1.0-west     │
   │                                        │
   │ Descripción: Copia para DR en Oregon   │
   │                                        │
   │ Cifrado: ☑ Cifrar volúmenes de destino│
   │   Clave KMS: (predeterminada) AWS     │
   └────────────────────────────────────────┘

5. Click en "Copiar AMI"
```

La copia puede tardar varios minutos. Una vez completada:
```
6. Cambia a la región us-west-2 en la consola

7. Navega a: EC2 > AMIs

8. Encontrarás la AMI copiada con nuevo ID:
   - Región origen: ami-abc123 (us-east-1)
   - Región destino: ami-xyz789 (us-west-2)
```

### Paso 6: Compartir AMI con Otra Cuenta

Para compartir con colegas u otras cuentas:
```
1. Navega a: EC2 > AMIs

2. Selecciona tu AMI

3. Click en "Acciones" > "Editar permisos de AMI"

4. Configurar permisos:
   ┌────────────────────────────────────────┐
   │ ○ Pública                              │
   │   (Cualquiera puede usar - ⚠️ cuidado) │
   │                                        │
   │ ● Privada                              │
   │   ☑ Compartir con cuentas de AWS       │
   │                                        │
   │   ID de cuenta de AWS:                 │
   │   123456789012                         │
   │   [+ Agregar otro ID de cuenta]        │
   │                                        │
   │ ☑ Agregar permisos de "create volume"  │
   │   (permite crear volúmenes desde       │
   │    los snapshots de la AMI)            │
   └────────────────────────────────────────┘

5. Click en "Guardar cambios"
```

La otra cuenta ahora podrá:
- Ver la AMI en su lista
- Lanzar instancias desde ella
- Copiarla a sus propias AMIs

### Paso 7: Deprecar una AMI

Cuando una AMI está desactualizada pero aún quieres que las instancias existentes funcionen:
```
1. Navega a: EC2 > AMIs

2. Selecciona la AMI antigua

3. Click en "Acciones" > "Deprecar AMI"

4. Configurar:
   ┌────────────────────────────────────────┐
   │ Fecha de desaprobación:                │
   │   2025-12-31                           │
   │                                        │
   │ La AMI aparecerá como "deprecated"     │
   │ pero seguirá funcionando.              │
   │                                        │
   │ Los usuarios verán advertencia al      │
   │ intentar usarla.                       │
   └────────────────────────────────────────┘

5. Click en "Deprecar"
```

### Paso 8: Eliminar (Deregistrar) una AMI

Cuando ya no necesitas una AMI:

⚠️ **IMPORTANTE:** Esto NO elimina automáticamente los snapshots subyacentes.
```
1. Navega a: EC2 > AMIs

2. Selecciona la AMI a eliminar

3. Click en "Acciones" > "Desregistrar AMI"

4. Confirmar:
   ┌────────────────────────────────────────┐
   │ ⚠️ ADVERTENCIA                         │
   │                                        │
   │ Desregistrar la AMI impedirá lanzar    │
   │ nuevas instancias desde ella.          │
   │                                        │
   │ Las instancias existentes NO se        │
   │ afectan.                               │
   │                                        │
   │ Los snapshots asociados NO se          │
   │ eliminan automáticamente.              │
   └────────────────────────────────────────┘

5. Click en "Desregistrar"

6. IMPORTANTE: Elimina los snapshots manualmente:
   - Navega a: EC2 > Snapshots
   - Filtra por descripción o ID de AMI
   - Selecciona los snapshots huérfanos
   - Click en "Acciones" > "Eliminar snapshot"
```

### Paso 9: Automatizar Creación de AMIs

Usa AWS Systems Manager o scripts para automatizar:
```bash
#!/bin/bash
# Script para crear AMI automáticamente

INSTANCE_ID="i-0123456789abcdef"
AMI_NAME="webserver-auto-$(date +%Y%m%d-%H%M%S)"
DESCRIPTION="AMI automática creada el $(date)"

# Crear AMI
AMI_ID=$(aws ec2 create-image \
    --instance-id $INSTANCE_ID \
    --name "$AMI_NAME" \
    --description "$DESCRIPTION" \
    --no-reboot \
    --tag-specifications "ResourceType=image,Tags=[{Key=AutoCreated,Value=true},{Key=Date,Value=$(date +%Y-%m-%d)}]" \
    --query 'ImageId' \
    --output text)

echo "AMI creada: $AMI_ID"

# Esperar a que esté disponible
aws ec2 wait image-available --image-ids $AMI_ID

echo "AMI $AMI_ID está disponible"

# Eliminar AMIs antiguas (mantener solo últimas 5)
aws ec2 describe-images \
    --owners self \
    --filters "Name=name,Values=webserver-auto-*" \
    --query 'sort_by(Images, &CreationDate)[:-5].ImageId' \
    --output text | \
while read old_ami; do
    echo "Eliminando AMI antigua: $old_ami"
    aws ec2 deregister-image --image-id $old_ami
done
```

### 🎯 Ejercicio Práctico Completo

Crea un flujo completo de trabajo con AMIs:
```
PARTE 1: Configuración Inicial
1. ✅ Lanza instancia EC2 Amazon Linux 2023
2. ✅ Instala y configura:
   - Servidor web (Apache/Nginx)
   - PHP o Python
   - Una aplicación web simple
3. ✅ Personaliza la página de inicio
4. ✅ Configura inicio automático de servicios

PARTE 2: Crear y Usar AMI
5. ✅ Crea AMI de la instancia configurada
6. ✅ Etiqueta: Nombre, Versión, Ambiente
7. ✅ Espera a que esté disponible
8. ✅ Lanza 2 instancias nuevas desde la AMI
9. ✅ Verifica que ambas funcionan idénticamente

PARTE 3: Gestión Avanzada
10. ✅ Copia la AMI a otra región (us-west-2)
11. ✅ Comparte la AMI con otra cuenta (si tienes)
12. ✅ En us-west-2, lanza instancia desde AMI copiada
13. ✅ Crea versión 2.0 con cambios adicionales
14. ✅ Depreca la versión 1.0

PARTE 4: Limpieza
15. ✅ Termina todas las instancias de prueba
16. ✅ Desregistra AMIs que no necesites
17. ✅ Elimina snapshots huérfanos
18. ✅ Verifica costos finales
```

### 💡 Tips y Mejores Prácticas

| Práctica | Beneficio |
|----------|-----------|
| **Nombra con versionado semántico** | `app-v1.2.3` facilita seguimiento |
| **Incluye fecha en descripción** | Sabrás cuándo se creó |
| **Limpia instancia antes de AMI** | Elimina logs, temporales, historiales |
| **Usa etiquetas consistentes** | Facilita automatización y búsqueda |
| **Documenta cambios entre versiones** | En descripción o en wiki externa |
| **Prueba AMI antes de producción** | Lanza instancia de test primero |
| **Mantén AMIs cross-region** | Para disaster recovery |
| **Automatiza creación periódica** | Scripts o Systems Manager |

### 📊 Organización de AMIs

Ejemplo de esquema de nombrado:
```
Esquema: {aplicacion}-{ambiente}-v{version}-{fecha}

Ejemplos:
- webapp-prod-v1.0.0-20250115
- webapp-staging-v1.0.1-20250120
- webapp-dev-v1.1.0-20250123
- api-backend-prod-v2.3.1-20250110

Etiquetas asociadas:
{
  "Nombre": "webapp-prod-v1.0.0",
  "Aplicacion": "webapp",
  "Ambiente": "produccion",
  "Version": "1.0.0",
  "Fecha": "2025-01-15",
  "Responsable": "equipo-devops",
  "Ticket": "JIRA-1234"
}
```

---

## 8. Visión General de EC2 Image Builder

**Amazon EC2 Image Builder** es un servicio totalmente administrado que simplifica la creación, mantenimiento, validación y distribución de imágenes de servidor (AMIs) seguras y actualizadas.

### ¿Qué Problema Resuelve?

#### Sin EC2 Image Builder:
```
Proceso Manual (Lento y Propenso a Errores):

1. Lanzar instancia EC2
2. Conectarse por SSH
3. Instalar software manualmente
4. Configurar aplicaciones
5. Aplicar parches de seguridad
6. Testear manualmente
7. Crear AMI
8. Repetir para cada actualización ♻️
```

#### Con EC2 Image Builder:
```
Proceso Automatizado:

1. Definir "receta" una vez
2. Image Builder ejecuta automáticamente:
   ✅ Aprovisiona instancia temporal
   ✅ Aplica componentes (software)
   ✅ Ejecuta tests de validación
   ✅ Crea y distribuye AMI
   ✅ Limpia recursos temporales
3. Programa ejecuciones periódicas
```

### Arquitectura de EC2 Image Builder
┌─────────────────────────────────────────────────┐
│           EC2 Image Builder Pipeline            │
└─────────────────────────────────────────────────┘
│
┌─────────────┴─────────────┐
↓                           ↓
┌──────────────┐          ┌──────────────────┐
│   Receta     │          │ Infraestructura  │
│   (Recipe)   │          │ Configuración    │
├──────────────┤          ├──────────────────┤
│• AMI base    │          │• Tipo instancia  │
│• Componentes │          │• Subred/VPC      │
│• Tests       │          │• Rol IAM         │
└──────────────┘          └──────────────────┘
│                           │
└─────────────┬─────────────┘
↓
┌──────────────────────────┐
│   Build (Construcción)    │
├──────────────────────────┤
│ 1. Lanzar instancia temp │
│ 2. Aplicar componentes   │
│ 3. Ejecutar tests        │
│ 4. Crear AMI             │
│ 5. Distribuir            │
└──────────────────────────┘
↓
┌──────────────────────────┐
│   Distribución Settings   │
├──────────────────────────┤
│• us-east-1: ami-abc123   │
│• us-west-2: ami-def456   │RetryClaude does not have the ability to run the code it generates yet.LContinue        │• eu-west-1: ami-ghi789   │
        └──────────────────────────┘
                      ↓
        ┌──────────────────────────┐
        │   AMIs Finales           │
        │   ✅ Listas para usar    │
        └──────────────────────────┘
Componentes Principales
1. Recipe (Receta)
Define QUÉ construir:
yamlReceta de Ejemplo:
┌────────────────────────────────────┐
│ Nombre: webserver-recipe           │
│ Versión: 1.0.0                     │
│ AMI base: Amazon Linux 2023        │
│                                    │
│ Componentes de Build:              │
│   1. Actualizar sistema (AWS)      │
│   2. Instalar Apache (Custom)      │
│   3. Instalar PHP (Custom)         │
│   4. Configurar firewall (AWS)     │
│   5. Hardening seguridad (AWS)     │
│                                    │
│ Componentes de Test:               │
│   1. Verificar Apache activo       │
│   2. Test de vulnerabilidades      │
│   3. Verificar conformidad CIS     │
│                                    │
│ Volumen root: 20 GB                │
└────────────────────────────────────┘
2. Components (Componentes)
Scripts o documentos que instalan/configuran software:
yamlTipos de Componentes:

1. AWS Managed (Gestionados por AWS):
   ✅ Actualización del sistema operativo
   ✅ Parches de seguridad
   ✅ Inspector de vulnerabilidades
   ✅ Configuraciones de hardening

2. AWS Marketplace:
   💰 Componentes de terceros
   💰 Software comercial preconfigurado

3. Custom (Personalizados):
   🔧 Scripts propios (bash, PowerShell)
   🔧 Ansible playbooks
   🔧 Chef recipes
Ejemplo de componente personalizado:
yamlname: InstalarApache
description: Instala y configura Apache
schemaVersion: 1.0
phases:
  - name: build
    steps:
      - name: InstalarPaquete
        action: ExecuteBash
        inputs:
          commands:
            - sudo yum install -y httpd
            - sudo systemctl enable httpd
      
      - name: ConfigurarFirewall
        action: ExecuteBash
        inputs:
          commands:
            - sudo firewall-cmd --permanent --add-service=http
            - sudo firewall-cmd --reload

  - name: test
    steps:
      - name: VerificarServicio
        action: ExecuteBash
        inputs:
          commands:
            - systemctl is-active httpd
            - curl -f http://localhost || exit 1
```

#### 3. **Infrastructure Configuration**

Define DÓNDE construir:
```
┌────────────────────────────────────┐
│ Tipo de instancia: t3.medium       │
│ Subred: subnet-abc123              │
│ Grupos de seguridad: sg-xyz789     │
│ Rol IAM: ImageBuilderRole         │
│ SNS Topic: notificaciones-build   │
│ Logs: CloudWatch Logs Group       │
└────────────────────────────────────┘
```

#### 4. **Distribution Settings**

Define DÓNDE distribuir:
```
┌────────────────────────────────────┐
│ Distribución:                      │
│                                    │
│ Region us-east-1:                  │
│   ✅ Crear AMI                     │
│   ✅ Copiar a cuentas: 1234567890 │
│   ✅ Tags: Ambiente=prod          │
│                                    │
│ Region us-west-2:                  │
│   ✅ Crear AMI                     │
│   ✅ Cifrar con KMS               │
│                                    │
│ Region eu-west-1:                  │
│   ✅ Crear AMI                     │
│   ✅ Hacer pública: NO            │
└────────────────────────────────────┘
```

#### 5. **Pipeline (Tubería)**

Orquesta todo el proceso:
```
Pipeline = Recipe + Infrastructure + Distribution + Schedule

Ejecución:
┌─────────────────────────────────────┐
│ Trigger: Manual / Programado / API  │
│              ↓                       │
│ Aprovisionar instancia temporal     │
│              ↓                       │
│ Aplicar componentes de build        │
│              ↓                       │
│ Ejecutar componentes de test        │
│              ↓                       │
│ Si tests pasan: Crear AMI           │
│              ↓                       │
│ Distribuir a regiones configuradas  │
│              ↓                       │
│ Limpiar instancia temporal          │
│              ↓                       │
│ Notificar por SNS                   │
└─────────────────────────────────────┘
```

### Beneficios de EC2 Image Builder

| Beneficio | Descripción |
|-----------|-------------|
| **Automatización** | Elimina tareas manuales repetitivas |
| **Consistencia** | Misma configuración cada vez |
| **Seguridad** | Integración con Inspector y Systems Manager |
| **Cumplimiento** | Aplicación automática de estándares (CIS, STIG) |
| **Ahorro de tiempo** | Construcción y distribución en horas, no días |
| **Versionado** | Control de versiones de imágenes |
| **Testing integrado** | Validación automática antes de distribución |
| **Multi-región** | Distribución automática a múltiples regiones |
| **Sin costo adicional** | Solo pagas por recursos subyacentes (EC2, EBS, S3) |

### Casos de Uso

#### 1. **Actualizaciones de Seguridad Regulares**
```
Problema: Mantener parches de seguridad actualizados
Solución: Pipeline programado semanalmente
Resultado: AMIs siempre con últimos parches
```

#### 2. **Imágenes Corporativas Estandarizadas**
```
Problema: Diferentes equipos configuran servidores de manera distinta
Solución: Image Builder con componentes corporativos
Resultado: Configuración uniforme en toda la organización
```

#### 3. **Cumplimiento Normativo**
```
Problema: Necesitas cumplir con CIS Benchmarks o STIG
Solución: Componentes AWS de hardening automático
Resultado: Auditorías simplificadas
```

#### 4. **CI/CD de Infraestructura**
```
Problema: Integrar creación de AMIs en pipeline DevOps
Solución: API de Image Builder + automatización
Resultado: AMIs actualizadas con cada release
```

### Integración con Otros Servicios AWS
```
┌──────────────────────────────────────────┐
│         EC2 Image Builder                │
└──────────────────────────────────────────┘
         ↓              ↓              ↓
    ┌────────┐    ┌─────────┐    ┌─────────┐
    │ Systems│    │Inspector│    │CloudWatch│
    │Manager │    │         │    │ Logs    │
    └────────┘    └─────────┘    └─────────┘
    Inventario    Scan de         Logs de
    de software   seguridad       construcción
         ↓              ↓              ↓
    ┌────────┐    ┌─────────┐    ┌─────────┐
    │   SNS  │    │   S3    │    │   KMS   │
    └────────┘    └─────────┘    └─────────┘
    Notifica-     Logs y          Cifrado
    ciones        artefactos      de AMIs
```

### Programación de Pipelines

Puedes ejecutar pipelines:

| Modo | Cuándo Usar |
|------|-------------|
| **Manual** | Builds bajo demanda, testing |
| **Programado** | Actualizaciones regulares (diarias, semanales) |
| **API/CLI** | Integración con CI/CD |
| **EventBridge** | Cuando se actualiza AMI base |

Ejemplo de programación:
```
Cron Expression: cron(0 2 ? * SUN *)
Significado: Cada domingo a las 2:00 AM UTC

Usa caso:
- Domingo: Build de AMI con parches de la semana
- Lunes-Viernes: Equipos usan AMI actualizada
Proceso de Testing Integrado
Image Builder puede ejecutar tests automáticos:
yamlTests Comunes:
┌────────────────────────────────────┐
│ 1. Tests de Funcionalidad          │
│    ✓ Servicios iniciados           │
│    ✓ Puertos escuchando            │
│    ✓ Aplicación responde           │
│                                    │
│ 2. Tests de Seguridad              │
│    ✓ Amazon Inspector scan         │
│    ✓ Vulnerabilidades conocidas    │
│    ✓ Configuraciones inseguras     │
│                                    │
│ 3. Tests de Conformidad            │
│    ✓ CIS Benchmark                 │
│    ✓ STIG compliance               │
│    ✓ Políticas corporativas        │
│                                    │
│ 4. Tests Personalizados            │
│    ✓ Scripts propios               │
│    ✓ Validaciones específicas      │
└────────────────────────────────────┘

Si algún test falla:
❌ La AMI NO se crea
❌ Se notifica por SNS
❌ Se guardan logs para debugging
```

### Costos de EC2 Image Builder

El servicio es **gratuito**, pero pagas por:

| Recurso | Costo |
|---------|-------|
| **Instancia EC2 temporal** | Mientras se construye (ej: $0.04/hora para t3.medium) |
| **Volúmenes EBS** | Almacenamiento temporal durante build |
| **Snapshots/AMIs** | Almacenamiento de las AMIs finales ($0.05/GB/mes) |
| **Transferencia de datos** | Si copias AMIs entre regiones |
| **Inspector** | Si usas validación de seguridad |

Ejemplo de costo:
```
Build de una AMI:
- Instancia t3.medium × 30 min = $0.02
- EBS 20 GB × 30 min = $0.01
- AMI final 20 GB/mes = $1.00

Total por build: ~$0.03
Total mensual (con AMI): ~$1.03

Si programas builds semanales (4/mes):
Costo total: ~$1.12/mes
```

### 💡 Mejores Prácticas

1. ✅ **Usa componentes AWS managed cuando sea posible** - Ya están optimizados
2. ✅ **Implementa tests exhaustivos** - Detecta problemas antes de distribución
3. ✅ **Versiona tus recetas** - Facilita rollback y tracking
4. ✅ **Etiqueta todo** - Recetas, componentes, AMIs resultantes
5. ✅ **Usa SNS para notificaciones** - Monitorea éxito/fallo de builds
6. ✅ **Guarda logs en CloudWatch** - Para troubleshooting
7. ✅ **Programa builds regulares** - Mantén AMIs actualizadas
8. ✅ **Distribuye a múltiples regiones** - Para disaster recovery

### ⚠️ Importante para el Examen

Para el examen **AWS Cloud Practitioner**:

✅ **Debes saber:**
- Qué es EC2 Image Builder
- Que automatiza creación de AMIs
- Que incluye testing y distribución
- Que es gratuito (solo pagas recursos subyacentes)

❌ **NO necesitas saber:**
- Sintaxis de componentes
- Detalles de programación de pipelines
- Configuración avanzada

---

## 9. EC2 Instance Store

**EC2 Instance Store** proporciona almacenamiento temporal a nivel de hardware directamente adjunto al servidor físico que ejecuta tu instancia EC2. A diferencia de EBS, este almacenamiento es **efímero** (temporal).

### ¿Qué es EC2 Instance Store?
```
┌─────────────────────────────────────────┐
│         Servidor Físico AWS             │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────┐    ┌──────────────┐  │
│  │  Instancia   │    │  Discos SSD  │  │
│  │     EC2      │────│  Físicos     │  │
│  │              │    │  (Instance   │  │
│  └──────────────┘    │   Store)     │  │
│                      └──────────────┘  │
│                                         │
│  Conexión directa = MUY RÁPIDA         │
└─────────────────────────────────────────┘

vs.

┌─────────────────────────────────────────┐
│              EBS (Red)                  │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────┐                      │
│  │  Instancia   │    Internet          │
│  │     EC2      │────→ Red AWS         │
│  │              │    │                 │
│  └──────────────┘    ↓                 │
│               ┌──────────────┐         │
│               │ Volumen EBS  │         │
│               │  (separado)  │         │
│               └──────────────┘         │
└─────────────────────────────────────────┘
```

### Características Principales

| Característica | Instance Store | EBS |
|----------------|----------------|-----|
| **Persistencia** | ❌ Temporal (efímero) | ✅ Persistente |
| **Rendimiento** | ⚡ Muy alto (IOPS hasta millones) | ✅ Alto (pero limitado por red) |
| **Latencia** | ⚡ Muy baja (microsegundos) | ✅ Baja (milisegundos) |
| **Ubicación** | Disco físico local | Almacenamiento en red |
| **Snapshots** | ❌ No soportado | ✅ Sí |
| **Costo adicional** | ❌ Incluido con instancia | ✅ Sí (por GB/mes) |
| **Tamaño** | Fijo según tipo de instancia | Variable (1 GB - 64 TB) |
| **Detener instancia** | ❌ Datos se pierden | ✅ Datos persisten |
| **Terminar instancia** | ❌ Datos se pierden | ✅ Datos persisten (opcional) |

### ⚠️ Comportamiento de los Datos
```
Acciones que PIERDEN datos en Instance Store:

❌ Detener la instancia (Stop)
❌ Terminar la instancia (Terminate)
❌ Fallo del disco físico subyacente
❌ Fallo del host físico
❌ Hibernar la instancia

Acciones que MANTIENEN datos:

✅ Reiniciar la instancia (Reboot)
```

### Tipos de Instancias con Instance Store

No todas las instancias tienen Instance Store. Ejemplos:

| Familia | Tipo de Instancia | Instance Store | Tamaño | Tipo |
|---------|-------------------|----------------|--------|------|
| **Almacenamiento Optimizado** | i3.large | ✅ | 475 GB | NVMe SSD |
| | i3.xlarge | ✅ | 950 GB | NVMe SSD |
| | d2.xlarge | ✅ | 6 TB | HDD |
| **Computación Optimizada** | c5d.large | ✅ | 50 GB | NVMe SSD |
| | c5d.xlarge | ✅ | 100 GB | NVMe SSD |
| **Propósito General** | m5d.large | ✅ | 75 GB | NVMe SSD |
| | m5d.xlarge | ✅ | 150 GB | NVMe SSD |
| **Memoria Optimizada** | r5d.large | ✅ | 75 GB | NVMe SSD |

💡 **Nota:** Las variantes con "d" en el nombre (ej: c5**d**, m5**d**, r5**d**) incluyen Instance Store.

### Rendimiento de Instance Store

Instance Store ofrece rendimiento excepcional:
```
Ejemplo: Instancia i3.16xlarge

Especificaciones:
┌────────────────────────────────────┐
│ Instance Store: 8 × 1.9 TB NVMe   │
│ Total: 15.2 TB                    │
│ IOPS lecturas: 3.3 millones       │
│ IOPS escrituras: 1.4 millones     │
│ Throughput lectura: 16 GB/s       │
│ Throughput escritura: 6.8 GB/s    │
└────────────────────────────────────┘

Comparación con EBS:
- EBS io2: máx 64,000 IOPS
- Instance Store i3: 3,300,000 IOPS
- Instance Store es ~51x más rápido
```

### Casos de Uso Apropiados

✅ **BUENO para Instance Store:**

1. **Almacenamiento temporal/cache**
```
   Ejemplos:
   - Cache de aplicación (Redis, Memcached)
   - Datos de sesión temporales
   - Scratch space para procesamiento
   - Buffers de datos
```

2. **Datos replicados**
```
   Ejemplos:
   - Nodos de bases de datos cluster (Cassandra, MongoDB)
   - Réplicas de lectura
   - Datos que existen en múltiples instancias
```

3. **Procesamiento de datos temporales**
```
   Ejemplos:
   - Big Data processing (Hadoop, Spark)
   - ETL workloads
   - Renderizado de video temporal
   - Compilaciones de código
```

4. **Alto rendimiento I/O**
```
   Ejemplos:
   - Bases de datos NoSQL de alto rendimiento
   - Procesamiento transaccional intensivo
   - Aplicaciones que requieren baja latencia extrema
```

❌ **MALO para Instance Store:**

1. ❌ Bases de datos críticas sin réplicas
2. ❌ Archivos importantes sin respaldo
3. ❌ Datos que deben persistir
4. ❌ Aplicaciones sin tolerancia a pérdida de datos
5. ❌ Volúmenes root (boot) importantes

### Arquitectura con Instance Store

Ejemplo de arquitectura apropiada:
```
┌────────────────────────────────────────┐
│         Aplicación Web                 │
└────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────┐
│    Auto Scaling Group (3 instancias)   │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ Instancia 1 (m5d.large)          │ │
│  │ - OS en EBS: Persistente ✅      │ │
│  │ - Instance Store: Cache Redis ⚡  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ Instancia 2 (m5d.large)          │ │
│  │ - OS en EBS: Persistente ✅      │ │
│  │ - Instance Store: Cache Redis ⚡  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ Instancia 3 (m5d.large)          │ │
│  │ - OS en EBS: Persistente ✅      │ │
│  │ - Instance Store: Cache Redis ⚡  │ │
│  └──────────────────────────────────┘ │
└────────────────────────────────────────┘
                  ↓
        ┌──────────────────┐
        │   RDS Database   │
        │   (persistente)  │
        └──────────────────┘

Estrategia:
- EBS para OS y datos críticos
- Instance Store para cache de alto rendimiento
- Si una instancia falla, ASG la reemplaza
- Cache se reconstruye automáticamente
Configuración y Uso
Al lanzar una instancia con Instance Store:
bash# 1. Verificar dispositivos disponibles
lsblk

# Salida ejemplo en i3.large:
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
nvme0n1     259:0    0   475G  0 disk  # Instance Store
nvme1n1     259:1    0     8G  0 disk  # EBS root
└─nvme1n1p1 259:2    0     8G  0 part /

# 2. Formatear Instance Store (primera vez)
sudo mkfs.xfs /dev/nvme0n1

# 3. Crear punto de montaje
sudo mkdir /instance-store

# 4. Montar el dispositivo
sudo mount /dev/nvme0n1 /instance-store

# 5. Cambiar permisos
sudo chown ec2-user:ec2-user /instance-store

# 6. Verificar
df -h /instance-store
⚠️ IMPORTANTE: NO agregues Instance Store a /etc/fstab porque los datos no persisten al detener la instancia.
Script de Inicialización con Instance Store
Usa User Data para configurar Instance Store automáticamente:
bash#!/bin/bash
# User Data script para configurar Instance Store

# Detectar dispositivo Instance Store
DEVICE=$(lsblk -d -n --output NAME | grep nvme0n1)

if [ ! -z "$DEVICE" ]; then
    # Formatear si no está formateado
    if ! blkid /dev/$DEVICE; then
        mkfs.xfs /dev/$DEVICE
    fi
    
    # Montar
    mkdir -p /cache
    mount /dev/$DEVICE /cache
    chown www-data:www-data /cache
    
    # Configurar Redis para usar Instance Store
    echo "dir /cache/redis" >> /etc/redis/redis.conf
    systemctl restart redis
fi
```

### Estrategias de Respaldo

Ya que los datos son temporales, implementa respaldo:
```
Estrategia 1: Snapshots Periódicos a S3
┌────────────────────────────────────┐
│ Instancia con Instance Store       │
│                                    │
│ Cronjob cada hora:                │
│  tar -czf /tmp/backup.tar.gz      │
│           /instance-store/        │
│  aws s3 cp /tmp/backup.tar.gz     │
│           s3://bucket/backups/    │
└────────────────────────────────────┘

Estrategia 2: Replicación en Tiempo Real
┌────────────────────────────────────┐
│ Múltiples instancias con cluster  │
│                                    │
│ Instancia 1    Instancia 2        │
│   [Data] ←────→ [Data]            │
│   Instance      Instance          │
│   Store         Store             │
│                                    │
│ Si una falla, datos en otra       │
└────────────────────────────────────┘

Estrategia 3: Datos Reconstruibles
┌────────────────────────────────────┐
│ Source of Truth en S3/RDS         │
│          ↓                        │
│ Instance Store = Cache derivado   │
│                                    │
│ Si se pierde, se reconstruye      │
│ desde la fuente                    │
└────────────────────────────────────┘
```

### 💡 Mejores Prácticas

| Práctica | Razón |
|----------|-------|
| **Nunca usar para datos únicos** | Se perderán al detener/terminar |
| **Siempre tener respaldo o réplicas** | Protección contra pérdida |
| **Usar RAID 0 para múltiples discos** | Maximizar rendimiento |
| **Documentar que es efímero** | Evitar malentendidos en el equipo |
| **Monitorear salud del disco** | Detectar fallos de hardware |
| **Automatizar reconstrucción** | Recovery rápido tras fallos |
| **No usar para volumen root** | Sistema operativo debe persistir |

### Cuándo Elegir Instance Store vs EBS
```
Elige INSTANCE STORE si:
✅ Necesitas máximo rendimiento I/O
✅ Los datos son temporales o replicados
✅ Tienes estrategia de respaldo/réplica
✅ La aplicación tolera pérdida de datos
✅ Quieres reducir costos (incluido en instancia)

Elige EBS si:
✅ Los datos deben persistir
✅ Necesitas snapshots
✅ Datos críticos sin réplicas
✅ Flexibilidad en tamaño
✅ Necesitas adjuntar/desadjuntar volúmenes
```

### 📊 Comparación de Rendimiento

| Métrica | Instance Store (i3) | EBS io2 | EBS gp3 |
|---------|---------------------|---------|---------|
| **IOPS máx** | 3.3M lecturas | 64,000 | 16,000 |
| **Throughput** | 16 GB/s lectura | 1,000 MB/s | 1,000 MB/s |
| **Latencia** | < 100 μs | < 1 ms | < 10 ms |
| **Durabilidad** | ❌ Efímero | ✅ 99.999% | ✅ 99.8-99.9% |
| **Persistencia** | ❌ No | ✅ Sí | ✅ Sí |
| **Costo adicional** | ❌ No | ✅ Sí (~$0.125/GB) | ✅ Sí (~$0.08/GB) |

---

## 10. Visión General de EFS (Elastic File System)

**Amazon EFS** es un sistema de archivos NFS (Network File System) totalmente administrado, escalable y elástico para usar con servicios de AWS Cloud y recursos on-premises. A diferencia de EBS, EFS puede montarse en **múltiples instancias EC2 simultáneamente**.

### ¿Qué es EFS?
```
EBS (Almacenamiento de Bloques):
┌────────────────────────────────────┐
│ Volumen EBS (10 GB)                │
│        │                           │
│        ↓                           │
│   EC2 Instance                     │
│   (1 a 1)                          │
└────────────────────────────────────┘

EFS (Sistema de Archivos Compartido):
┌────────────────────────────────────┐
│      EFS File System               │
│            │                       │
│      ┌─────┴─────┬─────┬─────┐    │
│      ↓           ↓     ↓     ↓    │
│   EC2-1      EC2-2  EC2-3  EC2-4  │
│   (Many to Many)                   │
└────────────────────────────────────┘
Características Principales
CaracterísticaDescripciónSistema de archivos compartidoMúltiples instancias EC2 pueden acceder simultáneamenteTotalmente administradoAWS gestiona hardware, parches, backupsAltamente disponibleDatos replicados en múltiples AZsEscalableCrece y se reduce automáticamente (hasta petabytes)RendimientoPuede escalar a miles de conexiones concurrentesCompatible con NFSProtocolo estándar NFSv4.1Solo para LinuxCompatible con instancias EC2 Linux, no WindowsPago por usoSolo pagas por el almacenamiento que uses
Arquitectura de EFS
┌──────────────────────────────────────────────────┐
│                 VPC                              │
│                                                  │
│  ┌────────────────────────────────────────────┐ │
│  │         Zona Disponibilidad us-east-1a     │ │
│  │                                            │ │
│  │  ┌──────────┐      ┌──────────────┐      │ │
│  │  │ EC2-1    │─────→│ EFS Mount    │      │ │
│  │  │          │      │ Target       │      │ │
│  │  └──────────┘      └───────┬──────┘      │ │
│  └────────────────────────────┼─────────────┘ │
│                                │               │
│  ┌────────────────────────────┼─────────────┐ │
│  │         Zona Disponibilidad│us-east-1b   │ │
│  │                            │             │ │
│  │  ┌──────────┐      ┌───────┴──────┐    │ │
│  │  │ EC2-2    │─────→│ EFS Mount    │    │ │
│  │  │          │      │ Target       │    │ │
│  │  └──────────┘      └───────┬──────┘    │ │
│  └────────────────────────────┼───────────┘ │
│                                │             │
│  ┌────────────────────────────┼───────────┐ │
│  │         Zona Disponibilidad│us-east-1c │ │
│  │                            │           │ │
│  │  ┌──────────┐      ┌───────┴──────┐  │ │
│  │  │ EC2-3    │─────→│ EFS Mount    │  │ │RetryClaude does not have the ability to run the code it generates yet.LContinue│  │  │          │      │ Target       │  │ │
│  │  └──────────┘      └───────┬──────┘  │ │
│  └────────────────────────────┼─────────┘ │
│                                │           │
│              ┌─────────────────┴─────────┐ │
│              │   Amazon EFS File System  │ │
│              │   (Datos replicados en    │ │
│              │   múltiples AZs)          │ │
│              └───────────────────────────┘ │
└──────────────────────────────────────────────┘

### Componentes de EFS

#### 1. **File System (Sistema de Archivos)**

El recurso principal que almacena tus datos:
```
File System: fs-abc12345
┌────────────────────────────────────┐
│ Tamaño: Dinámico (crece/decrece)  │
│ Rendimiento: Elástico              │
│ Cifrado: En reposo y tránsito     │
│ Ciclo de vida: Activo              │
│ Throughput mode: Elástico          │
└────────────────────────────────────┘
```

#### 2. **Mount Targets (Puntos de Montaje)**

Interfaces de red en cada AZ para acceder al sistema de archivos:
```
Mount Target en cada AZ:
┌────────────────────────────────────┐
│ AZ: us-east-1a                     │
│ Subnet: subnet-abc123              │
│ IP: 10.0.1.50                      │
│ Security Group: sg-efs-access      │
│ Estado: Available                  │
└────────────────────────────────────┘

Las instancias EC2 se conectan al mount target
en su misma AZ para mejor rendimiento.
```

#### 3. **Access Points (Puntos de Acceso)**

Proporcionan acceso específico de aplicación al file system:
```
Access Point: fsap-xyz789
┌────────────────────────────────────┐
│ Path: /app1-data                   │
│ POSIX User: uid=1001, gid=1001     │
│ Permisos: rwx para owner           │
│ Root Directory: /app1              │
└────────────────────────────────────┘

Permite aislar aplicaciones en el mismo EFS
```

### Clases de Almacenamiento en EFS

EFS ofrece dos clases de almacenamiento:

| Clase | Descripción | Costo | Uso |
|-------|-------------|-------|-----|
| **EFS Standard** | Acceso frecuente | ~$0.30/GB/mes | Archivos accedidos regularmente |
| **EFS Infrequent Access (IA)** | Acceso poco frecuente | ~$0.025/GB/mes | Archivos accedidos raramente |
```
┌──────────────────────────────────────────┐
│        Amazon EFS File System            │
├──────────────────────────────────────────┤
│                                          │
│  ┌────────────────┐  ┌────────────────┐ │
│  │ EFS Standard   │  │   EFS-IA       │ │
│  ├────────────────┤  ├────────────────┤ │
│  │ app.log (hoy)  │  │ app.log (60d)  │ │
│  │ data.db        │  │ backup-old.tar │ │
│  │ config.json    │  │ archive-2024   │ │
│  └────────────────┘  └────────────────┘ │
│                                          │
│  Lifecycle Policy: Mover a IA después    │
│  de 30 días sin acceso                   │
└──────────────────────────────────────────┘
```

**Ahorro con EFS-IA:** Hasta 92% comparado con Standard

### Políticas de Ciclo de Vida (Lifecycle Management)

Mueve automáticamente archivos a IA:
```
Opciones de política:
┌────────────────────────────────────────┐
│ Mover a IA después de:                 │
│  ○ 7 días sin acceso                   │
│  ○ 14 días sin acceso                  │
│  ● 30 días sin acceso (recomendado)    │
│  ○ 60 días sin acceso                  │
│  ○ 90 días sin acceso                  │
│                                        │
│ Mover de vuelta a Standard cuando:     │
│  ● En primer acceso                    │
└────────────────────────────────────────┘

Ejemplo de ahorro:
100 GB de archivos antiguos:
- Sin política: 100 GB × $0.30 = $30/mes
- Con política IA: 100 GB × $0.025 = $2.50/mes
- Ahorro: $27.50/mes (92%)
```

### Modos de Rendimiento

EFS ofrece dos modos de rendimiento:

| Modo | Descripción | Latencia | Throughput | Uso |
|------|-------------|----------|------------|-----|
| **General Purpose** | Balanceado | Baja | Hasta 7,000 ops/seg por FS | Mayoría de casos de uso |
| **Max I/O** | Alto throughput | Mayor | +500,000 ops/seg | Big data, procesamiento masivo |
```
General Purpose (Predeterminado):
┌────────────────────────────────────┐
│ ✅ Latencia más baja               │
│ ✅ Ideal para mayoría de apps      │
│ ✅ 7,000 operaciones/seg           │
│ Ej: Servidores web, CMS, home dirs│
└────────────────────────────────────┘

Max I/O:
┌────────────────────────────────────┐
│ ⚡ Mayor throughput agregado       │
│ ⚡ +500,000 operaciones/seg        │
│ ⚠️  Mayor latencia                 │
│ Ej: Big Data, analytics, genomics │
└────────────────────────────────────┘
```

### Modos de Throughput

Controla cuánto rendimiento obtienes:

| Modo | Descripción | Escalamiento |
|------|-------------|--------------|
| **Bursting** | Escala con tamaño del FS | Automático basado en almacenamiento |
| **Elastic** | Escala automáticamente con workload | Automático según demanda |
| **Provisioned** | Throughput fijo independiente del tamaño | Manual |
```
Bursting Throughput:
Throughput base: 50 MB/s por TB almacenado
Burst: Hasta 100 MB/s

Ejemplo:
- 1 TB almacenado = 50 MB/s base + burst a 100 MB/s
- 10 TB almacenado = 500 MB/s base + burst a 1000 MB/s

Elastic Throughput (Recomendado):
- Escala automáticamente
- Hasta 3 GB/s lecturas
- Hasta 1 GB/s escrituras
- Pagas solo por lo que usas
```

### Casos de Uso de EFS

#### 1. **Aplicaciones Web Escalables**
```
Problema: Múltiples servidores web necesitan acceder
         a los mismos archivos (imágenes, assets)

Solución: EFS montado en todos los servidores

┌─────────┐  ┌─────────┐  ┌─────────┐
│ Web-1   │  │ Web-2   │  │ Web-3   │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────────┴────────────┘
                  │
            ┌─────┴─────┐
            │    EFS    │
            │ /var/www  │
            └───────────┘

Beneficio: Contenido consistente en todos los servidores
```

#### 2. **Directorios Home Compartidos**
```
Múltiples usuarios acceden a sus directorios home
desde diferentes servidores:

/efs/home/
  ├── /usuario1/
  ├── /usuario2/
  └── /usuario3/

Beneficio: Usuarios tienen acceso a sus archivos
          desde cualquier servidor
```

#### 3. **Content Management Systems (CMS)**
```
WordPress, Drupal, Joomla en múltiples instancias:

EFS almacena:
- Temas
- Plugins
- Uploads de usuarios
- Archivos de medios

Auto Scaling agrega/quita servidores sin perder datos
```

#### 4. **Desarrollo y Testing**
```
Equipos de desarrollo comparten código y recursos:

/efs/projects/
  ├── /proyecto-a/
  ├── /proyecto-b/
  └── /shared-libs/

Beneficio: Colaboración sin conflictos de versiones
```

#### 5. **Machine Learning**
```
Múltiples instancias de entrenamiento acceden a:
- Datasets de entrenamiento
- Modelos pre-entrenados
- Resultados compartidos

EFS con Lifecycle manda datos antiguos a IA
```

#### 6. **Big Data y Analytics**
```
Cluster de procesamiento (EMR, Spark):
- Datos de entrada compartidos
- Resultados intermedios
- Logs agregados

Beneficio: Nodos pueden escalar sin replicar datos
```

### Seguridad en EFS
```
Capas de Seguridad:

1. Security Groups
   ┌──────────────────────────────┐
   │ Regla Entrada:               │
   │ Tipo: NFS                    │
   │ Puerto: 2049                 │
   │ Origen: sg-ec2-instances     │
   └──────────────────────────────┘

2. IAM Policies
   ┌──────────────────────────────┐
   │ Controla:                    │
   │ - Quién puede crear EFS      │
   │ - Quién puede modificar      │
   │ - Quién puede eliminar       │
   └──────────────────────────────┘

3. File System Policies
   ┌──────────────────────────────┐
   │ Controla acceso al FS:       │
   │ - Por cuenta AWS             │
   │ - Por VPC                    │
   │ - Por access point           │
   │ - Enforce encryption         │
   └──────────────────────────────┘

4. Cifrado
   ┌──────────────────────────────┐
   │ En reposo: KMS (opcional)    │
   │ En tránsito: TLS (opcional)  │
   └──────────────────────────────┘

5. Permisos POSIX
   ┌──────────────────────────────┐
   │ Estándar Linux:              │
   │ chmod, chown, ACLs           │
   └──────────────────────────────┘
```

### EFS vs EBS vs Instance Store vs S3

| Característica | EFS | EBS | Instance Store | S3 |
|----------------|-----|-----|----------------|-----|
| **Tipo** | Sistema archivos | Bloque | Bloque | Objeto |
| **Protocolo** | NFS | iSCSI/NVMe | N/A | HTTP(S) |
| **Adjunto a** | Múltiples EC2 | 1 EC2 (CCP) | 1 EC2 | N/A |
| **Persistencia** | ✅ Sí | ✅ Sí | ❌ No | ✅ Sí |
| **Disponibilidad** | Multi-AZ | 1 AZ | 1 AZ | Global |
| **Escalamiento** | Automático | Manual | Fijo | Ilimitado |
| **Precio** | $0.30/GB/mes | $0.08/GB/mes | Incluido | $0.023/GB/mes |
| **Latencia** | ~ms | ~ms | ~μs | ~100ms |
| **Casos de uso** | Compartir archivos | OS, DB | Cache temp | Backups, media |

### Integración con On-Premises

Puedes montar EFS desde servidores on-premises:
```
┌──────────────────────────────────────┐
│     Centro de Datos On-Premises      │
│                                      │
│  ┌────────────┐                     │
│  │ Servidor   │                     │
│  │ Linux      │                     │
│  └─────┬──────┘                     │
└────────┼────────────────────────────┘
         │
         │ AWS Direct Connect
         │ o VPN
         │
┌────────┼────────────────────────────┐
│        │        AWS Cloud           │
│        │                            │
│  ┌─────┴──────┐                    │
│  │ EFS Mount  │                    │
│  │ Target     │                    │
│  └─────┬──────┘                    │
│        │                            │
│  ┌─────┴──────┐                    │
│  │    EFS     │                    │
│  │ File System│                    │
│  └────────────┘                    │
└────────────────────────────────────┘

Requisitos:
- Direct Connect o VPN Site-to-Site
- Security Groups correctos
- NFS cliente en servidor on-premises
```

### 💡 Mejores Prácticas para EFS

1. ✅ **Usa General Purpose mode** para la mayoría de casos
2. ✅ **Habilita Lifecycle Management** para ahorrar costos
3. ✅ **Usa Elastic Throughput** para workloads impredecibles
4. ✅ **Monta desde la misma AZ** para menor latencia
5. ✅ **Cifra datos sensibles** (en reposo y tránsito)
6. ✅ **Usa Access Points** para aislar aplicaciones
7. ✅ **Implementa Security Groups restrictivos** (solo puerto 2049)
8. ✅ **Monitorea métricas** en CloudWatch
9. ✅ **Establece alarmas** para uso y rendimiento
10. ✅ **Prueba restauración** de backups periódicamente

### 📊 Estimación de Costos EFS
```
Ejemplo de Cálculo:

Escenario:
- 500 GB de datos activos (Standard)
- 2 TB de archivos antiguos (IA)
- Región: us-east-1

Costos mensuales:
Standard: 500 GB × $0.30 = $150.00
IA:      2000 GB × $0.025 = $50.00
                    Total = $200.00/mes

Sin Lifecycle (todo en Standard):
2500 GB × $0.30 = $750.00/mes

Ahorro con Lifecycle: $550.00/mes (73%)
```

### ⚠️ Importante para el Examen

Para el examen **AWS Cloud Practitioner**:

✅ **Debes saber:**
- EFS es sistema de archivos compartido (NFS)
- Funciona con múltiples instancias EC2 simultáneamente
- Solo compatible con Linux
- Escala automáticamente
- Alta disponibilidad (Multi-AZ)
- Pago por uso
- EFS-IA para reducir costos

❌ **NO necesitas saber:**
- Configuración detallada de mount targets
- Sintaxis de NFS mount commands
- Detalles de throughput modes
- Configuración de access points

---

## 11. Modelo de Responsabilidad Compartida para el Almacenamiento EC2

El **Modelo de Responsabilidad Compartida** de AWS define claramente qué aspectos de seguridad y operación gestiona AWS y cuáles son responsabilidad del cliente.

### Concepto General
```
┌──────────────────────────────────────────┐
│    MODELO DE RESPONSABILIDAD COMPARTIDA  │
├──────────────────────────────────────────┤
│                                          │
│  AWS                      CLIENTE        │
│  ════                     ═══════        │
│  Seguridad                Seguridad      │
│  DEL Cloud                EN el Cloud    │
│                                          │
└──────────────────────────────────────────┘
```

### Responsabilidades de AWS (DEL Cloud)

AWS es responsable de la **infraestructura** que ejecuta todos los servicios de AWS:

| Área | Responsabilidades de AWS |
|------|--------------------------|
| **Hardware Físico** | • Servidores físicos<br>• Almacenamiento físico (discos, SSD)<br>• Equipos de red<br>• Mantenimiento de hardware |
| **Infraestructura de Red** | • Switches, routers, firewalls físicos<br>• Cables y conectividad<br>• Redundancia de red |
| **Instalaciones** | • Seguridad física de data centers<br>• Control de acceso a instalaciones<br>• Energía y refrigeración<br>• Protección contra desastres naturales |
| **Software de Infraestructura** | • Hypervisor (virtualización)<br>• Software de gestión de almacenamiento<br>• Software de red |
| **Replicación y Durabilidad** | • Replicación automática de EBS en AZ<br>• Durabilidad de S3 (11 9's)<br>• Redundancia de EFS en múltiples AZs |
```
┌────────────────────────────────────────┐
│         AWS se encarga de:             │
├────────────────────────────────────────┤
│                                        │
│  🏢 Data Center físico                │
│  🔌 Electricidad y refrigeración       │
│  🔒 Seguridad física                   │
│  💾 Hardware de almacenamiento         │
│  🌐 Infraestructura de red             │
│  🖥️  Servidores físicos                │
│  ⚙️  Hypervisor y virtualización       │
│  🔄 Replicación automática de datos    │
│  🛠️  Mantenimiento de hardware         │
│  🔍 Monitoreo de infraestructura       │
│                                        │
└────────────────────────────────────────┘
```

### Responsabilidades del Cliente (EN el Cloud)

El cliente es responsable de la **seguridad y configuración** de lo que construye en AWS:

| Área | Responsabilidades del Cliente |
|------|-------------------------------|
| **Datos** | • Cifrado de datos en reposo<br>• Cifrado de datos en tránsito<br>• Clasificación de datos<br>• Protección de información sensible |
| **Configuración** | • Configuración de volúmenes EBS<br>• Configuración de EFS<br>• Políticas de ciclo de vida<br>• Configuración de snapshots |
| **Control de Acceso** | • Políticas IAM<br>• Security Groups<br>• Network ACLs<br>• Permisos de sistema operativo |
| **Backups y DR** | • Crear snapshots de EBS<br>• Programar backups<br>• Probar restauraciones<br>• Plan de disaster recovery |
| **Aplicaciones** | • Seguridad de aplicaciones<br>• Gestión de parches de aplicaciones<br>• Configuración de software |
```
┌────────────────────────────────────────┐
│        CLIENTE se encarga de:          │
├────────────────────────────────────────┤
│                                        │
│  🔐 Cifrado de datos                   │
│  🔑 Gestión de claves (KMS)            │
│  📋 Políticas IAM                      │
│  🛡️  Security Groups                   │
│  📸 Snapshots y backups                │
│  🔄 Replicación cross-region           │
│  🗂️  Organización de datos             │
│  ⚙️  Configuración de volúmenes        │
│  🏷️  Etiquetado de recursos            │
│  📊 Monitoreo de uso y costos          │
│  🔧 Lifecycle policies                 │
│  🚨 Alertas y notificaciones           │
│                                        │
└────────────────────────────────────────┘
```

### Responsabilidad Compartida por Servicio

#### EBS (Elastic Block Store)

| AWS Responsable | Cliente Responsable |
|-----------------|---------------------|
| ✅ Hardware de almacenamiento físico | ✅ Cifrar volúmenes con KMS |
| ✅ Replicación dentro de AZ | ✅ Crear snapshots regularmente |
| ✅ Disponibilidad del servicio EBS | ✅ Configurar "Delete on Termination" |
| ✅ Durabilidad del almacenamiento | ✅ Gestionar permisos de snapshots |
| ✅ Mantenimiento de hardware | ✅ Copiar snapshots entre regiones |
| ✅ Reemplazo de discos defectuosos | ✅ Eliminar snapshots no necesarios |

#### EFS (Elastic File System)

| AWS Responsable | Cliente Responsable |
|-----------------|---------------------|
| ✅ Infraestructura multi-AZ | ✅ Configurar Security Groups |
| ✅ Replicación automática | ✅ Configurar políticas de ciclo de vida |
| ✅ Escalamiento automático | ✅ Gestionar permisos POSIX |
| ✅ Disponibilidad del servicio | ✅ Cifrado en tránsito (TLS) |
| ✅ Durabilidad de datos | ✅ Configurar access points |
| ✅ Mantenimiento de infraestructura | ✅ Monitorear uso y rendimiento |

#### Instance Store

| AWS Responsable | Cliente Responsable |
|-----------------|---------------------|
| ✅ Hardware de discos físicos | ⚠️ **Entender que los datos son temporales** |
| ✅ Mantenimiento de hardware | ✅ Implementar backups propios |
| ✅ Reemplazo de discos defectuosos | ✅ Replicar datos críticos |
| | ✅ Aceptar riesgo de pérdida de datos |
| | ✅ Formatear y montar discos |

#### Snapshots

| AWS Responsable | Cliente Responsable |
|-----------------|---------------------|
| ✅ Almacenamiento en S3 | ✅ Crear snapshots regularmente |
| ✅ Durabilidad de snapshots | ✅ Organizar snapshots con etiquetas |
| ✅ Infraestructura de snapshot | ✅ Configurar Recycle Bin |
| ✅ Replicación en S3 | ✅ Copiar a otras regiones |
| | ✅ Gestionar ciclo de vida (Archive) |
| | ✅ Eliminar snapshots obsoletos |

#### AMIs

| AWS Responsable | Cliente Responsable |
|-----------------|---------------------|
| ✅ Infraestructura de AMI registry | ✅ Mantener AMIs actualizadas |
| ✅ Almacenamiento de AMIs | ✅ Gestionar permisos de AMI |
| ✅ Disponibilidad del servicio | ✅ Eliminar AMIs obsoletas |
| | ✅ Compartir AMIs apropiadamente |
| | ✅ Eliminar snapshots asociados |
| | ✅ Documentar contenido de AMI |

### Ejemplo Práctico: EBS Volumen de Producción
```
ESCENARIO: Volumen EBS con base de datos crítica

┌────────────────────────────────────────┐
│         AWS Responsabilidades          │
├────────────────────────────────────────┤
│ ✅ Hardware SSD funcionando            │
│ ✅ Replicar datos en múltiples discos  │
│    físicos dentro de la AZ             │
│ ✅ Reemplazar discos defectuosos       │
│ ✅ Mantener disponibilidad de la API   │
│ ✅ Infraestructura de snapshots en S3  │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│       CLIENTE Responsabilidades        │
├────────────────────────────────────────┤
│ ✅ Cifrar el volumen con KMS           │
│ ✅ Crear snapshots cada 24 horas       │
│ ✅ Copiar snapshots a us-west-2        │
│ ✅ Configurar Security Groups          │
│ ✅ Probar restauración mensualmente    │
│ ✅ Monitorear métricas en CloudWatch   │
│ ✅ Configurar alarmas para IOPS        │
│ ✅ Gestionar permisos IAM              │
│ ✅ Eliminar snapshots después de 90d   │
│ ✅ Documentar procedimientos de DR     │
└────────────────────────────────────────┘
```

### Matriz de Responsabilidades por Categoría

#### Seguridad

| Aspecto | AWS | Cliente |
|---------|-----|---------|
| Seguridad física | ✅ | ❌ |
| Seguridad de red (infraestructura) | ✅ | ❌ |
| Seguridad de red (configuración) | ❌ | ✅ |
| Cifrado en reposo (capacidad) | ✅ | ❌ |
| Cifrado en reposo (activación) | ❌ | ✅ |
| Cifrado en tránsito (capacidad) | ✅ | ❌ |
| Cifrado en tránsito (configuración) | ❌ | ✅ |
| Gestión de claves | ✅ (infraestructura) | ✅ (uso de claves) |

#### Disponibilidad y Durabilidad

| Aspecto | AWS | Cliente |
|---------|-----|---------|
| Replicación dentro de AZ (EBS) | ✅ | ❌ |
| Replicación multi-AZ (EFS) | ✅ | ❌ |
| Backups/Snapshots (capacidad) | ✅ | ❌ |
| Backups/Snapshots (ejecución) | ❌ | ✅ |
| Disaster Recovery (infraestructura) | ✅ | ❌ |
| Disaster Recovery (plan y pruebas) | ❌ | ✅ |

#### Cumplimiento

| Aspecto | AWS | Cliente |
|---------|-----|---------|
| Certificaciones de AWS (ISO, SOC) | ✅ | ❌ |
| Cumplimiento de tus datos | ❌ | ✅ |
| Retención de logs | ❌ | ✅ |
| Auditorías de acceso | ✅ (infraestructura) | ✅ (revisar logs) |

### Herramientas para Cumplir Responsabilidades
```
Para CIFRADO:
└─→ AWS KMS: Gestión de claves
└─→ EBS: Cifrado en creación de volumen
└─→ EFS: Cifrado en reposo y tránsito

Para BACKUPS:
└─→ EBS Snapshots: Backups de volúmenes
└─→ AWS Backup: Automatización de backups
└─→ Data Lifecycle Manager: Snapshots programados

Para MONITOREO:
└─→ CloudWatch: Métricas y logs
└─→ CloudTrail: Auditoría de API calls
└─→ AWS Config: Compliance de configuraciones

Para CONTROL DE ACCESO:
└─→ IAM: Políticas y permisos
└─→ Security Groups: Firewall de red
└─→ KMS: Políticas de uso de claves

Para DISASTER RECOVERY:
└─→ Snapshots cross-region
└─→ AMIs en múltiples regiones
└─→ AWS Backup para planes de DR
```

### Checklist de Responsabilidades del Cliente
Para Almacenamiento EC2:
Configuración Inicial:
☐ Cifrar volúmenes EBS con KMS
☐ Configurar Security Groups apropiados
☐ Establecer políticas IAM de acceso
☐ Etiquetar todos los recursos
Operaciones Continuas:
☐ Crear snapshots regularmente (EBS)
☐ Copiar snapshots a otra región (DR)
☐ Configurar Lifecycle policies (EFS)
☐ Eliminar recursos no usados
Monitoreo y Alertas:
☐ Configurar alarmas CloudWatch
☐ Revisar métricas de uso/rendimiento
☐ Auditar accesos con CloudTrail
☐ Verificar cumplimiento con Config
Disaster Recovery:
☐ Documentar procedimientos de restauración
☐ Probar restauraciones periódicamente
☐ Mantener AMIs actualizadas
☐ Plan de failover a otra región
Seguridad:
☐ Rotar claves de cifrado
☐ Revisar permisos IAM regularmente
☐ Actualizar Security Groups según sea necesario
☐ Eliminar snapshots/AMIs públicos no intencionales
Costos:
☐ Eliminar snapshots obsoletos
☐ Archivar snapshots antiguos (Archive)
☐ Usar EFS-IA para datos poco accedidos
☐ Monitorear y optimizar costos

### 💡 Puntos Clave para Recordar

1. ✅ **AWS = Infraestructura física y servicios base**
2. ✅ **Cliente = Configuración, datos y seguridad de aplicaciones**
3. ✅ **Cifrado: AWS proporciona la capacidad, cliente la activa**
4. ✅ **Backups: AWS proporciona la herramienta, cliente la ejecuta**
5. ✅ **Durabilidad: AWS replica automáticamente, cliente gestiona copias cross-region**
6. ✅ **Instance Store: Cliente asume riesgo de pérdida de datos**

---

66. Visión General de Amazon FSx
Amazon FSx es un servicio completamente administrado que proporciona sistemas de archivos de alto rendimiento optimizados para casos de uso específicos. A diferencia de EBS (almacenamiento a nivel de bloque) y EFS (sistema de archivos compartido simple), FSx ofrece soluciones especializadas con características avanzadas.
Características Principales
CaracterísticaDescripciónTotalmente AdministradoAWS gestiona el aprovisionamiento, parcheo y mantenimientoAlto RendimientoOptimizado para rendimiento y throughputEscalabilidadEscala automáticamente con tus necesidadesIntegración NativaCompatible con servicios de AWS y aplicaciones localesCifradoCifrado en reposo y en tránsito
Tipos de FSx
FSx for Windows File Server
Ideal para entornos empresariales que requieren compatibilidad total con Windows.
Características:

Protocolo SMB (Server Message Block) totalmente compatible
Integración nativa con Active Directory
Soporta DFS (Distributed File System)
Múltiples AZ para alta disponibilidad
Compatible con NTFS y permisos de Windows

Casos de uso:

Almacenamiento compartido para aplicaciones Windows
Migración de sistemas legados de archivos compartidos
Entornos empresariales con Active Directory

FSx for Lustre
Diseñado para cargas de trabajo de alto rendimiento y computación intensiva.
Características:

Rendimiento de subsegundos (latencia ultra baja)
Integración con S3 para acceso a datos
Escalable a terabytes y teraflops
Optimizado para HPC (High-Performance Computing)
Bajo costo para almacenamiento vinculado

Casos de uso:

Machine Learning y análisis de Big Data
Simulaciones científicas
Procesamiento de imágenes y vídeos
Análisis financiero

FSx for NetApp ONTAP (Disponible)
Sistema de archivos empresarial con características avanzadas de gestión.
Características:

Compatibilidad con SMB y NFS simultáneamente
Snapshots eficientes en espacio
Replicación de datos
Soporte para Windows, Linux y macOS

Comparativa: EBS vs EFS vs FSx
AspectoEBSEFSFSxTipoBloquesArchivos NFSArchivos EspecializadosAccesoInstancia únicaMúltiples instanciasMúltiples instanciasProtocoloN/ANFSSMB / NFS / LustreSistema OperativoCualquieraLinuxWindows / Linux / NetAppRendimientoMuy altoAltoUltra alto (Lustre)Caso de UsoAlmacenamiento localCompartido simpleEmpresarial / HPC

67. Resumen - Almacenamiento de Instancias EC2
A continuación se presenta un resumen completo de todas las opciones de almacenamiento disponibles para instancias EC2:
EBS (Elastic Block Store)

Descripción: Almacenamiento persistente a nivel de bloque
Persistencia: Datos se conservan después de detener/terminar la instancia
Disponibilidad: Replicado automáticamente dentro de la AZ
Escalabilidad: Puede aumentarse dinámicamente
Casos de uso: Bases de datos, aplicaciones críticas, sistemas de archivos
Tipos: gp3/gp2 (propósito general), io1/io2 (alto rendimiento), st1/sc1 (HDD)

EBS Snapshots

Descripción: Copias de seguridad incrementales de volúmenes EBS
Almacenamiento: Se guardan en S3 (administrado por AWS)
Casos de uso: Backup, migración entre AZ/regiones, recuperación ante desastres
Restauración: Puede tomar 24-72 horas para grandes snapshots
Características: Papelera de reciclaje, archivo a bajo costo

AMI (Amazon Machine Image)

Descripción: Imagen preconfigurada del sistema operativo y software
Tipos: Públicas, privadas, marketplace
Creación: Desde una instancia EC2 existente personalizada
Lanzamiento: Crear nuevas instancias idénticas rápidamente
Componentes: Template raíz, permisos, mapeo de dispositivos

EC2 Image Builder

Descripción: Automatiza la creación, prueba y distribución de AMIs
Pipelines: Configuración automatizada y pruebas
Beneficios: Mantiene imágenes actualizadas y seguras
Integración: Amazon Inspector, Systems Manager

EC2 Instance Store

Descripción: Almacenamiento temporal incluido con la instancia
Persistencia: Datos se pierden al detener/terminar
Rendimiento: Muy rápido (acceso directo al hardware)
Casos de uso: Caché, datos de sesión, archivos temporales
Limitación: No para almacenamiento crítico

EFS (Elastic File System)

Descripción: Sistema de archivos NFS compartido entre múltiples instancias
Acceso: Multi-AZ automático
Escalabilidad: Crecimiento automático sin planificación
Clases de almacenamiento: Standard (acceso frecuente) e Infrequent Access (ahorro de costos)
Casos de uso: Almacenamiento compartido, aplicaciones colaborativas

Amazon FSx

Descripción: Sistemas de archivos especializados y administrados
FSx for Windows: Compatibilidad total con SMB y Active Directory
FSx for Lustre: Alto rendimiento para HPC y Machine Learning
Características: Integración empresarial, rendimiento optimizado
Casos de uso: Entornos empresariales, computación intensiva

Comparativa de Selección
Usa esta matriz para elegir el almacenamiento correcto:
NecesidadSolución RecomendadaAlmacenamiento rápido de una instanciaEBS gp3Múltiples instancias, datos compartidosEFSDatos no persistentes, cachéInstance StoreBackup y recuperación ante desastresEBS SnapshotsLanzar instancias configuradasAMIActualizar y mantener imágenesEC2 Image BuilderServidor Windows empresarialFSx for WindowsComputación de alto rendimientoFSx for Lustre

68. Limpieza de Secciones
Para evitar costos innecesarios en AWS, es importante limpiar los recursos que ya no utilices:
Volúmenes EBS
bash# Eliminar volúmenes no asociados
1. En la consola de EC2, ve a "Volúmenes"
2. Selecciona volúmenes con estado "Disponible"
3. Click en "Eliminar volumen"
4. Confirma la eliminación
Consideraciones:

No puedes eliminar un volumen asociado a una instancia activa
Detén o termina la instancia primero
Ten cuidado: la eliminación es permanente

Snapshots de EBS
bash# Eliminar snapshots antiguos o innecesarios
1. En la consola de EC2, ve a "Snapshots"
2. Selecciona los snapshots a eliminar
3. Click en "Eliminar snapshot"
4. Confirma la eliminación
Mejores prácticas:

Mantén snapshots recientes para recuperación
Archiva snapshots antiguos a bajo costo
Utiliza etiquetas para organizar snapshots

AMIs Personalizadas
bash# Eliminar AMIs no utilizadas
1. En la consola de EC2, ve a "AMIs"
2. Selecciona tu AMI personalizada
3. Click en "Deregistrar imagen"
4. Elimina el snapshot raíz asociado
Instancias EC2
bash# Terminar instancias de prueba
1. En la consola de EC2, ve a "Instancias"
2. Selecciona la instancia a terminar
3. Click en "Estado de instancia" → "Terminar instancia"
4. Confirma la terminación
Diferencia importante:

Stop: Pausa la instancia (EBS persiste)
Terminate: Elimina la instancia (se pierden datos del volumen raíz por defecto)

Sistemas de Archivos Compartidos
bash# Eliminar EFS
1. En la consola de EFS, ve a "Sistema de archivos"
2. Desmonta todos los objetivos de montaje
3. Elimina el sistema de archivos

# Eliminar FSx
1. En la consola de FSx, selecciona el sistema
2. Click en "Eliminar"
3. Confirma la eliminación
Checklist de Limpieza
markdown- [ ] Eliminar volúmenes EBS no asociados
- [ ] Eliminar snapshots antiguos (o archivar)
- [ ] Deregistrar AMIs personalizadas innecesarias
- [ ] Terminar instancias EC2 de desarrollo/prueba
- [ ] Eliminar sistemas EFS no utilizados
- [ ] Eliminar sistemas FSx innecesarios
- [ ] Revisar costos en Cost Explorer
- [ ] Configurar alarmas de facturación

Cuestionario - Almacenamiento de Instancias EC2
Instrucciones
Este cuestionario tiene como objetivo reforzar tu comprensión de los servicios de almacenamiento en EC2. Selecciona la respuesta correcta para cada pregunta. Al final encontrarás las respuestas correctas.

Pregunta 1: Características de EBS
¿Cuál de las siguientes afirmaciones sobre Amazon EBS es correcta?

A) Los datos se pierden si detienes la instancia EC2
B) El almacenamiento es replicado automáticamente entre todas las regiones
C) Los datos persisten después de detener o terminar la instancia (salvo el volumen raíz)
D) EBS es más económico que Instance Store para datos temporales

Respuesta correcta: C
Explicación: EBS es almacenamiento persistente, lo que significa que los datos se conservan incluso después de detener la instancia. Sin embargo, el volumen raíz se elimina por defecto al terminar la instancia, a menos que desactives la opción "Delete on termination". EBS está replicado dentro de la AZ, no entre regiones.

Pregunta 2: EBS Multi-Attach
¿Cuáles son los requisitos para usar EBS Multi-Attach? (SELECCIONA DOS)

A) Debe usarse con instancias de propósito general
B) Solo disponible para volúmenes io1 e io2
C) Todas las instancias deben estar en la misma AZ
D) Requiere configuración de un sistema de archivos distribuido
E) Compatible con cualquier tipo de volumen EBS

Respuestas correctas: B y C
Explicación: EBS Multi-Attach solo está disponible para volúmenes de alto rendimiento (io1 e io2) y todas las instancias deben estar en la misma zona de disponibilidad. Además, cada instancia debe tener un sistema de archivos compatible con acceso concurrente (como Lustre o GFS2).

Pregunta 3: EBS Snapshots
¿Cuál es el principal beneficio de usar EBS Snapshots?

A) Aumentan automáticamente el rendimiento de EBS
B) Permiten replicar datos entre zonas de disponibilidad y regiones
C) Eliminan la necesidad de backups
D) Reducen el costo del almacenamiento en un 50%

Respuesta correcta: B
Explicación: Los snapshots son copias de seguridad incrementales almacenadas en S3 que permiten restaurar volúmenes en diferentes AZ o regiones. Esto es fundamental para la recuperación ante desastres y la migración. Los snapshots se pueden archivar a bajo costo, pero aún requieren una estrategia de backup completa.

Pregunta 4: AMI y Lanzamiento de Instancias
¿Cuál es la principal diferencia entre una AMI personalizada y una AMI pública?

A) Las AMI personalizadas no pueden iniciarse en múltiples AZ
B) Las AMI personalizadas contienen tu configuración específica y son privadas
C) Las AMI públicas son siempre más seguras
D) Las AMI personalizadas tienen un costo adicional

Respuesta correcta: B
Explicación: Una AMI personalizada es una imagen que creas a partir de una instancia existente con software, configuraciones y datos preinstalados. Las AMI públicas son proporcionadas por AWS o la comunidad. Las personalizadas pueden lanzarse en cualquier AZ dentro de su región.

Pregunta 5: EC2 Image Builder
¿Qué ventaja principal ofrece EC2 Image Builder?

A) Incrementa automáticamente la velocidad de las instancias
B) Automatiza la creación, prueba y distribución de AMIs seguras y actualizadas
C) Reemplaza la necesidad de usar EBS
D) Permite ejecutar múltiples sistemas operativos simultáneamente

Respuesta correcta: B
Explicación: EC2 Image Builder automatiza el pipeline completo de creación de imágenes: construye, prueba, valida y distribuye AMIs. Integra Amazon Inspector para verificar vulnerabilidades y Systems Manager para configuración, manteniendo las imágenes actualizadas automáticamente.

Pregunta 6: EC2 Instance Store
¿En qué situación es apropiado usar EC2 Instance Store?

A) Para almacenar bases de datos críticas
B) Para datos que deben persistir indefinidamente
C) Para cachés, datos temporales y buffers
D) Como reemplazo principal de EBS

Respuesta correcta: C
Explicación: Instance Store es almacenamiento temporal muy rápido incluido físicamente con la instancia. Se pierde al detener o terminar la instancia, por lo que es ideal para cachés, sesiones temporales y datos efímeros. Nunca debe usarse para datos críticos o permanentes.

Pregunta 7: EFS vs EBS
¿Cuál de las siguientes situaciones requiere EFS en lugar de EBS?

A) Una aplicación que necesita almacenamiento rápido en una sola instancia
B) Un sistema de archivos compartido accesible por múltiples instancias Linux simultáneamente
C) Una base de datos con requisitos IOPS muy altos
D) Un almacenamiento temporal para datos de sesión

Respuesta correcta: B
Explicación: EFS es un sistema de archivos NFS que puede ser montado simultáneamente por múltiples instancias EC2 en múltiples AZ. EBS está diseñado para instancias individuales, aunque puede adjuntarse a una sola instancia a la vez (excepto con Multi-Attach). EFS es ideal para aplicaciones colaborativas y almacenamiento compartido.

Pregunta 8: Modelo de Responsabilidad Compartida
En el contexto de almacenamiento EC2, ¿cuál es responsabilidad de AWS?

A) Cifrar manualmente todos los datos del usuario
B) Gestionar backups automáticos de todos los volúmenes
C) Proporcionar infraestructura, redundancia y disponibilidad física
D) Configurar las políticas de ciclo de vida del almacenamiento

Respuesta correcta: C
Explicación: AWS es responsable de la seguridad de la infraestructura física, la redundancia dentro de AZ y la disponibilidad del servicio. El usuario es responsable de configurar cifrado, crear snapshots, gestionar backups, establecer políticas de ciclo de vida y controlar el acceso.

Pregunta 9: Clases de Almacenamiento EFS
¿Cuál es el principal beneficio de usar EFS Infrequent Access (EFS-IA)?

A) Mejora el rendimiento de lectura/escritura
B) Reduce costos hasta un 92% para datos a los que se accede ocasionalmente
C) Permite el acceso desde múltiples regiones
D) Elimina la necesidad de configurar seguridad

Respuesta correcta: B
Explicación: EFS-IA es una clase de almacenamiento de bajo costo para archivos accedidos ocasionalmente. Las políticas de ciclo de vida mueven automáticamente archivos no accedidos a EFS-IA, reduciendo significativamente los costos. Puede ahorrar hasta un 92% comparado con EFS Standard.

Pregunta 10: Amazon FSx for Windows
¿Cuál es el caso de uso ideal para FSx for Windows File Server?

A) Procesamiento de Big Data y Machine Learning
B) Almacenamiento empresarial con integración Active Directory y protocolo SMB
C) Almacenamiento temporal de caché para aplicaciones web
D) Sincronización de archivos entre dispositivos móviles

Respuesta correcta: B
Explicación: FSx for Windows está optimizado para entornos empresariales Windows. Proporciona protocolo SMB, integración nativa con Active Directory, DFS y características avanzadas de administración. Es ideal para migrar sistemas legales de archivos compartidos o crear infraestructura Windows empresarial.

Pregunta 11: Amazon FSx for Lustre
¿Para qué tipo de carga de trabajo está optimizado FSx for Lustre?

A) Almacenamiento de documentos empresariales
B) Computación de alto rendimiento (HPC), Machine Learning y análisis de Big Data
C) Sincronización de archivos entre múltiples usuarios
D) Backup y recuperación ante desastres

Respuesta correcta: B
Explicación: FSx for Lustre está diseñado para cargas de trabajo intensivas: HPC, Machine Learning, procesamiento de imágenes y análisis científico. Ofrece rendimiento ultra rápido (subsegundos), integración con S3 y escalabilidad masiva. No es adecuado para usos empresariales generales.

Pregunta 12: Tipos de Volúmenes EBS
¿Cuál es la diferencia principal entre volúmenes EBS gp3 y io1?

A) gp3 es más económico y adecuado para propósito general; io1 está optimizado para IOPS muy altos
B) io1 puede usarse solo en Windows; gp3 solo en Linux
C) gp3 persiste después de terminar la instancia; io1 no persiste
D) No hay diferencia significativa en rendimiento

Respuesta correcta: A
Explicación: gp3 (propósito general) ofrece excelente relación precio-rendimiento para la mayoría de aplicaciones. io1 (optimizado para IOPS) está diseñado para bases de datos y aplicaciones que requieren IOPS muy predecibles y altos. io2 es la versión mejorada de io1 con mayor durabilidad.

Pregunta 13: Snapshots y Restauración
¿Cuál es la característica principal de la papelera de reciclaje para Snapshots de EBS?

A) Incrementa automáticamente el tamaño de tus snapshots
B) Permite recuperar snapshots eliminados accidentalmente dentro de un período de retención
C) Archiva snapshots automáticamente a bajo costo
D) Comprime snapshots para reducir almacenamiento

Respuesta correcta: B
Explicación: La papelera de reciclaje para EBS Snapshots permite especificar un período de retención (1 día a 1 año) durante el cual los snapshots eliminados pueden recuperarse. Esta es una característica importante para prevenir la pérdida accidental de copias de seguridad críticas.

Pregunta 14: Elección de Almacenamiento
Una aplicación requiere almacenamiento compartido accesible por 50 servidores Linux con baja latencia y costo optimizado. ¿Cuál es la mejor opción?

A) EBS con Multi-Attach
B) Instance Store en cada servidor
C) EFS (Elastic File System)
D) Amazon FSx for Windows

Respuesta correcta: C
Explicación: EFS es la mejor opción porque es un sistema de archivos NFS compartido que puede ser montado simultáneamente por múltiples instancias Linux. Es escalable, de alto rendimiento, con políticas de ciclo de vida para optimizar costos, y no requiere administración de servidor.

Pregunta 15: Cifrado y Seguridad
¿Cuál de las siguientes afirmaciones sobre cifrado en almacenamiento EC2 es correcta?

A) El cifrado de EBS siempre está habilitado automáticamente
B) EFS no soporta cifrado
C) El cifrado debe habilitarse al crear el volumen o se puede habilitar después
D) El cifrado de datos en tránsito no es importante para EFS

Respuesta correcta: C
Explicación: El cifrado de EBS puede habilitarse al crear el volumen (recomendado) o en algunos casos después de la creación. EFS soporta cifrado en reposo y en tránsito. El cifrado es un componente clave del modelo de responsabilidad compartida donde el usuario debe asegurar que sus datos están protegidos.

Hoja de Respuestas
PreguntaRespuestaTemas Relacionados1CVisión General de EBS2B, CEBS Multi-Attach3BEBS Snapshots4BAMI5BEC2 Image Builder6CEC2 Instance Store7BEFS8CModelo de Responsabilidad Compartida9BEFS Clases de Almacenamiento10BFSx for Windows11BFSx for Lustre12ATipos de Volúmenes EBS13BEBS Snapshots - Características14CSelección de Almacenamiento15CCifrado en Almacenamiento

Análisis de Desempeño

14-15 correctas: ¡Excelente! Dominas completamente el almacenamiento en EC2
11-13 correctas: Muy bien. Repasa los temas con puntuaciones más bajas
8-10 correctas: Bien. Revisa los conceptos clave nuevamente
Menos de 8: Dedica más tiempo a estudiar los servicios de almacenamiento


Recursos Adicionales

Documentación oficial de EBS
Documentación oficial de EFS
Documentación oficial de FSx
Calculadora de precios de AWS
AWS Well-Architected Framework
