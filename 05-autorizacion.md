# Bloque 5 — Autorización

> Objetivo: una vez sabes quién es el usuario (authn, bloque 4), controlar QUÉ
> puede hacer. El broken access control es el #1 del OWASP Top 10 — IDOR,
> elevación de privilegios, el error mortal de confiar en el cliente, y CSRF.

## 1. Broken access control: el #1 del OWASP Top 10

La autorización rota es la vulnerabilidad **más común** según OWASP. La idea:
el usuario está autenticado (sabemos quién es) pero puede hacer o ver cosas que
NO le corresponden. Es tan frecuente porque la autorización hay que comprobarla
en **cada** acción y cada recurso, y es fácil olvidar una.

## 2. IDOR: el fallo más común

**IDOR (Insecure Direct Object Reference)**: acceder a recursos de otro usuario
cambiando un identificador, porque el servidor no comprueba que el recurso te
pertenece:

```python
# ❌ VULNERABLE: devuelve el pedido SIN comprobar que es del usuario logueado
@app.get("/pedidos/{pedido_id}")
def ver_pedido(pedido_id):
    return db.get_pedido(pedido_id)     # ¿y si pido el pedido 12345 de OTRO?

# ✅ SEGURO: comprueba la propiedad
@app.get("/pedidos/{pedido_id}")
def ver_pedido(pedido_id, usuario=current_user):
    pedido = db.get_pedido(pedido_id)
    if pedido.usuario_id != usuario.id:
        abort(403)                       # no es tuyo → prohibido
    return pedido
```

El atacante simplemente incrementa el ID en la URL (`/pedidos/1`, `/pedidos/2`...)
y ve datos ajenos. Es trivial de explotar y devastador (fugas masivas de datos
han sido esto). La regla: **cada acceso a un recurso comprueba que el usuario
tiene derecho a ESE recurso**, no solo que está logueado. Usar IDs no
adivinables (UUID) ayuda pero NO sustituye la comprobación (seguridad por
oscuridad no es seguridad).

## 3. El error mortal: confiar en el cliente

El fallo conceptual que subyace a muchos: **la seguridad del lado del cliente no
existe.** Ocultar un botón de "borrar" a los no-admin en el frontend NO es
autorización — el atacante llama al endpoint directamente (con curl/Burp,
saltándose tu UI). Todo lo que llega del cliente es manipulable (bloque 0):
parámetros, cabeceras, cookies, el JWT, campos "ocultos" de formularios, precios
en el carrito...

> **La autorización SIEMPRE se comprueba en el servidor, en cada endpoint.** El
> frontend puede ocultar cosas por UX, pero la puerta de verdad está en el
> backend.

Ejemplo clásico: un endpoint de admin que confía en un parámetro `?admin=true`
o en que "solo la UI de admin lo llama". El atacante lo llama a mano.

## 4. Modelos de autorización

Cómo estructurar los permisos:
- **RBAC (Role-Based Access Control)**: permisos por **roles** (admin, editor,
  viewer). El usuario tiene roles, los roles tienen permisos. Simple y suficiente
  para la mayoría.
- **ABAC (Attribute-Based)**: permisos por **atributos/reglas** ("puede editar
  si es el autor Y el documento no está publicado Y es horario laboral"). Más
  flexible, más complejo. Para lógica de negocio fina.
- **Comprueba en cada capa**, no solo en la UI: en el endpoint, y a menudo
  también a nivel de dato (¿este usuario puede ver ESTA fila?). Defensa en
  profundidad (bloque 0).

Principio rector: **mínimo privilegio** (bloque 0) — por defecto denegar, y
conceder solo lo necesario.

## 5. CSRF: hacer que la víctima actúe sin querer

**CSRF (Cross-Site Request Forgery)**: el atacante engaña al navegador de una
víctima **autenticada** para que haga una petición no deseada a tu sitio. Como
el navegador envía las cookies automáticamente, si la víctima está logueada en
`tubanco.com` y visita una web maliciosa, esa web puede disparar
`POST tubanco.com/transferir` y el navegador adjunta la cookie de sesión.

Defensas:
- **Tokens anti-CSRF**: un token secreto único por formulario/sesión que el
  atacante no puede conocer; el servidor lo exige en cada acción que cambia
  estado. Los frameworks lo traen (Django CSRF middleware).
- **SameSite cookies** (bloque 3): `SameSite=Lax/Strict` hace que la cookie NO
  se envíe en peticiones cross-site → mitiga CSRF de raíz. Hoy es la primera
  línea.
- Nota: las APIs con tokens en cabecera (no cookies) son inmunes a CSRF clásico
  (el navegador no adjunta cabeceras automáticamente) — por eso afecta sobre
  todo a apps basadas en cookies de sesión.

## 6. Práctica del bloque

1. En Juice Shop/WebGoat, explota un IDOR: accede a datos/pedidos de otro
   usuario cambiando un ID. Luego escribe la comprobación de propiedad que lo
   arregla.
2. Demuestra el error de confiar en el cliente: encuentra una acción "oculta"
   en el frontend de una app de prueba y llámala directamente con curl/Burp
   saltándote la UI.
3. Implementa RBAC básico (roles y una comprobación de permiso por endpoint) en
   una app pequeña.
4. Reproduce un CSRF (un formulario en una página "maliciosa" que apunta a tu
   app) y protégelo con un token anti-CSRF y SameSite.

## Recursos

- 🌐 OWASP "Authorization Cheat Sheet", A01:Broken Access Control, "CSRF
  Prevention Cheat Sheet".
- 🌐 PortSwigger Academy: labs de access control y de CSRF.

## Autoevaluación

1. ¿Por qué el broken access control es tan común? ¿Qué es un IDOR y cómo se
   arregla?
2. ¿Por qué "la seguridad del lado del cliente no existe"? Da un ejemplo de
   confiar en el cliente.
3. Distingue RBAC de ABAC y cuándo usar cada uno.
4. ¿Qué es CSRF y por qué funciona (el papel de las cookies automáticas)?
5. ¿Cómo mitigan CSRF los tokens anti-CSRF y SameSite? ¿Por qué las APIs con
   token en cabecera son menos vulnerables?

✅ **Checklist de salida:**
- [ ] Compruebas la propiedad del recurso en cada acceso (no solo el login).
- [ ] Nunca confías en el cliente; autorizas en el servidor, en cada endpoint.
- [ ] Estructuras permisos con RBAC/ABAC y mínimo privilegio.
- [ ] Proteges las acciones con estado contra CSRF (SameSite + tokens).

**Siguiente:** [Bloque 6 — Secretos y configuración](06-secretos.md)
