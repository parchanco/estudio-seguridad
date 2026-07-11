# Bloque 0 — Mapa mental: pensar como atacante para defender

> Objetivo: instalar la mentalidad de seguridad — toda entrada es hostil, la
> defensa es en capas — y los principios que gobiernan todo el curso. Montar el
> laboratorio para atacar sin romper nada real. **Todo es defensivo y ético.**

## 1. La mentalidad: toda entrada es hostil

El cambio de chip que define la seguridad de aplicaciones:

> **Toda entrada que no controlas es hostil hasta que se demuestre lo
> contrario.** Datos de formularios, parámetros de URL, cabeceras HTTP,
> ficheros subidos, respuestas de APIs de terceros, incluso datos de tu propia
> BD (pudieron entrar comprometidos).

El desarrollador normal piensa "¿cómo hago que funcione?"; el que piensa en
seguridad añade "¿cómo se puede abusar de esto?". No es paranoia — es que hay
gente activamente intentando romper lo que construyes, y solo hace falta que
acierten una vez. Este curso te enseña a pensar como el atacante **para
defender mejor**, practicando siempre en entornos propios.

## 2. Los principios que gobiernan todo

Cuatro ideas de las que se derivan casi todas las defensas del curso:

- **Defensa en profundidad**: varias capas, no una. Si una falla (y fallará),
  otra protege. No confíes solo en la validación del frontend, ni solo en el
  firewall, ni solo en una cosa.
- **Mínimo privilegio**: cada componente/usuario tiene solo los permisos que
  necesita, nada más. Un servicio que solo lee no tiene permiso de escritura;
  un contenedor no corre como root (curso-docker, m8). Limita el daño de un
  compromiso.
- **Fail secure**: ante un error o duda, deniega. Si la comprobación de permisos
  falla con excepción, NO dejes pasar "por si acaso" (curso 07: crash early
  aplicado a seguridad — mejor rechazar que permitir dudoso).
- **Superficie de ataque mínima**: cada endpoint, puerto, dependencia y feature
  es una puerta potencial. Menos superficie = menos que proteger (curso de
  arquitectura: menos piezas; curso-docker, m7: imágenes mínimas).

## 3. El vocabulario base

- **CIA** (la tríada de la seguridad): **Confidencialidad** (solo quien debe ve
  los datos), **Integridad** (los datos no se alteran sin autorización),
  **Disponibilidad** (el sistema sigue funcionando — los DoS atacan esto).
- **Autenticación vs autorización** (se confunden constantemente):
  - **Autenticación** (authn): ¿quién eres? Demostrar identidad (login).
    (Bloque 4.)
  - **Autorización** (authz): ¿qué puedes hacer? Permisos una vez identificado.
    (Bloque 5.)
  - Un fallo típico: autenticar bien pero autorizar mal (el usuario logueado
    accede a datos de otro — IDOR, bloque 5).
- **Vulnerabilidad** (el fallo) vs **exploit** (el ataque que lo aprovecha) vs
  **amenaza** (quién/qué podría atacar).

## 4. El laboratorio (ético)

Para aprender atacas — pero solo entornos diseñados para ello. Nunca contra
sistemas de terceros (es ilegal y va contra el espíritu del curso). Monta un
laboratorio local con Docker (curso-docker):

```bash
# OWASP Juice Shop: una app deliberadamente vulnerable para practicar
docker run -d -p 3000:3000 bkimminich/juice-shop
# OWASP WebGoat: lecciones guiadas de cada vulnerabilidad
docker run -d -p 8080:8080 -p 9090:9090 webgoat/webgoat
```

Herramientas del curso: el navegador con sus DevTools, **Burp Suite** (o OWASP
ZAP) como proxy para interceptar/modificar peticiones, y `curl` para
peticiones a mano. Complementa con **PortSwigger Web Security Academy**
(labs online gratuitos y excelentes).

## 5. Práctica del bloque

1. Levanta Juice Shop o WebGoat en Docker. Explora la app como usuario normal
   primero, y luego pregúntate en cada campo: "¿qué pasaría si meto algo
   inesperado aquí?".
2. Instala un proxy (ZAP/Burp) y aprende a interceptar una petición: mándala
   modificada y observa. Esta es la habilidad base de todo el curso.
3. Para una app tuya, dibuja su superficie de ataque: todos los puntos de
   entrada (endpoints, formularios, uploads, integraciones). Es tu mapa de qué
   proteger.
4. Clasifica tres incidentes de seguridad famosos que conozcas según la tríada
   CIA: ¿atacaron confidencialidad, integridad o disponibilidad?

## Recursos

- 🌐 OWASP Top 10 (owasp.org) — el estándar; lo recorreremos entero.
- 🌐 PortSwigger Web Security Academy — los mejores labs gratuitos de AppSec.
- 🐳 OWASP Juice Shop / WebGoat — apps vulnerables para practicar en local.

## Autoevaluación

1. ¿Qué significa "toda entrada es hostil" y qué entradas incluye (más de las
   obvias)?
2. Explica defensa en profundidad, mínimo privilegio, fail secure y superficie
   mínima con un ejemplo de cada uno.
3. Distingue autenticación de autorización con un fallo típico de confundirlas.
4. ¿Qué es la tríada CIA? Da un ataque contra cada componente.
5. ¿Por qué el laboratorio debe ser propio/diseñado para ello?

✅ **Checklist de salida:**
- [ ] Piensas "¿cómo se abusa de esto?" además de "¿cómo funciona?".
- [ ] Conoces los cuatro principios y los reconoces en las defensas.
- [ ] Distingues authn de authz y la tríada CIA.
- [ ] Tienes un laboratorio ético montado y sabes interceptar peticiones.

**Siguiente:** [Bloque 1 — Criptografía para desarrolladores](01-criptografia.md)
