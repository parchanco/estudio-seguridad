# Bloque 8 — Seguridad de APIs

> Objetivo: las APIs tienen su propio Top 10 de OWASP porque cambian las reglas
> respecto a la web clásica — sin UI que oculte nada, todo es el endpoint. Rate
> limiting, mass assignment, exceso de datos y validación de esquemas.

## 1. Qué cambia en las APIs

Una API REST/GraphQL expone la lógica **directamente**, sin una interfaz web que
medie. Esto cambia el modelo de amenaza:
- **No hay UI que "oculte" nada** (bloque 5: nunca debías confiar en ello, pero
  en APIs es aún más evidente): el endpoint ES la superficie.
- Los clientes son diversos (móvil, SPA, otros servicios) y todos manipulables.
- Autenticación por **tokens** (bloque 4) más que por sesiones/cookies.

OWASP tiene un **API Security Top 10** aparte. Los riesgos más propios:

## 2. Broken Object Level Authorization (el IDOR de APIs)

El #1 del API Top 10 — es el IDOR del bloque 5, y en APIs es rampante:
`GET /api/users/123/orders` — ¿comprueba el servidor que el usuario autenticado
ES el 123 o puede tener sus permisos? Como las APIs exponen IDs por todas
partes, olvidar la comprobación de propiedad en **un solo** endpoint filtra
datos. La regla del bloque 5 aplica reforzada: **cada endpoint comprueba
autorización a nivel de objeto.**

## 3. Mass assignment

Cuando un endpoint acepta un objeto (JSON) y lo vuelca directamente al modelo/BD
sin filtrar qué campos permite:

```python
# ❌ VULNERABLE: acepta CUALQUIER campo que mande el cliente
@app.post("/api/users/{id}")
def update(id, data: dict):
    user = User.get(id)
    user.update(**data)        # ¿y si el cliente manda {"is_admin": true}?
```

El atacante añade campos que no debería poder tocar (`is_admin`, `role`,
`balance`, `verified`) y escala privilegios o manipula datos. La cura:
**allowlist explícita** de campos permitidos (nunca volcar el input crudo), o
usar esquemas que solo aceptan los campos definidos (punto 5). Es "parse, don't
validate" (curso 01 extra): define exactamente qué entra.

## 4. Excessive data exposure y otros

- **Excessive data exposure**: la API devuelve el objeto entero (con campos
  internos: hashes de contraseña, flags, datos de otros) confiando en que el
  cliente "solo muestre lo que debe". El atacante mira la respuesta cruda. Cura:
  devuelve solo los campos necesarios (DTOs de salida explícitos — curso de
  arquitectura, qué cruza los límites).
- **Rate limiting** (system-design bloque 11, algoritmos extra): sin él, las
  APIs son vulnerables a fuerza bruta (probar contraseñas/tokens), scraping
  masivo y DoS. Limita por usuario/IP/endpoint (token bucket). Especialmente en
  login, reset de contraseña, y endpoints caros.
- **Versionado y endpoints viejos**: una `/api/v1` obsoleta y sin mantener con
  vulnerabilidades sin parchear es una puerta trasera. Retira lo que no usas
  (superficie mínima).
- **BOLA/BFLA a nivel de función**: además de objetos, comprueba autorización a
  nivel de función (¿este usuario puede llamar a este endpoint de admin?).

## 5. Validación de esquemas en la frontera

La defensa transversal de las APIs: **validar la entrada contra un esquema
estricto en la frontera**, antes de que toque tu lógica. En Python, **Pydantic**
(curso 01 extra, FastAPI lo usa de serie):

```python
class UserUpdate(BaseModel):        # SOLO estos campos, con estos tipos
    name: str
    email: EmailStr
    # is_admin NO está → aunque el cliente lo mande, se ignora (anti mass assignment)

@app.post("/api/users/{id}")
def update(id: int, data: UserUpdate):   # FastAPI rechaza lo que no encaje
    ...
```

El esquema hace tres cosas de seguridad a la vez: rechaza tipos/formatos
inválidos (anti-inyección, bloque 2), define exactamente qué campos entran
(anti mass assignment), y documenta el contrato. "Parse, don't validate": en vez
de comprobar que la entrada no es mala, la conviertes en un tipo que **solo
puede ser bueno**.

## 6. Práctica del bloque

1. En una API tuya (o de práctica), busca un BOLA/IDOR: un endpoint que devuelve
   un recurso por ID sin comprobar propiedad. Explótalo y arréglalo.
2. Reproduce mass assignment: un endpoint que vuelca el JSON al modelo; mándale
   un campo privilegiado (`is_admin`) y observa el efecto. Arréglalo con un
   esquema/allowlist.
3. Revisa las respuestas de una API (DevTools/curl): ¿devuelve campos que no
   deberían salir? Recórtalos con un DTO de salida.
4. Añade rate limiting a un endpoint de login y verifica que bloquea el
   brute-force (muchas peticiones seguidas).

## Recursos

- 🌐 OWASP **API Security Top 10** (owasp.org/API-Security) — la referencia
  específica de este bloque.
- 📄 Docs de Pydantic/FastAPI sobre validación; system-design bloque 11 (rate
  limiting).

## Autoevaluación

1. ¿Qué cambia en el modelo de amenaza de una API respecto a la web clásica?
2. ¿Qué es BOLA y por qué es el #1 del API Top 10?
3. Explica mass assignment con un ejemplo y su cura.
4. ¿Qué es excessive data exposure y cómo lo evitas?
5. ¿Cómo un esquema (Pydantic) da tres defensas a la vez? ¿Qué es "parse, don't
   validate"?

✅ **Checklist de salida:**
- [ ] Compruebas autorización a nivel de objeto en cada endpoint.
- [ ] Usas allowlists/esquemas contra mass assignment.
- [ ] Devuelves solo los campos necesarios (DTOs de salida).
- [ ] Validas la entrada con esquemas estrictos y pones rate limiting.

**Siguiente:** [Bloque 9 — Seguridad en contenedores y cloud](09-contenedores-cloud.md)
