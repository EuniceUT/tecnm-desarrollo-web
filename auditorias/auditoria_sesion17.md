# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Control de Acceso Basado en Roles (RBAC)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Verificación manual de permisos de usuario (Admin/User)  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue distinguir correctamente cuándo debía responderse con un error 401 y cuándo con un error 403. Al principio parecía que ambos casos eran iguales, pero después entendí que el error 401 se utiliza cuando el usuario no ha iniciado sesión o no envía un token válido, mientras que el error 403 se utiliza cuando el usuario sí está autenticado, pero no tiene los permisos necesarios para acceder al recurso solicitado.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación desarrollada con FastAPI en Python. Se utiliza autenticación mediante tokens JWT y control de acceso basado en roles. El sistema debe diferenciar entre usuarios no autenticados (401) y usuarios sin permisos (403). |
| **Rol (R)** | Actuar como desarrollador backend encargado de implementar el inicio de sesión, la generación de tokens y la restricción de acceso según el rol de cada usuario. |
| **Especificación (E)** | Crear dos usuarios de prueba: admin/admin123 con rol admin y ana/secreta con rol user. El endpoint POST /api/token debe generar un token con el usuario y su rol. El endpoint GET /api/admin/panel solo permite acceso al administrador (200), devuelve 403 para usuarios normales y 401 cuando no se envía token. El endpoint GET /api/perfil debe funcionar para cualquier usuario autenticado y mostrar su usuario y rol. |
| **Output (O)** | Respuestas en formato JSON con los códigos HTTP esperados: 200 para accesos válidos, 401 cuando no hay autenticación y 403 cuando el usuario está autenticado pero no tiene permisos suficientes. |

### 2.2 Prompt Estructurado
```text
[Actúa como desarrollador backend especializado en FastAPI.
Crea una API con autenticación JWT y control de acceso basado en roles.
Requisitos:
- Usuario admin/admin123 con rol admin.
- Usuario ana/secreta con rol user.
- Endpoint POST /api/token que genere un token JWT con usuario y rol.
- Endpoint GET /api/admin/panel accesible únicamente para administradores.
- Endpoint GET /api/perfil accesible para cualquier usuario autenticado.
- Responder con HTTP 401 cuando no exista token válido.
- Responder con HTTP 403 cuando el usuario esté autenticado pero no tenga permisos.
- Devolver respuestas en formato JSON.]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, HTTPException, Header
from pydantic import BaseModel
import jwt

app = FastAPI(title="Control de Acceso Basado en Roles")

SECRET_KEY = "clave_secreta"

usuarios = {
    "admin": {
        "password": "admin123",
        "rol": "admin"
    },
    "ana": {
        "password": "secreta",
        "rol": "user"
    }
}

class Login(BaseModel):
    usuario: str
    password: str


@app.post("/api/token")
def obtener_token(datos: Login):

    usuario = usuarios.get(datos.usuario)

    if not usuario or usuario["password"] != datos.password:
        raise HTTPException(
            status_code=401,
            detail="Credenciales incorrectas"
        )

    token = jwt.encode(
        {
            "usuario": datos.usuario,
            "rol": usuario["rol"]
        },
        SECRET_KEY,
        algorithm="HS256"
    )

    return {"access_token": token}


def validar_token(authorization):

    if not authorization:
        raise HTTPException(
            status_code=401,
            detail="No autenticado"
        )

    try:
        token = authorization.replace("Bearer ", "")

        datos = jwt.decode(
            token,
            SECRET_KEY,
            algorithms=["HS256"]
        )

        return datos

    except:
        raise HTTPException(
            status_code=401,
            detail="Token inválido"
        )


@app.get("/api/admin/panel")
def panel_admin(Authorization: str = Header(default=None)):

    usuario = validar_token(Authorization)

    if usuario["rol"] != "admin":
        raise HTTPException(
            status_code=403,
            detail="Sin permiso"
        )

    return {"panel": "admin"}


@app.get("/api/perfil")
def perfil(Authorization: str = Header(default=None)):

    usuario = validar_token(Authorization)

    return {
        "usuario": usuario["usuario"],
        "rol": usuario["rol"]
    }

```

---
### Evidencia Verificador

==============================================================================

VERIFICACIÓN DEL RESULTADO · 6 casos · app.py

==============================================================================

✅ 01. El administrador obtiene su token

✅ 02. El usuario normal obtiene su token

✅ 03. El admin entra al panel

✅ 04. El usuario normal recibe 403

✅ 05. Sin token recibe 401

✅ 06. El perfil funciona para el usuario normal

------------------------------------------------------------------------------

RESULTADO: 6/6 casos correctos. ¡Resultado logrado!


## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso denegado por rol insuficiente (HTTP 403)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA inicialmente no distinguía claramente entre los códigos 401 y 403. Durante la verificación comprobé que una solicitud sin token debía devolver 401 (no autenticado), mientras que un usuario autenticado sin permisos de administrador debía recibir 403 (acceso denegado). Revisé la lógica de validación para asegurar que cada situación devolviera el código HTTP correcto.

---
**Firma digital:** [EuniceUT]
