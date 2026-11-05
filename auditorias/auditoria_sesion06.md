# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Validación Estricta, Constraints y Params Query/Body  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Filtro manual de rangos y longitudes de texto  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue asegurar que todos los datos recibidos cumplieran con las reglas solicitadas. Si las validaciones se hacen manualmente, es fácil olvidar algún caso, como verificar la longitud del título, el rango del año o que el precio sea mayor que cero. Además, esto hace que el código sea más largo y difícil de mantener.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación desarrollada con FastAPI en Google Colab. Se validarán datos de libros y límites de consulta. |
| **Rol (R)** | Actuar como desarrollador backend que crea una API con validaciones de negocio. |
| **Especificación (E)** | Crear un libro mediante POST /api/libros validando título, año y precio. Consultar libros mediante GET /api/libros usando el parámetro limite entre 1 y 100. Los datos válidos responden 201 o 200 y los inválidos 422. |
| **Output (O)** | Respuestas JSON con la información del libro o mensajes de error automáticos de validación. |

### 2.2 Prompt Estructurado
```text
[Necesito crear una API en FastAPI para gestionar libros. Debe validar que el título tenga entre 3 y 80 caracteres, que el año esté entre 1450 y 2100 y que el precio sea mayor que cero. También debe aceptar un parámetro limite con valores entre 1 y 100. Los datos válidos deben devolver respuestas 200 o 201, y los inválidos un error 422. Toda la información debe enviarse y recibirse en formato JSON.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, Query, status
from pydantic import BaseModel, Field

app = FastAPI(title="Tus límites de negocio responderán así")

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

# Base de datos temporal en memoria
libros = []

class Libro(BaseModel):
    titulo: str = Field(..., min_length=3, max_length=80)
    anio: int = Field(..., ge=1450, le=2100)
    precio: float = Field(..., gt=0)

@app.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear_libro(libro: Libro):
    nuevo = {
        "id": len(libros) + 1,
        "titulo": libro.titulo,
        "anio": libro.anio,
        "precio": libro.precio
    }
    libros.append(nuevo)
    return nuevo

@app.get("/api/libros")
def listar_libros(
    limite: int = Query(..., ge=1, le=100)
):
    return libros[:limite]
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de desbordamiento de cadenas y números negativos

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La validación de los datos fue el punto más importante que detectó la auditoría. Para evitar que se aceptaran títulos muy cortos, años fuera de rango, precios negativos o límites inválidos, agregué restricciones con Field y Query, lo que permitió que FastAPI devolviera automáticamente el código HTTP 422 cuando correspondía.

---
**Firma digital:** [EuniceUT]
