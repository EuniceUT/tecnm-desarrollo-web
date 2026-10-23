# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Headers, Content-Type y Negociación de Contenido  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lectura y emisión manual de encabezados HTTP  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> Tuve dificultad para entender la diferencia entre los encabezados Content-Type y Accept, ya que uno indica el formato de los datos enviados y el otro el formato que el cliente espera recibir. También fue necesario validar correctamente los códigos 415 y 406.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI en Python para una API REST de gestión de libros con negociación de contenido. |
| **Rol (R)** | Desarrollador backend encargado de validar encabezados HTTP y respuestas de la API. |
| **Especificación (E)** | POST /api/ libros acepta solo JSON y devuelve 415 en otros formatos. GET /api/ libros responde solo si el cliente acepta JSON y devuelve 406 si solicita XML. |
| **Output (O)** | Respuestas en formato JSON con códigos HTTP correctos (200, 201, 406 y 415). |

### 2.2 Prompt Estructurado
```text
Actúa como un arquitecto de software especializado en APIs REST con FastAPI. Implementa una API de libros con las rutas POST /api/libros y GET /api/libros. El endpoint POST debe aceptar únicamente Content-Type: application/json y devolver HTTP 415 para otros formatos. El endpoint GET debe responder solo cuando el cliente acepte application/json y devolver HTTP 406 si solicita XML. Las respuestas deben usar formato JSON y los códigos HTTP correspondientes.
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI(title="API de libros con negociación de contenido")

# Lista temporal para almacenar los libros creados
libros = []


@app.post("/api/libros")
async def crear_libro(request: Request):
    # Verificar el formato que envía el cliente
    content_type = request.headers.get("content-type", "")
    content_type = content_type.split(";")[0].strip().lower()

    if content_type != "application/json":
        return JSONResponse(
            status_code=415,
            content={"detail": "Unsupported Media Type"}
        )

    # Leer y validar los datos recibidos
    try:
        datos = await request.json()
    except Exception:
        return JSONResponse(
            status_code=400,
            content={"detail": "JSON inválido"}
        )

    if not isinstance(datos, dict):
        return JSONResponse(
            status_code=422,
            content={"detail": "Se esperaba un objeto JSON"}
        )

    if not isinstance(datos.get("titulo"), str) or not isinstance(datos.get("anio"), int) or isinstance(datos.get("anio"), bool):
        return JSONResponse(
            status_code=422,
            content={"detail": "El título debe ser texto y el año un número entero"}
        )

    # Crear el libro y asignarle un identificador
    libro = {
        "id": len(libros) + 1,
        "titulo": datos["titulo"],
        "anio": datos["anio"]
    }
    libros.append(libro)

    return JSONResponse(
        status_code=201,
        content=libro,
        media_type="application/json"
    )


@app.get("/api/libros")
async def listar_libros(request: Request):
    # Verificar el formato que acepta el cliente
    accept = request.headers.get("accept", "/")
    formatos = [formato.split(";")[0].strip().lower()
                for formato in accept.split(",")]

    acepta_json = any(
        formato in ("application/json", "/", "application/*")
        for formato in formatos
    )

    if not acepta_json:
        return JSONResponse(
            status_code=406,
            content={"detail": "Not Acceptable"}
        )

    return JSONResponse(
        status_code=200,
        content=libros,
        media_type="application/json"
    )

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas con tipos de contenido no soportados (415)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA no consideró inicialmente todas las validaciones de los encabezados HTTP. Se corrigió agregando la comprobación explícita de Content-Type y Accept para devolver los códigos 415 Unsupported Media Type y 406 Not Acceptable cuando correspondía. Con estas correcciones, los cuatro casos de prueba fueron aprobados correctamente.

---
**Firma digital:** [Eunice Esmeralda Uitzil Tuz]
