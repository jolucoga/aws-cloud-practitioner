
## 1. Introducción a IAM
```
¿Qué es IAM?
IAM (Identity and Access Management) es un servicio global de AWS que permite controlar de forma segura el acceso a los servicios y recursos de AWS.
Características Principales
CaracterísticaDescripciónServicio GlobalNo está vinculado a regiones específicasGratuitoNo tiene costo adicionalControl GranularPermisos específicos a nivel de recurso y acciónSeguridadBasado en el principio de mínimo privilegio
¿Por qué usar IAM?
Cuando creas una cuenta de AWS, obtienes un usuario root con acceso completo. Sin embargo, usar el root para operaciones diarias es una mala práctica de seguridad.
Ventajas de IAM:
✅ Crear usuarios individuales para cada persona
✅ Asignar permisos específicos según necesidades
✅ Auditar quién hace qué en tu cuenta
✅ Aplicar políticas de seguridad robustas
Componentes Fundamentales de IAM
mermaidgraph TD
    A[IAM] --> B[Usuarios]
    A --> C[Grupos]
    A --> D[Políticas]
    A --> E[Roles]
    B --> F[Personas o Aplicaciones]
    C --> G[Colección de Usuarios]
    D --> H[Documentos JSON]
    E --> I[Para Servicios AWS]
```

---

## 2. Usuarios y Grupos de IAM

### 2.1 Usuarios de IAM

Un usuario de IAM representa una identidad con credenciales permanentes para interactuar con AWS.

#### Características de los Usuarios

- **Credenciales únicas**: Nombre de usuario y contraseña
- **Claves de acceso**: Para acceso programático
- **Permisos asignados**: Mediante políticas
- **Representan**: Personas reales o aplicaciones

#### Tipos de Acceso

1. **Acceso a la Consola**
   - Usuario y contraseña
   - Interfaz web de AWS
   - Requiere MFA (recomendado)

2. **Acceso Programático**
   - Access Key ID
   - Secret Access Key
   - Para CLI, SDK, APIs

### 2.2 Grupos de IAM

Los grupos son colecciones de usuarios que comparten los mismos permisos.

#### Ejemplo de Estructura Organizacional
```
Organización AWS
│
├── Grupo: Desarrolladores
│   ├── Alice
│   ├── Bob
│   └── Charles
│
├── Grupo: Operaciones
│   ├── David
│   └── Edward
│
└── Grupo: Equipo de Auditoría
    ├── Charles (también en Desarrolladores)
    └── David (también en Operaciones)
Reglas Importantes de Grupos
✅ Permitido❌ No PermitidoUsuario en múltiples gruposGrupos dentro de otros gruposUsuario sin grupo (no recomendado)Usar grupos como identidades en políticasAsignar políticas a gruposGrupos con permisos de root
```

### 2.3 Práctica: Crear Usuarios y Grupos
#### Paso 1: Crear un Grupo
bash# Acceso a la Consola de AWS
1. Navega a: IAM > User groups
2. Click: "Create group"
3. Nombre: "Developers"
4. Adjuntar políticas (opcional en creación)
5. Click: "Create group"
#### Paso 2: Crear un Usuario
bash# En la Consola de IAM
1. Navega a: IAM > Users
2. Click: "Create user"
3. Configuración:
   - User name: "john-developer"
   - ☑ Provide user access to the AWS Management Console
   - Console password: Auto-generated o Custom
   - ☑ Users must create a new password at next sign-in (recomendado)
4. Click: "Next"
#### Paso 3: Asignar Permisos
Opción A: Agregar a un Grupo
bash1. Seleccionar: "Add user to group"
2. Marcar: "Developers"
3. Click: "Next"
Opción B: Políticas Directas (no recomendado)
bash1. Seleccionar: "Attach policies directly"
2. Buscar y seleccionar políticas
3. Click: "Next"
Opción C: Copiar Permisos
bash1. Seleccionar: "Copy permissions from existing user"
2. Elegir usuario de referencia
3. Click: "Next"
#### Paso 4: Revisar y Crear
bash1. Revisar configuración
2. Tags (opcional pero recomendado):
   - Key: "Department" | Value: "Engineering"
   - Key: "Environment" | Value: "Production"
3. Click: "Create user"
4. ⚠️ IMPORTANTE: Descargar credenciales (única oportunidad)


### 2.4 Acceso como Usuario IAM

#### URL de Inicio de Sesión
```
https://[ACCOUNT-ID].signin.aws.amazon.com/console
```

O usando el alias de cuenta:
```
https://[ACCOUNT-ALIAS].signin.aws.amazon.com/console
Crear Alias de Cuenta
bash# En IAM Dashboard
1. Navega a: IAM > Dashboard
2. Sección: "AWS Account"
3. Click: "Create" junto a "Account Alias"
4. Ingresa: "mi-empresa-dev"
5. Click: "Create alias"
```

#### Primer Inicio de Sesión
```
Usuario: john-developer
Contraseña: [contraseña temporal]
→ Sistema solicita cambio de contraseña
→ Ingresar nueva contraseña fuerte
→ Acceso concedido

3. Políticas IAM
3.1 ¿Qué son las Políticas IAM?
Las políticas son documentos JSON que definen permisos. Se aplican a usuarios, grupos o roles para controlar qué acciones pueden realizar en qué recursos.
3.2 Estructura de una Política IAM
json{
  "Version": "2012-10-17",
  "Id": "S3-Account-Permissions",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "192.168.1.0/24"
        }
      }
    }
  ]
}
3.3 Componentes de una Política
ElementoDescripciónObligatorioVersionVersión del lenguaje (siempre "2012-10-17")SíIdIdentificador de la políticaNoStatementArray de declaracionesSíSidID de la declaraciónNoEffect"Allow" o "Deny"SíPrincipalCuenta/usuario/rol al que aplicaDepende*ActionAcciones permitidas/denegadasSíResourceRecursos afectadosSíConditionCondiciones opcionalesNo
*Requerido en políticas basadas en recursos
3.4 Tipos de Políticas
1. Políticas Gestionadas por AWS
Creadas y mantenidas por AWS:
json{
  "PolicyName": "AmazonS3ReadOnlyAccess",
  "Description": "Provides read-only access to Amazon S3",
  "ManagedBy": "AWS"
}
2. Políticas Gestionadas por el Cliente
Creadas por ti, reutilizables:
json{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }
  ]
}
3. Políticas Inline
Integradas directamente en un usuario/grupo/rol específico (no recomendado):
bash# Las políticas inline NO son reutilizables
# Se eliminan cuando se elimina el usuario/grupo/rol
```

### 3.5 Herencia de Políticas
```
Grupo: Developers
└── Política: DeveloperAccess
    ├── Alice (hereda DeveloperAccess)
    ├── Bob (hereda DeveloperAccess)
    └── Charles (hereda DeveloperAccess)
        └── Política Inline: ExtraPermissions
            └── Resultado: DeveloperAccess + ExtraPermissions
3.6 Práctica: Crear y Aplicar Políticas
Crear Política Personalizada
bash# En la Consola de AWS
1. Navega a: IAM > Policies
2. Click: "Create policy"
3. Dos opciones:
   a) Visual Editor (GUI)
   b) JSON (código directo)
Opción A: Visual Editor
bash1. Service: "S3"
2. Actions:
   - Access level: Read
   - ☑ GetObject
   - ☑ ListBucket
3. Resources:
   - bucket: "my-app-bucket"
   - object: "my-app-bucket/*"
4. Request conditions (opcional):
   - Source IP: 203.0.113.0/24
5. Click: "Next"
6. Name: "MyAppS3ReadOnly"
7. Description: "Read-only access to my-app-bucket"
8. Click: "Create policy"
Opción B: JSON Editor
json{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-app-bucket"
    },
    {
      "Sid": "GetObjects",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-app-bucket/*"
    }
  ]
}
Adjuntar Política a un Grupo
bash1. Navega a: IAM > User groups
2. Selecciona: "Developers"
3. Tab: "Permissions"
4. Click: "Add permissions" > "Attach policies"
5. Buscar: "MyAppS3ReadOnly"
6. ☑ Marcar la política
7. Click: "Add permissions"
Adjuntar Política a un Usuario
bash1. Navega a: IAM > Users
2. Selecciona: "john-developer"
3. Tab: "Permissions"
4. Click: "Add permissions" > "Attach policies directly"
5. Buscar y seleccionar política
6. Click: "Add permissions"
3.7 Ejemplo Práctico: Usuario en Múltiples Grupos
Escenario
Charles necesita acceso como desarrollador Y como auditor.
bash# Estructura:
Charles
├── Grupo: Developers
│   └── Política: DevelopmentAccess
│       ├── EC2 (describe, start, stop)
│       └── S3 (read/write en dev-bucket)
│
└── Grupo: Auditors
    └── Política: AuditAccess
        ├── CloudTrail (read-only)
        ├── CloudWatch Logs (read-only)
        └── IAM (read-only)

# Permisos Finales de Charles:
# = DevelopmentAccess + AuditAccess
Implementación
bash# Paso 1: Crear grupos con sus políticas
1. Crear "Developers" con "DevelopmentAccess"
2. Crear "Auditors" con "AuditAccess"

# Paso 2: Agregar usuario a múltiples grupos
1. Navega a: IAM > Users > Charles
2. Tab: "Permissions"
3. Click: "Add permissions" > "Add user to group"
4. ☑ Developers
5. ☑ Auditors
6. Click: "Add permissions"
3.8 Política de Denegación Explícita
La denegación SIEMPRE tiene prioridad:
json{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}
Resultado: Puede hacer TODO en S3 EXCEPTO eliminar buckets.
3.9 Ejemplos de Políticas Comunes
Acceso Completo a EC2
json{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    }
  ]
}
Solo Lectura en RDS
json{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "rds:Describe*",
        "rds:List*"
      ],
      "Resource": "*"
    }
  ]
}
Gestión de IAM Limitada
json{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:GetUser",
        "iam:ListUsers",
        "iam:GetPolicy",
        "iam:ListPolicies"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## 4. Autenticación Multifactor (MFA)

### 4.1 ¿Qué es MFA?

MFA (Multi-Factor Authentication) añade una capa extra de seguridad requiriendo:
```
Autenticación = Algo que SABES + Algo que POSEES

┌─────────────────┐     ┌──────────────────┐
│   Contraseña    │  +  │  Código MFA      │
│  (algo que      │     │  (algo que       │
│   sabes)        │     │   posees)        │
└─────────────────┘     └──────────────────┘
```

### 4.2 ¿Por qué usar MFA?

#### Escenario Sin MFA
```
Hacker roba contraseña
→ Acceso COMPLETO a la cuenta
→ Puede eliminar recursos
→ Puede cambiar configuraciones
→ DESASTRE 💥
```

#### Escenario Con MFA
```
Hacker roba contraseña
→ Intenta acceder
→ Sistema solicita código MFA
→ Hacker NO tiene el dispositivo
→ Acceso DENEGADO ✅
→ Cuenta PROTEGIDA 🛡️
4.3 Opciones de Dispositivos MFA en AWS
TipoOpcionesCaracterísticasVirtual MFA• Google Authenticator<br>• Authy<br>• Microsoft Authenticator• Aplicación en smartphone<br>• Gratuito<br>• Fácil configuraciónU2F Security Key• YubiKey<br>• Titan Security Key• Dispositivo físico USB<br>• Múltiples usuarios/cuentas<br>• Muy seguroHardware MFA• Gemalto<br>• SurePassID• Dispositivo físico dedicado<br>• Para GovCloud<br>• Muy seguro
4.4 Práctica: Configurar MFA
Para Usuario Root
bash# ⚠️ CRÍTICO: Siempre activa MFA en la cuenta root

1. Iniciar sesión como root
2. Navega a: Nombre de cuenta (arriba derecha) > Security credentials
3. Sección: "Multi-factor authentication (MFA)"
4. Click: "Assign MFA device"
5. Device name: "root-mfa-device"
6. MFA device type:
   - ○ Authenticator app (recomendado)
   - ○ Security key
   - ○ Hardware TOTP token
7. Click: "Next"
Configuración con Authenticator App
bash# Paso 1: Escanear QR
1. Abrir Google Authenticator o Authy en smartphone
2. Click: "+" o "Add account"
3. Escanear código QR mostrado en pantalla AWS

# Paso 2: Ingresar códigos
1. Esperar a que se genere código MFA (6 dígitos)
2. Ingresar en "MFA code 1"
3. Esperar a siguiente código (30 segundos)
4. Ingresar en "MFA code 2"
5. Click: "Add MFA"

# ✅ MFA activado correctamente
Para Usuarios IAM
bash# Los usuarios IAM pueden configurar su propio MFA

1. Iniciar sesión como usuario IAM
2. Navega a: Nombre de usuario > Security credentials
3. Sección: "Multi-factor authentication (MFA)"
4. Click: "Assign MFA device"
5. Seguir mismos pasos que root
Configuración con Security Key (YubiKey)
bash1. Seleccionar: "Security key"
2. Click: "Next"
3. Insertar YubiKey en puerto USB
4. Tocar sensor de YubiKey cuando parpadee
5. Click: "Add MFA"

# ✅ Security key registrada
```

### 4.5 Inicio de Sesión con MFA

#### Proceso de Login
```
┌─────────────────────────────────────────────┐
│ 1. Ingresar nombre de usuario y contraseña │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ 2. AWS solicita código MFA                 │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ 3. Abrir app autenticador                  │
│    Obtener código de 6 dígitos             │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ 4. Ingresar código MFA                     │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│ 5. ✅ Acceso concedido                      │
└─────────────────────────────────────────────┘
4.6 Política de Contraseñas
Además de MFA, configura una política de contraseñas fuerte:
bash# En IAM Dashboard
1. Navega a: IAM > Account settings
2. Sección: "Password policy"
3. Click: "Edit"

# Configuración Recomendada:
☑ Minimum password length: 14 characters
☑ Require at least one uppercase letter
☑ Require at least one lowercase letter
☑ Require at least one number
☑ Require at least one non-alphanumeric character
☑ Enable password expiration: 90 days
☑ Password expiration requires administrator reset: No
☑ Allow users to change their own password: Yes
☑ Prevent password reuse: 5 passwords remembered

4. Click: "Save changes"
4.7 Recuperación de Acceso sin MFA
Si pierdes el dispositivo MFA:
bash# Para Cuenta Root:
1. Proceso de recuperación alternativo
2. Contactar AWS Support
3. Verificación de identidad exhaustiva

# Para Usuario IAM:
1. Administrador puede desactivar MFA
2. Usuario configura nuevo dispositivo
3. Reactivar MFA con nuevo dispositivo
```

---

## 5. Acceso Programático: CLI y SDK

### 5.1 Métodos de Acceso a AWS

AWS proporciona tres formas de interactuar con sus servicios:

| Método | Descripción | Protección |
|--------|-------------|------------|
| **Consola AWS** | Interfaz web gráfica | Contraseña + MFA |
| **AWS CLI** | Línea de comandos | Claves de acceso |
| **AWS SDK** | Librerías para código | Claves de acceso |

### 5.2 Claves de Acceso AWS

#### Componentes
```
Access Key = Access Key ID + Secret Access Key

┌────────────────────────────────────────────┐
│ Access Key ID (público)                    │
│ AKIAIOSFODNN7EXAMPLE                      │
│                                            │
│ ≈ Nombre de usuario                       │
└────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│ Secret Access Key (SECRETO)               │
│ wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY │
│                                            │
│ ≈ Contraseña                              │
└────────────────────────────────────────────┘
```

#### ⚠️ Reglas de Seguridad
```
❌ NUNCA compartir claves de acceso
❌ NUNCA incluir en código
❌ NUNCA subir a repositorios públicos
❌ NUNCA enviar por email

✅ Rotar regularmente
✅ Usar roles cuando sea posible
✅ Principio de mínimo privilegio
✅ Eliminar claves no usadas
5.3 AWS CLI (Command Line Interface)
¿Qué es AWS CLI?
Herramienta que permite interactuar con AWS mediante comandos en terminal.
bash# Ejemplo de comando
aws s3 ls
# Lista todos los buckets S3

aws ec2 describe-instances
# Lista instancias EC2
Características

✅ Acceso directo a APIs de AWS
✅ Scripting y automatización
✅ Código abierto
✅ Multiplataforma (Windows, macOS, Linux)
✅ Alternativa a la consola web

5.4 Instalación de AWS CLI
Windows
powershell# Método 1: MSI Installer
# Descargar de: https://aws.amazon.com/cli/

# Método 2: winget
winget install -e --id Amazon.AWSCLI

# Verificar instalación
aws --version
# Salida esperada: aws-cli/2.x.x Python/3.x.x Windows/10
macOS
bash# Método 1: Instalador PKG
# Descargar de: https://aws.amazon.com/cli/

# Método 2: Homebrew
brew install awscli

# Método 3: Python pip
pip3 install awscli

# Verificar instalación
aws --version
# Salida esperada: aws-cli/2.x.x Python/3.x.x Darwin/xx.x.x
Linux
bash# Ubuntu/Debian
sudo apt update
sudo apt install awscli

# Amazon Linux 2
sudo yum install aws-cli

# Método Universal: Binary
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verificar instalación
aws --version
5.5 Configuración de AWS CLI
Crear Claves de Acceso
bash# En la Consola AWS
1. Navega a: IAM > Users > [tu-usuario]
2. Tab: "Security credentials"
3. Sección: "Access keys"
4. Click: "Create access key"
5. Use case: "Command Line Interface (CLI)"
6. ☑ "I understand the above recommendation..."
7. Click: "Next"
8. Description tag: "Mi laptop personal"
9. Click: "Create access key"

# ⚠️ IMPORTANTE:
# - Descargar archivo .csv o copiar claves
# - Esta es la ÚNICA vez que verás la Secret Access Key
# - Guardar en un lugar seguro
Configuración Inicial
bash# Ejecutar comando de configuración
aws configure

# Ingresar información:
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: json

# ✅ Configuración guardada en:
# - ~/.aws/credentials (claves)
# - ~/.aws/config (configuración)
Archivo de Credenciales
ini# ~/.aws/credentials
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[usuario-produccion]
aws_access_key_id = AKIAI44QH8DHBEXAMPLE
aws_secret_access_key = je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
Archivo de Configuración
ini# ~/.aws/config
[default]
region = us-east-1
output = json

[profile usuario-produccion]
region = eu-west-1
output = table
5.6 Práctica con AWS CLI
Comandos Básicos
bash# Ver identidad actual
aws sts get-caller-identity
# Output:
{
    "UserId": "AIDAI23HXG2EEXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/john-developer"
}

# Listar buckets S3
aws s3 ls
# Output:
2024-01-15 10:30:00 mi-bucket-produccion
2024-02-20 14:45:00 mi-bucket-desarrollo

# Listar instancias EC2
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table

# Crear bucket S3
aws s3 mb s3://mi-nuevo-bucket-12345

# Copiar archivo a S3
aws s3 cp archivo.txt s3://mi-nuevo-bucket-12345/

# Descargar archivo de S3
aws s3 cp s3://mi-nuevo-bucket-12345/archivo.txt ./
Usar Perfiles
bash# Listar con perfil específico
aws s3 ls --profile usuario-produccion

# Configurar perfil adicional
aws configure --profile desarrollo
AWS Access Key ID [None]: AKIAI44QH8DHBEXAMPLE
AWS Secret Access Key [None]: je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json

# Usar perfil en comando
aws ec2 describe-instances --profile desarrollo
5.7 AWS SDK (Software Development Kit)
¿Qué es AWS SDK?
Conjunto de bibliotecas para programar con AWS en diferentes lenguajes.
Lenguajes Soportados
LenguajeSDKUso ComúnJavaScriptAWS SDK for JavaScriptNode.js, navegadorPythonBoto3Scripts, Machine LearningJavaAWS SDK for JavaAplicaciones enterprisePHPAWS SDK for PHPAplicaciones web.NETAWS SDK for .NETAplicaciones WindowsRubyAWS SDK for RubyRuby on RailsGoAWS SDK for GoMicroserviciosC++AWS SDK for C++Aplicaciones de alto rendimiento
Ejemplo: Python (Boto3)
bash# Instalación
pip install boto3

# Script básico
python# ejemplo.py
import boto3

# Crear cliente S3
s3 = boto3.client('s3')

# LiRetryClaude does not have the ability to run the code it generates yet.LContinuestar buckets
response = s3.list_buckets()
Mostrar nombres de buckets
print('Buckets existentes:')
for bucket in response['Buckets']:
print(f'  - {bucket["Name"]}')
Subir archivo
s3.upload_file('archivo.txt', 'mi-bucket', 'archivo.txt')
print('Archivo subido correctamente')
Descargar archivo
s3.download_file('mi-bucket', 'archivo.txt', 'descargado.txt')
print('Archivo descargado correctamente')

#### Ejemplo: Node.js
```bash
# Instalación
npm install @aws-sdk/client-s3
```
```javascript
// ejemplo.js
const { S3Client, ListBucketsCommand, PutObjectCommand } = require("@aws-sdk/client-s3");
const fs = require('fs');

// Crear cliente S3
const client = new S3Client({ region: "us-east-1" });

// Listar buckets
async function listBuckets() {
    try {
        const data = await client.send(new ListBucketsCommand({}));
        console.log("Buckets:", data.Buckets);
    } catch (err) {
        console.log("Error", err);
    }
}

// Subir archivo
async function uploadFile() {
    const fileContent = fs.readFileSync('archivo.txt');
    
    const params = {
        Bucket: "mi-bucket",
        Key: "archivo.txt",
        Body: fileContent
    };

    try {
        const data = await client.send(new PutObjectCommand(params));
        console.log("Archivo subido:", data);
    } catch (err) {
        console.log("Error", err);
    }
}

listBuckets();
uploadFile();
```

#### Ejemplo: Java
```java
// S3Example.java
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.ListBucketsRequest;
import software.amazon.awssdk.services.s3.model.ListBucketsResponse;
import software.amazon.awssdk.services.s3.model.Bucket;

public class S3Example {
    public static void main(String[] args) {
        // Crear cliente S3
        S3Client s3 = S3Client.builder()
            .region(Region.US_EAST_1)
            .build();

        // Listar buckets
        ListBucketsRequest listBucketsRequest = ListBucketsRequest.builder().build();
        ListBucketsResponse listBucketsResponse = s3.listBuckets(listBucketsRequest);
        
        System.out.println("Buckets:");
        for (Bucket bucket : listBucketsResponse.buckets()) {
            System.out.println("  - " + bucket.name());
        }

        s3.close();
    }
}
```

---

## 6. AWS CloudShell

### 6.1 ¿Qué es AWS CloudShell?

**AWS CloudShell** es una terminal basada en navegador, preautenticada con tus credenciales de la consola.

#### Características Principales
```
✅ Preinstalado: AWS CLI, Python, Node.js, Git
✅ Preautenticado: No necesita configurar credenciales
✅ Almacenamiento persistente: 1 GB por región
✅ Gratuito: Sin costo adicional
✅ Disponible en múltiples regiones
```

### 6.2 Disponibilidad Regional

CloudShell NO está disponible en todas las regiones:

| ✅ Disponible | ❌ No Disponible |
|---------------|------------------|
| us-east-1 (N. Virginia) | Regiones GovCloud |
| us-west-2 (Oregon) | China |
| eu-west-1 (Ireland) | Regiones más nuevas |
| ap-northeast-1 (Tokyo) | |
| [Verificar lista actualizada](https://docs.aws.amazon.com/cloudshell/latest/userguide/supported-aws-regions.html) | |

### 6.3 Práctica: Usar CloudShell

#### Acceso a CloudShell
```bash
# En la Consola de AWS
1. Click en el ícono de CloudShell (parte superior derecha)
   [>_] CloudShell
2. Se abre una terminal en el navegador
3. Espera a que se inicie (~10-30 segundos)

# ✅ Terminal lista para usar
```

#### Comandos Básicos
```bash
# Verificar identidad
[cloudshell-user@ip-10-0-123-45 ~]$ aws sts get-caller-identity
{
    "UserId": "AIDAI23HXG2EEXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/john-developer"
}

# Verificar región actual
[cloudshell-user@ip-10-0-123-45 ~]$ aws configure get region
us-east-1

# Listar buckets S3
[cloudshell-user@ip-10-0-123-45 ~]$ aws s3 ls

# Ver versiones de herramientas
[cloudshell-user@ip-10-0-123-45 ~]$ aws --version
aws-cli/2.x.x Python/3.x.x Linux/x.x.x

[cloudshell-user@ip-10-0-123-45 ~]$ python3 --version
Python 3.x.x

[cloudshell-user@ip-10-0-123-45 ~]$ node --version
v18.x.x
```

#### Crear y Gestionar Archivos
```bash
# Crear archivo
[cloudshell-user@ip-10-0-123-45 ~]$ echo "Hola desde CloudShell" > test.txt

# Ver contenido
[cloudshell-user@ip-10-0-123-45 ~]$ cat test.txt
Hola desde CloudShell

# Subir a S3
[cloudshell-user@ip-10-0-123-45 ~]$ aws s3 cp test.txt s3://mi-bucket/
upload: ./test.txt to s3://mi-bucket/test.txt

# Listar directorio home (persistente)
[cloudshell-user@ip-10-0-123-45 ~]$ ls -la ~/
total 12
drwxr-xr-x 2 cloudshell-user cloudshell-user 4096 Jan 20 10:00 .
drwxr-xr-x 3 root            root            4096 Jan 20 09:55 ..
-rw-r--r-- 1 cloudshell-user cloudshell-user   23 Jan 20 10:00 test.txt
```

#### Instalar Herramientas Adicionales
```bash
# Instalar paquetes Python
[cloudshell-user@ip-10-0-123-45 ~]$ pip3 install boto3 pandas

# Instalar paquetes Node.js
[cloudshell-user@ip-10-0-123-45 ~]$ npm install @aws-sdk/client-s3

# Clonar repositorio Git
[cloudshell-user@ip-10-0-123-45 ~]$ git clone https://github.com/usuario/repo.git
```

### 6.4 Subir/Descargar Archivos

#### Desde la Interfaz
```bash
# Subir archivo
1. Click: "Actions" > "Upload file"
2. Seleccionar archivo local
3. Archivo se sube a directorio actual

# Descargar archivo
1. Click: "Actions" > "Download file"
2. Ingresar ruta: /home/cloudshell-user/test.txt
3. Archivo se descarga al navegador
```

#### Usando AWS CLI
```bash
# Subir desde local a S3 (vía CloudShell)
[cloudshell-user@ip-10-0-123-45 ~]$ aws s3 cp s3://mi-bucket/archivo.txt ./

# Procesar y volver a subir
[cloudshell-user@ip-10-0-123-45 ~]$ sed 's/foo/bar/g' archivo.txt > modificado.txt
[cloudshell-user@ip-10-0-123-45 ~]$ aws s3 cp modificado.txt s3://mi-bucket/
```

### 6.5 Script de Python en CloudShell
```python
# Crear script
[cloudshell-user@ip-10-0-123-45 ~]$ cat > listar_ec2.py << 'EOF'
import boto3

# Crear cliente EC2
ec2 = boto3.client('ec2')

# Obtener todas las instancias
response = ec2.describe_instances()

# Iterar y mostrar
print("Instancias EC2:")
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        instance_id = instance['InstanceId']
        state = instance['State']['Name']
        instance_type = instance['InstanceType']
        
        # Obtener nombre de tags
        name = 'Sin nombre'
        if 'Tags' in instance:
            for tag in instance['Tags']:
                if tag['Key'] == 'Name':
                    name = tag['Value']
        
        print(f"  {instance_id}: {name} ({instance_type}) - {state}")
EOF

# Ejecutar script
[cloudshell-user@ip-10-0-123-45 ~]$ python3 listar_ec2.py
```

### 6.6 Persistencia de Datos
```bash
# El directorio /home persiste entre sesiones
/home/cloudshell-user/  ← PERSISTENTE (1 GB)

# Otros directorios NO persisten
/tmp/                   ← SE BORRA al cerrar sesión
/var/                   ← SE BORRA al cerrar sesión

# ⚠️ IMPORTANTE: Guardar archivos importantes en:
# - /home/cloudshell-user/
# - S3
# - Repositorio Git
```

### 6.7 CloudShell vs CLI Local

| Aspecto | CloudShell | CLI Local |
|---------|------------|-----------|
| **Configuración** | Ninguna | Instalar + configurar |
| **Credenciales** | Automáticas | Manual (access keys) |
| **Persistencia** | 1 GB limitado | Ilimitado (disco local) |
| **Disponibilidad** | Solo con internet + consola | Siempre disponible |
| **Herramientas** | Preinstaladas | Manual |
| **Costo** | Gratuito | Gratuito |

---

## 7. Roles de IAM

### 7.1 ¿Qué son los Roles de IAM?

Los **roles de IAM** son identidades de AWS con permisos específicos que pueden ser **asumidos** por servicios o usuarios cuando los necesitan.
```
Roles ≠ Usuarios

Usuario: Identidad PERMANENTE para personas
Rol: Identidad TEMPORAL para servicios/aplicaciones
```

### 7.2 ¿Cuándo Usar Roles?

#### Casos de Uso Principales
```mermaid
graph TD
    A[Roles IAM] --> B[Instancias EC2]
    A --> C[Funciones Lambda]
    A --> D[Servicios AWS]
    A --> E[Acceso Cross-Account]
    A --> F[Aplicaciones on-premises]
```

1. **Servicios de AWS necesitan permisos**
   - EC2 necesita leer de S3
   - Lambda necesita escribir en DynamoDB
   - ECS necesita obtener imágenes de ECR

2. **Acceso entre cuentas**
   - Usuarios de Cuenta A acceden a recursos de Cuenta B

3. **Federación de identidades**
   - Usuarios de Google/Facebook acceden a recursos AWS

### 7.3 Anatomía de un Rol
```
Rol de IAM = Políticas de Confianza + Políticas de Permisos

┌─────────────────────────────────────────┐
│ POLÍTICA DE CONFIANZA                   │
│ ¿Quién puede asumir este rol?          │
│                                         │
│ • EC2                                   │
│ • Lambda                                │
│ • Usuario específico                   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ POLÍTICAS DE PERMISOS                   │
│ ¿Qué puede hacer quien asume el rol?   │
│                                         │
│ • Leer S3                              │
│ • Escribir DynamoDB                    │
│ • Invocar Lambda                       │
└─────────────────────────────────────────┘
```

### 7.4 Ejemplo: Rol para EC2

#### Escenario
```
Instancia EC2 → Necesita leer archivos de S3

Sin Rol (❌ MALO):
- Credenciales hardcodeadas en código
- Riesgo de seguridad
- Difícil de rotar

Con Rol (✅ BUENO):
- Rol asignado a instancia
- Credenciales temporales automáticas
- Sin código de credenciales
```

### 7.5 Práctica: Crear Rol para EC2

#### Paso 1: Crear el Rol
```bash
# En la Consola de AWS
1. Navega a: IAM > Roles
2. Click: "Create role"

# Tipo de entidad de confianza
3. Trusted entity type: "AWS service"
4. Use case: "EC2"
5. Click: "Next"
```

#### Paso 2: Agregar Permisos
```bash
# Adjuntar políticas
6. Buscar y seleccionar:
   ☑ AmazonS3ReadOnlyAccess
   (Permite solo lectura de S3)

7. Click: "Next"
```

#### Paso 3: Nombrar y Crear
```bash
# Detalles del rol
8. Role name: "EC2-S3-ReadOnly-Role"
9. Description: "Permite a instancias EC2 leer objetos de S3"
10. Tags (opcional):
    - Key: "Environment" | Value: "Production"
    - Key: "Purpose" | Value: "S3Access"
11. Click: "Create role"

# ✅ Rol creado exitosamente
```

### 7.6 Asignar Rol a Instancia EC2

#### Durante la Creación
```bash
# Al lanzar instancia EC2
1. En "Configure instance details"
2. IAM role: Seleccionar "EC2-S3-ReadOnly-Role"
3. Continuar con el lanzamiento
```

#### En Instancia Existente
```bash
# Para instancia ya creada
1. Navega a: EC2 > Instances
2. Seleccionar instancia
3. Actions > Security > Modify IAM role
4. IAM role: "EC2-S3-ReadOnly-Role"
5. Click: "Update IAM role"

# ⚠️ Nota: Cambio toma efecto inmediatamente
```

### 7.7 Verificar Rol desde EC2
```bash
# Conectar a instancia EC2 vía SSH
ssh -i mi-clave.pem ec2-user@ec2-ip-address

# Verificar credenciales temporales
[ec2-user@ip-172-31-0-1 ~]$ curl http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2-S3-ReadOnly-Role
{
  "Code" : "Success",
  "LastUpdated" : "2024-01-20T10:30:00Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIATEMP...",
  "SecretAccessKey" : "...",
  "Token" : "...",
  "Expiration" : "2024-01-20T16:45:00Z"
}

# Probar acceso a S3 (sin configurar aws configure)
[ec2-user@ip-172-31-0-1 ~]$ aws s3 ls
2024-01-15 10:30:00 mi-bucket-produccion
2024-02-20 14:45:00 mi-bucket-desarrollo

# ✅ Funciona sin credenciales locales
```

### 7.8 Ejemplo: Rol para Lambda

#### Crear Rol para Lambda
```bash
# Paso 1: Crear rol
1. IAM > Roles > Create role
2. Trusted entity: "AWS service"
3. Use case: "Lambda"
4. Next

# Paso 2: Permisos
5. Adjuntar políticas:
   ☑ AWSLambdaBasicExecutionRole (logs)
   ☑ AmazonDynamoDBFullAccess (DynamoDB)
6. Next

# Paso 3: Nombrar
7. Role name: "Lambda-DynamoDB-Role"
8. Create role
```

#### Política de Confianza (Trust Policy)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Esta política dice: **"Lambda puede asumir este rol"**

### 7.9 Rol para Acceso Cross-Account

#### Escenario
```
Cuenta A (123456789012)
└── Usuario: developer-A
    └── Necesita acceso a →

Cuenta B (987654321098)
└── Bucket S3: datos-compartidos
```

#### Implementación

**En Cuenta B (donde está el recurso):**
```bash
# Crear rol
1. IAM > Roles > Create role
2. Trusted entity: "AWS account"
3. Account ID: 123456789012 (Cuenta A)
4. Require MFA: ☑ (recomendado)
5. Next

# Permisos
6. Adjuntar política para acceso a S3
7. Next

# Nombrar
8. Role name: "CrossAccount-S3-Access"
9. Create role
```

**Política de Confianza:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

**En Cuenta A (quien asume el rol):**
```bash
# Usuario developer-A necesita permiso para asumir rol
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::987654321098:role/CrossAccount-S3-Access"
    }
  ]
}
```

**Asumir el rol desde CLI:**
```bash
# developer-A en Cuenta A
aws sts assume-role \
  --role-arn "arn:aws:iam::987654321098:role/CrossAccount-S3-Access" \
  --role-session-name "developer-A-session" \
  --serial-number "arn:aws:iam::123456789012:mfa/developer-A" \
  --token-code 123456

# Respuesta con credenciales temporales
{
    "Credentials": {
        "AccessKeyId": "ASIATEMP...",
        "SecretAccessKey": "...",
        "SessionToken": "...",
        "Expiration": "2024-01-20T18:00:00Z"
    }
}

# Configurar credenciales temporales
export AWS_ACCESS_KEY_ID="ASIATEMP..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."

# Ahora puede acceder a recursos de Cuenta B
aws s3 ls s3://datos-compartidos/
```

### 7.10 Mejores Prácticas con Roles
```
✅ Usar roles en lugar de credenciales hardcodeadas
✅ Principio de mínimo privilegio
✅ Roles temporales para accesos puntuales
✅ Requerir MFA para roles sensibles
✅ Auditar asunción de roles con CloudTrail
✅ Usar condiciones en políticas de confianza

❌ Credenciales en código
❌ Permisos excesivos
❌ Roles sin fecha de expiración para humanos
❌ Confiar en cuentas sin verificar
```

---

## 8. Herramientas de Seguridad de IAM

### 8.1 IAM Credentials Report

#### ¿Qué es?

Informe a **nivel de cuenta** que lista todos los usuarios y el estado de sus credenciales.

#### Información que Proporciona
```
Para cada usuario muestra:
- Nombre de usuario
- ARN
- Fecha de creación
- Contraseña habilitada (Sí/No)
- Última vez que se usó contraseña
- Rotación de contraseña requerida
- Claves de acceso activas
- Última rotación de claves de acceso
- MFA activo (Sí/No)
- Certificados activos
```

#### Práctica: Generar Credentials Report
```bash
# En la Consola de AWS
1. Navega a: IAM > Credential report
2. Click: "Download credential report"
3. Se descarga archivo CSV

# Abrir en Excel/LibreOffice para analizar
```

#### Ejemplo de Credentials Report
```csv
user,arn,user_creation_time,password_enabled,password_last_used,password_last_changed,password_next_rotation,mfa_active,access_key_1_active,access_key_1_last_rotated,access_key_1_last_used_date,access_key_2_active
<root_account>,arn:aws:iam::123456789012:root,2020-01-01T00:00:00+00:00,not_supported,2024-01-20T10:00:00+00:00,not_supported,not_supported,true,false,N/A,N/A,false
john-developer,arn:aws:iam::123456789012:user/john-developer,2023-06-15T08:30:00+00:00,true,2024-01-20T09:45:00+00:00,2023-12-01T00:00:00+00:00,2024-03-01T00:00:00+00:00,true,true,2023-06-15T08:30:00+00:00,2024-01-19T15:20:00+00:00,false
alice-admin,arn:aws:iam::123456789012:user/alice-admin,2022-03-10T14:00:00+00:00,true,2024-01-18T11:30:00+00:00,2023-10-15T00:00:00+00:00,2024-01-15T00:00:00+00:00,false,true,2023-08-01T00:00:00+00:00,2024-01-20T08:15:00+00:00,true
```

#### Análisis de Seguridad
```bash
# Identificar problemas:

⚠️ alice-admin:
- MFA: NO activado ← RIESGO ALTO
- Access Key 2: Activa ← Revisar si es necesaria
- Password rotation: Vencida ← Requiere cambio

✅ john-developer:
- MFA: Activado
- Password: Actualizada
- Access keys: Solo una activa, uso reciente

🚨 root:
- Password usado recientemente ← NO recomendado
- Debe usar solo para configuración inicial
```

### 8.2 IAM Access Advisor

#### ¿Qué es?

Herramienta a **nivel de usuario** que muestra:
- Servicios a los que tiene permisos
- Última vez que accedió a cada servicio

#### Objetivo

Identificar permisos no utilizados para aplicar el principio de mínimo privilegio.

#### Práctica: Usar Access Advisor
```bash
# En la Consola de AWS
1. Navega a: IAM > Users
2. Selecciona: john-developer
3. Tab: "Access Advisor"
4. Ver tabla de servicios
```

#### Ejemplo de Access Advisor

| Servicio | Permisos | Último Acceso | Acción |
|----------|----------|---------------|--------|
| Amazon S3 | ✅ Permitido | Hace 2 horas | ✅ Mantener |
| Amazon EC2 | ✅ Permitido | Hace 45 días | ⚠️ Revisar |
| Amazon RDS | ✅ Permitido | Nunca | 🗑️ **Eliminar** |
| Amazon DynamoDB | ✅ Permitido | Hace 1 día | ✅ Mantener |
| AWS Lambda | ✅ Permitido | Hace 90+ días | ⚠️ Revisar |

#### Optimización de Permisos
```bash
# Basado en Access Advisor:

1. Servicios sin uso (>90 días):
   → Crear política sin esos permisos
   → Adjuntar al usuario
   → Monitorear por 30 días
   → Si no hay quejas, dejar la nueva política

2. Servicios con uso esporádico:
   → Considerar acceso bajo demanda
   → Usuario solicita cuando necesita
   → Permisos temporales vía rol

3. Servicios con uso constante:
   → Mantener permisos
   → Monitorear regularmente
```

### 8.3 IAM Policy Simulator

#### ¿Qué es?

Herramienta para **testear políticas** antes de aplicarlas en producción.

#### Acceso
```
https://policysim.aws.amazon.com/
```

#### Práctica: Simular Política
```bash
# En IAM Policy Simulator
1. Acceder a: https://policysim.aws.amazon.com/
2. Select a User/Group/Role: john-developer
3. Select Service: S3
4. Select Actions:
   ☑ GetObject
   ☑ PutObject
   ☑ DeleteBucket
5. Simulation Settings:
   - Resource: arn:aws:s3:::mi-bucket/*
6. Click: "Run Simulation"

# Resultados:
✅ GetObject: allowed
✅ PutObject: allowed
❌ DeleteBucket: denied (explícitamente)
```

#### Ejemplo de Simulación
```json
// Política a testear
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
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}

// Simulación en Policy Simulator muestra:
// ✅ Puede leer objetos de mi-bucket
// ✅ Puede escribir objetos en mi-bucket
// ❌ NO puede eliminar ningún bucket
```

### 8.4 AWS IAM Access Analyzer

#### ¿Qué es?

Identifica recursos compartidos con entidades externas a tu organización.

#### Recursos Analizados

- Buckets S3
- Roles IAM
- Claves KMS
- Funciones Lambda
- Colas SQS
- Secrets Manager

#### Práctica: Activar Access Analyzer
```bash
# En la Consola de AWS
1. Navega a: IAM > Access analyzer
2. Click: "Create analyzer"
3. Analyzer name: "OrganizationAnalyzer"
4. Zone of trust: "Current account"
   O "Organization" (si usas AWS Organizations)
5. Tags (opcional):
   - Key: "Environment" | Value: "Production"
6. Click: "Create analyzer"

# ⏱️ Análisis inicial puede tomar varios minutos
```

#### Interpretación de Hallazgos
```bash
# Ejemplo de hallazgo:

🔍 Finding: S3 Bucket "datos-publicos"
   Status: Active
   Resource Type: AWS::S3::Bucket
   Resource: arn:aws:s3:::datos-publicos
   
   External Access:
   - Principal: *
   - Access Level: Read
   - Condition: None
   
   ⚠️ RIESGO: Bucket accesible públicamente
   
   Acciones Recomendadas:
   1. Revisar si acceso público es intencional
   2. Si no es necesario, eliminar política pública
   3. Usar bucket policies restrictivas
   4. Habilitar "Block Public Access"
```

### 8.5 Práctica Completa de Herramientas de Seguridad

#### Auditoría Mensual Recomendada
```bash
# Semana 1: Credentials Report
1. Descargar IAM Credentials Report
2. Identificar usuarios sin MFA
3. Identificar claves no rotadas (>90 días)
4. Identificar contraseñas no cambiadas
5. Contactar usuarios para remediar

# Semana 2: Access Advisor
1. Revisar cada usuario/rol
2. Identificar servicios sin uso
3. Crear políticas optimizadas
4. Implementar gradualmente

# Semana 3: Access Analyzer
1. Revisar findings activos
2. Analizar cada recurso compartido
3. Verificar intencionalidad
4. Remediar accesos no intencionales

# Semana 4: Policy Review
1. Revisar políticas custom
2. Simular en Policy Simulator
3. Identificar permisos excesivos
4. Refactorizar políticas
```

---

## 9. Mejores Prácticas de IAM

### 9.1 Principios Fundamentales
```
🔐 Seguridad ante Todo
📉 Mínimo Privilegio
🔄 Rotación Regular
📊 Auditoría Constante
🛡️ Defensa en Profundidad
```

### 9.2 Cuenta Root

#### ❌ NO Hacer
```
❌ Usar root para operaciones diarias
❌ Crear access keys para root
❌ Compartir credenciales de root
❌ Usar root sin MFA
❌ Permitir acceso programático a root
```

#### ✅ SÍ Hacer
✅ Solo usar para configuración inicial
✅ Activar MFA inmediatamente
✅ Crear alias de cuenta
✅ Almacenar credenciales en lugar seguro
✅ Configurar información deRetryClaude does not have the ability to run the code it generates yet.LContinuecontacto alternativa
✅ Habilitar notificaciones de facturación
✅ Crear usuarios IAM para operaciones diarias

#### Tareas que SOLO Root Puede Hacer
```
1. Cambiar configuración de la cuenta
   - Nombre de cuenta
   - Email
   - Contraseña root
   
2. Cerrar cuenta de AWS

3. Cambiar o cancelar plan AWS Support

4. Registrarse como vendedor en Reserved Instance Marketplace

5. Configurar MFA Delete en bucket S3

6. Editar/eliminar política de bucket S3 con VPC/VPC endpoint inválido

7. Registrarse en AWS GovCloud
```

### 9.3 Gestión de Usuarios

#### 1 Usuario Físico = 1 Usuario IAM
```
✅ CORRECTO:
Juan Pérez → juan.perez@empresa.com
María García → maria.garcia@empresa.com
Carlos López → carlos.lopez@empresa.com

❌ INCORRECTO:
Equipo de Dev → dev-team@empresa.com (compartida)
Cuenta Admin → admin@empresa.com (compartida)
```

#### Organización por Grupos
```
Empresa
│
├── Grupo: Administradores
│   ├── Política: AdministratorAccess
│   ├── juan.admin
│   └── maria.admin
│
├── Grupo: Desarrolladores
│   ├── Política: DevelopmentAccess
│   ├── carlos.dev
│   ├── ana.dev
│   └── luis.dev
│
├── Grupo: QA-Testers
│   ├── Política: QA-Access
│   ├── pedro.qa
│   └── sofia.qa
│
└── Grupo: Solo-Lectura
    ├── Política: ReadOnlyAccess
    ├── jose.analyst
    └── laura.analyst
```

### 9.4 Gestión de Contraseñas

#### Política de Contraseña Recomendada
```bash
Configuración en IAM > Account settings:

☑ Minimum password length: 14 caracteres
☑ Require at least one uppercase letter: Sí
☑ Require at least one lowercase letter: Sí
☑ Require at least one number: Sí
☑ Require at least one non-alphanumeric character: Sí
☑ Enable password expiration: 90 días
☑ Password expiration requires administrator reset: No
☑ Allow users to change their own password: Sí
☑ Prevent password reuse: 5 contraseñas recordadas
```

#### Ejemplo de Contraseña Fuerte
```
❌ DÉBIL:
- password123
- empresa2024
- Admin@123

✅ FUERTE:
- K8$mPq2#vL9@nX4z
- Tr0p1c@l-F0r3st!2024
- My$3cur3P@ssw0rd#AWS

Recomendación: Usar gestor de contraseñas
- 1Password
- LastPass
- Bitwarden
```

### 9.5 Autenticación Multifactor (MFA)
```
✅ OBLIGATORIO MFA para:
   - Usuario Root
   - Usuarios Administradores
   - Usuarios con acceso a producción
   - Usuarios con permisos de eliminación

✅ RECOMENDADO MFA para:
   - Todos los usuarios IAM
   - Acceso a consola
   - Operaciones críticas (vía Condition)
```

#### Implementar MFA Obligatorio
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllExceptListedIfNoMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:GetUser",
        "iam:ListMFADevices",
        "iam:ListVirtualMFADevices",
        "iam:ResyncMFADevice",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

### 9.6 Gestión de Claves de Acceso

#### Ciclo de Vida de Claves
```
Creación → Uso Activo → Rotación → Desactivación → Eliminación
    ↓          ↓           ↓            ↓              ↓
  Día 0    Días 1-89   Día 90      Día 91-97      Día 98
```

#### Script de Monitoreo (Python)
```python
import boto3
from datetime import datetime, timezone, timedelta

iam = boto3.client('iam')

# Obtener todos los usuarios
users = iam.list_users()['Users']

print("=== Auditoría de Claves de Acceso ===\n")

for user in users:
    username = user['UserName']
    
    # Obtener claves de acceso del usuario
    keys = iam.list_access_keys(UserName=username)['AccessKeyMetadata']
    
    for key in keys:
        key_id = key['AccessKeyId']
        created_date = key['CreateDate']
        status = key['Status']
        
        # Calcular días desde creación
        age = (datetime.now(timezone.utc) - created_date).days
        
        # Alertas
        alert = ""
        if age > 90:
            alert = "⚠️ REQUIERE ROTACIÓN"
        elif age > 60:
            alert = "⏰ Rotar pronto"
        
        print(f"Usuario: {username}")
        print(f"  Key ID: {key_id}")
        print(f"  Status: {status}")
        print(f"  Edad: {age} días {alert}")
        print()
```

#### Proceso de Rotación
```bash
# Paso 1: Crear nueva clave
aws iam create-access-key --user-name john-developer

# Output:
{
    "AccessKey": {
        "AccessKeyId": "AKIAI44QH8DHBEXAMPLE",
        "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
        "Status": "Active",
        "CreateDate": "2024-01-20T10:00:00+00:00"
    }
}

# Paso 2: Actualizar aplicaciones con nueva clave
# (Actualizar variables de entorno, archivos config, etc.)

# Paso 3: Probar que aplicaciones funcionan

# Paso 4: Desactivar clave antigua
aws iam update-access-key \
  --user-name john-developer \
  --access-key-id AKIAIOSFODNN7EXAMPLE \
  --status Inactive

# Paso 5: Monitorear por 7 días

# Paso 6: Si todo OK, eliminar clave antigua
aws iam delete-access-key \
  --user-name john-developer \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

### 9.7 Uso de Roles vs Usuarios
```
Preferir ROLES en lugar de USUARIOS para:

✅ Aplicaciones en EC2
✅ Funciones Lambda
✅ Contenedores ECS/EKS
✅ Servicios de AWS
✅ Acceso cross-account
✅ Federación de identidades

Usar USUARIOS solo para:
- Personas que acceden a consola
- Desarrolladores que usan CLI/SDK localmente
```

### 9.8 Políticas: Menos es Más

#### Estrategia de Políticas
```
1. Empezar con Políticas AWS Managed
   ✅ Mantenidas por AWS
   ✅ Actualizadas automáticamente
   ✅ Probadas y seguras

2. Personalizar solo cuando sea necesario
   ✅ Customer Managed Policies
   ✅ Reutilizables
   ✅ Versionadas

3. Evitar Inline Policies
   ❌ No reutilizables
   ❌ Difíciles de auditar
   ❌ Se eliminan con el usuario/rol
```

#### Ejemplo de Refinamiento
```json
// ❌ Política demasiado permisiva
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}

// ⚠️ Mejor, pero aún amplia
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}

// ✅ ÓPTIMO: Mínimo privilegio
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListSpecificBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::mi-app-bucket"
    },
    {
      "Sid": "ReadWriteObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::mi-app-bucket/datos/*"
    }
  ]
}
```

### 9.9 Etiquetado (Tagging)

#### Estrategia de Tags
```bash
# Tags recomendados para recursos IAM

Usuario IAM:
- Department: Engineering
- CostCenter: CC-1234
- Environment: Production
- Owner: juan.perez@empresa.com
- Project: MigracionCloud

Rol IAM:
- Purpose: EC2-S3Access
- Environment: Production
- ManagedBy: Terraform
- Application: WebApp
```

#### Uso de Tags en Políticas
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowStartStopIfMatchingTags",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Owner": "${aws:username}",
          "ec2:ResourceTag/Environment": "Development"
        }
      }
    }
  ]
}
```

### 9.10 Auditoría y Monitoreo

#### Configurar CloudTrail
```bash
# CloudTrail registra TODAS las llamadas API

1. Navega a: CloudTrail > Trails
2. Click: "Create trail"
3. Trail name: "management-events"
4. Storage location: Crear nuevo bucket S3
5. Log file SSE-KMS encryption: Habilitado
6. Log file validation: ☑ Enabled
7. CloudWatch Logs: ☑ Enabled
8. Management events: ☑ Read/Write
9. Click: "Create trail"

# ✅ Ahora todas las acciones IAM se registran
```

#### Alertas Críticas con CloudWatch
```bash
# Crear alarma para cambios en IAM

1. CloudWatch > Log groups
2. Seleccionar grupo de CloudTrail
3. Actions > Create metric filter

# Patrón de filtro:
{
  ($.eventName = DeleteGroupPolicy) ||
  ($.eventName = DeleteRolePolicy) ||
  ($.eventName = DeleteUserPolicy) ||
  ($.eventName = PutGroupPolicy) ||
  ($.eventName = PutRolePolicy) ||
  ($.eventName = PutUserPolicy) ||
  ($.eventName = CreatePolicy) ||
  ($.eventName = DeletePolicy) ||
  ($.eventName = CreatePolicyVersion) ||
  ($.eventName = DeletePolicyVersion) ||
  ($.eventName = AttachRolePolicy) ||
  ($.eventName = DetachRolePolicy) ||
  ($.eventName = AttachUserPolicy) ||
  ($.eventName = DetachUserPolicy) ||
  ($.eventName = AttachGroupPolicy) ||
  ($.eventName = DetachGroupPolicy)
}

4. Metric name: IAM-Policy-Changes
5. Create alarm cuando métrica > 0
6. SNS topic: security-alerts
7. Email: security-team@empresa.com
```

### 9.11 Checklist de Mejores Prácticas
```markdown
## Checklist IAM Security

### Cuenta Root
- [ ] MFA activado
- [ ] Sin access keys
- [ ] Solo usada para tareas que requieren root
- [ ] Email y teléfono alternativos configurados

### Usuarios
- [ ] Un usuario por persona física
- [ ] Todos con MFA habilitado
- [ ] Organizados en grupos
- [ ] Sin políticas inline
- [ ] Access keys rotadas < 90 días
- [ ] Contraseñas fuertes y rotadas

### Grupos
- [ ] Nombres descriptivos
- [ ] Políticas AWS Managed cuando sea posible
- [ ] Documentados en wiki/confluence

### Roles
- [ ] Usados para servicios AWS
- [ ] Usados para aplicaciones
- [ ] Trust policies restrictivas
- [ ] Políticas de mínimo privilegio

### Políticas
- [ ] Principio de mínimo privilegio
- [ ] Deny explícitos cuando sea necesario
- [ ] Condiciones para MFA en operaciones críticas
- [ ] Versionadas y documentadas
- [ ] Revisadas trimestralmente

### Monitoreo
- [ ] CloudTrail habilitado
- [ ] Alarmas para cambios en IAM
- [ ] Credentials Report mensual
- [ ] Access Advisor revisado
- [ ] Access Analyzer activo

### Documentación
- [ ] Diagrama de grupos y roles
- [ ] Procedimiento de onboarding
- [ ] Procedimiento de offboarding
- [ ] Plan de respuesta a incidentes
```

---

## 10. Modelo de Responsabilidad Compartida

### 10.1 Concepto General
```
┌─────────────────────────────────────────────┐
│          RESPONSABILIDAD COMPARTIDA         │
├─────────────────────────────────────────────┤
│                                             │
│  AWS                         CLIENTE       │
│  ────                        ───────       │
│  Seguridad DEL Cloud        EN el Cloud    │
│                                             │
└─────────────────────────────────────────────┘
```

### 10.2 Responsabilidades de AWS
```
🔧 AWS es Responsable de:

1. Infraestructura
   - Hardware físico
   - Redes globales
   - Data centers
   - Instalaciones

2. Software de Infraestructura
   - Hypervisor
   - Networking
   - Storage

3. Seguridad de la Infraestructura
   - Protección física
   - Redundancia
   - Disponibilidad
   - Mantenimiento

4. Análisis de Vulnerabilidades
   - Auditorías internas
   - Certificaciones (ISO, SOC, PCI-DSS)
   - Validación de conformidad

5. Servicios Gestionados
   - RDS, DynamoDB, S3, Lambda
   - Patching del sistema operativo
   - Configuración base
```

### 10.3 Responsabilidades del Cliente (TÚ)
```
👤 TÚ eres Responsable de:

1. Gestión de Identidades
   ✅ Crear usuarios IAM
   ✅ Asignar permisos apropiados
   ✅ Gestionar grupos
   ✅ Configurar roles

2. Autenticación y Autorización
   ✅ Habilitar MFA
   ✅ Configurar políticas de contraseña
   ✅ Definir políticas IAM
   ✅ Gestionar access keys

3. Rotación de Credenciales
   ✅ Rotar contraseñas regularmente
   ✅ Rotar access keys
   ✅ Eliminar credenciales no usadas
   ✅ Deshabilitar usuarios inactivos

4. Monitoreo y Auditoría
   ✅ Habilitar CloudTrail
   ✅ Revisar Credentials Report
   ✅ Usar Access Advisor
   ✅ Configurar alertas

5. Principio de Mínimo Privilegio
   ✅ No dar permisos excesivos
   ✅ Revisar permisos regularmente
   ✅ Eliminar permisos no usados
   ✅ Usar roles cuando sea posible

6. Seguridad de Aplicaciones
   ✅ No hardcodear credenciales
   ✅ Usar roles para EC2/Lambda
   ✅ Cifrar datos sensibles
   ✅ Validar inputs

7. Configuración de Servicios
   ✅ Configurar encryption
   ✅ Configurar backups
   ✅ Configurar networking
   ✅ Configurar logging
```

### 10.4 Matriz de Responsabilidades

| Área | AWS | Cliente |
|------|-----|---------|
| **Data Centers** | ✅ Construir y mantener | ❌ |
| **Redes Globales** | ✅ Operar y mantener | ❌ |
| **Hardware** | ✅ Proveer y mantener | ❌ |
| **Hypervisor** | ✅ Gestionar | ❌ |
| **Usuarios IAM** | ❌ | ✅ Crear y gestionar |
| **Grupos IAM** | ❌ | ✅ Organizar |
| **Políticas IAM** | ⚠️ Proveer gestionadas | ✅ Crear y aplicar |
| **MFA** | ⚠️ Proveer mecanismo | ✅ Habilitar y usar |
| **Contraseñas** | ⚠️ Proveer políticas | ✅ Crear y gestionar |
| **Access Keys** | ⚠️ Proveer mecanismo | ✅ Crear y rotar |
| **Roles** | ⚠️ Proveer funcionalidad | ✅ Crear y asignar |
| **Auditoría** | ⚠️ Proveer herramientas | ✅ Ejecutar y revisar |
| **Certificaciones** | ✅ Mantener (ISO, SOC) | ⚠️ Usar apropiadamente |

⚠️ = Responsabilidad compartida

### 10.5 Ejemplo Práctico: S3
```
Bucket S3: "datos-empresa"

┌─────────────────────────────────────────────┐
│ AWS Responsable:                            │
├─────────────────────────────────────────────┤
│ ✅ Hardware de almacenamiento               │
│ ✅ Replicación entre AZs                    │
│ ✅ Durabilidad (11 nines)                   │
│ ✅ Disponibilidad del servicio              │
│ ✅ Protección física de data centers        │
│ ✅ Networking entre regiones                │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ CLIENTE (TÚ) Responsable:                   │
├─────────────────────────────────────────────┤
│ ✅ Configuración de bucket policy           │
│ ✅ Habilitar/deshabilitar acceso público    │
│ ✅ Configurar encriptación                  │
│ ✅ Configurar versionado                    │
│ ✅ Gestionar permisos IAM para acceso       │
│ ✅ Configurar logging                       │
│ ✅ Configurar lifecycle policies            │
│ ✅ Clasificar y proteger datos              │
└─────────────────────────────────────────────┘
```

### 10.6 Ejemplo Práctico: EC2
```
Instancia EC2: "web-server"

┌─────────────────────────────────────────────┐
│ AWS Responsable:                            │
├─────────────────────────────────────────────┤
│ ✅ Hardware físico (CPU, RAM, disco)        │
│ ✅ Hypervisor                               │
│ ✅ Aislamiento entre instancias             │
│ ✅ Red física                               │
│ ✅ Sustitución de hardware defectuoso       │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ CLIENTE (TÚ) Responsable:                   │
├─────────────────────────────────────────────┤
│ ✅ Sistema operativo (patching, updates)    │
│ ✅ Aplicaciones instaladas                  │
│ ✅ Security Groups                          │
│ ✅ Network ACLs                             │
│ ✅ Roles IAM para la instancia              │
│ ✅ Datos almacenados                        │
│ ✅ Encriptación de disco (EBS)              │
│ ✅ Backups/snapshots                        │
│ ✅ Firewall del SO                          │
│ ✅ Antivirus/antimalware                    │
└─────────────────────────────────────────────┘
```

### 10.7 Responsabilidades por Tipo de Servicio

#### IaaS (EC2)
```
Cliente tiene MÁS responsabilidad

AWS: Hardware, hypervisor, red física
Tú: SO, apps, datos, networking, seguridad
```

#### PaaS (RDS, Elastic Beanstalk)
```
Responsabilidad COMPARTIDA equilibrada

AWS: Hardware, SO, patching, backups automáticos
Tú: Datos, usuarios DB, networking, security groups
```

#### SaaS (S3, DynamoDB)
```
AWS tiene MÁS responsabilidad

AWS: Todo excepto tus datos y configuración
Tú: Datos, permisos, configuración de servicio
```

---

## 11. Resumen

### 11.1 Conceptos Clave de IAM
```markdown
## IAM - Identity and Access Management

### Definición
Servicio GLOBAL de AWS para gestionar identidades y permisos

### Componentes Principales
1. **Usuarios**: Identidades para personas o aplicaciones
2. **Grupos**: Colecciones de usuarios
3. **Políticas**: Documentos JSON con permisos
4. **Roles**: Identidades temporales para servicios

### Características
- ✅ Gratuito
- ✅ Global (no regional)
- ✅ Control granular
- ✅ Integrado con todos los servicios AWS
```

### 11.2 Puntos Críticos para el Examen
```
🎯 MEMORIZAR PARA EL EXAMEN:

1. IAM es GLOBAL, no regional

2. Usuario root:
   - Crear y nunca usar
   - Siempre habilitar MFA
   - Solo para tareas administrativas específicas

3. Principio de mínimo privilegio:
   - Dar SOLO permisos necesarios
   - Revisar y ajustar regularmente

4. Políticas IAM:
   - Formato JSON
   - Effect: Allow o Deny
   - Deny SIEMPRE gana

5. Grupos:
   - NO pueden contener otros grupos
   - Usuarios pueden estar en múltiples grupos

6. Roles:
   - Para servicios AWS
   - Credenciales temporales
   - NO para personas (usar usuarios)

7. MFA:
   - Contraseña + dispositivo
   - Crítico para seguridad
   - Opciones: app, hardware, U2F

8. Access Keys:
   - Para acceso programático (CLI/SDK)
   - Access Key ID + Secret Access Key
   - Rotar regularmente

9. CloudShell:
   - Terminal en navegador
   - Preautenticado
   - NO disponible en todas las regiones

10. Herramientas de auditoría:
    - Credentials Report (nivel cuenta)
    - Access Advisor (nivel usuario)
    - Access Analyzer (recursos externos)
```

### 11.3 Comandos AWS CLI Esenciales
```bash
# Identidad
aws sts get-caller-identity

# IAM Usuarios
aws iam list-users
aws iam create-user --user-name nuevo-usuario
aws iam delete-user --user-name usuario-antiguo

# IAM Grupos
aws iam list-groups
aws iam create-group --group-name nuevo-grupo
aws iam add-user-to-group --user-name usuario --group-name grupo

# IAM Políticas
aws iam list-policies --scope Local
aws iam create-policy --policy-name MiPolitica --policy-document file://policy.json
aws iam attach-user-policy --user-name usuario --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# IAM Roles
aws iam list-roles
aws iam create-role --role-name MiRol --assume-role-policy-document file://trust-policy.json

# Access Keys
aws iam list-access-keys --user-name usuario
aws iam create-access-key --user-name usuario
aws iam delete-access-key --user-name usuario --access-key-id AKIAIOSFODNN7EXAMPLE

# MFA
aws iam list-mfa-devices --user-name usuario
aws iam enable-mfa-device --user-name usuario --serial-number arn:aws:iam::123456789012:mfa/usuario --authentication-code-1 123456 --authentication-code-2 789012
```

### 11.4 Políticas de Ejemplo Importantes

#### Solo Lectura Global
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "*/Describe*",
        "*/List*",
        "*/Get*"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Requerir MFA
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

#### Limitar por IP
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": [
            "203.0.113.0/24",
            "198.51.100.0/24"
          ]
        }
      }
    }
  ]
}
```

### 11.5 Flujo de Decisión de Políticas
┌──────────────────────────────────────┐
│ Solicitud llega a AWS                │
└────────────┬─────────────────────────┘
│
▼
┌──────────────────────────────────────┐
│ ¿Hay un DENY explícito?              │
├──────────────────────────────────────┤
│ Sí → DENEGAR ACCESO ❌               │
│ No → Continuar ↓                     │
└────────────┬─────────────────────────┘
│
▼
┌──────────────────────────────────────┐
│ ¿Hay un ALLOW explícito?             │
├──────────────────────────────────────┤
│ Sí → PERMITIR ACCESO ✅              │
│ No → DENEGAR ACCESO ❌               │
│      (deny implícito por defecto)    │
└──────────────────────────────────────┘
REGLA DE ORO: DENY SIEMPRE GANA

---

## 12. Cuestionario de Evaluación

### Instrucciones
- Selecciona la(s) respuesta(s) correcta(s)
- Algunas preguntas tienen múltiples respuestas correctas
- Revisa tus respuestas al final

---

### Pregunta 1
**¿Cuál es la definición correcta de los roles IAM?**

A) Identidades permanentes para usuarios finales que necesitan acceder a AWS

B) Identidades temporales que pueden ser asumidas por servicios o usuarios cuando necesitan permisos

C) Grupos de usuarios organizados por departamento

D) Políticas JSON que definen permisos

<details>
<summary>Ver respuesta</summary>

**Respuesta Correcta: B**

Explicación: Los roles IAM son identidades que pueden ser asumidas temporalmente por servicios AWS, aplicaciones o usuarios cuando necesitan permisos específicos. A diferencia de los usuarios, los roles no tienen credenciales permanentes.

</details>

---

### Pregunta 2
**¿Cuál de las siguientes es una herramienta de seguridad IAM?**

A) CloudWatch Metrics

B) IAM Access Advisor

C) EC2 Instance Connect

D) S3 Transfer Acceleration

<details>
<summary>Ver respuesta</summary>

**Respuesta Correcta: B**

Explicación: IAM Access Advisor es una herramienta de seguridad que muestra los servicios a los que un usuario tiene permisos y cuándo fue la última vez que accedió a ellos, ayudando a aplicar el principio de mínimo privilegio.

</details>

---

### Pregunta 3
**¿Qué respuesta es INCORRECTA respecto a los usuarios de IAM?**

A) Los usuarios IAM pueden pertenecer a múltiples grupos

B) Los usuarios IAM deben tener necesariamente acceso a la consola de AWS

C) Cada usuario IAM debe representar a una persona física individual

D) Los usuarios IAM pueden tener políticas adjuntas directamente

<details>
<summary>Ver respuesta</summary>

**Respuesta Correcta: B**

Explicación: Los usuarios IAM NO necesitan obligatoriamente acceso a la consola. Pueden tener solo acceso programático (CLI/SDK) mediante access keys, sin necesidad de contraseña para la consola.

</details>

---

### Pregunta 4
**¿Cuál de las siguientes es una buena práctica de IAM?**

A) Usar la cuenta root para operaciones diarias

B) Crear access keys para la cuenta root

C) Habilitar MFA para todos losRetryClaude does not have the ability to run the code it generates yet.LContinueusuarios con acceso a producción
D) Compartir credenciales entre miembros del mismo equipo
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: Habilitar MFA (Multi-Factor Authentication) para usuarios con acceso a producción es una práctica de seguridad crítica. Las opciones A, B y D son malas prácticas que comprometen la seguridad.
</details>

Pregunta 5
¿Qué son las políticas IAM?
A) Grupos de usuarios organizados por función
B) Documentos JSON que definen permisos y controlan el acceso a recursos AWS
C) Credenciales temporales para acceso programático
D) Dispositivos de autenticación multifactor
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: Las políticas IAM son documentos en formato JSON que especifican qué acciones están permitidas o denegadas en qué recursos de AWS. Son el mecanismo principal para definir permisos.
</details>

Pregunta 6
En el modelo de responsabilidad compartida, ¿de qué es responsable el cliente en IAM?
A) Seguridad física de los data centers donde se ejecuta IAM
B) Gestión y supervisión de usuarios, grupos, roles y políticas
C) Mantenimiento del hardware de los servidores de autenticación
D) Actualización del software base del servicio IAM
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: El cliente es responsable de gestionar y supervisar sus usuarios, grupos, roles y políticas IAM. AWS es responsable de la infraestructura física, el hardware y el software base del servicio.
</details>

Pregunta 7
¿Cuál de las siguientes afirmaciones es VERDADERA?
A) Los grupos IAM pueden contener otros grupos
B) Un usuario IAM puede pertenecer a un máximo de un grupo
C) Las políticas se pueden adjuntar a grupos para aplicar permisos a todos sus usuarios
D) Los grupos IAM pueden asumir roles
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: Las políticas adjuntas a un grupo se aplican automáticamente a todos los usuarios miembros de ese grupo, facilitando la gestión de permisos. Los grupos NO pueden contener otros grupos, un usuario puede estar en múltiples grupos, y los grupos no pueden asumir roles (solo usuarios y servicios pueden hacerlo).
</details>

Pregunta 8
¿Qué principio debes aplicar con respecto a los permisos de IAM?
A) Principio de máxima disponibilidad
B) Principio de mínimo privilegio (least privilege)
C) Principio de segregación de datos
D) Principio de acceso público por defecto
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: El principio de mínimo privilegio (least privilege) significa otorgar solo los permisos necesarios para realizar una tarea, nada más. Esto minimiza el riesgo de seguridad en caso de que una cuenta sea comprometida.
</details>

Pregunta 9
¿Qué debes hacer para aumentar la seguridad de tu cuenta root?
A) Crear múltiples access keys para tener respaldo
B) Compartir las credenciales con el equipo de administradores
C) Habilitar MFA (autenticación multifactor)
D) Usar la cuenta root para todas las operaciones diarias
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: Habilitar MFA en la cuenta root es crítico para la seguridad. Añade una capa adicional de protección más allá de la contraseña. La cuenta root NO debe tener access keys, NO debe compartirse, y NO debe usarse para operaciones diarias.
</details>

Pregunta 10
¿Qué componentes forman las claves de acceso de AWS? (SELECCIONA DOS)
A) Access Key ID
B) Username
C) Secret Access Key
D) Session Token
E) Password
<details>
<summary>Ver respuesta</summary>
Respuestas Correctas: A y C
Explicación: Las claves de acceso de AWS constan de dos partes: Access Key ID (similar a un nombre de usuario) y Secret Access Key (similar a una contraseña). Ambas son necesarias para el acceso programático vía CLI o SDK.
</details>

Pregunta 11
¿Cuál es el formato correcto de una política IAM?
A) XML
B) JSON
C) YAML
D) Plain Text
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: Las políticas IAM se escriben en formato JSON (JavaScript Object Notation). Tienen una estructura específica con elementos como Version, Statement, Effect, Action y Resource.
</details>

Pregunta 12
En una política IAM, ¿qué efecto tiene SIEMPRE prioridad?
A) Allow
B) Deny
C) El primero que aparezca en el documento
D) Depende del orden de las políticas adjuntas
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: En IAM, un Deny SIEMPRE tiene prioridad sobre un Allow, sin importar el orden. Esto es fundamental para la seguridad: si existe una denegación explícita, el acceso es denegado incluso si hay permisos que lo permitirían.
</details>

Pregunta 13
¿Para qué se utiliza AWS CloudShell?
A) Para gestionar instancias EC2 desde el navegador
B) Como terminal basada en navegador preautenticada con AWS CLI
C) Para crear scripts de automatización en Python
D) Para monitorear logs de aplicaciones
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: AWS CloudShell es una terminal basada en navegador que viene preautenticada con tus credenciales de AWS y preinstalada con AWS CLI, Python, Node.js y otras herramientas. No requiere configuración de credenciales.
</details>

Pregunta 14
¿Cuál de las siguientes NO es una opción válida de dispositivo MFA en AWS?
A) Google Authenticator (aplicación virtual)
B) YubiKey (hardware U2F)
C) SMS al teléfono móvil
D) Gemalto (hardware token)
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: AWS no soporta MFA por SMS. Las opciones válidas son: aplicaciones virtuales (Google Authenticator, Authy), llaves de seguridad U2F (YubiKey), y tokens de hardware (Gemalto, SurePassID).
</details>

Pregunta 15
¿Qué herramienta IAM te permite ver qué servicios ha accedido un usuario y cuándo?
A) IAM Credentials Report
B) IAM Access Analyzer
C) IAM Access Advisor
D) IAM Policy Simulator
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: IAM Access Advisor muestra los servicios a los que un usuario tiene permisos y la última vez que accedió a cada uno. Es útil para identificar permisos no utilizados y aplicar el principio de mínimo privilegio.
</details>

Pregunta 16
¿Cuál es el límite de tiempo máximo de sesión para credenciales temporales de un rol asumido?
A) 15 minutos
B) 1 hora
C) 12 horas
D) 36 horas
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: Las credenciales temporales de un rol asumido pueden durar desde 15 minutos hasta 12 horas (por defecto es 1 hora). Esto proporciona un balance entre seguridad y conveniencia operativa.
</details>

Pregunta 17
¿Qué archivo almacena las credenciales de AWS CLI en Linux/Mac?
A) ~/.aws/config
B) ~/.aws/credentials
C) /etc/aws/credentials
D) ~/.awscli/credentials
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: El archivo ~/.aws/credentials almacena las access keys (Access Key ID y Secret Access Key). El archivo ~/.aws/config almacena la configuración como región y formato de salida.
</details>

Pregunta 18
¿Cuál es la URL para acceder al IAM Policy Simulator?
A) https://simulator.aws.amazon.com
B) https://iam.aws.amazon.com/simulator
C) https://policysim.aws.amazon.com
D) https://console.aws.amazon.com/iam/simulator
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: El IAM Policy Simulator está disponible en https://policysim.aws.amazon.com y permite probar políticas IAM sin aplicarlas en producción.
</details>

Pregunta 19
¿Con qué frecuencia se recomienda rotar las access keys?
A) Cada 30 días
B) Cada 90 días
C) Cada 180 días
D) Una vez al año
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: La mejor práctica de AWS recomienda rotar las access keys cada 90 días. Esto minimiza el riesgo de exposición de credenciales comprometidas.
</details>

Pregunta 20
¿Qué servicio proporciona credenciales temporales para asumir roles?
A) AWS IAM
B) AWS STS (Security Token Service)
C) AWS KMS
D) AWS Secrets Manager
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: AWS STS (Security Token Service) es el servicio que proporciona credenciales de seguridad temporales de corta duración para usuarios que asumen roles IAM.
</details>

Pregunta 21 (Múltiple Respuesta)
¿Cuáles de las siguientes son mejores prácticas de IAM? (SELECCIONA TRES)
A) Usar la cuenta root solo para configuración inicial de la cuenta
B) Habilitar MFA en todas las cuentas
C) Compartir access keys entre desarrolladores del mismo equipo
D) Crear usuarios IAM individuales para cada persona
E) Almacenar credenciales en el código fuente para fácil acceso
<details>
<summary>Ver respuesta</summary>
Respuestas Correctas: A, B y D
Explicación:

A: Correcto, la cuenta root solo debe usarse para tareas administrativas específicas
B: Correcto, MFA añade una capa crítica de seguridad
C: Incorrecto, las credenciales nunca deben compartirse
D: Correcto, cada persona debe tener su propio usuario IAM
E: Incorrecto, nunca almacenar credenciales en código fuente

</details>

Pregunta 22
¿Qué elemento de una política IAM especifica si se permite o deniega una acción?
A) Action
B) Resource
C) Effect
D) Principal
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: C
Explicación: El elemento "Effect" en una política IAM especifica si la declaración resulta en "Allow" (permitir) o "Deny" (denegar). Es obligatorio en cada declaración de política.
</details>

Pregunta 23
¿Cuál de las siguientes tareas SOLO puede realizar el usuario root?
A) Crear usuarios IAM
B) Cerrar la cuenta de AWS
C) Lanzar instancias EC2
D) Crear buckets S3
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: Cerrar una cuenta de AWS es una de las pocas acciones que solo puede realizar el usuario root. Otras incluyen cambiar configuración de la cuenta, modificar plan de soporte, y registrarse en GovCloud.
</details>

Pregunta 24
¿Qué genera el IAM Credentials Report? (SELECCIONA DOS)
A) Estado de las contraseñas de todos los usuarios
B) Configuración de seguridad de instancias EC2
C) Estado de las claves de acceso de todos los usuarios
D) Logs de acceso a S3
<details>
<summary>Ver respuesta</summary>
Respuestas Correctas: A y C
Explicación: El IAM Credentials Report es un informe a nivel de cuenta que incluye información sobre contraseñas y access keys de todos los usuarios, incluyendo fechas de creación, último uso y estado de MFA.
</details>

Pregunta 25
¿Qué comando AWS CLI muestra tu identidad actual?
A) aws iam get-user
B) aws sts get-caller-identity
C) aws iam whoami
D) aws configure list
<details>
<summary>Ver respuesta</summary>
Respuesta Correcta: B
Explicación: El comando aws sts get-caller-identity muestra información sobre la identidad que estás usando actualmente, incluyendo UserId, Account y ARN.
</details>

📊 Tabla de Respuestas Rápidas
PreguntaRespuesta(s)PreguntaRespuesta(s)1B14C2B15C3B16C4C17B5B18C6B19B7C20B8B21A, B, D9C22C10A, C23B11B24A, C12B25B13B

📚 Recursos Adicionales
Documentación Oficial de AWS

IAM User Guide
IAM Best Practices
IAM Policy Reference

Tutoriales Prácticos

Getting Started with IAM
IAM Tutorial: Delegate Access

Herramientas

IAM Policy Simulator
IAM Policy Generator


🎓 Preparación para el Examen Cloud Practitioner
Temas IAM Clave para el Examen
markdown✅ Dominar completamente:
- Concepto de usuarios, grupos, roles y políticas
- Diferencias entre usuarios y roles
- Principio de mínimo privilegio
- MFA y su importancia
- Access keys vs contraseñas
- Modelo de responsabilidad compartida
- Mejores prácticas de seguridad

⚠️ Conocer bien:
- Estructura básica de políticas JSON
- Herramientas de auditoría (Credentials Report, Access Advisor)
- AWS CLI básico
- CloudShell y sus limitaciones
- Trust policies en roles

📖 Familiarizarse con:
- Policy evaluation logic
- Cross-account access
- Federación de identidades (concepto)
- IAM Access Analyzer
Tips para el Examen

Lee cuidadosamente: Identifica si preguntan por usuarios, grupos o roles
Principio de mínimo privilegio: Siempre elige la opción con menos permisos
MFA es crítico: Si hay una opción de seguridad con MFA, suele ser correcta
Root account: Nunca debe usarse para operaciones normales
Deny gana: Recuerda que Deny siempre tiene prioridad sobre Allow


🎯 Checklist Final
Antes de dar el examen, verifica que puedas:
markdown- [ ] Explicar qué es IAM y por qué es importante
- [ ] Crear y gestionar usuarios, grupos y roles
- [ ] Escribir políticas IAM básicas
- [ ] Configurar MFA en una cuenta
- [ ] Instalar y configurar AWS CLI
- [ ] Usar CloudShell
- [ ] Generar y gestionar access keys
- [ ] Crear roles para servicios EC2/Lambda
- [ ] Interpretar IAM Credentials Report
- [ ] Usar IAM Access Advisor
- [ ] Explicar el modelo de responsabilidad compartida
- [ ] Aplicar las mejores prácticas de IAM

📝 Notas Finales
Este material cubre exhaustivamente todo lo relacionado con IAM para el examen AWS Certified Cloud Practitioner. Practica los conceptos en tu propia cuenta de AWS (usa la capa gratuita) para reforzar el aprendizaje.
¡Buena suerte en tu examen! 🚀

Última actualización: 2024
Versión del curso: AWS Certified Cloud Practitioner CLF-C02RetryClaude does not have the ability to run the code it generates yet.