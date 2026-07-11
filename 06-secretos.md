# Bloque 6 — Secretos y configuración

> Objetivo: gestionar credenciales sin filtrarlas, y no exponer el sistema por
> configuración descuidada. Los secretos en git (y qué hacer si ya están), la
> security misconfiguration, y las cabeceras de seguridad.

## 1. Gestión de secretos: nunca en el código

Un **secreto** es cualquier credencial: claves de API, contraseñas de BD,
tokens, claves de firma. La regla número uno: **nunca en el código fuente ni en
git.**

Por qué es tan grave: el código se comparte, se forkea, se hace público, y **git
recuerda todo** (curso de programación) — un secreto commiteado y luego borrado
**sigue en el historial** para siempre. Los bots escanean GitHub buscando claves
en cuestión de minutos; una clave de AWS filtrada puede costar miles de euros en
horas.

Cómo hacerlo bien, en orden de robustez:
- **Variables de entorno** (lo mínimo): la config y los secretos se leen del
  entorno, no del código (`os.environ`). El `.env` local va en `.dockerignore` y
  `.gitignore` (curso-docker, m2/m8).
- **Gestores de secretos** (producción): AWS Secrets Manager, HashiCorp Vault,
  Kubernetes Secrets (curso-kubernetes) — almacenan cifrado, rotan, auditan
  accesos. El secreto nunca toca el disco en claro.
- **En contenedores**: en runtime (`-e`, BuildKit secrets), nunca en la imagen
  (curso-docker, m8: queda en las capas para siempre).

## 2. Qué hacer si un secreto ya se filtró

Si commiteaste un secreto (a todos nos ha pasado), la reacción correcta:
1. **Rótalo YA** (invalida el viejo, genera uno nuevo). Asume que está
   comprometido — borrarlo del historial no basta, ya pudo ser leído.
2. **Luego** límpialo del historial (git filter-repo/BFG) si el repo es privado
   — pero lo primero es rotar. Un secreto expuesto es un secreto quemado.

Prevención: **escaneo automático de secretos** en el pipeline (gitleaks,
trufflehog, curso de seguridad como parte del CI) que rechaza commits con
credenciales antes de que entren. Y pre-commit hooks que escanean localmente.

## 3. Security misconfiguration

Una de las categorías del OWASP Top 10: el sistema es vulnerable no por un bug
de código sino por estar **mal configurado**. Los clásicos:

- **Defaults inseguros**: contraseñas por defecto sin cambiar (admin/admin),
  paneles de administración expuestos, cuentas de ejemplo activas.
- **Errores verbosos en producción**: un stack trace completo o un mensaje de
  error de BD le regala al atacante información del sistema (versiones,
  estructura, rutas). En producción: errores genéricos al usuario, el detalle
  solo a los logs (bloque 10).
- **Directorios/ficheros expuestos**: `.git/` accesible por web, `.env`
  servido, listado de directorios activado, ficheros de backup.
- **CORS abierto** (`Access-Control-Allow-Origin: *`, bloque 3), servicios
  internos accesibles desde fuera.
- **Software sin actualizar** con vulnerabilidades conocidas (bloque 7).

La cura: **hardening** — endurecer la configuración por defecto, principio de
mínima superficie (bloque 0), y no dejar nada "de ejemplo" en producción.

## 4. Cabeceras de seguridad

Cabeceras HTTP que el navegador respeta para reforzar la seguridad — ponlas
todas (defensa en profundidad):

- **Content-Security-Policy** (bloque 3): controla qué se puede cargar/ejecutar
  → mitiga XSS.
- **Strict-Transport-Security (HSTS)** (bloque 1): fuerza HTTPS siempre.
- **X-Content-Type-Options: nosniff**: evita que el navegador "adivine" tipos
  MIME (previene ciertos ataques).
- **X-Frame-Options / frame-ancestors**: evita que tu sitio se cargue en un
  iframe ajeno (clickjacking).
- **Referrer-Policy, Permissions-Policy**: controlan fugas de información y
  acceso a APIs del navegador.

Herramientas como securityheaders.com te dan una nota de qué te falta. Muchos
frameworks/librerías (helmet en Node, django-csp) las ponen por ti.

## 5. Práctica del bloque

1. Escanea un repo tuyo con gitleaks o trufflehog. Si encuentra algo (o
   provócalo commiteando una clave falsa), practica el flujo: rotar
   (conceptualmente) + limpiar el historial.
2. Configura una app para leer sus secretos de variables de entorno, con un
   `.env` correctamente ignorado en git y docker.
3. Reproduce una misconfiguration: deja un error verboso en producción y observa
   qué información revela. Arréglalo con errores genéricos + logging.
4. Añade las cabeceras de seguridad a una app y comprueba tu nota en
   securityheaders.com antes y después.

## Recursos

- 🌐 OWASP "Secrets Management Cheat Sheet", A05:Security Misconfiguration.
- 🔧 gitleaks, trufflehog (escaneo de secretos); securityheaders.com; helmet/django-csp.

## Autoevaluación

1. ¿Por qué un secreto en git es un problema permanente aunque lo borres?
2. Si filtras un secreto, ¿cuál es el primer paso y por qué? ¿Y el segundo?
3. Da tres ejemplos de security misconfiguration y su cura.
4. ¿Por qué los errores verbosos en producción son un riesgo?
5. Nombra cuatro cabeceras de seguridad y qué protege cada una.

✅ **Checklist de salida:**
- [ ] Nunca pones secretos en el código; usas entorno/gestores de secretos.
- [ ] Escaneas secretos en el pipeline y sabes reaccionar a una filtración.
- [ ] Endureces la configuración: sin defaults inseguros ni errores verbosos.
- [ ] Pones las cabeceras de seguridad en tus apps.

**Siguiente:** [Bloque 7 — Dependencias y cadena de suministro](07-dependencias.md)
