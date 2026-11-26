# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** OAuth2 Password Flow y Header Authorization Bearer  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Extracción manual del encabezado Authorization  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue comprender cómo se envía y valida el encabezado Authorization. Al inicio no tenía claro la diferencia entre enviar usuario y contraseña en el formulario de login y enviar posteriormente el token en el encabezado Bearer para acceder a una ruta protegida

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación en FastAPI ejecutada en Google Colab. Se debe implementar OAuth2 Password Flow usando formulario para el login y token Bearer para proteger rutas. |
| **Rol (R)** | Desarrollador backend encargado de autenticar usuarios, generar tokens y proteger recursos. |
| **Especificación (E)** | Crear POST /token que reciba username y password por formulario. El usuario válido es ana y la contraseña secreta. Debe devolver un token Bearer. Crear GET /api/perfil protegido con OAuth2. Con token válido responde el usuario; sin token o con otro esquema responde 401 con WWW-Authenticate: Bearer. |
| **Output (O)** | Archivo app.py funcional usando OAuth2PasswordRequestForm y OAuth2PasswordBearer, cumpliendo todas las pruebas del laboratorio. |

### 2.2 Prompt Estructurado
```text
[
Contexto: Aplicación en FastAPI ejecutada en Google Colab. Se debe implementar OAuth2 Password Flow usando formulario para el login y token Bearer para proteger rutas.
Rol: Desarrollador backend encargado de autenticar usuarios, generar tokens y proteger recursos.
Especificación: Crear POST /token que reciba username y password por formulario. El usuario válido es ana y la contraseña secreta. Debe devolver un token Bearer. Crear GET /api/perfil protegido con OAuth2. Con token válido responde el usuario; sin token o con otro esquema responde 401 con WWW-Authenticate: Bearer.
Output: Archivo app.py funcional usando OAuth2PasswordRequestForm y OAuth2PasswordBearer, cumpliendo todas las pruebas del laboratorio.
]
```

---

## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

app = FastAPI(title="OAuth2 Password Flow")

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

# OAuth2 espera obtener el token desde /token
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

USUARIO = "ana"
PASSWORD = "secreta"


@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    if form_data.username == USUARIO and form_data.password == PASSWORD:
        return {
            "access_token": USUARIO,
            "token_type": "bearer"
        }

    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Credenciales incorrectas"
    )


@app.get("/api/perfil")
async def perfil(token: str = Depends(oauth2_scheme)):
    if token != USUARIO:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token inválido",
            headers={"WWW-Authenticate": "Bearer"},
        )

    return {"usuario": USUARIO}
```

---
## Evidencia Verificador
### 1

==============================================================================

VERIFICACIÓN DEL RESULTADO · 4 casos · app.py

==============================================================================

✅ 01. El formulario de login emite token

✅ 02. El token Bearer abre el perfil

✅ 03. Sin token se responde 401 con WWW-Authenticate

✅ 04. Un esquema distinto se rechaza

------------------------------------------------------------------------------

RESULTADO: 4/4 casos correctos. ¡Resultado logrado!

### 2 

✅ Se usa el flujo OAuth2 de FastAPI

------------------------------------------------------------------------------

RESULTADO: 1/1 comprobaciones correctas. ¡Resultado logrado!


## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso no autorizado sin token (HTTP 401)

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> Durante las pruebas detecté que no bastaba con responder únicamente un error 401 cuando faltaba el token. También era necesario incluir el encabezado WWW-Authenticate: Bearer, ya que forma parte del contrato OAuth2. Al revisar el comportamiento de la aplicación, confirmé que este encabezado estuviera presente para que la verificación fuera exitosa.

---
**Firma digital:** [EuniceUT]
