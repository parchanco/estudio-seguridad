# Bloque 7 — Dependencias y cadena de suministro

> Objetivo: el riesgo que heredas del código que no escribiste. La mayoría de tu
> aplicación son dependencias de terceros — cada una es superficie de ataque.
> CVEs, escaneo, y los ataques de supply chain que han sacudido la industria.

## 1. El riesgo heredado

Tu aplicación es, en volumen, sobre todo **código de otros**: frameworks,
librerías, y las dependencias de tus dependencias (transitivas) — fácilmente
cientos o miles de paquetes. Cada uno:
- Ejecuta con **tus** permisos (curso de programación, límites).
- Puede tener vulnerabilidades (que heredas).
- Puede ser comprometido (punto 3).

Es una de las categorías del OWASP Top 10 ("Vulnerable and Outdated Components").
La superficie de ataque no es solo tu código — es todo el árbol de dependencias.

## 2. CVEs y escaneo

Una **CVE (Common Vulnerabilities and Exposures)** es un identificador público
de una vulnerabilidad conocida (`CVE-2021-44228` = Log4Shell). Cuando se
descubre una vuln en una librería popular, se publica, y a partir de ahí:
- Los defensores deben parchear.
- Los atacantes escanean internet buscando quién NO ha parcheado.

Tu trabajo: **saber qué dependencias vulnerables tienes y actualizarlas**. Las
herramientas lo automatizan:
- `pip-audit` (Python), `npm audit` (Node), `cargo audit` (Rust) — escanean tus
  dependencias contra bases de datos de CVEs.
- **Dependabot / Renovate**: bots que abren PRs automáticas cuando hay
  actualizaciones de seguridad.
- **Trivy** (curso-docker, m8): escanea imágenes de contenedor (tus deps + las
  del SO base).
- **SBOM** (Software Bill of Materials): un inventario de todo lo que compone tu
  software — cada vez más exigido, permite saber al instante si te afecta una
  CVE nueva.

Integra el escaneo en el CI (bloque 10): que el build avise (o falle) ante
vulnerabilidades críticas.

## 3. Ataques de supply chain

El riesgo más insidioso: no que una dependencia tenga un bug, sino que sea
**maliciosa a propósito**. Casos reales que conviene conocer:
- **Typosquatting**: publicar un paquete con un nombre parecido a uno popular
  (`python-requests` vs `requests`, `crossenv` vs `cross-env`) esperando que
  alguien se equivoque al instalar. El paquete falso roba secretos o instala
  malware.
- **Dependencia comprometida**: un mantenedor legítimo es hackeado (o vende el
  paquete, o se vuelve malicioso) e inyecta código dañino en una versión nueva
  — que se propaga a todos los que actualizan. Ha pasado con paquetes de npm muy
  usados (event-stream, ua-parser-js) y con SolarWinds (a escala corporativa
  devastadora).
- **Dependency confusion**: subir a un repo público un paquete con el nombre de
  uno interno de una empresa, para que su sistema instale el público malicioso.

La lección: **una dependencia es confianza depositada en desconocidos.** No
puedes auditar todo, pero puedes reducir el riesgo (punto 4).

## 4. Defensas prácticas

- **Pinning y lockfiles**: fija versiones exactas (`requirements.txt` con
  versiones, `poetry.lock`, `package-lock.json`) → builds reproducibles (curso-
  docker, m7) y no te tragas una versión nueva comprometida sin querer.
- **Revisa antes de añadir**: ¿el paquete está mantenido? ¿cuántos usuarios?
  ¿necesitas de verdad esa dependencia o son 3 líneas que puedes escribir?
  (Menos dependencias = menos superficie — curso de arquitectura.)
- **Actualiza con criterio**: parchea las de seguridad rápido, pero no
  actualices a ciegas al segundo de salir una versión (ha habido versiones
  maliciosas detectadas en horas). Un pequeño retraso + lockfile + escaneo es un
  buen equilibrio.
- **Escaneo continuo** en el CI y alertas (Dependabot).
- **Mínimo privilegio en el build**: que el pipeline no tenga más secretos/
  permisos de los necesarios (una dependencia maliciosa se ejecuta en tu CI).

## 5. Práctica del bloque

1. Ejecuta `pip-audit`/`npm audit` (o Trivy sobre una imagen) en un proyecto
   tuyo. ¿Cuántas vulnerabilidades tienes? Prioriza y actualiza las críticas.
2. Investiga una CVE famosa (Log4Shell, la de event-stream): qué era, cómo se
   explotaba, qué la hizo tan grave. Entender un caso real fija el concepto.
3. Configura Dependabot (o similar) en un repo de GitHub y observa las PRs que
   abre.
4. Revisa tu árbol de dependencias (`pip list`, `npm ls`): ¿cuántas hay? ¿alguna
   que añadiste para algo trivial y podrías eliminar?

## Recursos

- 🌐 OWASP A06:Vulnerable and Outdated Components; "Software Supply Chain
  Security".
- 🔧 pip-audit, Trivy, Dependabot, Snyk.
- 📄 Los post-mortem de event-stream y SolarWinds — lecturas que asustan (y
  enseñan).

## Autoevaluación

1. ¿Por qué tus dependencias son superficie de ataque? ¿Qué son las transitivas?
2. ¿Qué es una CVE y qué pasa cuando se publica una de una librería popular?
3. Explica typosquatting y "dependencia comprometida" con ejemplos.
4. ¿Qué dan los lockfiles y por qué no actualizar a ciegas?
5. ¿Qué herramientas usas para escanear dependencias y dónde las integras?

✅ **Checklist de salida:**
- [ ] Escaneas tus dependencias contra CVEs en el CI.
- [ ] Usas lockfiles y actualizas las de seguridad con prontitud pero criterio.
- [ ] Conoces los ataques de supply chain y reduces la superficie.
- [ ] Revisas antes de añadir una dependencia.

**Siguiente:** [Bloque 8 — Seguridad de APIs](08-apis.md)
