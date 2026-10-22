# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Gestión en GitHub Web y Métodos HTTP (GET, POST, PUT, DELETE)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Enrutamiento manual por verbo HTTP  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> Lo que se me complico fue entender que método HTTP usar en cada caso y de asegurarme de que cada ruta devolviera el código correcto. Tambien tuve que verificar que pasaba cuando un libro no existia.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Especificación semántica de verbos e idempotencia |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
[Ayúdame a crear una API en FastAPI para administrar una colección de libros.Requisitos:
- GET /api/libros debe mostrar todos los libros.
- POST /api/libros debe crear un libro con un id automático.
- GET /api/libros/{id} debe mostrar un libro por id.
- PUT /api/libros/{id} debe actualizar un libro.
- DELETE /api/libros/{id} debe eliminar un libro.
- Si el libro no existe, responder con 404.
- PATCH no debe implementarse para que FastAPI responda con 405.

Los datos deben almacenarse en memoria y devolver los códigos HTTP adecuados.]
```

---

## 3. Código Devuelto por el Modelo
```python
%%writefile app.py
from fastapi import FastAPI, HTTPException, Response
from pydantic import BaseModel

app = FastAPI(title="CRUD de Libros")

# Colección en memoria
libros = []
siguiente_id = 1


class LibroEntrada(BaseModel):
    titulo: str
    anio: int


@app.get("/api/libros")
def listar_libros():
    return {"libros": libros}


@app.post("/api/libros", status_code=201)
def crear_libro(libro: LibroEntrada):
    global siguiente_id

    nuevo = {
        "id": siguiente_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }

    libros.append(nuevo)
    siguiente_id += 1

    return nuevo


@app.get("/api/libros/{id}")
def obtener_libro(id: int):
    for libro in libros:
        if libro["id"] == id:
            return libro

    raise HTTPException(status_code=404, detail="Libro no encontrado")


@app.put("/api/libros/{id}")
def reemplazar_libro(id: int, datos: LibroEntrada):
    for i, libro in enumerate(libros):
        if libro["id"] == id:

            actualizado = {
                "id": id,
                "titulo": datos.titulo,
                "anio": datos.anio
            }

            libros[i] = actualizado
            return actualizado

    raise HTTPException(status_code=404, detail="Libro no encontrado")


@app.delete("/api/libros/{id}", status_code=204)
def eliminar_libro(id: int):
    for i, libro in enumerate(libros):
        if libro["id"] == id:
            libros.pop(i)
            return Response(status_code=204)

    raise HTTPException(status_code=404, detail="Libro no encontrado")

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de métodos no permitidos (HTTP 405)

* [ x ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ x ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ x ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> Al principio no había considerado la diferencia entre los errores 404 y 405. Después de ejecutar las pruebas verifiqué que cuando una ruta existe pero el método no está permitido, FastAPI debe devolver 405. También comprobé que cuando un libro no existe se responda correctamente con 404.

---
**Firma digital:** [Eunice Esmeralda Uitzil Tuz]
