# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hito 1: Mock API RESTful con OpenAPI y Validation Harness  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzi Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Router y esquemas de validación  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue asegurar que los datos recibidos cumplieran con las reglas 
definidas para cada campo. Al principio era fácil olvidar validaciones como campos obligatorios, 
longitud mínima o tipos de datos correctos. También fue necesario entender cómo separar las rutas 
en un router para mantener el código más organizado.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación desarrollada con FastAPI en Google Colab. Se debe usar middleware para agregar encabezados a todas las respuestas. |
| **Rol (R)** | Actuar como desarrollador backend encargado de implementar trazabilidad y medición de tiempo en las peticiones HTTP. |
| **Especificación (E)** | Crear un endpoint GET /api/libros que responda 200. Todas las respuestas deben incluir X-Process-Time y X-Request-ID. Si el cliente envía X-Request-ID, se conserva. También debe funcionar en errores 404. |
| **Output (O)** | Respuestas JSON con encabezados de tiempo y trazabilidad incluidos automáticamente mediante middleware. |

### 2.2 Prompt Estructurado
```text
[Actúa como desarrollador backend experto en FastAPI.
Crea una API RESTful para el primer hito del proyecto utilizando APIRouter y modelos Pydantic.
La aplicación debe validar los datos de entrada, generar automáticamente la documentación OpenAPI, 
responder en formato JSON y utilizar códigos HTTP adecuados (200, 201, 400 y 422). 
El código debe estar organizado, ser modular y evitar errores internos del servidor (500).]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, Request
from uuid import uuid4
import time

app = FastAPI(title="Todas tus respuestas llevarán estas cabeceras")

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

@app.middleware("http")
async def middleware_tiempo_y_traza(request: Request, call_next):
    inicio = time.time()

    request_id = request.headers.get("X-Request-ID")
    if not request_id:
        request_id = str(uuid4())[:8]

    response = await call_next(request)

    tiempo_proceso = time.time() - inicio

    response.headers["X-Process-Time"] = str(round(tiempo_proceso, 4))
    response.headers["X-Request-ID"] = request_id

    return response


@app.get("/api/libros")
async def obtener_libros():
    return {"libros": []}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Suite integral de 10 pruebas HTTP destructivas

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA generó inicialmente respuestas exitosas, pero no especificaba claramente algunos códigos de 
estado para operaciones de creación. También fue necesario verificar que las validaciones 
devolvieran un error 422 cuando los datos enviados eran incorrectos. 
Se corrigió agregando modelos de validación y revisando las respuestas HTTP esperadas para cada caso.

---
**Firma digital:** [EuniceUT]
