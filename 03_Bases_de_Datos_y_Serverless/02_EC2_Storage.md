# Almacenamiento en Instancias EC2

## Índice de Contenido

1. Visión General de EBS
2. Acerca de EBS Multi-Attach
3. EBS - Práctica
4. Visión General de EBS Snapshots
5. EBS Snapshots - Práctica
6. Visión General de AMI
7. AMI - Práctica
8. Visión General de EC2 Image Builder
9. EC2 Instance Store
10. Visión General de EFS
11. Modelo de Responsabilidad Compartida
12. Visión General de Amazon FSx
13. Resumen - Almacenamiento de Instancias EC2
14. Limpieza de Secciones
15. Cuestionario - Almacenamiento de Instancias EC2

---

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
2. Configurar los parámetros
3. Click en "Crear Volumen" (Create Volume)

### Paso 3: Adjuntar el Volumen a una Instancia EC2

Una vez creado el volumen:

1. Selecciona el volumen creado
2. Click en "Acciones" > "Asociar Volumen" (Attach Volume)
3. Configurar la instancia y dispositivo
4. Click en "Asociar" (Attach)

### Paso 4: Verificar desde la Instancia EC2

Conéctate a tu instancia y verifica:

```bash
# Listar todos los dispositivos de bloque
lsblk
```

### Paso 5: Formatear y Montar el Volumen

```bash
# 1. Crear sistema de archivos ext4
sudo mkfs -t ext4 /dev/xvdf

# 2. Crear punto de montaje
sudo mkdir /data

# 3. Montar el volumen
sudo mount /dev/xvdf /data

# 4. Verificar que está montado
df -h
```

### Paso 6: Desadjuntar un Volumen

1. Desde la instancia EC2, desmontar primero: `sudo umount /data`
2. En la Consola de AWS, seleccionar el volumen
3. Click en "Acciones" > "Desasociar Volumen"

⚠️ **IMPORTANTE:** Siempre desmonta primero desde el SO

### Paso 7: Modificar un Volumen Existente

1. Selecciona el volumen
2. Click en "Acciones" > "Modificar Volumen"
3. Cambiar parámetros (tipo, tamaño, IOPS, throughput)
4. Click en "Modificar"

### 🎯 Ejercicio Práctico

1. ✅ Crea un volumen EBS de 5 GB tipo gp3
2. ✅ Adjúntalo a una instancia EC2 activa
3. ✅ Conéctate por SSH y verifica con `lsblk`
4. ✅ Formatea el volumen con ext4
5. ✅ Móntalo en `/datos`
6. ✅ Crea un archivo de prueba
7. ✅ Verifica que el archivo persiste después de reiniciar

### ⚠️ Notas Importantes

| Concepto | Detalle |
|---|---|
| **Volumen Root** | Se elimina por defecto al terminar la instancia |
| **Volúmenes Adicionales** | NO se eliminan por defecto al terminar la instancia |
| **Zona de Disponibilidad** | Un volumen solo puede adjuntarse a instancias en la misma AZ |
| **Snapshots** | Usa snapshots para mover volúmenes entre AZs o regiones |

---

## 4. Visión General de EBS Snapshots

Un **Snapshot de EBS** es una copia de seguridad puntual (point-in-time) de un volumen EBS. Los snapshots son **incrementales**, lo que significa que solo se guardan los bloques que han cambiado desde el último snapshot.

### ¿Qué son los Snapshots?

Los snapshots son copias de seguridad incrementales que se almacenan automáticamente en Amazon S3 y permiten restaurar volúmenes en diferentes zonas de disponibilidad o regiones.

### Características Principales

| Característica | Descripción |
|---|---|
| **Incremental** | Solo guarda los bloques modificados desde el último snapshot |
| **Almacenamiento** | Se almacenan en Amazon S3 (gestionado automáticamente por AWS) |
| **Región** | Puede copiarse a otras regiones |
| **Zona de Disponibilidad** | Puede restaurarse en cualquier AZ dentro de la región |
| **Cifrado** | Los snapshots de volúmenes cifrados están automáticamente cifrados |

### Beneficios de los Snapshots

✅ **Backup y Recuperación** - Protección contra pérdida de datos y restauración rápida de volúmenes

✅ **Migración** - Mover datos entre zonas de disponibilidad y copiar datos a otras regiones

✅ **Creación de AMIs** - Base para crear Amazon Machine Images

✅ **Ahorro de Costos** - Almacenamiento incremental reduce costos

### Características Avanzadas de Snapshots

#### EBS Snapshot Archive

Mueve snapshots antiguos a un nivel de almacenamiento de menor costo (75% más barato) pero con restauración de 24-72 horas.

#### Recycle Bin

Protege contra eliminación accidental de snapshots con retención de 1 día a 1 año.

#### Fast Snapshot Restore (FSR)

Acelera la restauración de snapshots a rendimiento completo inmediato.

### Copiar Snapshots Entre Regiones

Puedes copiar snapshots a otras regiones para:

- 🌍 Recuperación ante desastres
- 🚀 Expansión global
- 📋 Cumplimiento normativo

### 💡 Mejores Prácticas

1. ✅ Programa snapshots regulares para protección de datos críticos
2. ✅ Usa etiquetas para organizar y gestionar snapshots
3. ✅ Elimina snapshots antiguos para ahorrar costos
4. ✅ Copia snapshots críticos a otra región
5. ✅ Habilita Recycle Bin para protección contra eliminación accidental

---

## 5. EBS Snapshots - Práctica

Aprende a crear, gestionar y restaurar snapshots de EBS desde la consola de AWS.

### Paso 1: Crear un Snapshot

1. Navega a: EC2 > Elastic Block Store > Snapshots
2. Click en "Crear Snapshot"
3. Configura el tipo de recurso, ID del volumen y etiquetas
4. Click en "Crear Snapshot"

### Paso 2: Monitorear el Progreso

El estado del snapshot pasará de "pending" a "completed". Verifica el progreso en la columna "Progress".

### Paso 3: Crear un Volumen desde un Snapshot

1. Selecciona el snapshot
2. Click en "Acciones" > "Crear Volumen desde Snapshot"
3. Configura el nuevo volumen
4. Click en "Crear Volumen"

### Paso 4: Copiar un Snapshot a Otra Región

1. Selecciona el snapshot
2. Click en "Acciones" > "Copiar Snapshot"
3. Selecciona la región de destino
4. Click en "Copiar Snapshot"

### Paso 5: Compartir un Snapshot

1. Selecciona el snapshot
2. Click en "Acciones" > "Modificar Permisos"
3. Configura permisos (Privada, Pública, o Compartida)
4. Click en "Guardar"

### Paso 6: Configurar Snapshot Archive

1. Selecciona el snapshot
2. Click en "Acciones" > "Archivar Snapshot"
3. Confirma el archivado
4. Para restaurar, click en "Acciones" > "Restaurar desde Archivo"

### Paso 7: Configurar Recycle Bin

1. Navega a: EC2 > Elastic Block Store > Recycle Bin
2. Click en "Crear Regla de Retención"
3. Configura nombre, tipo de recurso, retención y etiquetas
4. Click en "Crear Regla de Retención"

### Paso 8: Recuperar un Snapshot de Recycle Bin

1. Navega a: EC2 > Recycle Bin > Resources
2. Busca el snapshot eliminado
3. Selecciónalo y click en "Recover"

### Paso 9: Automatizar Snapshots con Data Lifecycle Manager

1. Navega a: EC2 > Elastic Block Store > Lifecycle Manager
2. Click en "Crear Política de Ciclo de Vida"
3. Configura volúmenes objetivo, programación y retención
4. Click en "Crear Política"

### 🎯 Ejercicio Práctico Completo

**PARTE 1: Crear y Preparar**
1. ✅ Crea un volumen EBS de 8 GB
2. ✅ Adjúntalo a una instancia EC2
3. ✅ Formatea y monta el volumen en /backup
4. ✅ Crea archivos de prueba

**PARTE 2: Snapshot y Restauración**
5. ✅ Crea un snapshot del volumen
6. ✅ Etiquétalo apropiadamente
7. ✅ Espera que se complete
8. ✅ Crea un nuevo volumen desde el snapshot
9. ✅ Adjunta a otra instancia
10. ✅ Verifica que los archivos están presentes

**PARTE 3: Gestión Avanzada**
11. ✅ Copia el snapshot a otra región
12. ✅ Configura una regla de Recycle Bin
13. ✅ Crea una política de Data Lifecycle Manager
14. ✅ Elimina el snapshot original
15. ✅ Recupéralo desde Recycle Bin

**PARTE 4: Limpieza**
16. ✅ Elimina los volúmenes de prueba
17. ✅ Elimina los snapshots no necesarios
18. ✅ Verifica que no quedaron recursos huérfanos

### 💡 Mejores Prácticas de Snapshots

| Práctica | Razón |
|---|---|
| **Etiqueta todo** | Facilita la organización y búsqueda |
| **Nombra descriptivamente** | Usa formatos como "backup-prod-db-YYYYMMDD" |
| **Programa backups automáticos** | Usa Data Lifecycle Manager |
| **Mantén snapshots cross-region** | Para disaster recovery |
| **Elimina snapshots antiguos** | Ahorra costos |
| **Usa Recycle Bin** | Protección contra eliminación accidental |
| **Documenta el contenido** | En la descripción o en etiquetas |
| **Verifica restauraciones** | Prueba periódicamente |

### 📊 Costos de Snapshots

| Tipo | Costo (us-east-1) |
|---|---|
| **Standard** | ~$0.05 por GB/mes |
| **Archive** | ~$0.0125 por GB/mes (75% ahorro) |

Ejemplo: 100 GB de snapshots Standard = $5.00/mes; Archive = $1.25/mes

### ⚠️ Errores Comunes a Evitar

❌ No verificar que el snapshot se completó antes de usarlo
❌ Crear volumen en AZ incorrecta
❌ No probar las restauraciones periódicamente
❌ Hacer snapshots públicos sin querer
❌ No documentar el contenido del snapshot

---

## 6. Visión General de AMI (Amazon Machine Image)

Una **AMI (Amazon Machine Image)** es una plantilla preconfigurada que contiene el sistema operativo, aplicaciones y configuraciones necesarias para lanzar una instancia EC2. Es como una "fotografía completa" de un servidor que puedes usar para crear copias idénticas.

### ¿Qué es una AMI?

Una AMI contiene:

- 📦 Sistema Operativo (Linux/Windows)
- ⚙️ Configuraciones del Sistema
- 📝 Software Instalado
- 🔧 Configuraciones de Aplicaciones
- 💾 Volumen(es) Root y Datos
- 🏷️ Metadata y Permisos

### Tipos de AMIs

#### AMIs Públicas

Proporcionadas por AWS o la comunidad:
- Amazon Linux 2023
- Ubuntu Server 22.04 LTS
- Red Hat Enterprise Linux 9
- Windows Server 2022

#### AMIs Privadas

Creadas por ti y solo visibles en tu cuenta:
- Servidores web preconfigurados
- Aplicaciones corporativas
- Configuraciones de seguridad específicas
- Entornos de desarrollo estandarizados

#### AWS Marketplace AMIs

Ofrecidas por terceros (pueden tener costo adicional):
- Software comercial preinstalado
- Soluciones de seguridad
- Bases de datos optimizadas
- Stacks de aplicaciones completas

### Componentes de una AMI

| Componente | Descripción |
|---|---|
| **Plantilla del volumen root** | El sistema operativo y aplicaciones base |
| **Permisos de lanzamiento** | Qué cuentas de AWS pueden usar la AMI |
| **Mapeo de dispositivos de bloque** | Especifica los volúmenes a adjuntar |
| **Region** | Las AMIs son específicas de región |
| **Arquitectura** | x86_64 o ARM64 (Graviton) |
| **Tipo de virtualización** | HVM o PV |

### Ventajas de Usar AMIs

| Ventaja | Beneficio |
|---|---|
| **Tiempo de arranque rápido** | Todo el software está preinstalado |
| **Configuración consistente** | Todas las instancias son idénticas |
| **Escalabilidad** | Lanza múltiples instancias rápidamente |
| **Backup completo del sistema** | Incluye SO, apps y configuraciones |
| **Portabilidad** | Copia AMIs entre regiones |
| **Recuperación ante desastres** | Restaura sistemas completos rápidamente |

### Casos de Uso Comunes

- **Entornos de Desarrollo/Testing** - Configuración estandarizada
- **Despliegue de Aplicaciones** - Apps preconfiguradas
- **Auto Scaling** - Nuevas instancias listas instantáneamente
- **Disaster Recovery** - Restauración rápida de sistemas
- **Migración a AWS** - MigraciÃ³n simplificada

### 💡 Mejores Prácticas para AMIs

1. ✅ Usa AMIs propias para producción - Mayor control y seguridad
2. ✅ Documenta el contenido de cada AMI - Usa descripciones claras
3. ✅ Etiqueta tus AMIs - Nombre, Versión, Fecha, Ambiente
4. ✅ Mantén AMIs actualizadas - Crea nuevas versiones regularmente
5. ✅ Elimina AMIs antiguas - Ahorra costos de almacenamiento
6. ✅ Prueba antes de usar en producción - Valida que funciona
7. ✅ Copia AMIs críticas a otras regiones - Para disaster recovery
8. ✅ Cifra AMIs con datos sensibles - Usa KMS para cifrado

### 📊 Costos de AMIs

Las AMIs no tienen costo directo, pero sí los snapshots subyacentes:
- Snapshot root (10 GB): $0.50/mes
- Snapshot datos (50 GB): $2.50/mes
- Total: $3.00/mes
- Si copias a otra región: costo duplicado

---

## 7. AMI - Práctica

Aprende a crear, gestionar y usar AMIs personalizadas en AWS.

### Paso 1: Preparar una Instancia para la AMI

Configura una instancia con todo lo que necesitas:

```bash
# Actualiza el sistema
sudo yum update -y  # Amazon Linux
sudo apt update && sudo apt upgrade -y  # Ubuntu

# Instala software
sudo yum install -y httpd git  # Amazon Linux
sudo apt install -y apache2 git  # Ubuntu

# Configura aplicaciones
sudo systemctl start httpd
sudo systemctl enable httpd

# Limpia datos temporales
sudo rm -rf /tmp/*
history -c
```

### Paso 2: Crear una AMI desde la Instancia

1. Navega a: EC2 > Instancias
2. Selecciona tu instancia configurada
3. Click en "Acciones" > "Imagen y plantillas" > "Crear imagen"
4. Configura nombre, descripción y tags
5. Click en "Crear imagen"

### Paso 3: Monitorear la Creación

1. Navega a: EC2 > Imágenes > AMIs
2. Observa el estado (pending → available)
3. Verifica los snapshots asociados

Tiempo estimado: 5-15 minutos

### Paso 4: Lanzar una Instancia desde tu AMI

1. Navega a: EC2 > AMIs
2. Selecciona tu AMI
3. Click en "Lanzar instancia desde AMI"
4. Configura la instancia
5. Click en "Lanzar instancia"

La instancia ya tiene todo configurado - no necesitas instalar nada.

### Paso 5: Copiar AMI a Otra Región

Para disaster recovery o expansión global:

1. Navega a: EC2 > AMIs
2. Selecciona tu AMI
3. Click en "Acciones" > "Copiar AMI"
4. Selecciona la región de destino
5. Click en "Copiar AMI"

### Paso 6: Compartir AMI con Otra Cuenta

1. Navega a: EC2 > AMIs
2. Selecciona tu AMI
3. Click en "Acciones" > "Editar permisos de AMI"
4. Configura permisos (Privada, Pública, o Compartida)
5. Click en "Guardar cambios"

### Paso 7: Deprecar una AMI

Cuando una AMI está desactualizada:

1. Navega a: EC2 > AMIs
2. Selecciona la AMI antigua
3. Click en "Acciones" > "Deprecar AMI"
4. Configura fecha de desaprobación
5. Click en "Deprecar"

### Paso 8: Eliminar (Deregistrar) una AMI

Cuando ya no necesitas una AMI:

1. Navega a: EC2 > AMIs
2. Selecciona la AMI a eliminar
3. Click en "Acciones" > "Desregistrar AMI"
4. Confirma la operación
5. **IMPORTANTE:** Elimina manualmente los snapshots asociados

### 🎯 Ejercicio Práctico Completo

**PARTE 1: Configuración Inicial**
1. ✅ Lanza instancia EC2 Amazon Linux 2023
2. ✅ Instala y configura servidor web y PHP
3. ✅ Personaliza la página de inicio
4. ✅ Configura inicio automático de servicios

**PARTE 2: Crear y Usar AMI**
5. ✅ Crea AMI de la instancia configurada
6. ✅ Etiqueta: Nombre, Versión, Ambiente
7. ✅ Espera a que esté disponible
8. ✅ Lanza 2 instancias nuevas desde la AMI
9. ✅ Verifica que funcionan idénticamente

**PARTE 3: Gestión Avanzada**
10. ✅ Copia la AMI a otra región (us-west-2)
11. ✅ Comparte con otra cuenta si tienes
12. ✅ En us-west-2, lanza instancia desde AMI copiada
13. ✅ Crea versión 2.0 con cambios adicionales
14. ✅ Depreca la versión 1.0

**PARTE 4: Limpieza**
15. ✅ Termina todas las instancias de prueba
16. ✅ Desregistra AMIs que no necesites
17. ✅ Elimina snapshots huérfanos
18. ✅ Verifica costos finales

### 💡 Tips y Mejores Prácticas

| Práctica | Beneficio |
|---|---|
| **Nombra con versionado semántico** | `app-v1.2.3` facilita seguimiento |
| **Incluye fecha en descripción** | Sabrás cuándo se creó |
| **Limpia instancia antes de AMI** | Elimina logs, temporales, historiales |
| **Usa etiquetas consistentes** | Facilita automatización y búsqueda |
| **Documenta cambios entre versiones** | En descripción o en wiki externa |
| **Prueba AMI antes de producción** | Lanza instancia de test primero |
| **Mantén AMIs cross-region** | Para disaster recovery |
| **Automatiza creación periódica** | Scripts o Systems Manager |

---

## 8. Visión General de EC2 Image Builder

**Amazon EC2 Image Builder** es un servicio totalmente administrado que simplifica la creación, mantenimiento, validación y distribución de imágenes de servidor (AMIs) seguras y actualizadas.

### ¿Qué Problema Resuelve?

#### Sin EC2 Image Builder:
Proceso manual (lento y propenso a errores):
1. Lanzar instancia EC2
2. Conectarse por SSH
3. Instalar software manualmente
4. Configurar aplicaciones
5. Aplicar parches de seguridad
6. Testear manualmente
7. Crear AMI
8. Repetir para cada actualización

#### Con EC2 Image Builder:
Proceso automatizado:
1. Definir "receta" una vez
2. Image Builder ejecuta automáticamente todos los pasos
3. Programa ejecuciones periódicas

### Beneficios de EC2 Image Builder

| Beneficio | Descripción |
|---|---|
| **Automatización** | Elimina tareas manuales repetitivas |
| **Consistencia** | Misma configuración cada vez |
| **Seguridad** | Integración con Inspector y Systems Manager |
| **Cumplimiento** | Aplicación automática de estándares (CIS, STIG) |
| **Ahorro de tiempo** | Construcción y distribución en horas, no días |
| **Versionado** | Control de versiones de imágenes |
| **Testing integrado** | Validación automática antes de distribución |
| **Multi-región** | Distribución automática a múltiples regiones |
| **Sin costo adicional** | Solo pagas por recursos subyacentes |

### Casos de Uso

- **Actualizaciones de Seguridad Regulares** - Pipeline programado semanalmente
- **Imágenes Corporativas Estandarizadas** - Configuración uniforme en toda la organización
- **Cumplimiento Normativo** - Componentes AWS de hardening automático
- **CI/CD de Infraestructura** - Integración con pipeline DevOps

### ⚠️ Importante para el Examen

**Debes saber:**
- Qué es EC2 Image Builder
- Que automatiza creación de AMIs
- Que incluye testing y distribución
- Que es gratuito (solo pagas recursos subyacentes)

**NO necesitas saber:**
- Sintaxis de componentes
- Detalles de programación de pipelines
- Configuración avanzada

---

## 9. EC2 Instance Store

**EC2 Instance Store** proporciona almacenamiento temporal a nivel de hardware directamente adjunto al servidor físico que ejecuta tu instancia EC2. A diferencia de EBS, este almacenamiento es **efímero** (temporal).

### Características Principales

| Característica | Instance Store | EBS |
|---|---|---|
| **Persistencia** | ❌ Temporal | ✅ Persistente |
| **Rendimiento** | ⚡ Muy alto (IOPS millones) | ✅ Alto (limitado por red) |
| **Latencia** | ⚡ Muy baja (microsegundos) | ✅ Baja (milisegundos) |
| **Ubicación** | Disco físico local | Almacenamiento en red |
| **Snapshots** | ❌ No soportado | ✅ Sí |
| **Costo adicional** | ❌ Incluido | ✅ Sí |
| **TamaÃ±o** | Fijo según instancia | Variable (1-64 TB) |
| **Detener instancia** | ❌ Datos se pierden | ✅ Datos persisten |
| **Terminar instancia** | ❌ Datos se pierden | ✅ Datos persisten |

### ⚠️ Comportamiento de los Datos

Acciones que **PIERDEN datos** en Instance Store:

- ❌ Detener la instancia (Stop)
- ❌ Terminar la instancia (Terminate)
- ❌ Fallo del disco físico subyacente
- ❌ Fallo del host físico
- ❌ Hibernar la instancia

Acciones que **MANTIENEN datos**:

- ✅ Reiniciar la instancia (Reboot)

### Tipos de Instancias con Instance Store

No todas las instancias tienen Instance Store. Ejemplos:

| Familia | Tipo de Instancia | Instance Store | Tamaño | Tipo |
|---|---|---|---|---|
| **Almacenamiento Optimizado** | i3.large | ✅ | 475 GB | NVMe SSD |
| | i3.xlarge | ✅ | 950 GB | NVMe SSD |
| **Computación Optimizada** | c5d.large | ✅ | 50 GB | NVMe SSD |
| **Propósito General** | m5d.large | ✅ | 75 GB | NVMe SSD |

💡 **Nota:** Las variantes con "d" en el nombre incluyen Instance Store.

### Rendimiento de Instance Store

Instance Store ofrece rendimiento excepcional. Ejemplo: Instancia i3.16xlarge

- Instance Store: 8 × 1.9 TB NVMe
- Total: 15.2 TB
- IOPS lecturas: 3.3 millones
- IOPS escrituras: 1.4 millones
- Throughput lectura: 16 GB/s

### Casos de Uso Apropiados

✅ **BUENO para Instance Store:**

- Almacenamiento temporal/cache (Redis, Memcached)
- Datos replicados (Cassandra, MongoDB)
- Procesamiento de datos temporales (Hadoop, Spark)
- Alto rendimiento I/O

❌ **MALO para Instance Store:**

- Bases de datos críticas sin réplicas
- Archivos importantes sin respaldo
- Datos que deben persistir
- Volúmenes root importantes

### 💡 Mejores Prácticas

| Práctica | Razón |
|---|---|
| **Nunca usar para datos únicos** | Se perderán al detener/terminar |
| **Siempre tener respaldo o réplicas** | Protección contra pérdida |
| **Usar RAID 0 para múltiples discos** | Maximizar rendimiento |
| **Documentar que es efímero** | Evitar malentendidos |
| **Monitorear salud del disco** | Detectar fallos |
| **Automatizar reconstrucción** | Recovery rápido |

### Cuándo Elegir Instance Store vs EBS

**Elige INSTANCE STORE si:**
- ✅ Necesitas máximo rendimiento I/O
- ✅ Los datos son temporales o replicados
- ✅ Tienes estrategia de respaldo/réplica
- ✅ La aplicación tolera pérdida de datos
- ✅ Quieres reducir costos

**Elige EBS si:**
- ✅ Los datos deben persistir
- ✅ Necesitas snapshots
- ✅ Datos críticos sin réplicas
- ✅ Flexibilidad en tamaño
- ✅ Necesitas adjuntar/desadjuntar

---

## 10. Visión General de EFS

**Amazon EFS** es un sistema de archivos NFS (Network File System) totalmente administrado, escalable y elástico para usar con servicios de AWS Cloud. A diferencia de EBS, EFS puede montarse en **múltiples instancias EC2 simultáneamente**.

### ¿Qué es EFS?

EFS es un sistema de archivos compartido que proporciona:

- 📤 Sistema de archivos compartido
- 🔄 Múltiples instancias pueden acceder simultáneamente
- 📊 Totalmente administrado
- 🎯 Altamente disponible
- 📈 Escalable automáticamente
- 🔒 Compatible con NFS

### Características Principales

| Característica | Descripción |
|---|---|
| **Sistema compartido** | Múltiples EC2 acceden simultáneamente |
| **Administrado** | AWS gestiona hardware, parches, backups |
| **Disponible** | Datos replicados en múltiples AZs |
| **Escalable** | Crece automáticamente hasta petabytes |
| **Compatible NFS** | Protocolo estándar NFSv4.1 |
| **Solo Linux** | Compatible con EC2 Linux, no Windows |
| **Pago por uso** | Solo pagas por lo que usas |

### Clases de Almacenamiento en EFS

| Clase | Descripción | Costo | Uso |
|---|---|---|---|
| **Standard** | Acceso frecuente | ~$0.30/GB/mes | Archivos accedidos regularmente |
| **Infrequent Access (IA)** | Acceso poco frecuente | ~$0.025/GB/mes | Archivos accedidos raramente |

**Ahorro con EFS-IA:** Hasta 92% comparado con Standard

### Modos de Rendimiento

| Modo | Descripción | Latencia | Throughput | Uso |
|---|---|---|---|---|
| **General Purpose** | Balanceado | Baja | Hasta 7,000 ops/seg | Mayoría de casos |
| **Max I/O** | Alto throughput | Mayor | +500,000 ops/seg | Big Data |

### Casos de Uso de EFS

- **Aplicaciones Web Escalables** - Múltiples servidores web compartiendo contenido
- **Directorios Home Compartidos** - Usuarios acceden desde diferentes servidores
- **Content Management Systems** - WordPress en múltiples instancias
- **Desarrollo y Testing** - Equipos comparten código
- **Machine Learning** - Instancias acceden a datasets
- **Big Data y Analytics** - Cluster de procesamiento

### EFS vs EBS vs Instance Store vs S3

| Característica | EFS | EBS | Instance Store | S3 |
|---|---|---|---|---|
| **Tipo** | Archivos NFS | Bloques | Bloques | Objeto |
| **Adjunto a** | Múltiples EC2 | 1 EC2 | 1 EC2 | N/A |
| **Persistencia** | ✅ Sí | ✅ Sí | ❌ No | ✅ Sí |
| **Disponibilidad** | Multi-AZ | 1 AZ | 1 AZ | Global |
| **Escalamiento** | Automático | Manual | Fijo | Ilimitado |
| **Precio** | $0.30/GB | $0.08/GB | Incluido | $0.023/GB |
| **Latencia** | ~ms | ~ms | ~μs | ~100ms |

### ⚠️ Importante para el Examen

**Debes saber:**
- EFS es sistema de archivos compartido (NFS)
- Funciona con múltiples EC2 simultáneamente
- Solo compatible con Linux
- Escala automáticamente
- Alta disponibilidad (Multi-AZ)
- Pago por uso
- EFS-IA para reducir costos

**NO necesitas saber:**
- Configuración detallada de mount targets
- Sintaxis de NFS mount commands
- Detalles de throughput modes
- Configuración de access points

---

## 11. Modelo de Responsabilidad Compartida para el Almacenamiento EC2

El **Modelo de Responsabilidad Compartida** de AWS define claramente qué aspectos de seguridad y operación gestiona AWS y cuáles son responsabilidad del cliente.

### Concepto General

**AWS** gestiona la **infraestructura** (Seguridad DEL Cloud)
**Cliente** gestiona la **configuración** (Seguridad EN el Cloud)

### Responsabilidades de AWS

AWS es responsable de:

- 🏢 Data center físico
- ⚡ Electricidad y refrigeración
- 🔐 Seguridad física
- 💾 Hardware de almacenamiento
- 🌐 Infraestructura de red
- 🖥️ Servidores físicos
- ⚙️ Hypervisor y virtualización
- 🔄 Replicación automática de datos
- 🛠️ Mantenimiento de hardware
- 📊 Monitoreo de infraestructura

### Responsabilidades del Cliente

El cliente es responsable de:

- 🔒 Cifrado de datos
- 🗝️ Gestión de claves (KMS)
- 📋 Políticas IAM
- 🛡️ Security Groups
- 💾 Snapshots y backups
- 🔄 Replicación cross-region
- 📁 Organización de datos
- ⚙️ Configuración de volúmenes
- 🏷️ Etiquetado de recursos
- 📊 Monitoreo de uso y costos
- 🔧 Lifecycle policies
- 🚨 Alertas y notificaciones

### Responsabilidad Compartida por Servicio

#### EBS (Elastic Block Store)

| AWS Responsable | Cliente Responsable |
|---|---|
| ✅ Hardware físico | ✅ Cifrar volúmenes con KMS |
| ✅ Replicación dentro de AZ | ✅ Crear snapshots regularmente |
| ✅ Disponibilidad del servicio | ✅ Configurar "Delete on Termination" |
| ✅ Durabilidad | ✅ Gestionar permisos de snapshots |
| ✅ Mantenimiento | ✅ Copiar snapshots entre regiones |

#### EFS (Elastic File System)

| AWS Responsable | Cliente Responsable |
|---|---|
| ✅ Infraestructura multi-AZ | ✅ Configurar Security Groups |
| ✅ Replicación automática | ✅ Configurar ciclo de vida |
| ✅ Escalamiento automático | ✅ Gestionar permisos POSIX |
| ✅ Disponibilidad | ✅ Cifrado en tránsito (TLS) |
| ✅ Durabilidad | ✅ Monitorear uso |

#### Instance Store

| AWS Responsable | Cliente Responsable |
|---|---|
| ✅ Hardware de discos | ⚠️ Entender datos son temporales |
| ✅ Mantenimiento | ✅ Implementar backups propios |
| ✅ Reemplazo de discos | ✅ Replicar datos críticos |
| | ✅ Aceptar riesgo de pérdida |

### Matriz de Responsabilidades

#### Seguridad

| Aspecto | AWS | Cliente |
|---|---|---|
| Seguridad física | ✅ | ❌ |
| Seguridad de red (infraestructura) | ✅ | ❌ |
| Seguridad de red (configuración) | ❌ | ✅ |
| Cifrado en reposo (capacidad) | ✅ | ❌ |
| Cifrado en reposo (activación) | ❌ | ✅ |
| Gestión de claves | ✅ (infraestructura) | ✅ (uso) |

#### Disponibilidad y Durabilidad

| Aspecto | AWS | Cliente |
|---|---|---|
| Replicación dentro de AZ (EBS) | ✅ | ❌ |
| Replicación multi-AZ (EFS) | ✅ | ❌ |
| Backups/Snapshots (capacidad) | ✅ | ❌ |
| Backups/Snapshots (ejecución) | ❌ | ✅ |
| Disaster Recovery (infraestructura) | ✅ | ❌ |
| Disaster Recovery (plan y pruebas) | ❌ | ✅ |

### 💡 Puntos Clave para Recordar

1. ✅ **AWS = Infraestructura física y servicios base**
2. ✅ **Cliente = Configuración, datos y seguridad**
3. ✅ **Cifrado: AWS proporciona capacidad, cliente la activa**
4. ✅ **Backups: AWS proporciona herramienta, cliente ejecuta**
5. ✅ **Durabilidad: AWS replica, cliente gestiona cross-region**
6. ✅ **Instance Store: Cliente asume riesgo de pérdida**

---

## 12. Visión General de Amazon FSx

Amazon FSx es un servicio completamente administrado que proporciona sistemas de archivos de alto rendimiento optimizados para casos de uso específicos.

### Características Principales

| Característica | Descripción |
|---|---|
| **Administrado** | AWS gestiona aprovisionamiento, parcheo, mantenimiento |
| **Alto Rendimiento** | Optimizado para rendimiento y throughput |
| **Escalable** | Escala automáticamente con tus necesidades |
| **Integración Nativa** | Compatible con servicios AWS y aplicaciones locales |
| **Cifrado** | Cifrado en reposo y en tránsito |

### Tipos de FSx

#### FSx for Windows File Server

Ideal para entornos empresariales Windows:

- Protocolo SMB totalmente compatible
- Integración nativa con Active Directory
- Soporta DFS (Distributed File System)
- Múltiples AZ para alta disponibilidad
- Compatible con NTFS y permisos Windows

**Casos de uso:**
- Almacenamiento compartido para aplicaciones Windows
- Migración de sistemas legados
- Entornos empresariales con Active Directory

#### FSx for Lustre

Diseñado para cargas de trabajo de alto rendimiento:

- Rendimiento de subsegundos (latencia ultra baja)
- Integración con S3 para acceso a datos
- Escalable a terabytes y teraflops
- Optimizado para HPC
- Bajo costo para almacenamiento

**Casos de uso:**
- Machine Learning y análisis de Big Data
- Simulaciones científicas
- Procesamiento de imágenes y vídeos
- Análisis financiero

#### FSx for NetApp ONTAP

Sistema de archivos empresarial:

- Compatibilidad con SMB y NFS simultáneamente
- Snapshots eficientes en espacio
- Replicación de datos
- Soporte para Windows, Linux, macOS

### Comparativa: EBS vs EFS vs FSx

| Aspecto | EBS | EFS | FSx |
|---|---|---|---|
| **Tipo** | Bloques | Archivos NFS | Archivos Especializados |
| **Acceso** | Instancia única | Múltiples instancias | Múltiples instancias |
| **Protocolo** | N/A | NFS | SMB / NFS / Lustre |
| **SO** | Cualquiera | Linux | Windows / Linux / NetApp |
| **Rendimiento** | Muy alto | Alto | Ultra alto (Lustre) |
| **Caso de Uso** | Almacenamiento local | Compartido simple | Empresarial / HPC |

---

## 13. Resumen - Almacenamiento de Instancias EC2

### EBS (Elastic Block Store)

- Almacenamiento persistente a nivel de bloque
- Datos persisten después de detener/terminar
- Replicado automáticamente dentro de AZ
- Escalable dinámicamente
- Casos: BD, aplicaciones críticas, sistemas de archivos

### EBS Snapshots

- Copias de seguridad incrementales
- Almacenadas en S3
- Para backup, migración, DR
- Papelera de reciclaje disponible

### AMI (Amazon Machine Image)

- Imagen preconfigurada del SO y software
- Tipos: Públicas, privadas, marketplace
- Se crea desde instancia existente
- Para lanzar instancias idénticas rápidamente

### EC2 Image Builder

- Automatiza creación, prueba, distribución de AMIs
- Pipelines con configuración automatizada
- Mantiene imágenes actualizadas y seguras

### EC2 Instance Store

- Almacenamiento temporal incluido
- Datos se pierden al detener/terminar
- Muy rápido, acceso directo
- Para cache, datos sesión, archivos temporales

### EFS (Elastic File System)

- Sistema de archivos NFS compartido
- Múltiples EC2 acceden simultáneamente
- Multi-AZ automático
- Crecimiento automático
- Clases: Standard e Infrequent Access

### Amazon FSx

- Sistemas de archivos especializados
- FSx for Windows: SMB + Active Directory
- FSx for Lustre: Alto rendimiento HPC
- FSx for NetApp ONTAP: Características avanzadas

### Matriz de Selección

| Necesidad | Solución |
|---|---|
| Almacenamiento rápido de una instancia | EBS gp3 |
| Múltiples instancias, datos compartidos | EFS |
| Datos no persistentes, cache | Instance Store |
| Backup y DR | EBS Snapshots |
| Lanzar instancias configuradas | AMI |
| Actualizar y mantener imágenes | EC2 Image Builder |
| Servidor Windows empresarial | FSx for Windows |
| Computación de alto rendimiento | FSx for Lustre |

---

## 14. Limpieza de Secciones

Para evitar costos innecesarios en AWS, es importante limpiar los recursos que ya no utilices.

### Volúmenes EBS

1. En la consola de EC2, ve a "Volúmenes"
2. Selecciona volúmenes con estado "Disponible"
3. Click en "Eliminar volumen"
4. Confirma la eliminación

**Nota:** No puedes eliminar un volumen asociado a una instancia activa.

### Snapshots de EBS

1. En la consola de EC2, ve a "Snapshots"
2. Selecciona los snapshots a eliminar
3. Click en "Eliminar snapshot"
4. Confirma la eliminación

**Nota:** Mantén snapshots recientes para recuperación

### AMIs Personalizadas

1. En la consola de EC2, ve a "AMIs"
2. Selecciona tu AMI personalizada
3. Click en "Deregistrar imagen"
4. Elimina el snapshot raíz asociado

### Instancias EC2

1. En la consola de EC2, ve a "Instancias"
2. Selecciona la instancia a terminar
3. Click en "Estado de instancia" > "Terminar instancia"
4. Confirma la terminación

**Diferencia importante:**
- **Stop:** Pausa la instancia (EBS persiste)
- **Terminate:** Elimina la instancia (EBS root se elimina por defecto)

### Sistemas de Archivos Compartidos

```bash
# Eliminar EFS
# 1. Desmonta todos los objetivos de montaje
# 2. En la consola de EFS, elimina el sistema de archivos

# Eliminar FSx
# 1. En la consola de FSx, selecciona el sistema
# 2. Click en "Eliminar"
# 3. Confirma la eliminación
```

### Checklist de Limpieza

- [ ] Eliminar volúmenes EBS no asociados
- [ ] Eliminar snapshots antiguos (o archivar)
- [ ] Deregistrar AMIs personalizadas innecesarias
- [ ] Terminar instancias EC2 de desarrollo/prueba
- [ ] Eliminar sistemas EFS no utilizados
- [ ] Eliminar sistemas FSx innecesarios
- [ ] Revisar costos en Cost Explorer
- [ ] Configurar alarmas de facturación

---

## 15. Cuestionario - Almacenamiento de Instancias EC2

### Preguntas de Práctica

**Pregunta 1: Características de EBS**

¿Cuál de las siguientes afirmaciones sobre Amazon EBS es correcta?

A) Los datos se pierden si detienes la instancia EC2
B) El almacenamiento es replicado automáticamente entre todas las regiones
C) Los datos persisten después de detener o terminar la instancia (salvo el volumen raíz)
D) EBS es más económico que Instance Store para datos temporales

**Respuesta correcta: C**

Explicación: EBS es almacenamiento persistente, lo que significa que los datos se conservan incluso después de detener la instancia. Sin embargo, el volumen raíz se elimina por defecto al terminar la instancia.

**Pregunta 2: EBS Multi-Attach**

¿Cuáles son los requisitos para usar EBS Multi-Attach? (SELECCIONA DOS)

A) Debe usarse con instancias de propósito general
B) Solo disponible para volúmenes io1 e io2
C) Todas las instancias deben estar en la misma AZ
D) Requiere configuración de un sistema de archivos distribuido
E) Compatible con cualquier tipo de volumen EBS

**Respuestas correctas: B y C**

Explicación: EBS Multi-Attach solo está disponible para volúmenes de alto rendimiento (io1 e io2) y todas las instancias deben estar en la misma zona de disponibilidad.

**Pregunta 3: EBS Snapshots**

¿Cuál es el principal beneficio de usar EBS Snapshots?

A) Aumentan automáticamente el rendimiento de EBS
B) Permiten replicar datos entre zonas de disponibilidad y regiones
C) Eliminan la necesidad de backups
D) Reducen el costo del almacenamiento en un 50%

**Respuesta correcta: B**

Explicación: Los snapshots son copias de seguridad incrementales que permiten restaurar volúmenes en diferentes AZ o regiones. Esto es fundamental para la recuperación ante desastres.

**Pregunta 4: AMI**

¿Cuál es la principal diferencia entre una AMI personalizada y una AMI pública?

A) Las AMI personalizadas no pueden iniciarse en múltiples AZ
B) Las AMI personalizadas contienen tu configuración específica y son privadas
C) Las AMI públicas son siempre más seguras
D) Las AMI personalizadas tienen un costo adicional

**Respuesta correcta: B**

Explicación: Una AMI personalizada es una imagen que creas a partir de una instancia existente. Las AMI públicas son proporcionadas por AWS o la comunidad.

**Pregunta 5: EC2 Image Builder**

¿Qué ventaja principal ofrece EC2 Image Builder?

A) Incrementa automáticamente la velocidad de las instancias
B) Automatiza la creación, prueba y distribución de AMIs seguras y actualizadas
C) Reemplaza la necesidad de usar EBS
D) Permite ejecutar múltiples sistemas operativos simultáneamente

**Respuesta correcta: B**

Explicación: EC2 Image Builder automatiza el pipeline completo de creación de imágenes.

**Pregunta 6: EC2 Instance Store**

¿En qué situación es apropiado usar EC2 Instance Store?

A) Para almacenar bases de datos críticas
B) Para datos que deben persistir indefinidamente
C) Para cachés, datos temporales y buffers
D) Como reemplazo principal de EBS

**Respuesta correcta: C**

Explicación: Instance Store es almacenamiento temporal muy rápido. Se pierde al detener o terminar la instancia.

**Pregunta 7: EFS vs EBS**

¿Cuál de las siguientes situaciones requiere EFS en lugar de EBS?

A) Una aplicación que necesita almacenamiento rápido en una sola instancia
B) Un sistema de archivos compartido accesible por múltiples instancias Linux simultáneamente
C) Una base de datos con requisitos IOPS muy altos
D) Un almacenamiento temporal para datos de sesión

**Respuesta correcta: B**

Explicación: EFS es un sistema de archivos NFS que puede ser montado simultáneamente por múltiples instancias EC2.

**Pregunta 8: Modelo de Responsabilidad Compartida**

En el contexto de almacenamiento EC2, ¿cuál es responsabilidad de AWS?

A) Cifrar manualmente todos los datos del usuario
B) Gestionar backups automáticos de todos los volúmenes
C) Proporcionar infraestructura, redundancia y disponibilidad física
D) Configurar las políticas de ciclo de vida del almacenamiento

**Respuesta correcta: C**

Explicación: AWS es responsable de la seguridad de la infraestructura física. El usuario es responsable de configurar cifrado, crear snapshots, y gestionar backups.

**Pregunta 9: EFS Infrequent Access**

¿Cuál es el principal beneficio de usar EFS Infrequent Access (EFS-IA)?

A) Mejora el rendimiento de lectura/escritura
B) Reduce costos hasta un 92% para datos a los que se accede ocasionalmente
C) Permite el acceso desde múltiples regiones
D) Elimina la necesidad de configurar seguridad

**Respuesta correcta: B**

Explicación: EFS-IA es una clase de almacenamiento de bajo costo. Las políticas de ciclo de vida mueven archivos no accedidos a EFS-IA.

**Pregunta 10: Amazon FSx for Windows**

¿Cuál es el caso de uso ideal para FSx for Windows File Server?

A) Procesamiento de Big Data y Machine Learning
B) Almacenamiento empresarial con integración Active Directory y protocolo SMB
C) Almacenamiento temporal de caché para aplicaciones web
D) Sincronización de archivos entre dispositivos móviles

**Respuesta correcta: B**

Explicación: FSx for Windows está optimizado para entornos empresariales Windows con protocolo SMB e integración Active Directory.

### Análisis de Desempeño

- **9-10 correctas:** Excelente - Dominas completamente almacenamiento EC2
- **7-8 correctas:** Muy bien - Buen dominio general
- **5-6 correctas:** Bien - Revisa conceptos clave
- **Menos de 5:** Dedica más tiempo a estudiar