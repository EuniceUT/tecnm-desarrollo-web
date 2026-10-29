# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Pydantic v2 / DTOs en PHP y Tipado Estático  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* DTO manual con validación de tipos primitivos  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue asegurar que los datos recibidos tuvieran el tipo correcto. También fue necesario controlar que no se enviaran campos adicionales y que los errores devolvieran el código HTTP adecuado cuando la información era incorrecta.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | API REST con FastAPI y Pydantic v2 en Google Colab. Validación estricta de datos. |
| **Rol (R)** | Crear DTOs para validar libros y controlar las respuestas de la API. |
| **Especificación (E)** | POST /api/libros crea libros validando titulo, anio, precio y disponible. GET /api/libros lista los libros. Rechaza tipos inválidos y campos extra con HTTP 422. |
| **Output (O)** | JSON con id, titulo, anio, precio y disponible. Códigos HTTP: 201, 200 y 422. |

### 2.2 Prompt Estructurado
```text
[Estoy desarrollando una API con FastAPI y Pydantic v2 para registrar libros. Ya tengo las rutas creadas, pero necesito ayuda para configurar un DTO que permita convertir automáticamente cadenas numéricas como "1966" a enteros y que rechace cualquier campo adicional enviado en el JSON. Muéstrame un ejemplo sencillo utilizando ConfigDict(extra="forbid") y explica cómo se generan los errores 422.]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, status
from pydantic import BaseModel, ConfigDict

app = FastAPI(title="DTO aceptará y rechazará exactamente esto")


# DTO de entrada
class LibroCreate(BaseModel):
    titulo: str
    anio: int
    precio: float
    disponible: bool

    model_config = ConfigDict(extra="forbid")


# DTO de salida
class LibroResponse(BaseModel):
    id: int
    titulo: str
    anio: int
    precio: float
    disponible: bool


# "Base de datos" en memoria
libros = []


@app.post(
    "/api/libros",
    response_model=LibroResponse,
    status_code=status.HTTP_201_CREATED
)
def crear_libro(libro: LibroCreate):
    nuevo = {
        "id": len(libros) + 1,
        "titulo": libro.titulo,
        "anio": libro.anio,
        "precio": libro.precio,
        "disponible": libro.disponible,
    }

    libros.append(nuevo)
    return nuevo


@app.get("/api/libros")
def listar_libros():
    return {"libros": libros}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de tipos incompatibles y coerción

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La parte que más revisé fue la validación de campos adicionales en las peticiones. Con ayuda de la IA identifiqué que era necesario configurar el DTO para rechazar campos no definidos usando ConfigDict(extra="forbid"). Después verifiqué con las pruebas que el servidor respondiera con HTTP 422 cuando se enviaba un campo extra, cumpliendo el contrato solicitado.

---
**Firma digital:** [EuniceUT]
