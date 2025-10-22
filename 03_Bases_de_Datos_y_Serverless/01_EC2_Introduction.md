# Introducción a EC2

## ¿Qué es Amazon EC2?

EC2 (Elastic Compute Cloud) es uno de los servicios más populares de AWS y representa el concepto de Infraestructura como Servicio (IaaS).

### Capacidades principales:

- 🖥️ Alquilar máquinas virtuales (instancias EC2)
- 💾 Almacenar datos en unidades virtuales (EBS)
- ⚖️ Distribuir carga entre máquinas (ELB - Elastic Load Balancer)
- 📈 Escalar servicios mediante Auto Scaling Groups (ASG)

> 💡 **Nota Importante**: Conocer EC2 es fundamental para entender el funcionamiento del Cloud y es uno de los temas más evaluados en el examen AWS Cloud Practitioner.

---

## Configuración del Presupuesto de AWS

### ¿Por qué configurar un presupuesto?

Antes de comenzar a usar servicios de AWS, es crucial establecer controles de costos para evitar cargos inesperados.

### Pasos para configurar un presupuesto

#### 1. Habilitar acceso a facturación para usuarios IAM

1. Iniciar sesión con la cuenta root
2. Ir a Mi cuenta (Account)
3. Buscar: "Acceso a usuario de IAM y rol a información de facturación"
4. Activar esta opción
5. Actualizar

#### 2. Crear un presupuesto

**Opción 1: Presupuesto de Gasto Cero**

Ideal para estudiantes que solo quieren aprender:
- Notificación cuando el gasto supere $0
- Correo de alerta configurado

**Opción 2: Presupuesto Mensual Personalizado**

Para uso en producción o desarrollo:
- Definir cantidad mensual (ej: $10)
- Configurar alertas en umbrales específicos:
  - 85% del presupuesto ($8.50)
  - 100% del presupuesto ($10.00)
  - Gasto previsto alcanza 100%

#### 3. Presupuesto Avanzado

**Características:**

- Presupuesto recurrente (mensual)
- Fecha de inicio personalizable
- Filtros por servicio específico
- Múltiples destinatarios de alertas
- Umbrales personalizados

**Ejemplo de configuración:**

```json
{
  "tipo": "mensual",
  "monto": "$100",
  "servicios": ["EC2", "S3", "RDS"],
  "alertas": {
    "80%": ["joan@empresa.com", "miguel@empresa.com"],
    "90%": ["joan@empresa.com", "miguel@empresa.com", "damian@empresa.com"]
  }
}
```

### 📊 Panel de Facturación

**Acceso:** AWS Console → Billing Dashboard

**Información disponible:**

- Resumen de costos actuales
- Servicios con mayor gasto
- Tendencias de gasto (3-6 meses)
- Facturas históricas
- Desglose por servicio

---

## Fundamentos de EC2

### Opciones de Configuración de Instancias

Al crear una instancia EC2, puedes configurar:

#### 1. Sistema Operativo

- 🐧 Linux (Amazon Linux, Ubuntu, Red Hat, etc.)
- 🪟 Windows (Windows Server)
- 🍎 Mac OS

#### 2. Recursos de Computación

- **CPU:** Número de núcleos y potencia de cálculo
- **RAM:** Memoria de acceso aleatorio (desde 1GB hasta varios TB)

#### 3. Almacenamiento

- **EBS (Elastic Block Store):** Conectado a la red
- **EFS (Elastic File System):** Sistema de archivos de red
- **EC2 Instance Store:** Almacenamiento de hardware (efímero)

#### 4. Configuración de Red

- Velocidad de tarjeta de red
- Dirección IP pública
- Configuración de VPC

#### 5. Seguridad

- **Security Groups:** Reglas de firewall
- **Key Pairs:** Claves SSH para acceso seguro

#### 6. Datos de Usuario (User Data)

Script de arranque que se ejecuta en el primer inicio de la instancia.

### EC2 User Data

Los datos de usuario permiten automatizar la configuración inicial:

```bash
#!/bin/bash
# Actualizar el sistema
yum update -y

# Instalar servidor web Apache
yum install -y httpd

# Iniciar el servicio
systemctl start httpd
systemctl enable httpd

# Crear página web simple
echo "<h1>Hola Mundo desde $(hostname -f)</h1>" > /var/www/html/index.html
```

#### Características:

- ✅ Se ejecuta **solo una vez** en el primer arranque
- ✅ Se ejecuta con privilegios de **root**
- ✅ Automatiza tareas como:
  - Instalación de actualizaciones
  - Instalación de software
  - Descarga de archivos
  - Configuración de servicios

---

## Lanzamiento de Instancias EC2

### Práctica: Crear tu Primera Instancia

#### Requisitos previos:

- Cuenta de AWS activa
- Región seleccionada (ej: us-east-1)

#### Pasos detallados:

**1. Acceder al servicio EC2**

```
AWS Console → EC2 → Instances → Launch Instance
```

**2. Configuración básica**

| Campo | Valor | Notas |
|-------|-------|-------|
| **Name** | mi-primera-instancia | Nombre descriptivo |
| **AMI** | Amazon Linux 2 | ✅ Capa gratuita |
| **Instance Type** | t2.micro | ✅ Capa gratuita |

**3. Configurar Key Pair**

```
Crear nuevo Key Pair:
- Nombre: demo_par_claves
- Tipo: RSA
- Formato: 
  * .pem (Linux/Mac/Windows 10+)
  * .ppk (Windows < 10 con PuTTY)
```

> ⚠️ **IMPORTANTE**: Guarda el archivo `.pem` o `.ppk` en lugar seguro. No se puede recuperar después.

**4. Configuración de Red**

- VPC: Default
- Subnet: No preference
- Auto-assign Public IP: Enable

**5. Configurar Security Group**

```
Crear nuevo Security Group:
- Nombre: launch-wizard-1
- Reglas de entrada:
  * SSH (puerto 22) - Tu IP
  * HTTP (puerto 80) - 0.0.0.0/0
```

**6. Configurar Almacenamiento**

```
Volumen Root:
- Tamaño: 8 GB (por defecto)
- Tipo: gp2 (SSD de propósito general)
- Delete on Termination: Yes
```

**7. Detalles Avanzados - User Data**

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hola mundo desde $(hostname -f)</h1>" > /var/www/html/index.html
```

**8. Lanzar Instancia**

```
Review → Launch Instance
⏱️ Tiempo de aprovisionamiento: ~30-60 segundos
```

### Verificación de la Instancia

#### Estados de la instancia:

- 🔴 **Pending**: Iniciándose
- 🟢 **Running**: En ejecución
- 🟡 **Stopping**: Deteniéndose
- ⚫ **Stopped**: Detenida
- 🔴 **Terminated**: Eliminada (permanente)

#### Acceder al sitio web:

1. Copiar la **Public IPv4 address**
2. Abrir navegador
3. Navegar a: `http://[PUBLIC-IP]`

> ⚠️ **Nota**: Usar `http://` no `https://` (puerto 80, no 443)

### Operaciones con Instancias

#### Detener una instancia:

```
Select Instance → Instance State → Stop Instance
```

- La instancia se detiene pero no se elimina
- No se cobra por tiempo de computación
- Se sigue cobrando por almacenamiento EBS
- **La IP pública cambia** al reiniciar

#### Iniciar una instancia detenida:

```
Select Instance → Instance State → Start Instance
```

#### Terminar una instancia:

```
Select Instance → Instance State → Terminate Instance
```

- ⚠️ **Acción irreversible**
- Se elimina la instancia y sus datos
- Se elimina el almacenamiento EBS (si está configurado)

---

## Costos Asociados a Direcciones IPv4 Públicas

### ⚠️ Aviso Importante sobre Costos

Desde 2024, AWS cobra por direcciones IPv4 públicas, **incluso en la capa gratuita**.

#### Tarifas actuales:

```
💰 $0.005 USD por hora por IP pública
📊 Aproximadamente $3.60 USD al mes
```

#### ¿Cuándo se cobra?

- ✅ IP pública asignada pero no asociada a instancia EC2 elegible
- ✅ IP pública sin uso activo
- ✅ IPs elásticas no asociadas a instancias en ejecución

#### ¿Cómo evitar estos cargos?

**1. Eliminar instancias después de las prácticas**

```bash
# Siempre terminar instancias de prueba
Instance State → Terminate
```

**2. Liberar IPs elásticas no utilizadas**

```
EC2 → Elastic IPs → Select → Release
```

**3. Revisar regularmente el panel de facturación**

```
Billing Dashboard → Check VPC resources
```

### 📋 Recomendación

> Para estudiantes: **Crear y eliminar** instancias después de cada práctica para mantener costos en $0.

---

## Tipos de Instancias EC2

### Nomenclatura de Instancias

Las instancias EC2 siguen una convención de nombres:

```
m5.2xlarge
│ │  │
│ │  └── Tamaño dentro de la clase
│ └──── Generación (AWS mejora con el tiempo)
└────── Clase de instancia
```

#### Ejemplo:

- **m** = Clase de instancia (propósito general)
- **5** = Generación 5
- **2xlarge** = 2 veces extra grande

### Categorías de Instancias

#### 1. 🎯 Propósito General (General Purpose)

**Casos de uso:**

- Servidores web
- Repositorios de código
- Entornos de desarrollo/pruebas

**Características:**

- Balance entre computación, memoria y red
- Rendimiento equilibrado

**Ejemplos:**

```
t2.micro  - 1 vCPU, 1 GiB RAM   ✅ Capa gratuita
t2.small  - 1 vCPU, 2 GiB RAM
t2.medium - 2 vCPU, 4 GiB RAM
m5.large  - 2 vCPU, 8 GiB RAM
```

#### 2. ⚡ Computación Optimizada (Compute Optimized)

**Casos de uso:**

- Procesamiento por lotes (batch processing)
- Transcodificación de medios
- Servidores web de alto rendimiento
- Cómputo científico y machine learning
- Servidores de videojuegos dedicados

**Características:**

- Procesadores de alto rendimiento
- Alto ratio CPU/Memoria

**Ejemplos:**

```
c5.large    - 2 vCPU, 4 GiB RAM
c5.xlarge   - 4 vCPU, 8 GiB RAM
c5.4xlarge  - 16 vCPU, 32 GiB RAM
```

#### 3. 🧠 Memoria Optimizada (Memory Optimized)

**Casos de uso:**

- Bases de datos relacionales/NoSQL de alto rendimiento
- Almacenes de caché distribuidos (Redis, Memcached)
- Bases de datos en memoria para BI
- Procesamiento en tiempo real de big data

**Características:**

- Gran cantidad de RAM
- Rápido rendimiento para conjuntos de datos grandes

**Ejemplos:**

```
r5.large    - 2 vCPU, 16 GiB RAM
r5.xlarge   - 4 vCPU, 32 GiB RAM
r5.16xlarge - 64 vCPU, 512 GiB RAM
```

#### 4. 💾 Almacenamiento Optimizado (Storage Optimized)

**Casos de uso:**

- Sistemas OLTP de alta frecuencia
- Bases de datos relacionales y NoSQL
- Caché para bases de datos en memoria
- Aplicaciones de data warehousing
- Sistemas de archivos distribuidos

**Características:**

- Alto rendimiento de I/O
- Acceso secuencial rápido a grandes datasets
- IOPS elevadas

**Ejemplos:**

```
i3.large    - 2 vCPU, 15.25 GiB RAM, 0.475 TB NVMe SSD
i3.xlarge   - 4 vCPU, 30.5 GiB RAM, 0.950 TB NVMe SSD
d2.xlarge   - 4 vCPU, 30.5 GiB RAM, 6 TB HDD
```

### 📊 Tabla Comparativa de Instancias

| Instancia | vCPU | Memoria | Almacenamiento | Red | Precio/hora* |
|-----------|------|---------|----------------|-----|--------------|
| t2.micro | 1 | 1 GiB | Solo EBS | Bajo-Moderado | ✅ Gratis |
| t2.xlarge | 4 | 16 GiB | Solo EBS | Moderado | ~$0.19 |
| c5d.4xlarge | 16 | 32 GiB | 400 NVMe SSD | Hasta 10 Gbps | ~$0.77 |
| r5.16xlarge | 64 | 512 GiB | Solo EBS | 20 Gbps | ~$4.10 |
| m5.8xlarge | 32 | 128 GiB | Solo EBS | 10 Gbps | ~$1.54 |

*Precios aproximados en us-east-1 (pueden variar)

### 🔗 Recursos Útiles

- **Comparador de instancias**: https://instances.vantage.sh
- **Documentación oficial**: https://aws.amazon.com/ec2/instance-types/

---

## Seguridad en EC2

### Security Groups (Grupos de Seguridad)

Los **Security Groups** son la base de la seguridad de red en AWS, actuando como **firewalls virtuales** para tus instancias EC2.

#### 🛡️ Características principales:

- Controlan el tráfico **de entrada** (inbound) y **de salida** (outbound)
- Solo contienen **reglas de permiso** (allow rules)
- Pueden referenciar por **IP** o por **otro Security Group**
- Son **stateful**: si permites tráfico de entrada, la respuesta se permite automáticamente

### Funcionamiento de Security Groups

```
┌─────────────┐
│   Internet  │
│     WWW     │
└──────┬──────┘
       │ Tráfico de entrada
       ↓
┌──────────────────┐
│  Security Group  │  ← Firewall Virtual
│                  │
│  Reglas entrada  │
│  Reglas salida   │
└────────┬─────────┘
         ↓
   ┌─────────────┐
   │ Instancia   │
   │    EC2      │
   └─────────────┘
```

### Reglas de Security Groups

**Componentes de una regla:**

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| Type | Tipo de protocolo | SSH, HTTP, HTTPS, Custom TCP |
| Protocol | Protocolo de transporte | TCP, UDP, ICMP |
| Port Range | Rango de puertos | 22, 80, 443, 1024-65535 |
| Source/Destination | Origen o destino | 0.0.0.0/0, My IP, Security Group |
| Description | Descripción de la regla | Allow SSH from office |

**Ejemplo de reglas de entrada:**

```json
{
  "Reglas de entrada": [
    {
      "Type": "SSH",
      "Protocol": "TCP",
      "Port": 22,
      "Source": "Mi IP",
      "Description": "SSH desde mi oficina"
    },
    {
      "Type": "HTTP",
      "Protocol": "TCP",
      "Port": 80,
      "Source": "0.0.0.0/0",
      "Description": "Acceso web público"
    },
    {
      "Type": "HTTPS",
      "Protocol": "TCP",
      "Port": 443,
      "Source": "0.0.0.0/0",
      "Description": "Acceso web seguro"
    }
  ]
}
```

### Puertos Clásicos que Debes Conocer

| Puerto | Protocolo | Uso | Sistema Operativo |
|--------|-----------|-----|-------------------|
| **22** | SSH (Secure Shell) | Conexión remota segura | Linux |
| **21** | FTP | Transferencia de archivos | Todos |
| **22** | SFTP | Transferencia segura de archivos | Todos |
| **80** | HTTP | Sitios web no seguros | Todos |
| **443** | HTTPS | Sitios web seguros | Todos |
| **3389** | RDP (Remote Desktop) | Escritorio remoto | Windows |

### Diagrama de Tráfico

```
┌──────────────────┐
│  Tu Ordenador    │
│  IP: 203.0.113.1 │
└────────┬─────────┘
         │
         │ SSH (Puerto 22) ✅ Autorizado
         ↓
┌──────────────────────────┐
│   Security Group 1       │
│   Entrada:               │
│   - Puerto 22: 203.0.113.1│
│   - Puerto 80: 0.0.0.0/0 │
└────────┬─────────────────┘
         ↓
   ┌────────────┐
   │ Instancia  │
   │    EC2     │
   │ Port 22, 80│
   └────────────┘
         ↑
         │ SSH (Puerto 22) ❌ Denegado
         │
┌────────┴─────────┐
│  Otro Ordenador  │
│  IP: 198.51.100.1│
└──────────────────┘
```

### Referencia a Otros Security Groups

Un Security Group puede autorizar tráfico desde otros Security Groups:

```
┌────────────────────────┐
│  Security Group 1      │
│  Entrada:              │
│  - SG-2 (autorizado)   │
│  - SG-1 (autorizado)   │
│  - SG-3 (denegado)     │
└────────┬───────────────┘
         ↓
   ┌──────────┐
   │Instancia │
   │    EC2   │
   └──────────┘

Instancia con SG-2 → ✅ Permitido
Instancia con SG-1 → ✅ Permitido
Instancia con SG-3 → ❌ Denegado
```

### Buenas Prácticas

#### ✅ DO (Hacer):

- Usar Security Groups separados para diferentes roles (web, database, etc.)
- Nombrar descriptivamente los Security Groups
- Documentar reglas con descripciones claras
- Principio de mínimo privilegio
- Usar referencias a Security Groups en lugar de IPs cuando sea posible

#### ❌ DON'T (No hacer):

- Permitir 0.0.0.0/0 en SSH (puerto 22)
- Dejar puertos innecesarios abiertos
- Usar un solo Security Group para todo
- Olvidar actualizar reglas después de cambios de arquitectura

### Práctica: Configurar Security Groups

#### Paso 1: Crear Security Group

```
EC2 Console → Security Groups → Create Security Group

Nombre: web-server-sg
Descripción: Security group para servidores web
VPC: Default VPC
```

#### Paso 2: Configurar Reglas de Entrada

```
Inbound Rules:

1. HTTP
   - Type: HTTP
   - Protocol: TCP
   - Port: 80
   - Source: 0.0.0.0/0
   - Description: Permitir tráfico web

2. HTTPS
   - Type: HTTPS
   - Protocol: TCP
   - Port: 443
   - Source: 0.0.0.0/0
   - Description: Permitir tráfico web seguro

3. SSH
   - Type: SSH
   - Protocol: TCP
   - Port: 22
   - Source: Mi IP
   - Description: Acceso SSH desde mi IP
```

#### Paso 3: Configurar Reglas de Salida

```
Outbound Rules:

Por defecto: All traffic permitido (0.0.0.0/0)
```

#### Paso 4: Asignar a Instancia

```
Select Instance → Actions → Security → Change Security Groups
```

### Solución de Problemas

#### Problema 1: Timeout de conexión

```
Síntoma: La conexión se queda "pensando" y no responde
Causa: Security Group bloqueando tráfico
Solución: Verificar reglas de entrada del puerto requerido
```

#### Problema 2: Connection Refused

```
Síntoma: Error inmediato de conexión rechazada
Causa: Aplicación no está corriendo o puerto incorrecto
Solución: Verificar que el servicio esté activo en la instancia
```

#### Problema 3: No puedo acceder a mi sitio web

```
Verificar:
1. Security Group permite puerto 80/443 ✔
2. Instancia está "Running" ✔
3. IP pública está asignada ✔
4. Usar http:// no https:// ✔
5. Servidor web está corriendo en la instancia ✔
```

---

## Conexión a Instancias EC2

### Visión General de SSH

SSH (Secure Shell) es el protocolo más importante para conectarse remotamente a instancias EC2 y controlarlas mediante línea de comandos.

**Métodos de conexión según el sistema operativo:**

| Sistema Operativo | Método | Puerto |
|-------------------|--------|--------|
| Linux | SSH | 22 |
| Mac OS | SSH | 22 |
| Windows 10+ | SSH / PowerShell | 22 |
| Windows <10 | PuTTY | 22 |
| Todos (navegador) | EC2 Instance Connect | 22 |

### 🐧 Método 1: SSH en Linux/Mac

**Requisitos:**

- Terminal
- Archivo de clave .pem
- Instancia con Security Group permitiendo puerto 22

**Pasos:**

**1. Ubicar tu archivo de clave**

```bash
cd ~/Desktop/tutorial/
ls
# Deberías ver: EC2tutorial.pem
```

**2. Configurar permisos de la clave**

```bash
chmod 400 EC2tutorial.pem
```

> ⚠️ **Importante**: Este paso es obligatorio. SSH rechaza claves con permisos demasiado abiertos por seguridad.

**3. Conectarse a la instancia**

```bash
ssh -i EC2tutorial.pem ec2-user@54.145.171.253
```

Desglosando el comando:
- `ssh`: Comando SSH
- `-i EC2tutorial.pem`: Especifica el archivo de clave
- `ec2-user`: Usuario por defecto en Amazon Linux
- `@54.145.171.253`: IP pública de la instancia

**4. Aceptar fingerprint**

```
The authenticity of host '54.145.171.253' can't be established.
Are you sure you want to continue connecting (yes/no)? yes
```

**5. ¡Conectado!**

```bash
[ec2-user@ip-172-31-80-209 ~]$
```

**Comandos útiles una vez conectado:**

```bash
# Ver usuario actual
whoami

# Verificar conectividad
ping 8.8.8.8

# Listar archivos
ls -la

# Ver hostname
hostname -f

# Salir de la sesión SSH
exit
# o
Ctrl + D
```

### 🪟 Método 2: PuTTY en Windows <10

#### Requisitos:

- PuTTY instalado
- PuTTYgen instalado
- Archivo de clave `.pem`

#### Paso 1: Instalar PuTTY

```
1. Buscar en Google: "PuTTY download"
2. Descargar versión 64-bit
3. Ejecutar instalador
4. Next → Next → Install → Finish
```

#### Paso 2: Convertir clave .pem a .ppk

```
1. Abrir PuTTYgen
2. File → Load Private Key
3. Seleccionar "All Files (*.*)"
4. Elegir tu archivo .pem
5. Click en "Save Private Key"
6. Guardar como: EC2tutorial.ppk
```

#### Paso 3: Configurar PuTTY

```
1. Abrir PuTTY
2. Session:
   - Host Name: ec2-user@[IP-PUBLICA]
   - Port: 22
   - Connection Type: SSH

3. Connection → SSH → Auth:
   - Browse y seleccionar EC2tutorial.ppk

4. Session:
   - Saved Sessions: MyEC2Instance
   - Save

5. Open
```

#### Paso 4: Conectar

```
login as: ec2-user
Authenticating with public key...

[ec2-user@ip-172-31-80-209 ~]$
```

### 🪟 Método 3: SSH en Windows 10+

**Requisitos:**

- Windows 10 o superior
- PowerShell o Command Prompt
- Archivo de clave .pem

**Pasos:**

**1. Verificar SSH instalado**

```powershell
ssh
# Si aparece el menú de ayuda, SSH está instalado
```

**2. Configurar permisos de la clave**

```
1. Click derecho en EC2tutorial.pem
2. Properties → Security → Advanced
3. Change owner → Tu usuario → OK
4. Disable inheritance → Remove all
5. Add → Select Principal → Tu usuario
6. Full Control → OK → Apply
```

**3. Conectarse**

```powershell
ssh -i C:\Users\Usuario\Downloads\EC2tutorial.pem ec2-user@54.145.171.253
```

**4. También funciona en CMD**

```cmd
ssh -i C:\Users\Usuario\Downloads\EC2tutorial.pem ec2-user@54.145.171.253
```

### 🌐 Método 4: EC2 Instance Connect (Recomendado)

#### Ventajas:

- ✅ No requiere descarga de claves
- ✅ Funciona desde el navegador
- ✅ Compatible con todos los sistemas operativos
- ✅ No requiere configuración de SSH client

#### Requisitos:

- Security Group permitiendo puerto 22
- Amazon Linux 2 o Ubuntu (compatibles)

#### Pasos:

**1. Ir a la instancia**

```
EC2 Console → Instances → Seleccionar instancia
```

**2. Conectar**

```
Actions → Connect → EC2 Instance Connect
Username: ec2-user
Connect
```

**3. Terminal en el navegador**

```bash
[ec2-user@ip-172-31-80-209 ~]$ whoami
ec2-user

[ec2-user@ip-172-31-80-209 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=107 time=1.24 ms
```

#### Limitaciones:

- Requiere que el Security Group permita puerto 22
- Solo funciona con ciertas AMIs (Amazon Linux 2, Ubuntu)

### Solución de Problemas de SSH

#### ❌ Problema 1: Permission denied (publickey)

```
Causa: Clave incorrecta o permisos incorrectos

Solución Linux/Mac:
chmod 400 archivo.pem
ssh -i archivo.pem ec2-user@IP

Solución Windows:
Verificar permisos del archivo (solo tu usuario debe tener acceso)
```

#### ❌ Problema 2: Connection timeout

```
Causa: Security Group no permite puerto 22

Solución:
1. EC2 Console → Security Groups
2. Seleccionar SG de la instancia
3. Inbound Rules → Edit
4. Add Rule: SSH, Port 22, My IP
5. Save
```

#### ❌ Problema 3: Host key verification failed

```
Causa: La huella digital del host ha cambiado

Solución Linux/Mac:
ssh-keygen -R [IP-PUBLICA]

Solución Windows:
Eliminar entrada en: C:\Users\Usuario\.ssh\known_hosts
```

#### ❌ Problema 4: ssh: command not found (Windows)

```
Causa: SSH no está instalado o no está en el PATH

Solución:
- Usar PuTTY en su lugar
- O instalar OpenSSH:
  Settings → Apps → Optional Features → Add OpenSSH Client
```

#### ❌ Problema 5: Network error: Connection refused

```
Causa: La instancia no está corriendo o el puerto SSH no está activo

Solución:
1. Verificar que la instancia esté en estado "Running"
2. Intentar reiniciar la instancia
3. Verificar que el servicio SSH esté corriendo en la instancia
```

### 📋 Guía Rápida de Comandos SSH

```bash
# Conectar con clave específica
ssh -i mi-clave.pem usuario@ip

# Conectar con puerto personalizado
ssh -i mi-clave.pem -p 2222 usuario@ip

# Conectar con verbose (debug)
ssh -v -i mi-clave.pem usuario@ip

# Copiar archivo A instancia
scp -i mi-clave.pem archivo.txt usuario@ip:/home/usuario/

# Copiar archivo DESDE instancia
scp -i mi-clave.pem usuario@ip:/home/usuario/archivo.txt ./

# Copiar directorio completo
scp -i mi-clave.pem -r directorio/ usuario@ip:/home/usuario/

# Tunnel/Port forwarding
ssh -i mi-clave.pem -L 8080:localhost:80 usuario@ip
```

### 🔐 Mejores Prácticas de Seguridad

**1. Nunca compartas tu clave privada (.pem o .ppk)**

```
❌ No enviar por email
❌ No subir a GitHub
❌ No compartir en Slack/Discord
✅ Guardar en ubicación segura
✅ Usar diferentes claves para diferentes entornos
```

**2. Usa permisos restrictivos**

```bash
chmod 400 mi-clave.pem  # Solo lectura para el propietario
```

**3. Limita acceso SSH por IP**

```
Security Group:
SSH → Port 22 → Source: Mi IP (no 0.0.0.0/0)
```

**4. Considera usar AWS Systems Manager Session Manager**

```
- No requiere abrir puerto 22
- No requiere claves SSH
- Auditoría completa
- Acceso basado en IAM
```

---

## Roles IAM para EC2

### ¿Por Qué Usar Roles IAM?

Cuando una instancia EC2 necesita acceder a otros servicios de AWS (S3, DynamoDB, etc.), hay dos opciones:

#### ❌ Opción INCORRECTA: Usar `aws configure`

```bash
# NO HACER ESTO en instancias EC2
aws configure
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

**Problemas:**

- 🚨 Expone credenciales en la instancia
- 🚨 Difícil de rotar credenciales
- 🚨 Riesgo de seguridad si alguien accede a la instancia
- 🚨 Las credenciales pueden quedar en logs

#### ✅ Opción CORRECTA: Usar Roles IAM

```
EC2 Instance → IAM Role → AWS Services (S3, DynamoDB, etc.)
```

**Ventajas:**

- ✅ No se exponen credenciales
- ✅ Rotación automática de credenciales temporales
- ✅ Fácil de auditar con CloudTrail
- ✅ Basado en el principio de mínimo privilegio

### Práctica: Asignar un Rol IAM a EC2

#### Paso 1: Crear el Rol IAM

**1. Ir al servicio IAM**

```
AWS Console → IAM → Roles → Create Role
```

**2. Seleccionar tipo de entidad de confianza**

```
AWS Service → EC2 → Next
```

**3. Agregar permisos**

```
Buscar: IAMReadOnlyAccess
☑️ IAMReadOnlyAccess
Next
```

**4. Nombrar el rol**

```
Role name: DemoRolEC2
Description: Permite a EC2 leer información de IAM
Create Role
```

#### Paso 2: Verificar la Instancia (Sin Rol)

**1. Conectar a la instancia**

```
EC2 Console → Instances → Connect → EC2 Instance Connect
```

**2. Intentar comando AWS CLI**

```bash
[ec2-user@ip-172-31-80-209 ~]$ aws --version
aws-cli/2.x.x Python/3.x.x Linux/x86_64

[ec2-user@ip-172-31-80-209 ~]$ aws iam list-users

Unable to locate credentials. You can configure credentials by running "aws configure".
```

#### Paso 3: Asignar Rol a la Instancia

**1. Seleccionar instancia**

```
EC2 Console → Instances → Seleccionar instancia
```

**2. Modificar rol IAM**

```
Actions → Security → Modify IAM Role
IAM Role: DemoRolEC2
Update IAM Role
```

**3. Verificar asignación**

```
Instance Details → Security → IAM Role: DemoRolEC2
```

#### Paso 4: Probar el Rol

**1. Conectar nuevamente (cerrar sesión anterior)**

```
EC2 Instance Connect → Connect
```

**2. Ejecutar comando AWS CLI**

```bash
[ec2-user@ip-172-31-80-209 ~]$ aws iam list-users

{
    "Users": [
        {
            "UserName": "admin",
            "UserId": "AIDACKCEVSQ6C2EXAMPLE",
            "Arn": "arn:aws:iam::123456789012:user/admin",
            "CreateDate": "2024-01-15T10:30:00Z"
        },
        {
            "UserName": "developer",
            "UserId": "AIDACKCEVSQ6C3EXAMPLE",
            "Arn": "arn:aws:iam::123456789012:user/developer",
            "CreateDate": "2024-01-20T14:20:00Z"
        }
    ]
}
```

✅ **¡Funciona! La instancia ahora puede acceder a IAM.**

### Demostración: Agregar y Quitar Permisos

#### Quitar permiso del rol:

**1. Ir a IAM**

```
IAM → Roles → DemoRolEC2 → Permissions
```

**2. Remover política**

```
IAMReadOnlyAccess → Remove
```

**3. Probar nuevamente en EC2**

```bash
[ec2-user@ip-172-31-80-209 ~]$ aws iam list-users

An error occurred (AccessDenied) when calling the ListUsers operation:
User is not authorized to perform: iam:ListUsers
```

❌ **Acceso denegado - el rol no tiene permisos**

#### Agregar permiso nuevamente:

**1. Ir a IAM**

```
IAM → Roles → DemoRolEC2 → Add Permissions
```

**2. Buscar y agregar política**

```
Attach Policies → Search: IAMReadOnlyAccess
☑️ IAMReadOnlyAccess → Add Permissions
```

**3. Probar en EC2 (esperar unos segundos)**

```bash
[ec2-user@ip-172-31-80-209 ~]$ aws iam list-users

{
    "Users": [...]
}
```

✅ **Acceso restaurado**

### Casos de Uso Comunes de Roles IAM

#### 1. Acceso a S3

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::mi-bucket/*"
    }
  ]
}
```

**Uso en EC2:**

```bash
# Listar objetos
aws s3 ls s3://mi-bucket/

# Copiar archivo a S3
aws s3 cp archivo.txt s3://mi-bucket/

# Descargar de S3
aws s3 cp s3://mi-bucket/archivo.txt .
```

#### 2. Acceso a DynamoDB

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/MiTabla"
    }
  ]
}
```

#### 3. Acceso a Systems Manager Parameter Store

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters"
      ],
      "Resource": "arn:aws:ssm:us-east-1:123456789012:parameter/*"
    }
  ]
}
```

**Uso en EC2:**

```bash
# Obtener parámetro
aws ssm get-parameter --name "/config/database/password" --with-decryption
```

### 🔐 Mejores Prácticas con Roles IAM

**1. Principio de Mínimo Privilegio**

```
✅ Dar solo los permisos necesarios
❌ No usar políticas como AdministratorAccess
```

**2. Usar Políticas Específicas**

```
✅ Limitar por recurso (ARN específico)
✅ Limitar por acción (solo GetObject, no *)
❌ Evitar "Resource": "*" y "Action": "*"
```

**3. Separar Roles por Función**

```
EC2-WebServer-Role → Solo S3 y CloudWatch
EC2-Worker-Role → Solo SQS y DynamoDB
EC2-Admin-Role → Permisos administrativos limitados
```

**4. Auditar Regularmente**

```
IAM → Roles → DemoRolEC2 → Access Advisor

Ver qué servicios se están usando realmente
Eliminar permisos no utilizados
```

**5. Usar Tags para Organización**

```
Environment: Production
Application: WebApp
Team: Backend
CostCenter: Engineering
```

### Comparación: Credenciales vs Roles

| Aspecto | `aws configure` | Roles IAM |
|---------|----------------|-----------|
| **Seguridad** | ❌ Baja | ✅ Alta |
| **Rotación** | ❌ Manual | ✅ Automática |
| **Auditoría** | ❌ Difícil | ✅ Fácil (CloudTrail) |
| **Configuración** | ❌ En cada instancia | ✅ Una vez (rol) |
| **Riesgo de exposición** | ❌ Alto | ✅ Bajo |
| **Recomendado para** | ❌ Nunca en EC2 | ✅ Siempre en EC2 |

---

## Opciones de Compra

### Resumen de Opciones

AWS ofrece múltiples modelos de precios para instancias EC2, permitiéndote optimizar costos según tus necesidades:

| Opción | Duración | Descuento | Caso de Uso |
|--------|----------|-----------|-------------|
| **On-Demand** | Por hora/segundo | 0% | Cargas impredecibles, corto plazo |
| **Reserved** | 1 o 3 años | Hasta 72% | Cargas estables, largo plazo |
| **Savings Plans** | 1 o 3 años | Hasta 72% | Flexibilidad con compromiso |
| **Spot** | Variable | Hasta 90% | Cargas tolerantes a fallos |
| **Dedicated Hosts** | On-Demand o Reserved | Variable | Cumplimiento normativo |
| **Dedicated Instances** | On-Demand | Variable | Aislamiento de hardware |
| **Capacity Reservations** | Bajo demanda | 0% | Capacidad garantizada en AZ |

### 1. 💵 Instancias On-Demand (Bajo Demanda)

**Características:**

- Pagas solo por lo que usas
- Sin compromiso a largo plazo
- Sin pagos por adelantado

**Facturación:**

```
Linux/Windows: Por segundo (después del primer minuto)
Otros OS: Por hora
```

**Casos de uso:**

- ✅ Aplicaciones con cargas impredecibles
- ✅ Desarrollo y pruebas
- ✅ Aplicaciones que no pueden interrumpirse
- ✅ Primer uso de AWS

**Ejemplo de costos:**

```
Instancia: t2.micro
Región: us-east-1
Costo: ~$0.0116/hora

Uso diario (24h): $0.28
Uso mensual (730h): $8.47
```

### 2. 📋 Instancias Reservadas (Reserved Instances)

**Características:**

- Hasta **72% de descuento** vs On-Demand
- Compromiso de 1 o 3 años
- Tres opciones de pago

**Opciones de pago:**

| Opción | Pago Inicial | Descuento | Mejor Para |
|--------|--------------|-----------|------------|
| **Sin pago inicial** | 0% | + | Flujo de caja limitado |
| **Pago parcial** | ~50% | ++ | Balance costo/flujo |
| **Pago total** | 100% | +++ | Máximo ahorro |

**Tipos de instancias reservadas:**

#### Standard Reserved Instances

```
Características:
- Hasta 72% descuento
- Tipo de instancia FIJO
- Región FIJA
- Sistema operativo FIJO

Caso de uso:
- Base de datos de producción
- Aplicaciones con uso constante
```

#### Convertible Reserved Instances

```
Características:
- Hasta 66% descuento
- Tipo de instancia FLEXIBLE
- Sistema operativo FLEXIBLE
- Tenancy FLEXIBLE

Caso de uso:
- Cargas que pueden cambiar de tamaño
- Necesidad de flexibilidad futura
```

**Ejemplo de ahorro:**

```
Instancia: m5.xlarge
On-Demand: $0.192/hora = $1,401/año

Reserved (1 año, todo adelantado):
- Standard: $0.119/hora = $870/año → 38% ahorro
- Convertible: $0.134/hora = $979/año → 30% ahorro

Reserved (3 años, todo adelantado):
- Standard: $0.073/hora = $533/año → 62% ahorro
- Convertible: $0.095/hora = $694/año → 50% ahorro
```

### 3. 💰 Savings Plans (Planes de Ahorro)

**Características:**

- Hasta **72% de descuento**
- Compromiso de uso ($/hora) por 1 o 3 años
- Más flexible que Reserved Instances

**Tipos de Savings Plans:**

#### EC2 Instance Savings Plans

```
Compromiso: Familia de instancias + Región
Flexibilidad:
  ✅ Tamaño de instancia (m5.large → m5.2xlarge)
  ✅ Sistema operativo (Linux ↔ Windows)
  ✅ Tenancy (Default ↔ Dedicated)
  ❌ Región fija
  ❌ Familia fija

Ejemplo:
  Compromiso: $10/hora en familia M5 en us-east-1
  Puedes usar: m5.large, m5.xlarge, m5.2xlarge
```

#### Compute Savings Plans

```
Compromiso: Cantidad de gasto ($/hora)
Flexibilidad:
  ✅ Familia de instancias
  ✅ Región
  ✅ Tamaño
  ✅ Sistema operativo
  ✅ Servicio (EC2, Fargate, Lambda)

Ejemplo:
  Compromiso: $15/hora en cualquier computación
  Puedes usar: EC2 en cualquier región, Fargate, Lambda
```

**Comparación Savings Plans vs Reserved:**

| Aspecto | Savings Plans | Reserved Instances |
|---------|---------------|-------------------|
| Flexibilidad | ✅✅✅ Alta | ✅✅ Media |
| Descuento | Hasta 72% | Hasta 72% |
| Gestión | ✅ Más simple | ❌ Más compleja |
| Cross-Region | ✅ (Compute) | ❌ No |
| Fargate/Lambda | ✅ (Compute) | ❌ No |

### 4. ⚡ Instancias Spot

**Características:**

- Hasta **90% de descuento**
- Precio variable según oferta/demanda
- Pueden ser interrumpidas con 2 minutos de aviso

**¿Cómo funcionan?**

```
1. Defines precio máximo que pagas
2. Si precio spot < tu máximo → Instancia corre
3. Si precio spot > tu máximo → AWS puede terminar instancia
```

**Casos de uso APROPIADOS:**

- ✅ Procesamiento por lotes (batch jobs)
- ✅ Análisis de datos
- ✅ Procesamiento de imágenes
- ✅ Cargas distribuidas y tolerantes a fallos
- ✅ Inicio/fin flexible

**Casos de uso NO APROPIADOS:**

- ❌ Bases de datos
- ❌ Aplicaciones críticas
- ❌ Servidores web de producción
- ❌ Trabajos que no pueden interrumpirse

**Ejemplo de uso:**

```bash
# Ejemplo: Procesamiento de video
Input: 1000 videos en S3
Proceso: Conversión a diferentes resoluciones
Output: Videos procesados en S3

Estrategia:
1. Lanzar 50 instancias Spot
2. Si se interrumpen, el trabajo se redistribuye
3. Ahorro del 80% vs On-Demand
```

### 5. 🏢 Dedicated Hosts (Hosts Dedicados)

**Características:**

- Servidor físico completo dedicado a tu uso
- Visibilidad de sockets, cores y host ID
- Control de ubicación de instancias

**Casos de uso:**

- Cumplimiento normativo (regulaciones que requieren aislamiento físico)
- Licencias BYOL (Bring Your Own License)
  - Por socket
  - Por core
  - Por VM

**Opciones de compra:**

```
On-Demand: Pago por hora del host completo
Reserved: 1 o 3 años (hasta 70% descuento)
```

**Ejemplo:**

```
Dedicated Host para m5.large:
Capacidad: Hasta 16 instancias m5.large en un host

On-Demand: ~$1.68/hora
Reserved (3 años): ~$0.70/hora

Software con licencia por socket:
- Windows Server Datacenter
- SQL Server Enterprise
- Oracle Database
```

### 6. 🔒 Dedicated Instances (Instancias Dedicadas)

**Características:**

- Hardware dedicado a tu cuenta
- Pueden compartir hardware con otras instancias **de tu cuenta**
- Sin control de ubicación de instancia

**Diferencia con Dedicated Hosts:**

| Aspecto | Dedicated Instances | Dedicated Hosts |
|---------|-------------------|-----------------|
| **Hardware** | Dedicado a tu cuenta | Servidor físico dedicado |
| **Visibilidad** | No | Sockets, cores, host ID |
| **Control de ubicación** | No | Sí |
| **BYOL** | Limitado | Completo |
| **Precio** | Más económico | Más caro |

### 7. 🎯 Capacity Reservations (Reservas de Capacidad)

**Características:**

- Reserva capacidad en una AZ específica
- Sin compromiso de tiempo
- Sin descuentos (precio On-Demand)

**Casos de uso:**

- Cargas de trabajo ininterrumpidas a corto plazo
- Necesidad de garantizar capacidad en momentos específicos
- DR (Disaster Recovery) planning

**Ejemplo:**

```
Escenario: Black Friday

Preparación:
1. Reservar 100 instancias m5.large en us-east-1a
2. Durante el evento: Lanzar instancias en capacidad reservada
3. Después del evento: Cancelar reserva

Costo:
- Pagas por capacidad reservada aunque no uses instancias
- Precio On-Demand por instancias lanzadas
```

### 💡 Analogía: Hotel de Vacaciones

Para entender las opciones de compra, imagina un resort de vacaciones:

```
🏨 On-Demand (Bajo Demanda)
Llegas y pagas precio completo
- Sin reserva
- Máxima flexibilidad
- Precio completo

🏨 Reserved (Reservado)
Reservas con anticipación
- Compromiso de 1-3 años
- Descuento significativo
- Si planeas quedarte mucho tiempo

🏨 Savings Plans (Planes de Ahorro)
Pagas cantidad fija por hora
- Puedes cambiar de habitación
- King, Suite, Vista al mar
- Flexibilidad con ahorro

🏨 Spot (Subasta)
El hotel subasta habitaciones vacías
- Mejor postor gana
- Pueden echarte si alguien paga más
- Súper barato pero arriesgado

🏨 Dedicated Hosts (Edificio Completo)
Rentas edificio entero del resort
- Control total
- Muy caro
- Para necesidades específicas

🏨 Capacity Reservations (Reserva sin descuento)
Reservas habitación pagando precio completo
- Garantía de disponibilidad
- Pagas aunque no te quedes
- Sin descuento
```

### 📊 Tabla Comparativa de Precios

Ejemplo: Instancia **m4.large** en **us-east-1**

| Tipo | Precio/hora | Ahorro | Notas |
|------|-------------|--------|-------|
| **On-Demand** | $0.10 | 0% | Línea base |
| **Spot** | $0.038-$0.039 | 61% | Variable, puede interrumpirse |
| **Reserved 1y (sin adelanto)** | $0.062 | 38% | Standard |
| **Reserved 1y (todo adelanto)** | $0.058 | 42% | Standard |
| **Reserved 3y (sin adelanto)** | $0.043 | 57% | Standard |
| **Reserved 3y (todo adelanto)** | $0.037 | 63% | Standard |
| **Savings Plan 1y** | $0.062 | 38% | Mayor flexibilidad |
| **Convertible Reserved 1y** | $0.071 | 29% | Más flexible |
| **Dedicated Host** | On-Demand | 0% | Precio base |
| **Dedicated Host Reserved** | Varía | Hasta 70% | Con compromiso |

### 🎯 Estrategia de Compra Recomendada

```
Carga de trabajo típica:

┌─────────────────────────────────┐
│   Base constante (70%)          │  ← Reserved Instances o
│   M5.xlarge x 10                │     Savings Plans (3 años)
│                                 │     Ahorro: 60-72%
├─────────────────────────────────┤
│   Carga predecible (20%)        │  ← Reserved Instances (1 año)
│   M5.xlarge x 3                 │     Ahorro: 30-40%
│                                 │
├─────────────────────────────────┤
│   Picos impredecibles (10%)     │  ← On-Demand + Spot
│   M5.xlarge x 0-5               │     Flexibilidad total
└─────────────────────────────────┘

Ahorro total estimado: 50-60% vs todo On-Demand
```

### 📈 Calculando tu Estrategia

**Herramientas de AWS:**

1. **Cost Explorer** → Analiza patrones de uso histórico
2. **Compute Optimizer** → Recomienda tipo y tamaño de instancia
3. **Savings Plans Recommendations** → Calcula ahorros potenciales

**Ejemplo práctico:**

```
Uso actual: 100 m5.xlarge On-Demand 24/7

Análisis:
- 70 instancias siempre activas → Reserved 3y
- 20 instancias activas horario laboral → Reserved 1y  
- 10 instancias para picos → On-Demand + Spot

Ahorro anual:
On-Demand total: $168,192/año
Con estrategia mixta: $75,686/año
Ahorro: $92,506/año (55%)
```

---

## Modelo de Responsabilidad Compartida

### Concepto de Responsabilidad Compartida

AWS opera bajo un **modelo de responsabilidad compartida** donde tanto AWS como el cliente tienen responsabilidades específicas de seguridad y gestión.

```
┌────────────────────────────────────────┐
│    RESPONSABILIDAD DEL CLIENTE         │
│    "Seguridad EN el Cloud"             │
├────────────────────────────────────────┤
│    Datos del cliente                   │
│    Plataforma, aplicaciones, IAM       │
│    Sistema operativo, red, firewall    │
│    Cifrado de datos (cliente/servidor) │
│    Integridad de datos y autenticación │
├────────────────────────────────────────┤
│    RESPONSABILIDAD DE AWS              │
│    "Seguridad DEL Cloud"               │
├────────────────────────────────────────┤
│    Software                            │
│    - Computación                       │
│    - Almacenamiento                    │
│    - Base de datos                     │
│    - Red                               │
├────────────────────────────────────────┤
│    Hardware/Infraestructura global     │
│    - Regiones                          │
│    - Zonas de disponibilidad           │
│    - Ubicaciones edge                  │
└────────────────────────────────────────┘
```

### Responsabilidad Compartida para EC2

#### 🔷 Responsabilidades de AWS

**1. Infraestructura física**

- Seguridad global de la red
- Centros de datos físicos
- Hardware del servidor

**2. Aislamiento en hosts físicos**

- Separación entre clientes
- Virtualización segura

**3. Sustitución de hardware defectuoso**

- Mantenimiento de hardware
- Reemplazo proactivo

**4. Validación de la normativa**

- Cumplimiento de estándares
- Certificaciones (ISO, SOC, PCI-DSS)

#### 🔶 Responsabilidades del Cliente

**1. Grupos de seguridad y reglas de firewall**

```
Tu responsabilidad:
- Configurar Security Groups correctamente
- Permitir solo puertos necesarios
- Restringir acceso por IP
```

**2. Parches y actualizaciones del SO**

```bash
# ⚠️ Tú debes ejecutar:
sudo yum update -y                    # Amazon Linux
sudo apt update && sudo apt upgrade -y # Ubuntu
```

**3. Software y utilidades instaladas**

```
Tu responsabilidad:
- Mantener software actualizado
- Aplicar parches de seguridad
- Configurar aplicaciones correctamente
```

**4. Roles IAM asignados**

```
Tu responsabilidad:
- Crear roles con permisos apropiados
- Principio de mínimo privilegio
- Auditar permisos regularmente
```

**5. Gestión de acceso de usuarios IAM**

```
Tu responsabilidad:
- Crear usuarios y grupos
- Asignar permisos
- Implementar MFA
- Rotar credenciales
```

**6. Seguridad de los datos**

```
Tu responsabilidad:
- Cifrado de datos en reposo
- Cifrado de datos en tránsito
- Backups
- Control de acceso a datos
```

### Ejemplo: Responsabilidades en RDS

#### AWS es responsable de:

- ✅ Gestionar instancia EC2 subyacente
- ✅ Desactivar acceso SSH al host
- ✅ Parches automáticos de base de datos
- ✅ Parches automáticos del sistema operativo
- ✅ Auditoría del disco y hardware subyacente

#### Tú eres responsable de:

- 🔶 Verificar reglas de Security Group
- 🔶 Crear usuarios en la base de datos
- 🔶 Asignar permisos a usuarios
- 🔶 Configurar base de datos pública o privada
- 🔶 Configurar conexiones SSL obligatorias
- 🔶 Habilitar cifrado de base de datos

### Ejemplo: Responsabilidades en S3

#### AWS es responsable de:

- ✅ Infraestructura global
- ✅ Durabilidad (99.999999999%)
- ✅ Disponibilidad de la plataforma
- ✅ Separación entre datos de clientes
- ✅ Empleados de AWS no pueden acceder a tus datos

#### Tú eres responsable de:

- 🔶 Configuración de buckets
- 🔶 Políticas de bucket
- 🔶 Configuración de acceso público
- 🔶 Usuarios y roles IAM
- 🔶 Habilitar cifrado de objetos
- 🔶 Versionado de objetos

### 📋 Checklist de Seguridad para EC2

#### Configuración Inicial

- [ ] Security Group configurado con mínimo privilegio
- [ ] Solo puertos necesarios abiertos
- [ ] SSH restringido a IPs conocidas
- [ ] Key pair creado y almacenado de forma segura

#### Sistema Operativo

- [ ] Parches de seguridad aplicados
- [ ] Sistema operativo actualizado
- [ ] Software innecesario desinstalado
- [ ] Firewall del SO configurado (iptables/firewalld)

#### Acceso y Autenticación

- [ ] Rol IAM asignado (no usar aws configure)
- [ ] Permisos de rol basados en mínimo privilegio
- [ ] No hay credenciales hardcodeadas en código
- [ ] SSH con key pair (no contraseñas)

#### Datos y Aplicaciones

- [ ] Cifrado de datos en reposo (EBS)
- [ ] Cifrado de datos en tránsito (SSL/TLS)
- [ ] Backups configurados
- [ ] Logs enviados a CloudWatch

#### Monitorización

- [ ] CloudWatch alarms configuradas
- [ ] CloudTrail habilitado
- [ ] Revisión regular de logs
- [ ] AWS Config para auditoría de configuración

### ⚠️ Errores Comunes y Responsabilidades

| Error | ¿De quién es la responsabilidad? | Solución |
|-------|----------------------------------|----------|
| Instancia hackeada por puerto SSH abierto | 🔶 Cliente | Restringir Security Group |
| Hardware del servidor falla | 🔷 AWS | AWS reemplaza automáticamente |
| Software desactualizado con vulnerabilidad | 🔶 Cliente | Aplicar parches regularmente |
| Pérdida de datos por no hacer backup | 🔶 Cliente | Implementar estrategia de backup |
| Interrupción de servicio en AZ | 🔷 AWS | AWS restaura servicio |
| Credenciales IAM expuestas en GitHub | 🔶 Cliente | Rotar credenciales, usar Secrets Manager |
| Datos robados por acceso no autorizado | 🔶 Cliente | Revisar políticas IAM y S3 |

### 🎯 Mejores Prácticas

#### Para AWS (lo que ellos hacen):

- Mantener infraestructura física segura
- Proporcionar servicios con alta disponibilidad
- Mantener certificaciones de cumplimiento
- Actualizar y parchear infraestructura subyacente

#### Para el Cliente (lo que TÚ debes hacer):

**1. Implementar el principio de mínimo privilegio**

```
✅ Roles con permisos específicos
✅ Security Groups restrictivos
✅ No usar credenciales root
```

**2. Mantener sistemas actualizados**

```bash
# Automatizar actualizaciones
sudo yum update -y

# O usar AWS Systems Manager Patch Manager
```

**3. Habilitar cifrado**

```
✅ EBS volumes cifrados
✅ S3 buckets con SSE-S3 o SSE-KMS
✅ RDS con cifrado en reposo
✅ SSL/TLS para datos en tránsito
```

**4. Implementar monitorización y alertas**

```
✅ CloudWatch Logs
✅ CloudTrail para auditoría
✅ AWS Config para compliance
✅ GuardDuty para detección de amenazas
```

**5. Realizar backups regulares**

```
✅ Snapshots de EBS automáticos
✅ Backups de RDS
✅ Versionado de S3
✅ AWS Backup para centralizar
```

---

## Resumen

### 🎯 Puntos Clave de EC2

#### 1. ¿Qué es EC2?

```
EC2 = Elastic Compute Cloud
- IaaS (Infrastructure as a Service)
- Máquinas virtuales en el cloud
- Escalable y flexible
- Pago por uso
```

#### 2. Componentes principales:

**AMI (Amazon Machine Image)**

```
- Sistema operativo (Linux, Windows, Mac)
- Software preinstalado
- Configuración del sistema
```

**Tipo de Instancia**

```
Tamaño = CPU + RAM

Categorías:
- Propósito general (t2, m5)
- Computación optimizada (c5)
- Memoria optimizada (r5)
- Almacenamiento optimizado (i3, d2)
```

**Almacenamiento**

```
- EBS (Elastic Block Store) - persistente
- Instance Store - efímero
- EFS (Elastic File System) - compartido
```

**Configuración de Red**

```
- VPC (Virtual Private Cloud)
- Subnet pública o privada
- Security Groups
- IP pública/privada
```

**User Data**

```bash
#!/bin/bash
# Script que se ejecuta en el primer arranque
yum update -y
yum install -y httpd
systemctl start httpd
```

#### 3. Security Groups

```
Firewall virtual a nivel de instancia

Reglas:
✅ Solo ALLOW (permitir)
✅ Stateful (respuestas automáticas)
✅ Por IP o por Security Group

Puertos importantes:
- 22: SSH (Linux)
- 3389: RDP (Windows)
- 80: HTTP
- 443: HTTPS
```

#### 4. Conexión a Instancias

| Método | Sistema | Requisito |
|--------|---------|-----------|
| SSH | Linux/Mac | Key pair .pem |
| PuTTY | Windows <10 | Key pair .ppk |
| SSH | Windows 10+ | Key pair .pem |
| Instance Connect | Todos | Navegador |

#### 5. Roles IAM

```
✅ HACER: Usar roles IAM para acceso a AWS services
❌ NO HACER: Usar aws configure en instancias

Ventajas:
- Sin credenciales expuestas
- Rotación automática
- Auditable con CloudTrail
```

#### 6. Opciones de Compra

| Opción | Ahorro | Uso |
|--------|--------|-----|
| On-Demand | 0% | Flexible, impredecible |
| Reserved | Hasta 72% | Estable, 1-3 años |
| Savings Plans | Hasta 72% | Flexible con compromiso |
| Spot | Hasta 90% | Tolerante a interrupciones |
| Dedicated | Varía | Cumplimiento normativo |

#### 7. Modelo de Responsabilidad Compartida

```
AWS → Seguridad DEL cloud
- Hardware
- Infraestructura global
- Servicios gestionados

CLIENTE → Seguridad EN el cloud
- Security Groups
- Parches del SO
- Datos y cifrado
- Roles IAM
- Configuración de aplicaciones
```

### 📊 Diagrama Completo de Arquitectura EC2

```
┌─────────────────────────────────────────────────────┐
│                    AWS Cloud                        │
│                                                     │
│  ┌───────────────────────────────────────────────┐ │
│  │              Región (us-east-1)               │ │
│  │                                               │ │
│  │  ┌─────────────────────────────────────────┐ │ │
│  │  │    Availability Zone 1a                 │ │ │
│  │  │                                         │ │ │
│  │  │  ┌──────────────────────────────────┐  │ │ │
│  │  │  │         VPC (10.0.0.0/16)        │  │ │ │
│  │  │  │                                  │  │ │ │
│  │  │  │  ┌────────────────────────────┐ │  │ │ │
│  │  │  │  │   Public Subnet            │ │  │ │ │
│  │  │  │  │   (10.0.1.0/24)            │ │  │ │ │
│  │  │  │  │                            │ │  │ │ │
│  │  │  │  │  ┌──────────────────────┐ │ │  │ │ │
│  │  │  │  │  │   Security Group     │ │ │  │ │ │
│  │  │  │  │  │   - SSH: 22          │ │ │  │ │ │
│  │  │  │  │  │   - HTTP: 80         │ │ │  │ │ │
│  │  │  │  │  │   - HTTPS: 443       │ │ │  │ │ │
│  │  │  │  │  └──────────┬───────────┘ │ │  │ │ │
│  │  │  │  │             │             │ │  │ │ │
│  │  │  │  │  ┌──────────▼───────────┐ │ │  │ │ │
│  │  │  │  │  │   EC2 Instance       │ │ │  │ │ │
│  │  │  │  │  │   t2.micro           │ │ │  │ │ │
│  │  │  │  │  │   Amazon Linux 2     │ │ │  │ │ │
│  │  │  │  │  │   ┌────────────────┐ │ │ │  │ │ │
│  │  │  │  │  │   │   IAM Role     │ │ │ │  │ │ │
│  │  │  │  │  │   │   S3ReadOnly   │ │ │ │  │ │ │
│  │  │  │  │  │   └────────────────┘ │ │ │  │ │ │
│  │  │  │  │  │   ┌────────────────┐ │ │ │  │ │ │
│  │  │  │  │  │   │   EBS Volume   │ │ │ │  │ │ │
│  │  │  │  │  │   │   8 GB gp2     │ │ │ │  │ │ │
│  │  │  │  │  │   └────────────────┘ │ │ │  │ │ │
│  │  │  │  │  └──────────────────────┘ │ │  │ │ │
│  │  │  │  └────────────────────────────┘ │  │ │ │
│  │  │  └──────────────────────────────────┘  │ │ │
│  │  └─────────────────────────────────────────┘ │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
         ▲
         │
         │ SSH / HTTP
         │
    ┌────┴─────┐
    │  Usuario │
    └──────────┘
```

### 🎓 Preparación para el Examen

#### Temas que DEBES dominar:

**1. Tipos de instancias EC2**

- Propósito general, computación, memoria, almacenamiento
- Nomenclatura (m5.2xlarge)

**2. Security Groups**

- Stateful vs NACL (stateless)
- Reglas de entrada/salida
- Puertos comunes (22, 80, 443, 3389)

**3. Opciones de compra**

- On-Demand, Reserved, Spot, Dedicated
- Casos de uso para cada una
- Niveles de ahorro

**4. Roles IAM**

- ¿Por qué usar roles en vez de credenciales?
- Cómo asignar roles a EC2

**5. User Data**

- Se ejecuta una sola vez
- Con privilegios root
- Para bootstrap/configuración inicial

**6. Modelo de responsabilidad compartida**

- Qué gestiona AWS
- Qué gestionas tú

#### Preguntas típicas del examen:

**Pregunta 1:**

```
Una empresa necesita ejecutar cargas de trabajo de procesamiento 
por lotes que pueden interrumpirse sin problemas. ¿Qué opción 
de compra proporciona el mayor ahorro?

A) On-Demand Instances
B) Reserved Instances
C) Spot Instances ✅
D) Dedicated Hosts

Respuesta: C - Spot ofrece hasta 90% descuento y es ideal para 
cargas tolerantes a interrupciones.
```

**Pregunta 2:**

```
¿Qué es responsabilidad del cliente en el modelo de 
responsabilidad compartida para EC2?

A) Mantenimiento del hardware físico
B) Parches del sistema operativo ✅
C) Seguridad del hipervisor
D) Mantenimiento de la red física

Respuesta: B - El cliente debe mantener el SO actualizado.
```

**Pregunta 3:**

```
Una aplicación web necesita permitir tráfico HTTP desde 
cualquier lugar. ¿Qué configuración de Security Group es correcta?

A) Type: HTTP, Port: 80, Source: 0.0.0.0/0 ✅
B) Type: HTTP, Port: 443, Source: Mi IP
C) Type: HTTPS, Port: 80, Source: 0.0.0.0/0
D) Type: SSH, Port: 22, Source: 0.0.0.0/0

Respuesta: A - HTTP usa puerto 80 y debe permitir todo el tráfico.
```

**Pregunta 4:**

```
¿Cuál es la mejor práctica para que una instancia EC2
acceda a un bucket de S3?

A) Configurar credenciales con aws configure
B) Hardcodear Access Key en la aplicación
C) Asignar un rol IAM a la instancia ✅
D) Compartir la contraseña root de AWS

Respuesta: C - Los roles IAM son la forma segura y recomendada.
```

### ✅ Checklist Final

Antes del examen, asegúrate de poder responder:

- [ ] ¿Qué es EC2 y para qué sirve?
- [ ] ¿Cuáles son los componentes de una instancia EC2?
- [ ] ¿Qué tipos de instancias existen y sus casos de uso?
- [ ] ¿Cómo funcionan los Security Groups?
- [ ] ¿Cuáles son los puertos más comunes? (22, 80, 443, 3389)
- [ ] ¿Cómo conectarse a una instancia EC2?
- [ ] ¿Por qué usar roles IAM en lugar de credenciales?
- [ ] ¿Qué son los User Data y cuándo se ejecutan?
- [ ] ¿Cuáles son las opciones de compra y sus ahorros?
- [ ] ¿Qué es responsabilidad de AWS vs el cliente?
- [ ] ¿Cuándo usar On-Demand vs Reserved vs Spot?
- [ ] ¿Qué sucede con la IP pública al detener una instancia?

### 📚 Recursos Adicionales

**Documentación oficial:**

- EC2 User Guide: https://docs.aws.amazon.com/ec2/
- Security Groups: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html
- Instance Types: https://aws.amazon.com/ec2/instance-types/
- Pricing: https://aws.amazon.com/ec2/pricing/

**Herramientas útiles:**

- Calculadora de precios: https://calculator.aws/
- Comparador de instancias: https://instances.vantage.sh/
- EC2 Instance Connect: Desde la consola de AWS

**Práctica:**

- AWS Free Tier: 750 horas/mes de t2.micro
- Siempre terminar instancias después de practicar
- Configurar presupuestos para evitar cargos

---

## 🎉 ¡Felicidades!

Has completado el módulo de EC2. Ahora deberías ser capaz de:

- ✅ Lanzar y configurar instancias EC2
- ✅ Implementar Security Groups correctamente
- ✅ Conectarte a instancias mediante SSH
- ✅ Asignar roles IAM a instancias
- ✅ Elegir la opción de compra adecuada
- ✅ Entender el modelo de responsabilidad compartida

### 🚀 Próximos Pasos

1. **Practicar** lanzando diferentes tipos de instancias
2. **Experimentar** con User Data scripts
3. **Configurar** Security Groups restrictivos
4. **Asignar** roles IAM para acceso a S3
5. **Calcular** costos con diferentes opciones de compra
6. **Realizar** el cuestionario de EC2

¡Mucho éxito en tu preparación para el examen AWS Cloud Practitioner! 🎓

---

**Última actualización**: Octubre 2024  
**Versión**: 1.0  
**Autor**: Basado en el curso AWS Certified Cloud Practitioner

---

## Cuestionario - Amazon EC2

### Instrucciones

- Este cuestionario contiene 20 preguntas sobre Amazon EC2
- Cada pregunta tiene una o más respuestas correctas
- Lee cuidadosamente cada pregunta antes de responder
- Tiempo estimado: 30 minutos

---

### Preguntas

#### Pregunta 1

**¿Qué significa EC2?**

A) Elastic Container Cloud  
B) Elastic Compute Cloud  
C) Elastic Configuration Cloud  
D) Elastic Computing Cluster

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

EC2 significa Elastic Compute Cloud. Es el servicio de máquinas virtuales de AWS y uno de los servicios más fundamentales de la plataforma.

</details>

---

#### Pregunta 2

**Una empresa necesita ejecutar una aplicación web 24/7 durante los próximos 3 años con carga predecible. ¿Qué opción de compra de EC2 proporciona el mayor ahorro?**

A) On-Demand Instances  
B) Spot Instances  
C) Reserved Instances (3 años, pago total adelantado)  
D) Dedicated Hosts

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

Reserved Instances con compromiso de 3 años y pago total adelantado ofrecen el mayor descuento (hasta 72%) para cargas de trabajo predecibles y de larga duración.

- On-Demand: 0% descuento
- Spot: Hasta 90% pero puede interrumpirse
- Reserved 3 años: Hasta 72% descuento
- Dedicated Hosts: Más caro, para cumplimiento normativo

</details>

---

#### Pregunta 3

**¿Cuál de las siguientes es responsabilidad del cliente en el modelo de responsabilidad compartida para EC2?**

A) Mantenimiento del hardware físico del servidor  
B) Actualización y parcheo del sistema operativo invitado  
C) Seguridad del hipervisor  
D) Mantenimiento de la infraestructura de red del data center

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

El cliente es responsable de actualizar y parchear el sistema operativo de sus instancias EC2.

**AWS es responsable de:**
- Hardware físico
- Hipervisor
- Infraestructura de red del data center
- Aislamiento entre clientes

**El cliente es responsable de:**
- Sistema operativo invitado
- Aplicaciones
- Security Groups
- Datos y cifrado

</details>

---

#### Pregunta 4

**¿Qué puertos deben estar abiertos en un Security Group para permitir que los usuarios accedan a un sitio web mediante HTTPS?**

A) Puerto 22  
B) Puerto 80  
C) Puerto 443  
D) Puerto 3389

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

El puerto 443 es utilizado por el protocolo HTTPS para conexiones web seguras.

**Puertos comunes:**
- 22: SSH (Linux)
- 80: HTTP (web no segura)
- 443: HTTPS (web segura)
- 3389: RDP (Windows Remote Desktop)

</details>

---

#### Pregunta 5

**¿Cuándo se ejecuta el script de User Data de EC2?**

A) Cada vez que se inicia la instancia  
B) Solo la primera vez que se inicia la instancia  
C) Cada vez que se detiene la instancia  
D) Manualmente por el usuario

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

El script de User Data se ejecuta solo una vez, durante el primer arranque de la instancia. Se utiliza para automatizar la configuración inicial (bootstrap).

**Características:**
- Se ejecuta con privilegios de root
- Solo en el primer inicio
- Útil para: instalar software, actualizar sistema, configurar servicios

</details>

---

#### Pregunta 6

**Una startup está desarrollando una nueva aplicación y no puede predecir los patrones de uso. ¿Qué opción de compra es la más adecuada?**

A) Reserved Instances  
B) On-Demand Instances  
C) Spot Instances  
D) Dedicated Hosts

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

On-Demand Instances son ideales cuando:
- El uso es impredecible
- No hay compromiso a largo plazo
- Se requiere máxima flexibilidad
- Es un proyecto nuevo o en desarrollo

A pesar de ser la opción más cara, ofrece flexibilidad total sin compromisos.

</details>

---

#### Pregunta 7

**¿Cuál es la mejor práctica para que una instancia EC2 acceda a objetos en un bucket de S3?**

A) Configurar credenciales de AWS usando aws configure  
B) Hardcodear las Access Keys en el código de la aplicación  
C) Asignar un rol IAM a la instancia EC2  
D) Usar las credenciales del usuario root de AWS

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

Asignar un rol IAM a la instancia es la mejor práctica porque:

- ✅ No expone credenciales
- ✅ Rotación automática de credenciales temporales
- ✅ Fácil de auditar
- ✅ Sigue el principio de mínimo privilegio

**❌ NUNCA usar:**
- aws configure en instancias EC2
- Access Keys hardcodeadas
- Credenciales root

</details>

---

#### Pregunta 8

**¿Qué tipo de instancia EC2 es la más adecuada para una aplicación de análisis de big data que requiere grandes cantidades de RAM?**

A) Instancia de propósito general (T2, M5)  
B) Instancia optimizada para computación (C5)  
C) Instancia optimizada para memoria (R5)  
D) Instancia optimizada para almacenamiento (I3)

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

Instancias optimizadas para memoria (R5, X1) están diseñadas para cargas de trabajo que requieren grandes cantidades de RAM:

- Análisis de big data en memoria
- Bases de datos en memoria (Redis, SAP HANA)
- Procesamiento de datos a gran escala
- Aplicaciones de business intelligence

Ejemplo: r5.16xlarge tiene 512 GiB de RAM

</details>

---

#### Pregunta 9

**Una empresa necesita procesar 10,000 imágenes. El trabajo puede interrumpirse y reanudarse sin problemas. ¿Qué opción de compra minimiza los costos?**

A) On-Demand Instances  
B) Reserved Instances  
C) Spot Instances  
D) Dedicated Instances

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

Spot Instances ofrecen hasta 90% de descuento y son perfectas para:

- ✅ Trabajos por lotes (batch jobs)
- ✅ Procesamiento de imágenes/videos
- ✅ Análisis de datos
- ✅ Cargas tolerantes a interrupciones
- ✅ Trabajos con inicio/fin flexible

**❌ NO usar Spot para:**
- Bases de datos
- Aplicaciones críticas
- Servidores web de producción

</details>

---

#### Pregunta 10

**¿Qué sucede con la dirección IP pública de una instancia EC2 cuando se detiene y luego se reinicia?**

A) Permanece igual  
B) Cambia a una nueva dirección IP pública  
C) Se pierde permanentemente  
D) Se convierte en una IP privada

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

La dirección IP pública cambia cada vez que se detiene y reinicia una instancia.

- IP privada: Permanece igual
- IP pública: Cambia en cada stop/start
- Elastic IP: Puede asignarse para mantener IP estática (pero tiene costo)

⚠️ Importante: Esto puede afectar aplicaciones que dependen de la IP

</details>

---

#### Pregunta 11 (Selección Múltiple)

**¿Cuáles de las siguientes son características de los Security Groups? (Selecciona DOS)**

A) Solo contienen reglas ALLOW (permitir)  
B) Son stateless  
C) Se aplican a nivel de subnet  
D) Son stateful  
E) Pueden aplicarse a múltiples instancias

<details>
<summary>Ver respuesta</summary>

**Respuestas correctas: A y D**

**A) Solo contienen reglas ALLOW** ✅
- No puedes crear reglas DENY explícitas
- Todo está denegado por defecto

**D) Son stateful** ✅
- Si permites tráfico entrante, la respuesta se permite automáticamente
- No necesitas regla de salida para respuestas

**Incorrectas:**
- B: Son stateful, no stateless (NACLs son stateless)
- C: Se aplican a nivel de instancia/ENI, no subnet
- E: Correcto, pero solo se pedían DOS respuestas

</details>

---

#### Pregunta 12

**Un desarrollador necesita conectarse remotamente a una instancia EC2 con Linux. ¿Qué puerto debe estar abierto en el Security Group?**

A) Puerto 21 (FTP)  
B) Puerto 22 (SSH)  
C) Puerto 80 (HTTP)  
D) Puerto 3389 (RDP)

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

Puerto 22 (SSH) se utiliza para conexiones remotas seguras a instancias Linux.

- Puerto 22: SSH (Linux/Mac/Unix)
- Puerto 3389: RDP (Windows)
- Puerto 21: FTP (transferencia de archivos, inseguro)
- Puerto 80: HTTP (web)

</details>

---

#### Pregunta 13

**¿Qué servicio permite conectarse a una instancia EC2 directamente desde el navegador sin necesidad de descargar claves SSH?**

A) AWS Systems Manager Session Manager  
B) EC2 Instance Connect  
C) AWS CloudShell  
D) AWS Management Console

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

EC2 Instance Connect permite:

- Conexión desde el navegador
- No requiere descargar claves SSH
- Funciona en Amazon Linux 2 y Ubuntu
- Requiere puerto 22 abierto en Security Group

AWS Systems Manager Session Manager también funciona pero no requiere puerto 22 abierto.

</details>

---

#### Pregunta 14

**Una empresa necesita cumplir con regulaciones que requieren servidores físicos dedicados. ¿Qué opción de EC2 deben usar?**

A) On-Demand Instances  
B) Reserved Instances  
C) Dedicated Hosts  
D) Spot Instances

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

Dedicated Hosts proporcionan:

- Servidor físico completo dedicado
- Visibilidad de sockets, cores, host ID
- Control de ubicación de instancias
- Soporte para licencias BYOL (Bring Your Own License)
- Cumplimiento de regulaciones estrictas

Dedicated Instances también ofrecen aislamiento pero sin visibilidad ni control del hardware físico.

</details>

---

#### Pregunta 15

**¿Cuál de las siguientes afirmaciones sobre los datos de usuario (User Data) de EC2 es CORRECTA?**

A) Se ejecutan cada vez que se reinicia la instancia  
B) Se ejecutan con permisos de usuario limitados  
C) Solo se ejecutan en el primer arranque de la instancia  
D) Requieren instalación manual de software adicional

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

User Data se ejecuta solo en el primer arranque de la instancia.

**Características:**
- ✅ Solo primera ejecución
- ✅ Con privilegios de root
- ✅ Para bootstrap/configuración automática
- ✅ No requiere software adicional

**Ejemplo de uso:**

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
```

</details>

---

#### Pregunta 16

**¿Qué herramienta de AWS ayuda a optimizar costos recomendando el tipo y tamaño de instancia más adecuado?**

A) AWS Cost Explorer  
B) AWS Trusted Advisor  
C) AWS Compute Optimizer  
D) AWS Budgets

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

**AWS Compute Optimizer** utiliza machine learning para:

- Recomendar tipos de instancia óptimos
- Analizar patrones de utilización
- Sugerir ajustes de tamaño (right-sizing)
- Ayudar a reducir costos hasta 25%

**Otras herramientas:**
- Cost Explorer: Visualiza y analiza costos
- Trusted Advisor: Recomendaciones generales (incluye optimización)
- Budgets: Alertas de presupuesto

</details>

---

#### Pregunta 17 (Selección Múltiple)

**¿Cuáles son componentes que puedes configurar al lanzar una instancia EC2? (Selecciona TRES)**

A) AMI (Amazon Machine Image)  
B) Tipo de instancia  
C) Región de AWS  
D) Security Group  
E) Dominio DNS personalizado

<details>
<summary>Ver respuesta</summary>

**Respuestas correctas: A, B y D**

Al lanzar una instancia EC2 configuras:

**A) AMI** ✅
- Sistema operativo
- Software preinstalado
- Configuración base

**B) Tipo de instancia** ✅
- CPU y RAM
- Familia (t2, m5, c5, etc.)
- Tamaño (micro, small, large, xlarge)

**D) Security Group** ✅
- Reglas de firewall
- Puertos permitidos
- Control de acceso

**Incorrectas:**
- C: La región se selecciona antes, no durante el launch
- E: DNS se configura en Route 53, no en el launch de EC2

</details>

---

#### Pregunta 18

**Una aplicación necesita acceder a DynamoDB. ¿Cuál es la forma MÁS segura de proporcionar acceso desde una instancia EC2?**

A) Crear un usuario IAM y ejecutar `aws configure` en la instancia  
B) Hardcodear Access Keys en las variables de entorno  
C) Crear un rol IAM con permisos de DynamoDB y asignarlo a la instancia  
D) Usar las credenciales del usuario root

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

**Crear un rol IAM y asignarlo a la instancia** es la forma más segura:

**Proceso:**
1. Crear rol IAM con política DynamoDB
2. Asignar rol a instancia EC2
3. Aplicación usa credenciales temporales automáticamente

**Ventajas:**
- ✅ Sin credenciales expuestas
- ✅ Rotación automática
- ✅ Auditable con CloudTrail
- ✅ Principio de mínimo privilegio

**❌ Nunca:**
- Usar `aws configure` en producción
- Hardcodear credenciales
- Usar credenciales root

</details>

---

#### Pregunta 19

**¿Qué característica diferencia a las Reserved Instances Convertibles de las Standard Reserved Instances?**

A) Mayor descuento  
B) Posibilidad de cambiar el tipo de instancia durante el periodo reservado  
C) No requieren compromiso de tiempo  
D) Pueden interrumpirse por AWS

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: B**

**Convertible Reserved Instances** permiten:

- ✅ Cambiar tipo de instancia
- ✅ Cambiar sistema operativo
- ✅ Cambiar tenancy (default/dedicated)
- ✅ Cambiar familia de instancias

**Comparación:**

| Aspecto | Standard | Convertible |
|---------|----------|-------------|
| Descuento | Hasta 72% | Hasta 66% |
| Flexibilidad | Baja | Alta |
| Cambio de tipo | ❌ No | ✅ Sí |
| Precio | Más barato | Más caro |

</details>

---

#### Pregunta 20

**Una empresa tiene cargas de trabajo base constantes y picos impredecibles. ¿Cuál es la estrategia de compra MÁS rentable?**

A) Solo On-Demand Instances  
B) Solo Spot Instances  
C) Reserved Instances para la base + On-Demand para picos  
D) Solo Reserved Instances

<details>
<summary>Ver respuesta</summary>

**Respuesta correcta: C**

**Estrategia mixta** es la más rentable:

```
Arquitectura recomendada:

├── Carga base (70%) → Reserved Instances
│   └── Ahorro: 60-72%
│
├── Carga predecible (20%) → Savings Plans
│   └── Ahorro: 40-50%
│
└── Picos (10%) → On-Demand + Spot
    └── Flexibilidad total
```

**Ejemplo:**
- Base: 10 instancias 24/7 → Reserved
- Picos: 0-5 instancias variables → On-Demand
- Ahorro total: ~50-60% vs todo On-Demand

Esta estrategia optimiza costo y flexibilidad.

</details>

---

## Resultados

**Puntuación para aprobar**: 14/20 (70%)

### Evaluación por categorías:

**Fundamentos de EC2** (Preguntas 1, 5, 10, 15)
- Conceptos básicos
- User Data
- Comportamiento de IPs

**Seguridad** (Preguntas 3, 4, 7, 12, 18)
- Responsabilidad compartida
- Security Groups
- Roles IAM
- Puertos

**Opciones de Compra** (Preguntas 2, 6, 9, 14, 19, 20)
- On-Demand, Reserved, Spot, Dedicated
- Estrategias de optimización
- Casos de uso

**Tipos y Configuración** (Preguntas 8, 11, 13, 16, 17)
- Tipos de instancias
- Herramientas de optimización
- Componentes de configuración

---

## Análisis de Errores Comunes

### Error #1: Confundir responsabilidades compartidas

```
❌ Pensar que AWS actualiza el SO
✅ El cliente debe parchear el SO invitado

Recordar:
- AWS → Seguridad DEL cloud (infraestructura)
- Cliente → Seguridad EN el cloud (datos, SO, apps)
```

### Error #2: No entender Security Groups

```
❌ Pensar que son stateless
✅ Son stateful (respuestas automáticas)

❌ Crear reglas DENY
✅ Solo reglas ALLOW (deny por defecto)
```

### Error #3: Confundir opciones de compra

```
❌ Usar Reserved para cargas impredecibles
✅ Reserved para cargas estables

❌ Usar Spot para bases de datos
✅ Spot para trabajos por lotes tolerantes a fallos
```

### Error #4: Mal uso de credenciales

```
❌ Usar aws configure en instancias
✅ Asignar roles IAM

❌ Hardcodear Access Keys
✅ Usar credenciales temporales vía roles
```

---

## Recursos para Mejorar

### Si obtuviste menos de 14/20:

- Repasar sección de Fundamentos de EC2
- Practicar lanzamiento de instancias
- Experimentar con Security Groups
- Estudiar modelo de responsabilidad compartida

### Si obtuviste 14-17/20:

- Profundizar en opciones de compra
- Practicar asignación de roles IAM
- Estudiar estrategias de optimización de costos
- Revisar casos de uso de cada tipo de instancia

### Si obtuviste 18-20/20:

¡Excelente! Estás listo para esta sección del examen.

- Revisar otros módulos del curso
- Hacer exámenes prácticos completos
- Enfocarte en otros servicios de AWS

---

## Próximos Pasos

- ✅ Completado: Amazon EC2

**Continúa con:**

- 📦 Almacenamiento de instancias EC2 (EBS, EFS, Instance Store)
- ⚖️ Elastic Load Balancing y Auto Scaling Groups
- 🗄️ Amazon S3

---

## ¿Listo para el siguiente módulo?

Continúa tu preparación con el módulo de **Almacenamiento EC2** donde aprenderás sobre:

- EBS (Elastic Block Store)
- Snapshots
- AMIs (Amazon Machine Images)
- EFS (Elastic File System)
- EC2 Instance Store

¡Sigue así! 🚀

---
***
## Notas Finales

Este material cubre la Introducción a Intancias EC2 para el examen AWS Certified Cloud Practitioner. Practica los conceptos en tu propia cuenta de AWS (usa la capa gratuita) para reforzar el aprendizaje.

¡Buena suerte en tu examen!

**Última actualización**: 22/10/2025  
**Versión del curso**: AWS Certified Cloud Practitioner CLF-C02