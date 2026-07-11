# Bloque 10 — Diseñar con seguridad (shift left)

> Objetivo: dejar de tratar la seguridad como un parche al final y meterla en el
> diseño desde el principio (shift left). Threat modeling ligero, el SDLC
> seguro, logging que no filtra, y la cultura de seguridad como responsabilidad
> de todos.

## 1. Shift left: la seguridad empieza en el diseño

El anti-patrón: construir todo y al final pasar un "test de seguridad" que
encuentra problemas caros de arreglar (o que se ignoran por falta de tiempo).
**Shift left** = mover la seguridad hacia el principio del ciclo de desarrollo,
donde arreglar es barato (curso de arquitectura: el coste del cambio crece con
el tiempo; curso 07: crash early aplicado al proceso).

En la práctica: pensar en seguridad al **diseñar** (threat modeling, punto 2),
al **codificar** (los bloques 1-8 como hábito), en la **review** (revisar
también la seguridad, no solo la lógica), y en el **CI** (escaneo automático,
punto 3). La seguridad deja de ser una fase y pasa a ser una propiedad
transversal.

## 2. Threat modeling ligero

Antes de construir, preguntarse sistemáticamente **"¿qué puede salir mal?"**.
No hace falta un proceso pesado — una versión ligera para cada feature
importante:

- **STRIDE** como checklist de categorías de amenaza: **S**poofing (suplantar
  identidad → authn, bloque 4), **T**ampering (alterar datos → integridad,
  inyección), **R**epudiation (negar haber hecho algo → logging/auditoría),
  **I**nformation disclosure (fugas → confidencialidad, autz), **D**enial of
  service (tumbar → rate limiting), **E**levation of privilege (escalar
  permisos → autz, bloque 5).
- Para cada feature: ¿qué datos maneja? ¿quién no debería acceder? ¿qué pasa si
  la entrada es maliciosa? ¿qué pasa si un componente es comprometido?
- Documenta las decisiones (es un ADR de seguridad — curso de arquitectura,
  m8): qué amenazas consideraste y cómo las mitigas.

Es el "pensar como atacante" del bloque 0, hecho método y aplicado en el diseño.

## 3. SDLC seguro y el pipeline

Integrar la seguridad en cada fase del desarrollo:
- **Diseño**: threat modeling.
- **Código**: hábitos seguros (bloques 1-8), linters de seguridad (bandit en
  Python, semgrep).
- **Review**: incluir seguridad en la checklist de code review (curso 07: la
  review construye). Existe `/security-review` como herramienta.
- **CI/CD** (curso-docker, curso 01 extra): automatizar —
  - **SAST** (Static Application Security Testing): analiza tu código
    (semgrep, bandit, CodeQL).
  - **Escaneo de dependencias** (bloque 7) y **de secretos** (bloque 6).
  - **Escaneo de imágenes** (bloque 9, Trivy).
  - **DAST** (Dynamic): prueba la app corriendo (OWASP ZAP automatizado).

  Que el pipeline **falle** ante vulnerabilidades críticas → no llega a
  producción.

## 4. Logging seguro

Los logs son clave para detectar y responder a incidentes — pero mal hechos son
un riesgo en sí:
- **Qué NO loguear**: nunca contraseñas, tokens, claves, números de tarjeta,
  datos personales sensibles (PII). Un log con secretos es una filtración
  esperando a pasar (y los logs se copian, se envían a terceros, se retienen).
  Cuidado con loguear objetos enteros (request bodies) que pueden contenerlos.
- **Qué SÍ loguear**: eventos de seguridad (logins, fallos de auth, cambios de
  permisos, accesos denegados) para poder auditar e investigar — con IDs de
  correlación (system-design, observabilidad) pero sin los datos sensibles.
- **Detección sin exponer**: los logs permiten detectar ataques (muchos fallos
  de login → fuerza bruta) sin que ellos mismos sean el agujero.

## 5. La cultura: seguridad de todos

El punto final y el más importante. La seguridad **no es el trabajo de un equipo
aparte** al que "se le pasa" el código al final — es responsabilidad de cada
desarrollador (curso 07: profesionalidad). Un equipo con cultura de seguridad:
- Trata las vulnerabilidades como bugs de primera (no "ya lo veremos").
- Comparte conocimiento (un XSS encontrado se explica al equipo — DRY
  organizacional, curso 07).
- No culpa (el que introdujo el bug no es el villano; el proceso que lo dejó
  pasar es lo que se mejora — curso 07, reviews que construyen).
- Equilibra: seguridad absoluta es imposible y paraliza; el objetivo es
  **gestión de riesgo** proporcional al valor de lo que proteges (no gastas lo
  mismo protegiendo un blog que un banco). El criterio, como siempre (curso 07),
  es el juicio informado, no el dogma.

## 6. Práctica del bloque

1. Haz un threat model ligero (STRIDE) de una feature de una app tuya: una
   página, cada amenaza de STRIDE, y cómo la mitigas. Escríbelo como ADR de
   seguridad.
2. Añade a un pipeline de CI: bandit/semgrep (SAST), escaneo de dependencias
   (bloque 7) y de secretos (bloque 6). Haz que falle ante algo crítico.
3. Audita los logs de una app: ¿loguea algún secreto o PII? Arréglalo.
   Asegúrate de que SÍ registra los eventos de seguridad.
4. Ejecuta `/security-review` (o OWASP ZAP) sobre un cambio y revisa los
   hallazgos.

## Recursos

- 🌐 OWASP SAMM (Software Assurance Maturity Model) y "Threat Modeling Cheat
  Sheet".
- 🔧 bandit, semgrep, CodeQL (SAST); OWASP ZAP (DAST); la herramienta
  `/security-review`.
- 📖 *Threat Modeling: Designing for Security* (Adam Shostack).

## Autoevaluación

1. ¿Qué es "shift left" y por qué arreglar seguridad pronto es más barato?
2. ¿Qué es STRIDE y cómo lo usas en un threat model ligero?
3. ¿Qué escaneos de seguridad integrarías en el CI (SAST, deps, secretos,
   imágenes, DAST)?
4. ¿Qué NO se loguea nunca y qué SÍ? ¿Por qué los logs son un arma de doble
   filo?
5. ¿Por qué la seguridad es "de todos" y qué caracteriza una buena cultura de
   seguridad?

✅ **Checklist de salida:**
- [ ] Piensas en amenazas al diseñar (threat modeling ligero con STRIDE).
- [ ] Automatizas la seguridad en el CI (SAST, deps, secretos, imágenes).
- [ ] Logueas eventos de seguridad sin filtrar secretos ni PII.
- [ ] Tratas la seguridad como responsabilidad propia y gestión de riesgo.

**Siguiente:** [Bloque 11 — Proyecto final](11-proyecto-final.md)
