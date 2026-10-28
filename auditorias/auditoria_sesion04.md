# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Anatomía RESTful, URIs Nombres y Recursos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diseño de URIs jerárquicas sin verbos en la ruta  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La parte que más se me dificultó fue diferenciar cuándo usar parámetros de ruta y cuándo usar parámetros de consulta. Al realizar la práctica entendí que autor_id debe ir en la ruta porque identifica un recurso específico, mientras que genero y limite funcionan como filtros opcionales y por eso deben ir como parámetros de consulta. 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI en Google Colab con Python. Implementación en app.py sin base de datos. |
| **Rol (R)** | Desarrollador backend encargado de diseñar una API RESTful siguiendo buenas prácticas. |
| **Especificación (E)** | Crear GET /api/autores/{autor_id}/libros con filtros opcionales genero y limite, devolver 404 cuando el autor no exista y crear GET /api/libros para listar todos los libros. |
| **Output (O)** | Respuestas en formato JSON con códigos HTTP correctos (200 y 404) según el contrato. |

### 2.2 Prompt Estructurado
```text
[Ayúdame a crear una API en FastAPI para consultar autores y sus libros. La API debe permitir obtener los libros de un autor, filtrar por género y limitar la cantidad de resultados. Si el autor no existe, debe devolver un error 404. También debe incluir una ruta para mostrar todos los libros y responder en formato JSON.]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py

from fastapi import FastAPI, HTTPException
from typing import Optional

app = FastAPI(title="Tu jerarquía de recursos responderá así")

# Datos iniciales exactamente como pide el contrato
autores = {
    1: {
        "id": 1,
        "nombre": "Gabriel García Márquez"
    },
    2: {
        "id": 2,
        "nombre": "Julio Cortázar"
    }
}

libros = [
    {
        "id": 1,
        "titulo": "Cien años de soledad",
        "genero": "novela",
        "autor_id": 1
    },
    {
        "id": 2,
        "titulo": "Doce cuentos peregrinos",
        "genero": "cuento",
        "autor_id": 1
    },
    {
        "id": 3,
        "titulo": "Rayuela",
        "genero": "novela",
        "autor_id": 2
    }
]


@app.get("/api/autores/{autor_id}/libros")
def obtener_libros_autor(
    autor_id: int,
    genero: Optional[str] = None,
    limite: Optional[int] = None
):
    if autor_id not in autores:
        raise HTTPException(
            status_code=404,
            detail="Autor no encontrado"
        )

    resultado = [
        {
            "id": libro["id"],
            "titulo": libro["titulo"],
            "genero": libro["genero"]
        }
        for libro in libros
        if libro["autor_id"] == autor_id
    ]

    if genero:
        resultado = [
            libro for libro in resultado
            if libro["genero"] == genero
        ]

    if limite is not None:
        resultado = resultado[:limite]

    return {
        "autor_id": autor_id,
        "libros": resultado
    }


@app.get("/api/libros")
def obtener_todos_los_libros():
    return {
        "libros": [
            {
                "id": libro["id"],
                "titulo": libro["titulo"],
                "genero": libro["genero"]
            }
            for libro in libros
        ]
    }
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de peticiones a recursos inexistentes (404)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> Me di cuenta de que cuando se buscaba un autor que no existía, no era correcto devolver una lista vacía. Lo corregí para que mostrara un error 404 con el mensaje "Autor no encontrado", cumpliendo con lo que pedía la práctica.

---
**Firma digital:** [EuniceUT]
