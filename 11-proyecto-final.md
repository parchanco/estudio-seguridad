# 🎓 Proyecto final — Auditar (y arreglar) una aplicación vulnerable

> Objetivo: aplicar todo el curso atacando y defendiendo una aplicación real en
> tu laboratorio, y luego endureciendo un proyecto propio. Encontrar
> vulnerabilidades, explotarlas éticamente, y aplicar el fix — el ciclo completo
> del profesional de AppSec.

## Parte A — Auditar una app deliberadamente vulnerable

Sobre **OWASP Juice Shop** (o WebGoat) en tu laboratorio Docker (bloque 0):

1. **Encuentra y explota 8-10 vulnerabilidades** que cubran el OWASP Top 10 y
   los bloques del curso. Como mínimo:
   - Una **inyección** (SQL o command — bloque 2).
   - Un **XSS** (reflejado o almacenado — bloque 3).
   - Un fallo de **autenticación** (bloque 4).
   - Un **broken access control / IDOR** (bloque 5).
   - Una **misconfiguration** o secreto expuesto (bloque 6).
   - Un fallo de **API** (mass assignment, BOLA — bloque 8).
2. **Documenta cada una** en `HALLAZGOS.md` con el formato de un informe de
   pentest:
   - Vulnerabilidad y bloque del curso al que corresponde.
   - **Impacto** (qué consigue el atacante — usa CIA, bloque 0).
   - **Prueba de concepto**: los pasos/payload para explotarla (éticamente, en
     el lab).
   - **Severidad** (crítica/alta/media/baja) y el **fix** recomendado.
3. Usa las herramientas del curso: DevTools, un proxy (ZAP/Burp), curl.

## Parte B — Endurecer un proyecto propio

Sobre una app tuya real (o una que montes para esto):

1. **Aplica las defensas** de los bloques a lo largo del código:
   - Consultas parametrizadas (bloque 2), auto-escapado verificado (bloque 3).
   - Hashing de contraseñas correcto (bloque 1), auth y authz sólidas
     (bloques 4-5), comprobación de propiedad en cada recurso.
   - Secretos fuera del código (bloque 6), cabeceras de seguridad, validación
     de esquemas en la API (bloque 8).
2. **Monta el pipeline de seguridad** (bloque 10): en el CI, escaneo de
   dependencias (bloque 7), de secretos (bloque 6), SAST (bandit/semgrep), y si
   es contenedor, escaneo de imagen (bloque 9). Que falle ante lo crítico.
3. **Haz un threat model** (bloque 10, STRIDE) de una página de la app,
   documentado como ADR de seguridad.
4. **Hardening de despliegue** (bloque 9): si va en contenedor, imagen mínima +
   no-root + escaneada; si va en cloud, mínimo privilegio IAM y nada interno
   expuesto.

## Entregables

- `HALLAZGOS.md`: el informe de las 8-10 vulnerabilidades explotadas (parte A).
- El proyecto propio endurecido, con:
  - Los fixes aplicados (commits que referencian el bloque/vulnerabilidad).
  - El pipeline de seguridad en el CI, funcionando.
  - `THREAT-MODEL.md` con el STRIDE de una feature.
  - Un `SECURITY.md` en el repo (cómo reportar vulnerabilidades, qué se soporta).

## Criterios de "aprobado"

- [ ] 8-10 vulnerabilidades explotadas y documentadas con impacto, PoC y fix.
- [ ] Las categorías cubren inyección, XSS, auth, access control, config y API.
- [ ] El proyecto propio aplica las defensas correspondientes, verificadas.
- [ ] El CI escanea dependencias, secretos y código, y falla ante lo crítico.
- [ ] Hay un threat model y un hardening de despliegue documentados.
- [ ] Todo el trabajo ofensivo se hizo en entornos propios/de práctica (ético).

## Después de esto

Ya llevas la seguridad puesta como propiedad transversal, no como parche. Los
puentes con tus otros repos quedan activos: el bloque 9 conecta con
`curso-docker`, `curso-kubernetes` y `aws-architect`; el 2 con
`estudio-bases-de-datos`; el 4-5 con cualquier backend de `estudio-programacion`.
Si quieres profundizar en el lado ofensivo (siempre ético): las certificaciones
tipo OSCP y los CTF son el siguiente nivel; para el defensivo, OWASP SAMM y la
ingeniería de detección.

**Volver al [temario del curso](TEMARIO.md)**
