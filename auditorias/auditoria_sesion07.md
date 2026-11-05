# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Especificación OpenAPI (Swagger) y ReDoc  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Anotación manual de metadata de endpoints  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue comprender cómo FastAPI genera la documentación automáticamente. Al principio pensé que bastaba con crear el modelo Libro, pero después descubrí que también debía utilizarlo dentro de un endpoint para que apareciera en la especificación OpenAPI. Además, tuve que revisar varias veces la estructura de la respuesta para que coincidiera exactamente con lo que esperaba el verificador.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Se utilizará FastAPI en Python dentro de Google Colab. El proyecto debe generar automáticamente la documentación OpenAPI y permitir acceder a Swagger UI y ReDoc. |
| **Rol (R)** | Actuar como desarrollador backend encargado de crear una API de biblioteca documentada. |
| **Especificación (E)** | Crear una API llamada Biblioteca API versión 1.0.0, con un modelo Libro y el endpoint GET /api/libros que responda HTTP 200. También deben estar disponibles /openapi.json, /docs y /redoc. |
| **Output (O)** | Archivo app.py funcional que cumpla el contrato y pase todas las pruebas de verificación. |

### 2.2 Prompt Estructurado
```text
[Crear una API en FastAPI llamada Biblioteca API versión 1.0.0 que documente automáticamente sus endpoints mediante OpenAPI. Debe incluir el modelo Libro, el endpoint GET /api/libros con respuesta HTTP 200 y permitir el acceso a la documentación mediante /openapi.json, /docs y /redoc, cumpliendo todas las pruebas de verificación.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI
from pydantic import BaseModel
from typing import List

app = FastAPI(
    title="Biblioteca API",
    version="1.0.0",
    description="API de libros del curso DAB-2201"
)


# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

class Libro(BaseModel):
    id: int
    titulo: str
    autor: str

class ListaLibros(BaseModel):
    libros: List[Libro]

@app.get(
    "/api/libros",
    response_model=ListaLibros,
    summary="Listar libros"
)
def listar_libros():
    return {"libros": []}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Verificación de renderizado de interfaz Swagger UI

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> Inicialmente el modelo Libro estaba definido, pero no era utilizado por ningún endpoint. Debido a esto, el esquema no aparecía dentro de components.schemas en la documentación OpenAPI y la prueba de validación fallaba. La corrección consistió en crear un modelo de respuesta llamado ListaLibros y usarlo como response_model en el endpoint /api/libros, permitiendo que FastAPI incluyera automáticamente el esquema Libro en la documentación.

---
**Firma digital:** [EuniceUT]
