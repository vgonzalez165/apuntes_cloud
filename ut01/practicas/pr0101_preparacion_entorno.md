```
----------------- ADMINISTRACIÓN DE SISTEMAS INFORMATICOS EN RED ----------------
---------------------------------------------------------------------------------

Módulo:                     Computación en la nube (Optativa II)
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT01. Introducción y preparación del entorno
Práctica:                   PR0101. Preparación del entorno
Resultados de aprendizaje:  XX
```

# PR0101. Preparación del entorno

## 1. Objetivo

Configurar el espacio de trabajo local (Docker/Jupyter), familiarizarse con el ciclo de vida del Learner Lab y realizar operaciones básicas en Amazon S3 mediante tres vías: Consola Web, AWS CloudShell (CLI) y SDK Boto3 (Python).

## Fase 0: Arranque del Laboratorio en AWS Academy

1. Accede a tu curso de **AWS Academy Canvas** y entra en **Learner Lab**.
2. Haz clic en el botón **Start Lab**. Espera a que el círculo junto a "AWS" pase de rojo/amarillo a **verde**.


3. Localiza dos botones clave en la barra superior:
   - **AWS:** Al pulsarlo, abre la Consola de Administración de AWS en una pestaña nueva.
   - **AWS Details:** Al pulsarlo, muestra la pestaña con tus credenciales temporales (`aws_access_key_id`, `aws_secret_access_key` y `aws_session_token`). Las necesitaremos en la Fase 3.


## Fase 1: Interacción mediante la Consola Web (GUI)

El objetivo es familiarizarse con el panel de administración web y la navegación por servicios.

1. Pulsa sobre el botón **AWS** en Learner Lab para entrar en la consola web. Comprueba en la esquina superior derecha que la región seleccionada es **N. Virginia (`us-east-1`)**.
2. En el buscador superior, escribe `S3` y selecciona el servicio.
3. **Crear el bucket:**
   - Pulsa en **Create bucket** (Crear bucket).
   - Nombre del bucket: `asir-[tunombre]-web` (ej: `asir-juan-web`).
   - Mantén las opciones por defecto.
   - Haz clic en **Create bucket** al final de la página.
4. **Subir un fichero:**
   - Entra en el bucket recién creado.
   - Crea en tu equipo un archivo de texto llamado `bienvenida-gui.txt` con el texto *"Creado desde la interfaz web"*.
   - Pulsa en **Upload** (Cargar) -> **Add files** (Añadir archivos), selecciona el archivo y pulsa **Upload**.



> **Captura 1 para la memoria:** Vista del interior de tu bucket en la consola web donde se aprecie claramente el nombre del bucket y el archivo `bienvenida-gui.txt` subido.



## Fase 2: Interacción mediante CLI con AWS CloudShell

Dado que en los equipos del aula no disponemos de la herramienta de línea de comandos (`aws-cli`) instalada localmente, utilizaremos **AWS CloudShell**, una terminal basada en navegador que ya incluye las herramientas oficiales de AWS y se autentica automáticamente con tu sesión.

1. En la consola de AWS (barra superior derecha, al lado del icono de notificaciones), haz clic en el icono de **CloudShell** (`>_`). Espera unos segundos a que inicialice el entorno Linux.
2. Comprueba la identidad con la que estás ejecutando comandos:
    ```bash
    aws sts get-caller-identity
    ```
3. Crea un nuevo bucket mediante CLI:
    ```bash
    aws s3 mb s3://asir-[tunombre]-cli --region us-east-1
    ```
4. Genera un archivo en la terminal y súbelo a S3:
    ```bash
    echo "Hola AWS CLI desde CloudShell" > mensaje-cli.txt
    aws s3 cp mensaje-cli.txt s3://asir-[tunombre]-cli/

    ```
5. Lista los objetos del bucket para verificar la subida:
    ```bash
    aws s3 ls s3://asir-[tunombre]-cli/
    ```

> **Captura 2 para la memoria:** Terminal de CloudShell mostrando la ejecución secuencial de la creación del bucket (`aws s3 mb`), la copia del archivo (`aws s3 cp`) y el listado final (`aws s3 ls`).



## Fase 3: Interacción mediante SDK con Python (Boto3) en Docker

En esta fase automatizaremos tareas en la nube usando el SDK oficial de AWS para Python (**Boto3**). Ejecutaremos el entorno dentro de un contenedor Docker con **JupyterLab**.

### 3.1. Despliegue de Jupyter con Docker Compose

1. En tu máquina del aula, crea una carpeta de trabajo y accede a ella:
```bash
mkdir practica-aws && cd practica-aws
mkdir notebooks
```


1. Obtén el fichero `docker-compose.yml` del [repositorio de ficheros compose](https://vgonzalez165.github.io/docker_resources/)
2. Levanta el contenedor en segundo plano:
```bash
docker compose up -d
```
2. Abre tu navegador y entra en: `http://localhost:8888`. 



### 3.2. Script en Python con Boto3

1. Dentro de JupyterLab, entra en la carpeta `work` y crea un nuevo Notebook de Python 3.
2. **Instalar el SDK:** En la primera celda del notebook, instala la librería `boto3`:
```python
!pip install boto3
```
3. **Obtener las credenciales:**
    - Vuelve a la pestaña de **Learner Lab** en Canvas.
    - Pulsa en **AWS Details** y haz clic en **Show** junto a *AWS CLI*.
    - Verás tres valores: `aws_access_key_id`, `aws_secret_access_key` y `aws_session_token`.
4. **Ejecutar el script de conexión y subida:**
    - Copia el siguiente código en una celda de Jupyter sustituyendo las credenciales y tu nombre:

```python
import boto3

# 1. Credenciales temporales de AWS Academy Learner Lab
AWS_ACCESS_KEY = "PEGA_AQUI_TU_AWS_ACCESS_KEY_ID"
AWS_SECRET_KEY = "PEGA_AQUI_TU_AWS_SECRET_ACCESS_KEY"
AWS_SESSION_TOKEN = "PEGA_AQUI_TU_AWS_SESSION_TOKEN"
REGION = "us-east-1"

# 2. Inicializar el cliente de S3
s3_client = boto3.client(
    's3',
    aws_access_key_id=AWS_ACCESS_KEY,
    aws_secret_access_key=AWS_SECRET_KEY,
    aws_session_token=AWS_SESSION_TOKEN,
    region_name=REGION
)

# 3. Crear el bucket
bucket_name = "asir-tunombre-sdk"  # Reemplaza 'tunombre'
s3_client.create_bucket(Bucket=bucket_name)
print(f"Bucket '{bucket_name}' creado con éxito.")

# 4. Crear un archivo local y subirlo al bucket
archivo_local = "datos_sdk.txt"
with open(archivo_local, "w") as f:
    f.write("Archivo subido mediante Python y Boto3 desde Jupyter en Docker.")

s3_client.upload_file(archivo_local, bucket_name, "datos_sdk.txt")
print(f"Archivo '{archivo_local}' subido correctamente.")

# 5. Listar objetos del bucket para confirmar
respuesta = s3_client.list_objects_v2(Bucket=bucket_name)
print("\nObjetos en el bucket:")
for obj in respuesta.get('Contents', []):
    print(f" - {obj['Key']} ({obj['Size']} bytes)")

```

> **Captura 3 para la memoria:** Celda de Jupyter con la ejecución del script completa y la salida por pantalla confirmando la creación y el listado de objetos.
> **Captura 4 para la memoria:** Comprobación final en la consola web de AWS (S3) donde se vean los **3 buckets creados** durante la práctica (el de la web, el de CloudShell y el de Python).



## 4. Tarea final y limpieza de recursos

Una buena práctica en administración cloud es evitar costes y uso innecesario de almacenamiento:

1. **Vaciar y borrar los 3 buckets:** puedes hacerlo desde la consola web.
2. **Detener el entorno:**
   - En tu terminal local: `docker compose down`
   - En AWS Academy Canvas: pulsa en **Stop Lab** para pausar el contador de créditos.


## 5. Estructura de la Memoria a Entregar

El documento debe entregarse en **Markdown** y contener:

1. **Portada y datos del alumno.**
2. **Apartado 1 (GUI):** Breve descripción del proceso y **Captura 1**.
3. **Apartado 2 (CLI/CloudShell):** Explicación de los comandos utilizados y **Captura 2**.
4. **Apartado 3 (SDK/Python):**
    - **Captura 3** (JupyterLab).
    - **Captura 4** (Consola con los 3 buckets).
5. **Conclusión comparativa (3-5 líneas):** ¿En qué casos de administración de sistemas recomendarías usar la consola web, en cuáles la CLI y cuándo recurrir a un script con SDK?