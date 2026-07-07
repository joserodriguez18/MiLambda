# Guía completa: crear y desplegar una AWS Lambda con .NET (con CI/CD)

De la terminal local a la arquitectura serverless automatizada.

---

## Índice

- [Fase 1: Preparación del entorno (local)](#fase-1-preparación-del-entorno-local)
- [Fase 2: Identidad y acceso (seguridad)](#fase-2-identidad-y-acceso-seguridad)
- [Fase 3: Del código a la nube (despliegue manual)](#fase-3-del-código-a-la-nube-despliegue-manual)
- [Fase 4: Automatización (CI/CD)](#fase-4-automatización-cicd)
- [Demostración en vivo (guion para la exposición)](#demostración-en-vivo-guion-para-la-exposición)
- [Glosario de conceptos clave](#glosario-de-conceptos-clave)
- [Checklist antes de exponer](#checklist-antes-de-exponer)

---

## Fase 1: Preparación del entorno (local)

### 1. Verificar la instalación de .NET

Antes de crear una Lambda, comprobamos que el SDK de .NET está instalado.

```bash
dotnet --version
```

**Resultado esperado:**
```
10.0.300
```

**Explicación:** este comando muestra la versión del SDK de .NET instalada en el equipo. Es necesario para crear, compilar y publicar proyectos escritos en C#.

---

### 2. Verificar las plantillas instaladas de AWS Lambda

.NET no incluye las plantillas de AWS Lambda por defecto, por lo que primero debemos comprobar si están disponibles.

```bash
dotnet new list
dotnet new list lambda
```

**Problema común:** si aparece el mensaje:
```
No se encontraron plantillas ni subcomandos que coincidan: "lambda.EmptyFunction"
```
significa que las plantillas de AWS Lambda no están instaladas todavía. Es normal, se resuelve en el siguiente paso.

---

### 3. Instalar las plantillas de AWS Lambda para .NET

```bash
dotnet new install Amazon.Lambda.Templates
dotnet new list lambda
```

**Resultado esperado:** debe aparecer una lista con plantillas como:
- AWS Lambda Empty Function
- AWS Lambda ASP.NET Core
- AWS Lambda S3 Function
- AWS Lambda DynamoDB Function

**Explicación:** las plantillas proporcionan la estructura inicial del proyecto Lambda, incluyendo archivos de configuración y un método preparado para ser ejecutado por AWS.

---

### 4. Crear un proyecto Lambda vacío

```bash
dotnet new lambda.EmptyFunction -n MiLambda
cd MiLambda
```

**Explicación:** este comando crea un proyecto C# preparado para AWS Lambda. La estructura generada contiene:

```
MiLambda/
├── Function.cs
├── MiLambda.csproj
├── aws-lambda-tools-defaults.json
└── README.md
```

| Archivo | Qué es |
|---|---|
| `Function.cs` | El handler: tu código C# preparado para ser ejecutado por AWS |
| `MiLambda.csproj` | Dependencias del proyecto |
| `aws-lambda-tools-defaults.json` | Configuración predeterminada para el despliegue |
| `README.md` | Documentación generada por la plantilla |

---

### 5. Instalar AWS Lambda Tools para .NET

Esta herramienta permite compilar y desplegar funciones Lambda desde la terminal, sin salir del entorno de C#.

```bash
dotnet tool install -g Amazon.Lambda.Tools
dotnet lambda --version
```

**Resultado esperado:**
```
Amazon Lambda Tools for .NET Core applications (7.0.0)
```

---

## Fase 2: Identidad y acceso (seguridad)

### 6. Configurar AWS CLI

AWS Lambda Tools necesita credenciales para comunicarse con AWS.

```bash
aws configure list
```

**Ejemplo de salida:**
```
profile:
access_key: ****************ZHNJ
region: us-east-1
```

**Explicación:** muestra qué credenciales y región está utilizando AWS CLI actualmente.

---

### 7. Configurar una cuenta AWS

Si necesitamos cambiar la cuenta utilizada:

```bash
aws configure
```

Solicita:
```
AWS Access Key ID: ****************
AWS Secret Access Key: ****************
Default region name: us-east-1
Default output format: json
```

---

### 8. Verificar qué cuenta AWS está utilizando la terminal

Antes de desplegar, confirmamos la identidad.

```bash
aws sts get-caller-identity
```

**Resultado esperado:**
```json
{
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/usuario"
}
```

**Explicación:** permite confirmar la cuenta AWS activa, el usuario IAM utilizado, y que las credenciales son válidas antes de "despegar".

---

### 9. Verificar la región AWS

```bash
aws configure list
```

La región debe coincidir exactamente con la región seleccionada en la consola de AWS.

**Ejemplo:** AWS Console → Lambda → Región: US East (N. Virginia) → `us-east-1`

---

## Fase 3: Del código a la nube (despliegue manual)

### 10. Compilar el proyecto Lambda

Antes del despliegue verificamos que el código compile.

```bash
dotnet build
```

**Explicación:** compila el código C# y verifica que no existan errores locales antes de intentar subir nada a AWS.

---

### 11. Desplegar la Lambda en AWS

```bash
dotnet lambda deploy-function
```

Durante el proceso pedirá el **nombre de la función**. Ejemplo: `cicd`. AWS creará una función con ese nombre exacto.

---

### 12. Proceso interno del despliegue

Cuando ejecutamos `dotnet lambda deploy-function`, ocurre lo siguiente por debajo:

```
Código C#
   │
   ▼
dotnet publish
   │
   ▼
Compilación para Linux
   │
   ▼
Creación de archivo ZIP
   │
   ▼
Creación de IAM Role
   │
   ▼
Subida del paquete a AWS
   │
   ▼
Creación de la función Lambda
```

---

### 13. Seleccionar permisos IAM

AWS solicita un rol para ejecutar la Lambda. Para una función básica, seleccionar:

```
AWSLambdaBasicExecutionRole
```

**¿Qué permisos agrega?**
- ✅ Permite crear logs y escribirlos en CloudWatch.
- ❌ No permite acceder a bases de datos u otros servicios hasta que agregues permisos adicionales.

---

### 14. Confirmación de creación

**Mensaje esperado:**
```
Waiting for new IAM Role to propagate to AWS regions ........ Done
New Lambda function created
```

Esto indica que la función Lambda fue creada correctamente.

---

### 15. Verificar que la Lambda existe desde AWS CLI

```bash
aws lambda list-functions --region us-east-1
```

Debe aparecer:
```json
{
  "FunctionName": "cicd"
}
```

---

### 16. Arquitectura final del flujo (Fases 1-3)

```
Desarrollador
   │
   ▼
Proyecto C# Lambda
   │
   ▼
AWS Lambda Tools
   │
   ▼
AWS CLI + Credenciales IAM
   │
   ▼
AWS Lambda
   │
   ▼
Ejecución del método FunctionHandler()
   │
   ▼
CloudWatch Logs
```

---

## Fase 4: Automatización (CI/CD)

Del despliegue manual al despliegue automático.

### 17. El problema: desplegar a mano no escala

| Antes: manual | Después: automático |
|---|---|
| Terminas el código en tu máquina | Terminas el código y haces `git push` |
| Abres la consola de AWS o la terminal | GitHub Actions se activa solo |
| Ejecutas `dotnet publish` a mano | Compila, restaura y compila el proyecto |
| Subes el ZIP manualmente a Lambda | Empaqueta y despliega a Lambda |
| Repites todo el proceso en cada cambio | Tú solo verificas el resultado |

**Explicación clave:** se le hace CI/CD **al código que corre dentro de la Lambda**, exactamente igual que a cualquier otra aplicación (una API, un microservicio). La única diferencia es dónde vive el código ya desplegado.

---

### 18. Crear el workflow de GitHub Actions

Archivo: `.github/workflows/deploy.yml`

```yaml
# Nombre del workflow que aparecerá en la pestaña "Actions" de GitHub
name: Deploy AWS Lambda

# Define cuándo se ejecutará automáticamente este pipeline
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:

      # PASO 1: Descargar el código del repositorio
      - name: Checkout code
        uses: actions/checkout@v4

      # PASO 2: Instalar el entorno .NET
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      # PASO 3: Restaurar paquetes NuGet
      - name: Restore packages
        run: dotnet restore

      # PASO 4: Compilar la aplicación (CI)
      - name: Build
        run: dotnet build --configuration Release

      # PASO 5: Instalar herramientas AWS Lambda para .NET
      - name: Install Lambda Tools
        run: dotnet tool install -g Amazon.Lambda.Tools

      # PASO 6: Desplegar la Lambda en AWS (CD)
      - name: Deploy Lambda
        run: dotnet lambda deploy-function cicd
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
```

---

### 19. El disparador: `on: push`

```yaml
on:
  push:
    branches:
      - main
```

**Explicación:** este bloque es la regla de activación. El pipeline **solo** se dispara cuando hay un push a la rama `main`. Un push a cualquier otra rama no activa nada. Todo lo que sigue ocurre automáticamente, sin que el desarrollador ejecute nada más después del push.

---

### 20. Fase CI: verificación de integridad

Estos cuatro pasos corren en orden. **Si uno falla, el pipeline se detiene y Lambda no se toca.**

1. **Checkout code** — clona el repositorio dentro de la máquina temporal de GitHub. Sin esto, el servidor no tiene el código.
2. **Setup .NET** — instala el SDK de .NET en el runner. La versión debe coincidir con el runtime de la Lambda.
3. **Restore packages** — descarga las dependencias NuGet del proyecto, incluyendo el SDK de AWS Lambda.
4. **Build** — compila el proyecto. Si hay errores, el pipeline se detiene aquí y Lambda nunca se actualiza.

> Esto ES la Integración Continua: verificar antes de tocar producción.

---

### 21. Fase CD: empaquetar y desplegar

**Paso 1 — Install Lambda Tools**
```bash
dotnet tool install -g Amazon.Lambda.Tools
```
Instala la CLI que sabe empaquetar y subir funciones .NET a AWS Lambda.

**Paso 2 — Deploy Lambda**
```bash
dotnet lambda deploy-function cicd
```
Compila, empaqueta en un ZIP y actualiza la función Lambda llamada `cicd` en AWS. Aquí ocurre el CD (Continuous Deployment).

> ⚠️ El nombre `cicd` debe ser **exactamente** el nombre de la función ya creada en AWS, o el despliegue falla con un error de "función no encontrada".

---

### 22. Seguridad: credenciales como GitHub Secrets

GitHub Actions necesita credenciales para hablar con AWS. **Nunca se escriben directamente en el código.**

Se configuran en: `Settings → Secrets and variables → Actions` del repositorio.

| Secret | Qué es |
|---|---|
| `AWS_ACCESS_KEY_ID` | Identificador de acceso de AWS |
| `AWS_SECRET_ACCESS_KEY` | Contraseña criptográfica de AWS |
| `AWS_REGION` | Ej. `us-east-1`. Debe coincidir con la región de la Lambda |

**Uso en el workflow:**
```yaml
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

> ⚠️ **No olvides el IAM:** el usuario de AWS detrás de estas credenciales necesita, como mínimo, permisos de `lambda:UpdateFunctionCode` y `lambda:GetFunction`. Sin ellos, el despliegue falla por permisos aunque el código sea perfecto.

---

### 23. El flujo completo de CI/CD

```
git push a main
   │
   ▼
GitHub Actions inicia
   │
   ▼
CI: checkout, restore, build
   │
   ▼
CD: install lambda tools + deploy
   │
   ▼
AWS Lambda actualizada
   │
   ▼
Probada en consola AWS o vía URL
```

> Si se entiende este dibujo, se entiende toda la arquitectura: un cambio de código se convierte, **sin intervención humana**, en una función Lambda actualizada.

---

## Demostración en vivo (guion para la exposición)

1. Cambiar el mensaje de respuesta en `Function.cs` (ejemplo: `"Hola Academia"` → `"Hola Devs!"`)
2. `git add .` → `git commit -m "demo"` → `git push`
3. Abrir la pestaña **Actions** de GitHub (`https://github.com/<usuario>/<repo>/actions`) y mostrar el pipeline corriendo en vivo
4. Cuando termine, ir a la consola de AWS Lambda → pestaña **Probar** → invocar la función
5. Confirmar que la respuesta ya dice `"Hola Devs!"` — **sin que se haya hecho el deploy a mano**

**Cierre sugerido:**
> "Cada `git push` ahora compila, prueba y despliega mi Lambda en AWS de forma automática. Eso es CI/CD aplicado a una función serverless."

---

## Glosario de conceptos clave

| Concepto | Explicación |
|---|---|
| **AWS Lambda** | Servicio serverless que ejecuta código sin administrar servidores |
| **IAM Role** | Permisos que determinan qué puede hacer una Lambda |
| **Handler** | Método principal que AWS ejecuta cuando llega un evento |
| **CloudWatch** | Servicio donde Lambda almacena registros (logs) de ejecución |
| **AWS CLI** | Herramienta que permite administrar AWS desde la terminal |
| **Lambda Tools** | Extensión de .NET que facilita crear y desplegar Lambdas desde C# |
| **CI (Continuous Integration)** | Compilar, restaurar y verificar el código automáticamente en cada push |
| **CD (Continuous Deployment)** | Empaquetar y subir automáticamente el resultado a producción tras pasar el CI |
| **GitHub Actions** | El motor de automatización que ejecuta el workflow |
| **GitHub Secrets** | Almacenamiento seguro de credenciales, usado en vez de escribirlas en el código |

---

## Checklist antes de exponer

- [ ] `dotnet --version` funciona y coincide con la versión del workflow
- [ ] La Lambda ya existe en AWS con el nombre exacto usado en `deploy-function`
- [ ] Los tres GitHub Secrets están configurados (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`)
- [ ] El usuario IAM tiene permisos de `lambda:UpdateFunctionCode` y `lambda:GetFunction`
- [ ] Ya corriste el pipeline al menos una vez con éxito (no lo pruebes por primera vez en vivo)
- [ ] Tienes la pestaña **Actions** de GitHub y la consola de AWS Lambda abiertas y listas para mostrar
- [ ] Sabes qué línea vas a cambiar en `Function.cs` para la demo
