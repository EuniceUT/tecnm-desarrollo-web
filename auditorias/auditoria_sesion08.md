# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Respuestas Semánticas y Excepciones HTTP Personalizadas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lanzamiento manual de excepciones estructuradas  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La parte que más trabajo me dio fue lograr que todos los errores mostraran mensajes claros y con el formato que pedía la actividad. También tuve que asegurarme de que no se aceptaran títulos vacíos y que el usuario recibiera una respuesta que le ayudara a entender qué estaba mal.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI ejecutándose en Google Colab. Debe responder con códigos HTTP semánticos y evitar errores 500. |
| **Rol (R)** | Arquitecto Backend FastAPI encargado de diseñar una API REST con validaciones y manejo global de excepciones. |
| **Especificación (E)** | GET /api/libros/{id} devuelve 200 si existe y 404 con {"error":{"codigo":"NO_ENCONTRADO"}} si no existe. POST /api/libros valida que titulo no esté vacío y responde 422 con {"error":{"codigo":"VALIDACION"}}. Libros válidos responden 201. |
| **Output (O)** | JSON consistente tanto para éxito como para error, usando un sobre error para todas las excepciones controladas. |

### 2.2 Prompt Estructurado
```text
[Actúa como desarrollador backend en FastAPI. Crea una API de libros que permita consultar y registrar libros, responda con 404 si un libro no existe, 422 si los datos son inválidos, 201 cuando el registro sea exitoso y muestre mensajes de error claros en formato JSON, evitando errores 500.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from pydantic import BaseModel, Field
from typing import Dict

app = FastAPI(title="Tus errores se verán exactamente así")

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

# Base de datos simulada
libros: Dict[int, dict] = {
    1: {
        "id": 1,
        "titulo": "Rayuela",
        "anio": 1963
    }
}


# Modelo de entrada
class LibroCreate(BaseModel):
    titulo: str = Field(..., min_length=1)
    anio: int | None = None


# Excepción de negocio
class LibroNoEncontrado(Exception):
    def __init__(self, libro_id: int):
        self.libro_id = libro_id


# Handler para libro inexistente
@app.exception_handler(LibroNoEncontrado)
async def libro_no_encontrado_handler(request: Request, exc: LibroNoEncontrado):
    return JSONResponse(
        status_code=status.HTTP_404_NOT_FOUND,
        content={
            "error": {
                "codigo": "NO_ENCONTRADO",
                "mensaje": f"El libro {exc.libro_id} no existe"
            }
        }
    )


# Handler para validaciones
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": {
                "codigo": "VALIDACION",
                "mensaje": "Revisa los datos enviados"
            }
        }
    )


# Obtener libro
@app.get("/api/libros/{libro_id}")
async def obtener_libro(libro_id: int):
    if libro_id not in libros:
        raise LibroNoEncontrado(libro_id)

    return libros[libro_id]


# Crear libro
@app.post("/api/libros", status_code=201)
async def crear_libro(libro: LibroCreate):
    nuevo_id = max(libros.keys()) + 1

    nuevo_libro = {
        "id": nuevo_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }

    libros[nuevo_id] = nuevo_libro
    return nuevo_libro
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de captura de errores sin crashing (HTTP 500)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La solución funcionaba casi completa, pero fue necesario ajustar la forma en que se mostraban los errores cuando los datos enviados eran incorrectos. Por defecto, FastAPI muestra esos errores de una manera diferente a la solicitada en la actividad, así que se modificó la respuesta para que tuviera exactamente el formato esperado y fuera más fácil de entender para el usuario.

---
**Firma digital:** [EuniceUT]
