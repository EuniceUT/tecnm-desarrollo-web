# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Inyección de Dependencias (FastAPI Depends / PHP Container)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inyector de dependencias manual por funciones  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> 
Lo que más me costó fue hacer que la información se mantuviera guardada después de crear un libro. Al principio, cada vez que se hacía una petición se volvía a empezar desde cero y los libros desaparecían. También fue complicado entender cómo pasar el repositorio a los endpoints sin crearlo dentro de cada función.


---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación desarrollada con FastAPI en Google Colab. Se debe utilizar inyección de dependencias mediante Depends y mantener un repositorio en memoria. |
| **Rol (R)** | Actuar como desarrollador backend que implementa un CRUD sencillo siguiendo el contrato definido por la práctica. |
| **Especificación (E)** | Crear una función get_repositorio(), inyectar el repositorio en los endpoints con Depends, exponer /api/diagnostico, permitir crear y listar libros, validar datos con Pydantic y devolver códigos HTTP correctos (200, 201 y 422). |
| **Output (O)** | Archivo app.py funcional que supere todas las pruebas automáticas de verificación. |

### 2.2 Prompt Estructurado
```text
[
Contexto:
Aplicación realizada con FastAPI utilizando un repositorio en memoria.
Rol:
Actúa como desarrollador backend.
Especificación:
Crear una función get_repositorio(), utilizar Depends para inyectar la dependencia, implementar los endpoints de diagnóstico, creación y consulta de libros, y validar los datos recibidos.
Output:
Código completo en app.py que cumpla con todos los requisitos del ejercicio.
]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, Depends, status
from pydantic import BaseModel

app = FastAPI(title="Tu endpoint recibirá el repositorio inyectado")

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.


class LibroCrear(BaseModel):
    titulo: str
    anio: int

#Repositorio en memoria
class RepositorioMemoria:
  def __init__(self):
    self.libros = []
    self.contador = 1

  def crear_libro(self, titulo, anio):
    libro = {
        "id": self.contador,
        "titulo": titulo,
        "anio": anio
    }

    self.libros.append(libro)
    self.contador += 1
    return libro

  def listar_libros(self):
    return self.libros

#Instancia unica para conservar el estado
repositorio = RepositorioMemoria()

#Funcion de inyeccion requerida
def get_repositorio():
  return repositorio

@app.get("/api/diagnostico")
def diagnostico(repo: RepositorioMemoria = Depends(get_repositorio)):
    return {"repositorio": repo.__class__.__name__}


@app.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear_libro(
    libro: LibroCrear,
    repo: RepositorioMemoria = Depends(get_repositorio)
):
    return repo.crear_libro(libro.titulo, libro.anio)


@app.get("/api/libros")
def obtener_libros(
    repo: RepositorioMemoria = Depends(get_repositorio)
):
    return {"libros": repo.listar_libros()}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de ciclo de vida de dependencias por request

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 
El principal problema fue que los datos no se conservaban entre peticiones porque el repositorio se estaba creando nuevamente cada vez que se utilizaba un endpoint. Lo corregí utilizando una sola instancia compartida del repositorio y devolviéndola mediante la función de inyección. Después de ese ajuste todas las pruebas se ejecutaron correctamente.


---
**Firma digital:** [EuniceUT]
