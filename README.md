# AI Support Engine — Motor de análisis de solicitudes de soporte

Este proyecto es un prototipo funcional que recibe solicitudes de soporte técnico y las analiza con dos métodos diferentes:

1. **Motor tradicional:** utiliza reglas y palabras clave. No utiliza un modelo generativo.
2. **Motor generativo:** envía el mensaje a un modelo de lenguaje local mediante Ollama y recibe una clasificación en formato JSON.

Los dos motores utilizan el mismo formato de respuesta y se pueden probar mediante una API.

## 1. Requisitos

Antes de comenzar, necesitas:

- Windows, macOS o Linux.
- Python 3.10 o una versión posterior.
- Ollama instalado para utilizar el motor generativo.
- Conexión a internet para instalar las dependencias y descargar el modelo.

## 2. Descargar y preparar el proyecto

Descomprime la carpeta `AI_Support_Engine` y abre una terminal dentro de ella.

### En Windows (PowerShell)

Crea el entorno virtual:

```powershell
py -m venv .venv
```

Actívalo:

```powershell
.\.venv\Scripts\Activate.ps1
```

Si PowerShell indica que la ejecución de scripts está deshabilitada, ejecuta primero este comando:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Después vuelve a activar el entorno:

```powershell
.\.venv\Scripts\Activate.ps1
```

Instala las dependencias:

```powershell
pip install -r requirements.txt
```

### En macOS o Linux

Crea y activa el entorno virtual:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## 3. Configurar el motor generativo

El motor generativo utiliza Ollama para ejecutar un modelo de inteligencia artificial de manera local.

1. Descarga Ollama desde https://ollama.com/download.
2. Instálalo y asegúrate de que esté ejecutándose.
3. Abre una terminal y descarga el modelo:

```bash
ollama pull llama3.2:3b
```

El proyecto incluye un archivo `.env.example` con la configuración inicial. En Windows puedes crear el archivo `.env` copiándolo con:

```powershell
Copy-Item .env.example .env
```

En macOS o Linux:

```bash
cp .env.example .env
```

La configuración predeterminada es:

```env
OLLAMA_URL=http://localhost:11434/api/chat
OLLAMA_MODEL=llama3.2:3b
```

Si utilizas otro modelo, cambia `OLLAMA_MODEL` en el archivo `.env`.

**Importante:** no compartas ni subas archivos `.env` que contengan credenciales o información privada. El archivo `.gitignore` evita que `.env` se agregue accidentalmente al repositorio.

## 4. Iniciar la API

Con el entorno virtual activado y desde la carpeta principal del proyecto, ejecuta:

```bash
uvicorn app:app --reload
```

Cuando aparezca un mensaje que indique que el servidor está disponible en `http://127.0.0.1:8000`, la API estará lista.

No cierres esa terminal mientras estés realizando las pruebas.

Direcciones disponibles:

- API: http://127.0.0.1:8000
- Documentación interactiva: http://127.0.0.1:8000/docs
- Comprobación del estado: http://127.0.0.1:8000/health

La interfaz automática de FastAPI puede mostrar algunos controles en inglés. Eso no afecta el idioma de las respuestas que genera el proyecto.

## 5. Probar el endpoint

Abre http://127.0.0.1:8000/docs.

1. Busca `POST /api/analyze`.
2. Presiona **Try it out**.
3. En el cuerpo de la solicitud escribe un JSON como el siguiente.
4. Presiona **Execute** para enviar la solicitud.

### Prueba del motor tradicional

```json
{
  "message": "No puedo iniciar sesión",
  "mode": "traditional"
}
```

### Prueba del motor generativo

```json
{
  "message": "No puedo iniciar sesión",
  "mode": "generative"
}
```

El motor generativo necesita que Ollama esté instalado, ejecutándose y que el modelo configurado ya se haya descargado.

## 6. Ejecutar las pruebas comparativas

Mantén abierta la terminal donde está funcionando la API.

Abre una segunda terminal dentro de la carpeta principal del proyecto y activa el entorno virtual. En Windows:

```powershell
.\.venv\Scripts\Activate.ps1
```

Si aparece el bloqueo de PowerShell, aplica el comando `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` indicado anteriormente.

Ejecuta las pruebas:

```bash
python tests/run_comparison.py
```

El programa procesa 12 solicitudes con el motor tradicional y con el generativo. Incluye mensajes habituales, solicitudes ambiguas y mensajes que intentan manipular las instrucciones del clasificador.

Al finalizar, se crea la carpeta `results` con dos archivos:

- `comparacion.csv`: resultados que se pueden abrir con Excel.
- `comparacion.json`: los mismos resultados en formato JSON.

La exactitud se calcula comparando la categoría y la prioridad obtenidas con los valores de referencia definidos para cada caso. También se verifica si cada respuesta cumple el formato esperado.

**Los porcentajes dependen de la ejecución real y del modelo utilizado.** No se deben inventar ni copiar resultados de otra ejecución. Si Ollama no está disponible, los casos generativos mostrarán un error y no contarán como aciertos.

## 7. Formato de respuesta de la API

Los dos motores devuelven los mismos campos:

```json
{
  "category": "ACCESS",
  "priority": "MEDIUM",
  "response": "Lamentamos el inconveniente. Verifique sus credenciales y utilice la recuperación de contraseña si es necesario.",
  "requiresHumanReview": false,
  "mode": "traditional"
}
```

Las categorías permitidas son:

| Categoría | Significado |
|---|---|
| `ACCESS` | Problemas de inicio de sesión o acceso |
| `BUG` | Errores o fallas del sistema |
| `FEATURE` | Solicitudes de nuevas funcionalidades |
| `SECURITY` | Vulnerabilidades o incidentes de seguridad |

Las prioridades permitidas son `LOW`, `MEDIUM`, `HIGH` y `CRITICAL`.

## 8. Validaciones y medidas de seguridad

La implementación contempla lo siguiente:

- El mensaje es obligatorio y debe tener entre 5 y 3000 caracteres.
- El modo solo puede ser `traditional` o `generative`.
- Se comprueba que el modelo devuelva las propiedades requeridas.
- Se validan la categoría, la prioridad, el texto de respuesta y el valor booleano de revisión humana.
- Se manejan errores de conexión, tiempo de espera y respuestas incorrectas de Ollama.
- No se incluyen claves de API ni contraseñas en el código.
- El mensaje recibido se trata como contenido que debe analizarse, no como instrucciones que puedan cambiar las reglas del sistema.
- Si se detectan indicadores de seguridad o el modelo devuelve la categoría `SECURITY`, se activa `requiresHumanReview`.

La revisión humana es una medida de apoyo. Los incidentes reales deben seguir el procedimiento de seguridad de la empresa.

## 9. Estructura de archivos

```text
AI_Support_Engine/
│
├── app.py
├── requirements.txt
├── .env.example
├── .gitignore
├── README.md
│
└── tests/
    └── run_comparison.py
```

- `app.py`: contiene la API, las reglas del motor tradicional y la conexión con Ollama.
- `requirements.txt`: enumera las librerías necesarias.
- `.env.example`: muestra las variables de configuración del modelo.
- `.gitignore`: evita agregar archivos temporales y secretos al repositorio.
- `README.md`: contiene las instrucciones del proyecto.
- `tests/run_comparison.py`: ejecuta y compara los dos motores.

## 10. Explicación breve del funcionamiento

1. FastAPI recibe la solicitud enviada a `POST /api/analyze`.
2. Pydantic revisa que el mensaje y el modo tengan valores permitidos.
3. Si el modo es `traditional`, el programa compara el mensaje con listas de palabras clave y aplica reglas para determinar la categoría y la prioridad.
4. Si el modo es `generative`, el programa envía el mensaje a Ollama junto con instrucciones para que el modelo lo analice y devuelva un JSON.
5. La respuesta del modelo se valida antes de enviarse al usuario.
6. El script de pruebas envía los mismos casos a los dos motores y calcula sus resultados.

De esta manera se pueden comparar las ventajas y limitaciones de un clasificador basado en reglas frente a uno que utiliza un modelo generativo.
