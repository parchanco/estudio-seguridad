# Bloque 4 — Autenticación

> Objetivo: verificar identidad de forma segura — sesiones vs tokens (JWT), sus
> trade-offs reales, MFA, y OAuth2/OIDC (el login social sin magia). Authn es
> "¿quién eres?"; la siguiente puerta (authz, bloque 5) es "¿qué puedes hacer?".

## 1. Sesiones vs tokens: los dos modelos

Tras un login correcto, ¿cómo recuerda el servidor quién eres en las siguientes
peticiones? Dos enfoques:

- **Sesiones (server-side)**: el servidor crea una sesión, la guarda (en
  memoria/Redis/BD) y le da al cliente un **ID de sesión** en una cookie. En
  cada petición, busca la sesión por ese ID. **Estado en el servidor.**
- **Tokens (JWT)**: el servidor crea un **token firmado** que contiene los datos
  (quién eres, permisos, expiración) y se lo da al cliente. En cada petición, el
  cliente lo envía; el servidor **verifica la firma** sin consultar nada.
  **Sin estado en el servidor.**

| | Sesiones | JWT |
|---|---|---|
| Estado | En el servidor | En el token (cliente) |
| **Revocación** | Fácil (borra la sesión) | **Difícil** (el token vale hasta que expira) |
| Escala horizontal | Necesita estado compartido (Redis) | Sin estado, escala fácil |
| Tamaño | ID pequeño | Token grande en cada petición |
| Riesgo si se roba | Robas una sesión (revocable) | Robas un token (válido hasta expirar) |

El malentendido común: "JWT es más moderno/mejor". No — es un **trade-off**. Los
JWT brillan en microservicios/APIs sin estado, pero la **revocación** es su
talón de Aquiles (no puedes "cerrar sesión" de verdad de un JWT robado hasta que
caduca). Solución habitual: tokens de acceso de **vida corta** (minutos) +
refresh tokens revocables. Para una app web monolítica, las sesiones suelen ser
más simples y seguras.

## 2. JWT bien hecho (y los errores clásicos)

Un JWT tiene tres partes (header.payload.signature), en Base64 — recuerda:
**Base64 no es cifrado** (bloque 1), el payload es **legible por cualquiera**.
No metas secretos en un JWT. La seguridad viene de la **firma**, no de ocultar.

Los errores que rompen JWT (los verás en labs):
- **`alg: none`**: si el servidor acepta el algoritmo "none", cualquiera forja
  tokens sin firma. Rechaza siempre `none` y fija el algoritmo esperado.
- **Secreto débil**: si firmas con HMAC y una clave débil, se rompe por fuerza
  bruta. Clave larga y aleatoria.
- **No validar la expiración** o la firma: verifica SIEMPRE `exp` y la firma
  antes de confiar en nada del payload.
- **Confundir firmar con cifrar**: si el payload lleva datos sensibles, va
  cifrado (JWE), no solo firmado (JWS).

Usa una librería probada (bloque 1: no lo hagas a mano) que valide todo esto.

## 3. MFA y passkeys

La contraseña sola es débil (se reutiliza, se filtra, se adivina). **MFA
(autenticación multifactor)** añade un segundo factor:
- Algo que sabes (contraseña) + algo que tienes (el móvil con un código
  **TOTP**, o una llave física) + algo que eres (biometría).
- **TOTP** (Google Authenticator): un código de 6 dígitos que rota cada 30s,
  derivado de un secreto compartido + la hora. Fácil de implementar.
- **Passkeys / WebAuthn**: el estándar moderno que reemplaza contraseñas por
  criptografía de clave pública (bloque 1) ligada al dispositivo — resistente a
  phishing (no hay secreto que robar). La dirección del futuro.

## 4. OAuth2 y OpenID Connect: el login social sin magia

"Iniciar sesión con Google/GitHub" usa estos protocolos, que confunden por su
nombre parecido:
- **OAuth2** es **autorización delegada**: permite a tu app acceder a recursos
  del usuario en otro servicio (leer sus repos de GitHub) **sin darle su
  contraseña**. El usuario autoriza un permiso concreto.
- **OpenID Connect (OIDC)** es una capa **de autenticación** sobre OAuth2:
  añade un **ID token** que dice "este usuario es quien dice ser". Es lo que de
  verdad usa el "login con Google".

El flujo correcto (**Authorization Code + PKCE**): tu app redirige al proveedor
(Google), el usuario se autentica allí (tu app nunca ve su contraseña), y el
proveedor devuelve un código que tu app canjea por tokens. PKCE protege el
intercambio en apps públicas (SPA, móvil). La ventaja de seguridad: delegas la
autenticación en quien la hace bien (Google) y no gestionas contraseñas.

## 5. Práctica del bloque

1. Implementa autenticación por sesión y por JWT en una app pequeña. Compara:
   cierra sesión en ambas y observa por qué revocar el JWT es el problema.
2. En un lab (PortSwigger tiene labs de JWT), explota `alg:none` o un secreto
   débil. Entiende cada error.
3. Añade TOTP (2FA) a un login usando una librería (pyotp): genera el QR,
   verifica el código. 
4. Integra "login con GitHub/Google" (OIDC) en una app de prueba y sigue el
   flujo Authorization Code: observa que tu app nunca ve la contraseña.

## Recursos

- 🌐 OWASP "Authentication Cheat Sheet" y "JSON Web Token Cheat Sheet".
- 🌐 PortSwigger Academy: labs de authentication y de JWT.
- 📄 "The OAuth 2.0 Authorization Framework" y oauth.net (los flujos explicados).

## Autoevaluación

1. Compara sesiones y JWT. ¿Cuál es el talón de Aquiles del JWT y cómo se
   mitiga?
2. ¿Por qué el payload de un JWT no es secreto? ¿De dónde viene su seguridad?
3. Da tres errores clásicos que rompen JWT.
4. ¿Qué es MFA y qué aportan TOTP y las passkeys?
5. Distingue OAuth2 de OpenID Connect. ¿Qué ventaja de seguridad da el login
   delegado?

✅ **Checklist de salida:**
- [ ] Eliges sesiones vs JWT por sus trade-offs, no por moda.
- [ ] Validas firma, expiración y algoritmo de los JWT; no metes secretos.
- [ ] Añades MFA donde importa y conoces las passkeys.
- [ ] Entiendes el flujo OAuth2/OIDC del login social.

**Siguiente:** [Bloque 5 — Autorización](05-autorizacion.md)
