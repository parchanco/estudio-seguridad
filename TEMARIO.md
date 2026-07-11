# Temario — Seguridad de Aplicaciones

> Basado en el OWASP Top 10 y las Cheat Sheets de OWASP, con laboratorios
> prácticos (WebGoat, PortSwigger Academy). Enfoque 100% defensivo: entender
> el ataque para escribir código que lo resista. Practicar SOLO en entornos
> propios o diseñados para ello.

Criterio de "terminado" por punto: puedes explicar el ataque, reproducirlo en
un laboratorio, y escribir el código que lo previene.

---

## Bloque 0 — Mapa mental: pensar como atacante para defender

- [ ] El modelo mental: toda entrada del usuario es hostil hasta que se demuestre lo contrario
- [ ] Superficie de ataque, defensa en profundidad, mínimo privilegio, fail-secure — los principios que gobiernan todo
- [ ] Confidencialidad, integridad, disponibilidad (CIA); autenticación vs autorización
- [ ] El setup: un laboratorio local (WebGoat/Juice Shop en Docker) para atacar sin romper nada real

## Bloque 1 — Criptografía para desarrolladores

- [ ] Simétrico vs asimétrico, hashing vs cifrado vs codificación (los tres se confunden) — amplía telecom bloque 9
- [ ] Hashing de contraseñas: por qué NUNCA MD5/SHA sueltos — bcrypt/argon2, salt, coste
- [ ] TLS por dentro: qué garantiza y qué no, certificados, HSTS
- [ ] No inventes cripto: usa librerías probadas; el peligro de los "roll your own"

## Bloque 2 — Inyección (SQL y amigos)

- [ ] SQL injection: cómo funciona, reproducirla, y la ÚNICA defensa real (consultas parametrizadas, nunca concatenar) — conexión: estudio-bases-de-datos
- [ ] ORMs: cuándo te protegen y cuándo los rompes (raw queries)
- [ ] Otras inyecciones: comandos (os.system con input), LDAP, NoSQL
- [ ] Validación en la frontera (parse, don't validate) como estrategia general

## Bloque 3 — XSS y seguridad del navegador

- [ ] Los tres XSS: reflejado, almacenado, DOM-based — reproducir cada uno
- [ ] Defensas: escapado por contexto (HTML/JS/URL), Content Security Policy, sanitización
- [ ] El modelo de seguridad del navegador: Same-Origin Policy, CORS (qué protege de verdad y qué no)
- [ ] Cookies seguras: HttpOnly, Secure, SameSite

## Bloque 4 — Autenticación

- [ ] Sesiones vs tokens: cookies de sesión vs JWT — trade-offs reales (revocación, tamaño, XSS vs CSRF)
- [ ] JWT bien hecho: firma, expiración, los errores clásicos (alg:none, secreto débil, no validar)
- [ ] MFA/2FA (TOTP), passkeys/WebAuthn a nivel de idea
- [ ] OAuth2 y OpenID Connect: los flujos (authorization code + PKCE), qué resuelve cada uno — el login social sin magia

## Bloque 5 — Autorización

- [ ] Broken access control (el #1 del OWASP Top 10): IDOR, elevación de privilegios
- [ ] Modelos: RBAC vs ABAC; autorización en cada capa, no solo en la UI
- [ ] El error mortal: confiar en el cliente (ocultar un botón no es seguridad)
- [ ] CSRF: cómo funciona, tokens anti-CSRF, y por qué SameSite lo mitiga

## Bloque 6 — Secretos y configuración

- [ ] Gestión de secretos: nunca en git (y qué hacer si ya están), variables de entorno, vaults (conexión: docker, aws)
- [ ] Security misconfiguration: defaults inseguros, verbose errors, directorios expuestos, CORS abierto
- [ ] Cabeceras de seguridad: CSP, HSTS, X-Frame-Options, etc.
- [ ] Escaneo de secretos en CI (gitleaks, trufflehog)

## Bloque 7 — Dependencias y cadena de suministro

- [ ] Vulnerabilidades conocidas (CVEs): el riesgo que heredas de tus dependencias
- [ ] Escaneo: pip-audit, npm audit, Dependabot, SBOM a nivel de idea
- [ ] Ataques de supply chain: typosquatting, dependencias comprometidas — el caso real que conviene conocer
- [ ] Pinning, lockfiles y actualizaciones con criterio

## Bloque 8 — Seguridad de APIs

- [ ] OWASP API Top 10: qué cambia respecto al web clásico
- [ ] Rate limiting como defensa (token bucket, extra de algoritmos) contra fuerza bruta y abuso
- [ ] Mass assignment, exceso de datos expuestos, versionado
- [ ] Validación de esquemas (Pydantic en la frontera — conexión: curso 01 extra)

## Bloque 9 — Seguridad en contenedores y cloud

- [ ] Imágenes: mínimas (distroless), no-root, escaneo (trivy) — conexión directa: curso-docker
- [ ] Secretos en Kubernetes y en AWS (IAM roles, Secrets Manager) — conexión: curso-kubernetes, aws-architect
- [ ] El modelo de responsabilidad compartida (telecom bloque 15) desde la óptica de seguridad
- [ ] Superficie de ataque de un contenedor: qué expones y qué aíslas (namespaces, del curso de SO)

## Bloque 10 — Diseñar con seguridad (shift left)

- [ ] Threat modeling ligero: STRIDE, "¿qué puede salir mal?" en el diseño (conexión: ADRs del curso de arquitectura)
- [ ] SDLC seguro: dónde entra la seguridad en cada fase; security review (conexión: /security-review)
- [ ] Logging seguro: qué NO loguear (secretos, PII), y detectar sin exponer
- [ ] La cultura: seguridad como responsabilidad de todos, no de un equipo aparte

## 🎓 Proyecto final

- [ ] Auditar (y arreglar) una aplicación deliberadamente vulnerable (OWASP Juice Shop o una propia): encontrar y explotar 8-10 vulnerabilidades del Top 10 en el laboratorio, documentar cada una con impacto y fix, y aplicar el fix
- [ ] Añadir a un proyecto tuyo real: escaneo de secretos y dependencias en CI, cabeceras de seguridad, y un threat model de una página

---

## Orden recomendado

1. Bloques 0-1 (mentalidad y cripto) — la base
2. Bloques 2-5 (los ataques core: inyección, XSS, auth, authz) — el 80% del Top 10
3. Bloques 6-8 (secretos, dependencias, APIs) — la higiene diaria
4. Bloques 9-10 (contenedores/cloud y diseño) — tras o en paralelo a docker/k8s/aws
5. Proyecto final atacando un laboratorio de verdad

## Notas de progreso

- Fecha de inicio:
- Ritmo objetivo:
