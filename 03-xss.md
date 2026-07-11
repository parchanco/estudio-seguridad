# Bloque 3 — XSS y seguridad del navegador

> Objetivo: Cross-Site Scripting (inyección de código en el navegador de la
> víctima) y el modelo de seguridad del navegador que lo rodea — Same-Origin
> Policy, CORS, CSP y cookies seguras. Lo que todo desarrollador frontend/web
> debe llevar puesto.

## 1. Qué es XSS

**XSS** es inyección (bloque 2) pero en el navegador: el atacante consigue que
**su JavaScript se ejecute en el navegador de OTRO usuario**, en el contexto de
tu sitio. Con eso puede robar sesiones/cookies, hacer acciones en nombre de la
víctima, capturar lo que teclea, o desfigurar la página. La raíz, como toda
inyección: **datos del usuario renderizados como HTML/JS sin escapar**.

## 2. Los tres tipos de XSS

- **Reflejado (reflected)**: el payload viaja en la petición (un parámetro de
  URL) y se refleja en la respuesta sin escapar. El atacante manda a la víctima
  un enlace malicioso; al abrirlo, el script se ejecuta. `?q=<script>...</script>`.
- **Almacenado (stored)**: el payload se **guarda** en el servidor (un comentario,
  un nombre de perfil) y se sirve a todos los que ven esa página. El más
  peligroso — afecta a cualquiera que visite, sin necesidad de un enlace
  trampa. Un comentario con `<script>` que roba la cookie de cada visitante.
- **DOM-based**: el JavaScript del propio cliente coge datos no confiables
  (de la URL, por ejemplo) y los mete en el DOM de forma insegura
  (`element.innerHTML = location.hash`). Ocurre enteramente en el navegador.

## 3. Las defensas contra XSS

- **Escapado por contexto** (la principal): al insertar datos en HTML,
  escápalos según DÓNDE van (cuerpo HTML, atributo, JavaScript, URL — cada uno
  tiene reglas distintas). La buena noticia: **los frameworks modernos escapan
  por defecto** — React, Vue, Django templates, Jinja2 auto-escapan el HTML.
  El peligro está cuando los saltas: `dangerouslySetInnerHTML` (React),
  `|safe` (Jinja), `v-html` (Vue), `innerHTML` — ahí eres responsable tú.
- **Sanitización** cuando SÍ necesitas permitir HTML del usuario (un editor
  rico): usa una librería como **DOMPurify** que quita lo peligroso y deja lo
  seguro. Nunca sanees a mano con regex.
- **Content Security Policy (CSP)**: una cabecera que le dice al navegador de
  qué orígenes puede cargar/ejecutar scripts. Una buena CSP puede **neutralizar
  XSS** aunque se cuele (el script inyectado no se ejecuta porque viola la
  política). Defensa en profundidad (bloque 0).

## 4. El modelo de seguridad del navegador

Las reglas que hacen (relativamente) seguro navegar:

- **Same-Origin Policy (SOP)**: el JavaScript de un origen (esquema + dominio +
  puerto) NO puede acceder a los datos de otro origen. Sin esto, cualquier web
  podría leer tu sesión de tu banco abierta en otra pestaña. Es la piedra
  angular.
- **CORS (Cross-Origin Resource Sharing)**: SOP es estricta, pero a veces
  necesitas que tu frontend en `app.com` llame a tu API en `api.com`. CORS es
  el mecanismo por el que el servidor **relaja** SOP de forma controlada
  (cabeceras `Access-Control-Allow-Origin`). Malentendido común: CORS NO
  "protege" tu API — es el servidor **permitiendo** ciertos orígenes. Un
  `Access-Control-Allow-Origin: *` mal puesto abre tu API a cualquiera.
- **Cookies seguras** — las banderas que hay que poner siempre en las de sesión:
  - **HttpOnly**: el JavaScript NO puede leer la cookie → aunque haya XSS, no
    roban la sesión por ahí.
  - **Secure**: solo se envía por HTTPS.
  - **SameSite** (Lax/Strict): no se envía en peticiones cross-site → mitiga
    CSRF (bloque 5).

## 5. Práctica del bloque

1. En Juice Shop/WebGoat, explota un XSS reflejado y uno almacenado. Observa
   cómo tu script se ejecuta en el contexto del sitio (un `alert` basta para
   demostrarlo — no hagas nada dañino).
2. En una app tuya con framework, comprueba que auto-escapa: intenta inyectar
   `<script>` en un campo y verifica que se muestra como texto, no se ejecuta.
   Luego usa la vía insegura (`|safe`/`innerHTML`) y observa la diferencia.
3. Añade una cabecera CSP a una página y comprueba que bloquea scripts inline.
4. Inspecciona las cookies de una app (DevTools): ¿tienen HttpOnly, Secure,
   SameSite? Añádelas si faltan.

## Recursos

- 🌐 OWASP "XSS Prevention Cheat Sheet" y "Content Security Policy Cheat Sheet".
- 🌐 PortSwigger Academy: labs de XSS y de CORS.
- 📄 MDN sobre Same-Origin Policy, CORS y cookies.

## Autoevaluación

1. ¿Qué es XSS y qué puede lograr un atacante con él?
2. Distingue XSS reflejado, almacenado y DOM-based. ¿Cuál es el más peligroso y
   por qué?
3. ¿Cómo protegen los frameworks modernos y cómo los saltas por accidente?
4. ¿Qué es la Same-Origin Policy y qué hace CORS realmente (no "protege")?
5. ¿Qué hacen HttpOnly, Secure y SameSite en una cookie de sesión?

✅ **Checklist de salida:**
- [ ] Entiendes XSS como inyección en el navegador y sus tres tipos.
- [ ] Confías en el auto-escapado del framework y sabes dónde lo saltas.
- [ ] Distingues SOP de CORS y no abres tu API con `*`.
- [ ] Pones HttpOnly/Secure/SameSite en las cookies de sesión.

**Siguiente:** [Bloque 4 — Autenticación](04-autenticacion.md)
