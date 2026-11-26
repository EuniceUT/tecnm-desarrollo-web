# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Manejo de Sesiones Stateful vs Stateless Tokens  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Simulador de tabla de sesiones en memoria  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La mayor dificultad fue entender cómo el servidor puede recordar qué usuario inició sesión entre una petición y otra. También fue necesario comprender cómo leer la cookie enviada por el navegador y relacionarla con una sesión guardada en memoria para permitir o negar el acceso al perfil.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Se desarrollará una aplicación web usando Python 3 y FastAPI en Google Colab. Se utilizarán sesiones almacenadas en memoria y cookies HttpOnly para mantener la autenticación del usuario. No se usará base de datos ni servicios externos. |
| **Rol (R)** | Actúa como desarrollador backend encargado de implementar un sistema de inicio de sesión basado en sesiones. Debe crear las rutas necesarias para login, perfil y logout, cumpliendo los códigos HTTP y reglas establecidas en el contrato. |
| **Especificación (E)** | Usuario válido: ana. Contraseña válida: secreta. POST /api/login: valida credenciales, crea una sesión y devuelve HTTP 200 junto con una cookie session_id configurada con HttpOnly y SameSite=Lax. GET /api/perfil: devuelve { "usuario": "ana" } si la sesión es válida; en caso contrario responde HTTP 401. POST /api/logout: elimina la sesión activa y responde HTTP 204. Después del logout, cualquier acceso a /api/perfil debe responder HTTP 401. |
| **Output (O)** | Archivo app.py funcional que implemente el flujo completo de autenticación mediante sesiones, cumpla todos los criterios de aceptación y permita que el verificador muestre todos los casos en verde. |

### 2.2 Prompt Estructurado
```text
[
Actúa como un desarrollador backend experto en FastAPI.
Implementa una aplicación con sesiones en memoria usando cookies HttpOnly.
Requisitos:
- Usuario válido: ana
- Contraseña válida: secreta
- POST /api/login debe validar credenciales, crear una sesión y responder HTTP 200.
- Debe enviar una cookie session_id con HttpOnly y SameSite=Lax.
- GET /api/perfil debe devolver el usuario autenticado si la sesión es válida.
- Si la sesión no existe debe responder HTTP 401.
- POST /api/logout debe eliminar la sesión y responder HTTP 204.
- Después del logout, GET /api/perfil debe responder HTTP 401.

Devuelve únicamente el código completo de app.py
]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, Response, Request, HTTPException
from pydantic import BaseModel
import uuid

app = FastAPI(title="Tu flujo de sesión se verá así")

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

# Sesiones en memoria
sesiones = {}

# Usuario de prueba
USUARIO = "ana"
PASSWORD = "secreta"


class LoginRequest(BaseModel):
    usuario: str
    password: str


@app.post("/api/login")
def login(datos: LoginRequest, response: Response):

    if datos.usuario != USUARIO or datos.password != PASSWORD:
        raise HTTPException(status_code=401, detail="Credenciales incorrectas")

    session_id = str(uuid.uuid4())

    sesiones[session_id] = {
        "usuario": datos.usuario
    }

    response.set_cookie(
        key="session_id",
        value=session_id,
        httponly=True,
        samesite="lax",
        path="/"
    )

    return {"mensaje": "sesión iniciada"}


@app.get("/api/perfil")
def perfil(request: Request):

    session_id = request.cookies.get("session_id")

    if not session_id or session_id not in sesiones:
        raise HTTPException(status_code=401, detail="No autorizado")

    return {
        "usuario": sesiones[session_id]["usuario"]
    }


@app.post("/api/logout", status_code=204)
def logout(request: Request, response: Response):

    session_id = request.cookies.get("session_id")

    if session_id and session_id in sesiones:
        del sesiones[session_id]

    response.delete_cookie("session_id")

    return Response(status_code=204)
```

---
## Evidencia Verificador

==============================================================================

VERIFICACIÓN DEL RESULTADO · 4 casos · app.py

==============================================================================

✅ 01. Login correcto y cookie HttpOnly

✅ 02. El perfil se obtiene con la cookie

✅ 03. El logout cierra la sesión

✅ 04. Después del logout el perfil es 401

------------------------------------------------------------------------------

RESULTADO: 4/4 casos correctos. ¡Resultado logrado!


## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de expiración y revocación de sesión

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> Inicialmente no presté atención a los atributos de seguridad de la cookie. Durante la revisión confirmé que la cookie debía incluir HttpOnly y SameSite=Lax para cumplir el contrato. También verifiqué que después del logout el sistema respondiera con 401 Unauthorized, ya que una sesión eliminada no debe seguir dando acceso al perfil

---
**Firma digital:** [EuniceUT]
