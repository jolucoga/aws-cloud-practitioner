## 1. Fundamentos de Cloud Computing
### 1.1 Arquitectura Cliente-Servidor
La comunicación web se basa en tres componentes fundamentales:
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Cliente  │ ◄─────► │   Red    │ ◄─────► │ Servidor │
│ (Usuario)│         │(Internet)│         │(Recursos)│
└──────────┘         └──────────┘         └──────────┘


- **Cliente**: Usuario que solicita información/servicios
- **Servidor**: Máquina que almacena y proporciona contenido
- **Red**: Medio de transmisión (Internet) que conecta clientes y servidores

#### 📮 Analogía del Correo Postal

| Concepto de Red | Equivalente Postal         |
|-----------------|----------------------------|
| Enviar datos    | Enviar una carta           |
| IP del cliente  | Dirección del remitente    |
| IP del servidor | Dirección del destinatario |
| Internet        | Sistema postal             |


### 1.2 Direcciones IP

Las direcciones IP son identificadores únicos para cada dispositivo en la red.

**Tipos:**
- **IPv4**: Formato tradicional (ejemplo: `192.168.1.1`)
- **IPv6**: Formato expandido para soportar más dispositivos

### 1.3 Componentes de un Servidor

#### Hardware Básico

| Componente         | Función                                     |
|--------------------|---------------------------------------------|
| **CPU**            | Procesador que ejecuta instrucciones        |
| **RAM**            | Almacenamiento temporal de datos en uso     |
| **Almacenamiento** | Discos duros para información permanente    |
| **Base de Datos**  | Sistema para almacenar datos estructurados  |

#### Componentes de Red
```
Usuario escribe: www.ejemplo.com
         ↓
DNS traduce a: 192.168.1.50
         ↓
Router encuentra la ruta
         ↓
Switch entrega al servidor correcto
         ↓
Servidor responde con la página web
```

- **Router**: Determina la ruta óptima para enviar datos entre redes
- **Switch**: Distribuye datos al dispositivo correcto dentro de una red local
- **Servidor DNS**: Traduce nombres de dominio a direcciones IP

---

## 2. Introducción a AWS

### 2.1 Historia y Evolución

#### Cronología Clave

| Año      | Hito                                                                                    |
|----------|-----------------------------------------------------------------------------------------|
| **2002** | 🔧 Lanzamiento interno - Amazon identifica su infraestructura como ventaja estratégica |
| **2004** | 🚀 **SQS** - Primer servicio público (Simple Queue Service)                            |
| **2006** | 💎 **Trinity Launch** - EC2, S3, SQS (servicios fundamentales)                         |
| **2007** | 🌍 Expansión a Europa - Transición de servicio regional a global                       |
| **2019** | 📊 **47% cuota de mercado** - Líder indiscutible del cloud computing                   |

### 2.2 Servicios Fundacionales (2006)

#### EC2 - Elastic Compute Cloud
```
┌─────────────────────────┐
│ Servidores Virtuales    │
│ • Capacidad ajustable   │
│ • Múltiples OS          │
│ • Escalabilidad elástica│
└─────────────────────────┘
```

#### S3 - Simple Storage Service
```
┌──────────────────────────┐
│ Almacenamiento Objetos   │
│ • 99.999999999% (11-9s)  │
│ • Escalabilidad ilimitada│
│ • Acceso API simple      │
└──────────────────────────┘
```

#### SQS - Simple Queue Service
```
┌─────────────────────────┐
│ Colas de Mensajes       │
│ • Desacoplamiento apps  │
│ • Procesamiento async   │
│ • Alta disponibilidad   │
└─────────────────────────┘
```

### 2.3 Problemas de la IT Tradicional

#### Modelo Tradicional vs Cloud
```
IT TRADICIONAL                    CLOUD COMPUTING
─────────────────                 ─────────────────
❌ Alquiler centros datos    →    ✅ Sin infraestructura física
❌ Costos electricidad       →    ✅ Pago por uso
❌ Mantenimiento hardware    →    ✅ Mantenimiento gestionado
❌ Escalado limitado         →    ✅ Escalabilidad instantánea
❌ Equipo 24/7               →    ✅ Monitoreo automático
❌ Vulnerabilidad desastres  →    ✅ Alta disponibilidad/DR
```

#### Ejemplo de Escalabilidad
```
Demanda Normal:
1,000 usuarios → 10 servidores

Black Friday:
1,000,000 usuarios → ¿1,000 servidores?

❌ Problema: No puedes comprar e instalar 990 servidores en un día
✅ Solución Cloud: Auto-scaling en minutos
```

### 2.4 ¿Qué es Cloud Computing?

> **Definición**: Suministro bajo demanda de recursos IT (computación, almacenamiento, bases de datos, aplicaciones) a través de Internet, con modelo de **pago por uso**.

#### Las 5 Características Fundamentales

| # | Característica | Descripción |
|-----|-------------------------------|----------------------------------------------------|
| 1️⃣ | **Autoservicio bajo demanda** | Aprovisionar recursos sin intervención humana       |
| 2️⃣ | **Amplio acceso a la red**    | Disponible desde cualquier dispositivo              |
| 3️⃣ | **Multi-tenancy**             | Múltiples clientes comparten infraestructura segura |
| 4️⃣ | **Elasticidad rápida**        | Escalar automáticamente según demanda               |
| 5️⃣ | **Servicio medido**           | Pagar solo por lo que se usa                        |

#### Las 6 Ventajas del Cloud
```
1. 💰 CAPEX → OPEX
   Sin inversión inicial, pago operativo

2. 📈 Economías de escala
   Precios reducidos por eficiencia masiva

3. 🎯 Dejar de adivinar capacidad
   Escalar basado en uso real

4. ⚡ Velocidad y agilidad
   Recursos en minutos, no semanas

5. 🏢 Sin gastos en centros de datos
   Enfoque en el negocio

6. 🌍 Global en minutos
   Infraestructura mundial instantánea
```

### 2.5 Modelos de Despliegue

#### Cloud Privado
```
┌────────────────────────┐
│ Una sola organización  │
│ No expuesto al público │
│ Control total          │
│ Ejemplo: Rackspace     │
└────────────────────────┘
```

#### Cloud Público
```
┌────────────────────────┐
│ Múltiples clientes     │
│ A través de Internet   │
│ Proveedor gestiona     │
│ Ejemplo: AWS, Azure    │
└────────────────────────┘
```

#### Cloud Híbrido
```
┌──────────────────┐
│ On-Premises      │◄──── Conexión segura ────►┌────────────────┐
│ (Privado)        │                           │ Cloud Público  │
│ • Datos sensibles│                           │ • Escalabilidad│
└──────────────────┘                           └────────────────┘
```

### 2.6 Casos de Uso Reales

| Empresa        | Uso de AWS                                      |
|----------------|-------------------------------------------------|
| **Netflix**    | Streaming completo en AWS, escalabilidad masiva |
| **Dropbox**    | Almacenamiento construido sobre AWS S3          |
| **Airbnb**     | Hosting de aplicación web y móvil               |
| **NASA**       | Procesamiento de datos científicos              |
| **McDonald's** | Aplicaciones retail y punto de venta            |

---

## 3. Modelos de Servicio Cloud

### 3.1 Comparativa IaaS vs PaaS vs SaaS
```
┌──────────────────────────────────────────────────────────┐
│                  NIVEL DE GESTIÓN                        │
├──────────────┬──────────────┬──────────────┬─────────────┤
│ On-Premises  │    IaaS      │    PaaS      │    SaaS     │
├──────────────┼──────────────┼──────────────┼─────────────┤
│ Aplicaciones │ Aplicaciones │ Aplicaciones │ ✓ Proveedor │
│ Datos        │ Datos        │ Datos        │ ✓ Proveedor │
│ Runtime      │ Runtime      │ ✓ Proveedor  │ ✓ Proveedor │
│ Middleware   │ Middleware   │ ✓ Proveedor  │ ✓ Proveedor │
│ O/S          │ O/S          │ ✓ Proveedor  │ ✓ Proveedor │
│ Virtualiz.   │ ✓ Proveedor  │ ✓ Proveedor  │ ✓ Proveedor │
│ Servidores   │ ✓ Proveedor  │ ✓ Proveedor  │ ✓ Proveedor │
│ Storage      │ ✓ Proveedor  │ ✓ Proveedor  │ ✓ Proveedor │
│ Networking   │ ✓ Proveedor  │ ✓ Proveedor  │ ✓ Proveedor │
└──────────────┴──────────────┴──────────────┴─────────────┘
    TÚ TODO         MÁXIMA         DESARROLLO      SOLO USO
                  FLEXIBILIDAD     ENFOCADO
```

### 3.2 IaaS - Infraestructura como Servicio

#### 🎯 Características
- Bloques básicos de construcción para IT en la nube
- **Máxima flexibilidad y control**
- Similar a IT tradicional on-premises

#### 👤 Gestión del Usuario
```
✅ Aplicaciones
✅ Datos
✅ Runtime
✅ Middleware
✅ Sistema Operativo
```

#### ☁️ Gestión del Proveedor
```
✅ Virtualización
✅ Servidores
✅ Almacenamiento
✅ Networking
```

#### 📦 Ejemplos
- **Amazon EC2** (AWS)
- Google Compute Engine
- Azure Virtual Machines
- DigitalOcean

#### 🏗️ Analogía
> Es como alquilar un **terreno con agua y electricidad**; tú decides qué construir y cómo gestionarlo.

### 3.3 PaaS - Plataforma como Servicio

#### 🎯 Características
- Elimina la gestión de infraestructura
- **Enfoque 100% en desarrollo**
- Productividad maximizada

#### 👤 Gestión del Usuario
```
✅ Aplicaciones
✅ Datos
```

#### ☁️ Gestión del Proveedor
```
✅ Runtime
✅ Middleware
✅ Sistema Operativo
✅ Virtualización
✅ Servidores
✅ Almacenamiento
✅ Networking
```

#### 📦 Ejemplos
- **AWS Elastic Beanstalk**
- Heroku
- Google App Engine
- Windows Azure

#### 🍳 Analogía
> Es como alquilar un **restaurante totalmente equipado** donde solo necesitas traer tu receta y cocinar.

### 3.4 SaaS - Software como Servicio

#### 🎯 Características
- **Producto completo listo para usar**
- Cero gestión técnica
- Acceso inmediato vía navegador

#### 👤 Gestión del Usuario
```
❌ Nada (solo usa el servicio)
```

#### ☁️ Gestión del Proveedor
```
✅ Todo el stack tecnológico
```

#### 📦 Ejemplos
- **Gmail** (correo)
- **Dropbox** (almacenamiento)
- **Zoom** (videoconferencias)
- **AWS Rekognition** (ML)
- Microsoft 365

#### 🍔 Analogía
> Es como usar un **servicio de comida a domicilio**; simplemente haces el pedido y recibes el producto final.

---

## 4. Infraestructura Global de AWS

### 4.1 Jerarquía de la Infraestructura
```
🌍 INFRAESTRUCTURA GLOBAL AWS
│
├─ 📍 REGIONES (33+ worldwide)
│   │
│   ├─ 🏢 ZONAS DE DISPONIBILIDAD (mín 3, máx 6)
│   │   │
│   │   └─ 💾 CENTROS DE DATOS (redundantes)
│   │
│   └─ Criterios de selección:
│       • Cumplimiento legal
│       • Proximidad a clientes
│       • Servicios disponibles
│       • Precios
│
└─ ⚡ EDGE LOCATIONS (450+ ubicaciones)
    └─ CDN y caché de contenido
```

### 4.2 Regiones (AWS Regions)

#### 🗺️ Definición
- Grupo de centros de datos distribuidos geográficamente
- Cada región tiene **nombre único** (ej: `us-east-1`, `eu-west-3`)
- Son **completamente independientes**
- La mayoría de servicios son de **ámbito regional**

#### 📋 Ejemplos de Regiones

| Código           | Ubicación          | Nombre                   |
|------------------|--------------------|--------------------------|
| `us-east-1`      | Virginia del Norte | US East (N. Virginia)    |
| `us-west-2`      | Oregón             | US West (Oregon)         |
| `eu-west-1`      | Irlanda            | Europe (Ireland)         |
| `eu-central-1`   | Frankfurt          | Europe (Frankfurt)       |
| `ap-southeast-1` | Singapur           | Asia Pacific (Singapore) |
| `ap-northeast-1` | Tokio              | Asia Pacific (Tokyo)     |
| `sa-east-1`      | São Paulo          | South America (São Paulo)|

#### 🎯 Criterios para Elegir una Región

##### 1. Cumplimiento Legal y Gobernanza de Datos
```
🔒 Los datos NUNCA salen de una región sin permiso explícito

Ejemplo: 
GDPR (Europa) → Elegir eu-west-1 o eu-central-1
HIPAA (USA) → Elegir us-east-1 o us-west-2
```

##### 2. Proximidad a los Clientes
```
🎯 Menor latencia = Mejor experiencia de usuario

Ejemplo:
Usuarios en Asia → ap-southeast-1 (Singapur)
Usuarios en Europa → eu-west-1 (Irlanda)
Usuarios en Latinoamérica → sa-east-1 (São Paulo)
```

##### 3. Servicios Disponibles
```
⚠️ No todas las regiones tienen todos los servicios

Servicios nuevos → Lanzados gradualmente
Servicios legacy → Pueden no estar en regiones nuevas
```

##### 4. Precios
```
💰 Los precios varían entre regiones

Ejemplo EC2:
us-east-1 → $0.10/hora
ap-northeast-1 → $0.13/hora
```

### 4.3 Zonas de Disponibilidad (Availability Zones - AZ)

#### 🏢 Características
```
┌─────────────────────────────────────────┐
│     Región: Sydney (ap-southeast-2)     │
├─────────────────────────────────────────┤
│  AZ-A          AZ-B          AZ-C       │
│  [Centro      [Centro       [Centro     │
│   Datos]       Datos]        Datos]     │
│    ↓             ↓             ↓        │
│ Separación física para resistir         │
│ desastres (terremotos, inundaciones)    │
└─────────────────────────────────────────┘
```

- Cada región contiene **mínimo 3 AZ, máximo 6 AZ**
- Son **centros de datos físicamente separados**
- Nomenclatura: `región` + letra (`us-east-1a`, `us-east-1b`)
- **Conectadas con redes de alto ancho de banda**

#### ✅ Ventajas

| Ventaja                 | Beneficio                                       |
|-------------------------|-------------------------------------------------|
| **Alta Disponibilidad** | Si una AZ falla, las otras continúan            |
| **Redundancia**         | Alimentación, red y conectividad independientes |
| **Baja Latencia**       | Interconexión de alta velocidad entre AZ        |

#### 💡 Ejemplo Práctico
```
Aplicación distribuida en 3 AZ:

AZ-1: [App Server] [DB Replica]
  ↕
AZ-2: [App Server] [DB Primary]
  ↕
AZ-3: [App Server] [DB Replica]

❌ Terremoto afecta AZ-2
✅ Tráfico redirigido a AZ-1 y AZ-3
✅ Servicio continúa sin interrupción
```

### 4.4 Puntos de Presencia (Edge Locations)

#### 📊 Números Clave
```
🌐 COBERTURA GLOBAL
├─ 450+ Puntos de Presencia
├─ 10+ Cachés Regionales
├─ 90+ Ciudades
└─ 40+ Países
```

#### 🎯 Función Principal

Reducir la **latencia** para usuarios finales mediante:
- Almacenamiento de contenido en caché
- Distribución de contenido estático
- Servicios CDN (CloudFront)

#### 🔄 Flujo de Funcionamiento
```
Usuario en Madrid
      │
      ▼
Edge Location (Madrid) [✓ contenido en caché]
      │
      ▼ (si NO está en caché)
Región AWS (Irlanda) [origen del contenido]
```

#### 🎬 Ejemplo Real: Netflix
```
Película solicitada por usuario en México:

1. Usuario → Edge Location (Ciudad de México)
2. ¿Contenido en caché? → SÍ
3. Entrega inmediata (baja latencia)
4. Sin necesidad de ir a Virginia (us-east-1)

Resultado: Streaming sin buffering
```

### 4.5 Servicios Globales vs Regionales

#### 🌍 Servicios GLOBALES (4 principales)

| Servicio       | Función                        | Alcance |
|----------------|--------------------------------|---------|
| **IAM**        | Identity and Access Management | Global  |
| **Route 53**   | DNS Service                    | Global  |
| **CloudFront** | Content Delivery Network       | Global  |
| **WAF**        | Web Application Firewall       | Global  |

#### 📍 Servicios REGIONALES (ejemplos clave)

| Servicio     | Función                     | Alcance   |
|--------------|-----------------------------|-----------|
| **EC2**      | Elastic Compute Cloud       | Regional  |
| **Lambda**   | Serverless Computing        | Regional  |
| **RDS**      | Relational Database Service | Regional  |
| **S3**       | Simple Storage Service      | Regional* |
| **DynamoDB** | NoSQL Database              | Regional  |

> **Nota S3**: Aunque los buckets son regionales, los nombres de bucket son únicos globalmente.

---

## 5. Modelo de Responsabilidad Compartida

### 5.1 La Regla de Oro
```
┌─────────────────────────────────────────────────┐
│          MODELO DE RESPONSABILIDAD              │
├─────────────────────────────────────────────────┤
│                                                 │
│  AWS → Seguridad DEL Cloud                      │
│                                                 │
│  ═══════════════════════════════════════════    │
│                                                 │
│  CLIENTE → Seguridad EN el Cloud                │
│  (Datos, configuración y aplicaciones)          │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 5.2 Responsabilidad de AWS (DEL Cloud)

#### 🏢 Componentes que AWS Gestiona
```
┌────────────────────────────────────────┐
│   INFRAESTRUCTURA FÍSICA               │
├────────────────────────────────────────┤
│ ✅ Hardware (servidores, storage)      │
│ ✅ Software fundamental                │
│ ✅ Regiones y Zonas de Disponibilidad  │
│ ✅ Edge Locations                      │
│ ✅ Seguridad física de data centers    │
│ ✅ Networking de infraestructura       │
└────────────────────────────────────────┘
```

| Componente                 | Ejemplos Específicos                       |
|----------------------------|--------------------------------------------|
| **Hardware**               | Servidores, almacenamiento, equipos de red |
| **Infraestructura Global** | Regiones, AZ, Edge Locations               |
| **Software Base**          | Hypervisor, sistemas de almacenamiento     |
| **Seguridad Física**       | Control de acceso a centros de datos       |

### 5.3 Responsabilidad del Cliente (EN el Cloud)

#### 👤 Componentes que el Cliente Gestiona
```
┌────────────────────────────────────────┐
│   DATOS Y CONFIGURACIÓN                │
├────────────────────────────────────────┤
│ ✅ Datos de clientes                   │
│ ✅ IAM (usuarios, roles, permisos)     │
│ ✅ Configuración de red                │
│ ✅ Sistema Operativo (guest)           │
│ ✅ Aplicaciones                        │
│ ✅ Cifrado de datos                    │
│ ✅ Firewall configuration              │
│ ✅ MFA (Multi-Factor Authentication)   │
└────────────────────────────────────────┘
```

| Componente       | Responsabilidad del Cliente                |
|------------------|--------------------------------------------|
| **Datos**        | Confidencialidad, integridad, backups      |
| **IAM**          | Gestión de usuarios, roles, MFA            |
| **Red y SO**     | Firewall, grupos de seguridad, patches     |
| **Cifrado**      | Lado cliente y configuración lado servidor |
| **Aplicaciones** | Seguridad de código y plataforma           |

### 5.4 Controles Compartidos

#### 🔄 Ejemplo: Patch Management
```
PATCHING - RESPONSABILIDAD COMPARTIDA

AWS:
├─ ✅ Patches de infraestructura
├─ ✅ Hypervisor
├─ ✅ Servicios gestionados (RDS, Lambda)
└─ ✅ Software subyacente

CLIENTE:
├─ ✅ Sistema Operativo (EC2)
├─ ✅ Aplicaciones
├─ ✅ Software instalado
└─ ✅ Agentes de seguridad
```

### 5.5 Variación según Modelo de Servicio
```
┌─────────────────────────────────────────────────┐
│        RESPONSABILIDAD POR SERVICIO             │
├─────────────────────────────────────────────────┤
│                                                 │
│  IaaS (EC2):                                    │
│  ┌───────────────┐                              │
│  │ TÚ: Máxima    │                              │
│  │ responsabilidad│                             │
│  └───────────────┘                              │
│                                                 │
│  PaaS (Elastic Beanstalk):                      │
│  ┌───────────────┐                              │
│  │ TÚ: Media     │                              │
│  │ responsabilidad│                             │
│  └───────────────┘                              │
│                                                 │
│  SaaS (Rekognition):                            │
│  ┌───────────────┐                              │
│  │ TÚ: Mínima    │                              │
│  │ responsabilidad│                             │
│  └───────────────┘                              │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 5.6 Política de Uso Aceptable (AUP)

#### ⚠️ Prohibiciones

El cliente **NO puede usar AWS para**:
```
❌ Actividades ilegales
❌ Violación de seguridad
❌ Abuso de red
❌ Spam o phishing
❌ Violación de propiedad intelectual
❌ Contenido dañino u ofensivo
```

---

## 6. Precios y Modelo de Costos

### 6.1 Los 3 Fundamentos de Precios AWS
```
┌────────────────────────────────────────┐
│        MODELO DE PAGO POR USO          │
├────────────────────────────────────────┤
│                                        │
│  1️⃣ COMPUTACIÓN                        │
│     • Pagar por tiempo de cómputo      │
│     • Servicios: EC2, Lambda, ECS      │
│                                        │
│  2️⃣ ALMACENAMIENTO                     │
│     • Pagar por datos almacenados      │
│     • Servicios: S3, EBS, Glacier      │
│                                        │
│  3️⃣ TRANSFERENCIA DE DATOS             │
│     • Entrada: GRATIS ✅               │
│     • Salida: SE COBRA 💰              │
│                                        │
└────────────────────────────────────────┘
```

### 6.2 Desglose por Componente

#### 💻 Computación
```
Ejemplos de Facturación:

EC2:
├─ Por hora de instancia activa
├─ Varía según tipo de instancia
└─ Ejemplo: t2.micro = $0.0116/hora

Lambda:
├─ Por número de requests
├─ Por duración de ejecución (ms)
└─ Ejemplo: $0.20 por 1M requests
```

#### 💾 Almacenamiento
```
Ejemplos de Facturación:

S3:
├─ Por GB almacenado/mes
├─ Por cantidad de requests
└─ Ejemplo: $0.023/GB/mes (us-east-1)

EBS:
├─ Por GB aprovisionado/mes
├─ Por IOPS (operaciones)
└─ Ejemplo: $0.10/GB/mes
```

#### 🌐 Transferencia de Datos
```
REGLA DE ORO:

📥 ENTRADA (Internet → AWS)
   └─ GRATIS ✅

📤 SALIDA (AWS → Internet)
   └─ SE COBRA 💰
   └─ Primeros 100GB/mes: $0.09/GB
   └─ Siguientes TB: precio decreciente

🔄 ENTRE REGIONES
   └─ SE COBRA 💰

🔄 DENTRO DE LA MISMA AZ
   └─ GRATIS ✅
```

### 6.3 Comparativa: CAPEX vs OPEX

#### 💰 Modelo Tradicional (CAPEX)
```
INVERSIÓN INICIAL (Capital Expenditure)
├─ Comprar 100 servidores: $500,000
├─ Equipamiento de red: $100,000
├─ Instalaciones/Data Center: $200,000
├─ Mantenimiento anual: $100,000
└─ Personal IT: $300,000/año

TOTAL AÑO 1: $1,200,000
```

#### ☁️ Modelo Cloud (OPEX)
```
GASTO OPERATIVO (Operational Expenditure)
├─ Sin inversión inicial: $0
├─ Pago mensual según uso: $10,000-$50,000
├─ Personal reducido: $100,000/año
└─ Escalabilidad: Pagar solo lo necesario

TOTAL AÑO 1: $220,000 - $700,000
AHORRO: 40-80%
```

### 6.4 Ejemplo Práctico: Escalabilidad y Costos
📊 Caso: E-commerce con Carga Variable
┌────────────────────────────────────────────────┐
│  LUNES - VIERNES (Carga Normal)                │
├────────────────────────────────────────────────┤
│  👥 1M usuarios                                │
│  🖥️  100 servidores activos                    │
│  💰 Costo: $10,000/mes                         │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│  SÁBADO - DOMINGO (Carga Baja)                 │
├────────────────────────────────────────────────┤
│  👥 200K usuarios                              │
│  🖥️  20 servidores activos                     │
│  💰 Costo: $2,000/mes                          │
│  💡 AHORRO: 80%                                │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│  BLACK FRIDAY (Pico de Carga)                  │
├────────────────────────────────────────────────┤
│  👥 5M usuarios                                │
│  🖥️  500 servidores activos                    │
│  💰 Costo: $50,000/mesRetryClaude does not have
│ the ability to run the code it generates yet.  │  
│  💡 Solo durante el pico                       │
└────────────────────────────────────────────────┘
VENTAJA CLOUD: Elasticidad Automática
✅ Escalar hacia arriba en minutos
✅ Escalar hacia abajo automáticamente
✅ Pagar solo por recursos activos

### 6.5 Calculadora de Precios AWS

#### 🧮 Herramientas Disponibles
```
AWS Pricing Calculator
├─ Estimar costos antes de usar servicios
├─ Comparar configuraciones
├─ Exportar estimaciones
└─ URL: calculator.aws

AWS Cost Explorer
├─ Visualizar gastos actuales
├─ Identificar tendencias
├─ Forecasting de costos
└─ Recomendaciones de ahorro
```

### 6.6 AWS Free Tier

#### 🎁 Tres Tipos de Ofertas Gratuitas
```
1️⃣ SIEMPRE GRATIS
├─ Lambda: 1M requests/mes
├─ DynamoDB: 25GB storage
└─ CloudWatch: 10 métricas personalizadas

2️⃣ 12 MESES GRATIS (desde registro)
├─ EC2: 750 horas/mes (t2.micro)
├─ S3: 5GB storage
├─ RDS: 750 horas/mes (db.t2.micro)
└─ EBS: 30GB

3️⃣ PRUEBAS GRATUITAS (limitadas)
├─ Lightsail: 1 mes gratis
├─ SageMaker: 250 horas
└─ Inspector: 90 días
```

---

## 📝 Resumen Ejecutivo para el Examen

### ✅ Conceptos Clave que Debes Dominar

#### 1. Cloud Computing Fundamentals
```
✓ Definición: Recursos IT bajo demanda + Pago por uso
✓ 5 Características: Autoservicio, Acceso red, Multi-tenancy, 
                     Elasticidad, Servicio medido
✓ 6 Ventajas: CAPEX→OPEX, Economías escala, No adivinar capacidad,
              Velocidad, Sin data centers, Global en minutos
```

#### 2. Modelos de Servicio
```
IaaS → Máxima flexibilidad (EC2)
PaaS → Enfoque desarrollo (Elastic Beanstalk)
SaaS → Solo usar servicio (Rekognition)
```

#### 3. Modelos de Despliegue
```
Privado → Una organización (Rackspace)
Público → Multi-tenant (AWS, Azure)
Híbrido → Combinación de ambos
```

#### 4. Infraestructura Global
```
Región → Grupo de AZ (mínimo 3)
AZ → Centros de datos separados
Edge Location → Caché CDN (450+)
```

#### 5. Servicios AWS
```
GLOBALES:
├─ IAM (identidades)
├─ Route 53 (DNS)
├─ CloudFront (CDN)
└─ WAF (firewall)

REGIONALES:
├─ EC2 (compute)
├─ S3 (storage)
├─ RDS (database)
└─ Lambda (serverless)
```

#### 6. Responsabilidad Compartida
```
AWS → Seguridad DEL Cloud (infraestructura)
CLIENTE → Seguridad EN el Cloud (datos, config)
```

#### 7. Precios
```
3 Componentes:
├─ Computación (tiempo uso)
├─ Almacenamiento (GB/mes)
└─ Transferencia datos (OUT se cobra, IN gratis)
```

---

## 🎯 Preguntas de Práctica

### Pregunta 1: Historia de AWS
**¿Cuál fue el primer servicio público de AWS?**

- A) EC2
- B) S3
- C) SQS ✅
- D) Lambda

**Respuesta**: C - SQS (Simple Queue Service) fue lanzado en 2004

---

### Pregunta 2: Infraestructura
**¿Cuántas Zonas de Disponibilidad debe tener una región de AWS como mínimo?**

- A) 1
- B) 2
- C) 3 ✅
- D) 5

**Respuesta**: C - Cada región tiene mínimo 3 AZ, máximo 6

---

### Pregunta 3: Modelos de Servicio
**Si un desarrollador solo quiere enfocarse en escribir código sin gestionar servidores ni sistemas operativos, ¿qué modelo debería usar?**

- A) IaaS
- B) PaaS ✅
- C) SaaS
- D) On-Premises

**Respuesta**: B - PaaS (ejemplo: Elastic Beanstalk)

---

### Pregunta 4: Responsabilidad Compartida
**¿Quién es responsable de aplicar parches al sistema operativo de una instancia EC2?**

- A) AWS
- B) El cliente ✅
- C) Ambos
- D) Nadie

**Respuesta**: B - El cliente gestiona el SO guest en EC2

---

### Pregunta 5: Precios
**¿Cuál de las siguientes transferencias de datos es GRATUITA?**

- A) De AWS a Internet
- B) De Internet a AWS ✅
- C) Entre regiones AWS
- D) Todas las anteriores

**Respuesta**: B - La transferencia de datos entrante es siempre gratuita

---

### Pregunta 6: Servicios Globales
**¿Cuáles de los siguientes son servicios globales? (Selecciona DOS)**

- A) IAM ✅
- B) EC2
- C) Route 53 ✅
- D) RDS
- E) Lambda

**Respuesta**: A y C - IAM y Route 53 son servicios globales

---

### Pregunta 7: Casos de Uso
**Netflix utiliza AWS principalmente para:**

- A) Almacenar datos de clientes
- B) Streaming de video y escalabilidad ✅
- C) Procesamiento de pagos
- D) Gestión de inventarios

**Respuesta**: B - Netflix ejecuta toda su infraestructura de streaming en AWS

---

### Pregunta 8: Ventajas del Cloud
**¿Qué ventaja del cloud permite ajustar recursos automáticamente según la demanda?**

- A) Economías de escala
- B) Elasticidad rápida ✅
- C) Alcance global
- D) CAPEX a OPEX

**Respuesta**: B - Elasticidad permite auto-scaling

---

### Pregunta 9: Modelos de Despliegue
**Una empresa quiere mantener datos sensibles on-premises pero usar AWS para cargas de trabajo variables. ¿Qué modelo debería usar?**

- A) Cloud Privado
- B) Cloud Público
- C) Cloud Híbrido ✅
- D) Multi-Cloud

**Respuesta**: C - Cloud Híbrido combina on-premises con cloud público

---

### Pregunta 10: Selección de Región
**¿Cuáles son factores para elegir una región AWS? (Selecciona TRES)**

- A) Cumplimiento legal ✅
- B) Color del logo
- C) Proximidad a clientes ✅
- D) Precios ✅
- E) Temperatura local

**Respuesta**: A, C y D - Legal, proximidad y precios son factores clave

---

## 🎓 Checklist Final

Antes de tomar el examen, verifica que puedas responder:

### Cloud Computing Basics
- [ ] ¿Qué es Cloud Computing?
- [ ] ¿Cuáles son las 5 características?
- [ ] ¿Cuáles son las 6 ventajas?
- [ ] ¿Diferencia entre CAPEX y OPEX?

### AWS Infrastructure
- [ ] ¿Qué es una Región?
- [ ] ¿Qué es una AZ?
- [ ] ¿Para qué sirven las Edge Locations?
- [ ] ¿Cuántas AZ mínimo por región?

### Service Models
- [ ] ¿Diferencia entre IaaS, PaaS y SaaS?
- [ ] ¿Ejemplos de cada modelo en AWS?
- [ ] ¿Quién gestiona qué en cada modelo?

### Shared Responsibility
- [ ] ¿Qué gestiona AWS?
- [ ] ¿Qué gestiona el cliente?
- [ ] ¿Qué son controles compartidos?

### Pricing
- [ ] ¿Cuáles son los 3 componentes de precio?
- [ ] ¿Qué transferencias son gratuitas?
- [ ] ¿Qué es el Free Tier?

### Core Services
- [ ] ¿Qué es EC2?
- [ ] ¿Qué es S3?
- [ ] ¿Qué es RDS?
- [ ] ¿Qué servicios son globales?

---