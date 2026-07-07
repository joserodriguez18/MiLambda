# Proyecto de Función Vacía de AWS Lambda

Este proyecto inicial consta de:

- **Function.cs**: archivo de clase que contiene una clase con un único método controlador (handler) de la función.
- **aws-lambda-tools-defaults.json**: configuración predeterminada de argumentos para utilizar con Visual Studio y las herramientas de implementación desde la línea de comandos de AWS.

También puedes tener un proyecto de pruebas, dependiendo de las opciones que hayas seleccionado.

El controlador de función generado es un método sencillo que recibe un argumento de tipo `string` y devuelve su equivalente en mayúsculas. Sustituye el cuerpo de este método, así como sus parámetros, según las necesidades de tu aplicación.

## Pasos para comenzar desde Visual Studio

Para implementar tu función en AWS Lambda:

1. Haz clic derecho sobre el proyecto en **Solution Explorer**.
2. Selecciona **Publish to AWS Lambda**.

Para ver la función implementada:

1. Abre **AWS Explorer**.
2. Dentro del nodo **AWS Lambda**, haz doble clic sobre el nombre de la función para abrir su ventana **Function View**.

Para probar la función implementada:

- Utiliza la pestaña **Test Invoke** en la ventana **Function View**.

Para configurar los orígenes de eventos que invocarán la función (por ejemplo, cuando se cree un objeto en un bucket de Amazon S3):

- Utiliza la pestaña **Event Sources** de la ventana **Function View**.

Para actualizar la configuración del entorno de ejecución (runtime) de la función:

- Utiliza la pestaña **Configuration**.

Para consultar los registros de ejecución (logs) de las invocaciones:

- Utiliza la pestaña **Logs**.

## Pasos para comenzar desde la línea de comandos

Una vez que hayas editado la plantilla y el código, puedes implementar tu aplicación utilizando la herramienta global **Amazon.Lambda.Tools** para .NET.

Más información:
https://github.com/aws/aws-extensions-for-dotnet-cli#aws-lambda-amazonlambdatools

### Instalar Amazon.Lambda.Tools (si aún no está instalada)

```bash
dotnet tool install -g Amazon.Lambda.Tools
```

### Actualizar la herramienta a la versión más reciente

```bash
dotnet tool update -g Amazon.Lambda.Tools
```

### Ejecutar las pruebas unitarias

```bash
cd "MiLambda/test/MiLambda.Tests"
dotnet test
```

### Implementar la función en AWS Lambda

```bash
cd "MiLambda/src/MiLambda"
dotnet lambda deploy-function
```