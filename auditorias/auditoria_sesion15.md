# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Autenticación Stateless con JSON Web Tokens (JWT)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Codificación Base64Url y firma HMAC-SHA256  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> 
Lo que más me costó fue entender cómo un token puede demostrar quién es el usuario sin que el servidor guarde una sesión. Al principio parecía que el token era simplemente una cadena de texto, pero después comprendí que incluye información firmada que permite verificar si fue modificado. También tuve que entender cómo leer el encabezado Authorization y extraer correctamente el token para validarlo.


---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación desarrollada con FastAPI y Python 3 en Google Colab. La autenticación se realizará mediante JSON Web Tokens (JWT) utilizando una clave secreta almacenada en variables de entorno. |
| **Rol (R)** | Actuar como desarrollador backend encargado de implementar autenticación sin sesiones almacenadas en el servidor utilizando JWT. |
| **Especificación (E)** | Crear un endpoint /api/token que valide las credenciales del usuario ana y genere un JWT firmado con fecha de expiración. Crear una ruta protegida /api/perfil que exija el encabezado Authorization Bearer y valide la firma del token. Los tokens alterados, inválidos o vencidos deben responder con HTTP 401. |
| **Output (O)** | Archivo app.py funcional utilizando JWT y cumpliendo todos los requisitos definidos en el contrato y las pruebas automáticas. |

### 2.2 Prompt Estructurado
```text
[
Actúa como un desarrollador backend especializado en FastAPI y autenticación JWT.
Implementa una API que cumpla los siguientes requisitos:
- Usuario válido: ana
- Contraseña válida: secreta
- POST /api/token debe generar un JWT firmado.
- El token debe incluir el usuario y una fecha de expiración.
- GET /api/perfil debe requerir Authorization: Bearer.
- Validar la firma del token antes de permitir acceso.
- Si el token es inválido, alterado o expiró, responder HTTP 401.
- Utilizar una librería JWT.
- Utilizar variables de entorno para la clave secreta.
]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, HTTPException, Header
from pydantic import BaseModel
from jose import jwt, JWTError
from datetime import datetime, timedelta
import os

app = FastAPI(title="Tu flujo JWT se verá así")

SECRET_KEY = os.getenv("SECRET_KEY", "clave_pruebas")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

USUARIO = "ana"
PASSWORD = "secreta"

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

class LoginRequest(BaseModel):
    usuario: str
    password: str


@app.post("/api/token")
def crear_token(datos: LoginRequest):

    if datos.usuario != USUARIO or datos.password != PASSWORD:
        raise HTTPException(status_code=401, detail="Credenciales incorrectas")

    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)

    payload = {
        "sub": datos.usuario,
        "exp": expire
    }

    token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

    return {
        "access_token": token,
        "token_type": "bearer"
    }


@app.get("/api/perfil")
def perfil(authorization: str = Header(None)):

    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="No autorizado")

    token = authorization.replace("Bearer ", "")

    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        usuario = payload.get("sub")

        if not usuario:
            raise HTTPException(status_code=401, detail="Token inválido")

        return {"usuario": usuario}

    except JWTError:
        raise HTTPException(status_code=401, detail="Token inválido")
```

---
## Evidencia Verificador
### 1

==============================================================================

VERIFICACIÓN DEL RESULTADO · 4 casos · app.py

==============================================================================

✅ 01. El login emite un token

✅ 02. El token válido abre el perfil

✅ 03. Un token alterado es rechazado

✅ 04. Sin token no hay acceso

------------------------------------------------------------------------------

RESULTADO: 4/4 casos correctos. ¡Resultado logrado!

### 2 

✅ Se usa una librería JWT

------------------------------------------------------------------------------

RESULTADO: 1/1 comprobaciones correctas. ¡Resultado logrado!


## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas con tokens alterados o firma inválida

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 
Una de las cosas que revisé fue que los tokens modificados no provocaran errores internos en la aplicación. En lugar de generar un error 500, el sistema debía responder correctamente con un 401 indicando que el token no era válido. También confirmé que no fuera posible acceder al perfil cuando faltaba el encabezado Authorization o cuando el token había sido alterado.


---
**Firma digital:** [EuniceUT]
