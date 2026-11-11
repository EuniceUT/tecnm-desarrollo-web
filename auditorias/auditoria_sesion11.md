# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Arquitectura en Capas: Router, Servicio y Capa de Datos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Separación manual de controladores y lógica  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> 
La principal dificultad fue separar correctamente las responsabilidades entre las capas. Al principio no tenía claro qué código debía estar en el router, cuál en el servicio y cuál en el repositorio. También tuve problemas porque una versión anterior de los archivos quedó cargada en memoria en Colab, lo que provocaba errores aunque el código ya estuviera corregido.


---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación web usando FastAPI en Google Colab. Los datos se guardan en memoria y debe mantenerse el contrato HTTP.  |
| **Rol (R)** | Actuar como desarrollador backend organizando la aplicación en capas. |
| **Especificación (E)** | GET /api/libros devuelve la lista de libros (200). POST /api/libros crea un libro (201). Si los datos son incorrectos devuelve 422. |
| **Output (O)** | Archivos separados: app.py, router.py, servicio.py y repositorio.py funcionando correctamente. |

### 2.2 Prompt Estructurado
```text
[Actúa como desarrollador de FastAPI. Crea una API organizada en capas usando app.py, router.py, servicio.py y repositorio.py. El router debe manejar las rutas /api/libros, el servicio debe contener la lógica para crear y listar libros, y el repositorio debe guardar los datos en memoria. Mantén el contrato HTTP con GET 200, POST 201 y validación 422 para datos incorrectos.]
```

---

## 3. Código Devuelto por el Modelo
```python

# Se generó una estructura por capas utilizando:
# app.py
%%writefile app.py
from fastapi import FastAPI
from router import router

app = FastAPI(title="Biblioteca por capas")

app.include_router(router)

# repositorio.py
%%writefile repositorio.py
# Guarda y recupera libros. No conoce HTTP.

libros = []
def obtener_libros():
  return libros

def guardar_libro(libro):
  libros.append(libro)
  return libro

# servicio.py
%%writefile servicio.py
# Reglas de negocio: crear y listar libros.
from repositorio import obtener_libros, guardar_libro

def listar_libros():
  return {"libros": obtener_libros()}

def crear_libro(titulo, anio):
  libro = {
      "id": len(obtener_libros()) + 1, "titulo": titulo, "anio": anio
      
  }

  guardar_libro(libro)
  return libro

# router.py
%%writefile router.py
from fastapi import APIRouter, status

from pydantic import BaseModel
from servicio import listar_libros, crear_libro

router = APIRouter(prefix="/api")

class Libro(BaseModel):
  titulo: str
  anio: int

@router.get("/libros")
def get_libros():
  return listar_libros()

@router.post("/libros", status_code=status.HTTP_201_CREATED)
def post_libro(libro: Libro):
  return crear_libro(libro.titulo, libro.anio)

#
# El router expone las rutas HTTP,
# el servicio contiene la lógica de negocio
# y el repositorio almacena los datos en memoria.

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de aislamiento de errores en lógica de negocio

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 
Durante las pruebas apareció un error HTTP 500 porque existía una versión antigua de una función cargada en memoria dentro de Colab. Aunque el archivo ya estaba corregido, el entorno seguía utilizando la versión anterior. La solución fue reiniciar la sesión de ejecución, volver a cargar los archivos y ejecutar nuevamente las pruebas hasta obtener los cuatro casos correctos.


---
**Firma digital:** [EuniceUT]
