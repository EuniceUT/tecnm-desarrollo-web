# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hashing Seguro de Contraseñas (Bcrypt / Argon2)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Eunice Esmeralda Uitzil Tuz]  
**Matrícula:** [23531107]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cifrado manual con Salting y Factor de Costo  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> Lo que más me costó fue entender cómo se puede validar una contraseña sin guardar la contraseña original. Al principio parecía que el sistema debía comparar textos iguales, pero después comprendí que se guarda una versión transformada de la contraseña y que existen funciones especiales para verificar si coincide sin necesidad de verla nuevamente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Aplicación desarrollada con FastAPI y Python 3 en Google Colab. Las contraseñas no deben guardarse directamente, sino mediante hashing seguro usando bcrypt a través de passlib. |
| **Rol (R)** | Actuar como desarrollador backend responsable de implementar el registro y autenticación de usuarios siguiendo buenas prácticas de seguridad. |
| **Especificación (E)** | Crear una ruta /api/registro que reciba usuario y contraseña, genere un hash de la contraseña y responda HTTP 201. Crear una ruta /api/login que compare la contraseña recibida con el hash almacenado y responda HTTP 200 cuando sea correcta o HTTP 401 cuando sea incorrecta. La API nunca debe devolver la contraseña ni el hash al cliente. Un usuario ya existente no debe sobrescribir la información almacenada. |
| **Output (O)** | Archivo app.py funcional que use bcrypt/passlib, cumpla todos los requisitos del laboratorio y pase correctamente las pruebas automáticas. |

### 2.2 Prompt Estructurado
```text
[
Actúa como un desarrollador backend especializado en FastAPI y seguridad de aplicaciones web.
Implementa una API con las rutas /api/registro y /api/login.
Requisitos:
- Usar bcrypt o passlib para el hashing de contraseñas.
- Guardar únicamente hashes y nunca contraseñas en texto plano.
- POST /api/registro debe responder HTTP 201.
- POST /api/login debe responder HTTP 200 para credenciales correctas.
- Debe responder HTTP 401 para credenciales incorrectas.
- No devolver password ni hash en ninguna respuesta.
- Evitar que un usuario existente sobrescriba su contraseña almacenada.

Devuelve únicamente el código completo del archivo app.py.
]
```

---
## Evidencia Verificador
### 1

==============================================================================

VERIFICACIÓN DEL RESULTADO · 3 casos · app.py

==============================================================================

✅ 01. El registro responde sin filtrar la contraseña

✅ 02. El login correcto autentica

✅ 03. La contraseña incorrecta se rechaza

------------------------------------------------------------------------------

RESULTADO: 3/3 casos correctos. ¡Resultado logrado!


### 2

✅ Se usa un algoritmo de hashing para contraseñas

------------------------------------------------------------------------------

RESULTADO: 1/1 comprobaciones correctas. ¡Resultado logrado!


## 3. Código Devuelto por el Modelo
```python
# %%writefile app.py
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
from passlib.context import CryptContext


app = FastAPI(title="Tu registro y login se verán así")

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

usuarios = {}

# Implementa aquí el contrato declarado en el reto.
# Tú decides los modelos, las rutas, las validaciones y el manejo de errores.

class UsuarioRequest(BaseModel):
    usuario: str
    password: str


@app.post("/api/registro", status_code=201)
def registro(datos: UsuarioRequest):

    if datos.usuario in usuarios:
        return {"usuario": datos.usuario}

    hash_password = pwd_context.hash(datos.password)

    usuarios[datos.usuario] = {
        "hash": hash_password
    }

    return {"usuario": datos.usuario}


@app.post("/api/login")
def login(datos: UsuarioRequest):

    usuario = usuarios.get(datos.usuario)

    if not usuario:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciales incorrectas"
        )

    if not pwd_context.verify(datos.password, usuario["hash"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Credenciales incorrectas"
        )

    return {
        "usuario": datos.usuario,
        "autenticado": True
    }

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas contra ataques de Rainbow Tables y Timing

* [x] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [x] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [x] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 
Al revisar el resultado noté que no bastaba con que el login funcionara. También era importante asegurar que la respuesta nunca mostrara la contraseña ni el hash almacenado. Verifiqué que solamente se devolviera la información necesaria del usuario y que los intentos con contraseñas incorrectas respondieran con un error 401 en lugar de generar comportamientos inesperados.


---
**Firma digital:** [EuniceUT]
