# Bloque 9 — Seguridad en contenedores y cloud

> Objetivo: la seguridad de dónde corre tu código hoy — contenedores, Kubernetes
> y cloud. Amplía la seguridad de curso-docker (m8) y conecta con los
> namespaces del curso de SO y el modelo de responsabilidad compartida de
> telecom.

## 1. Seguridad de imágenes de contenedor

Recapitula y profundiza curso-docker (m8) — una imagen es tu código + un mini-SO
+ dependencias, todo superficie de ataque:
- **Imágenes mínimas**: distroless/slim (curso-docker, m7) → menos paquetes,
  menos CVEs (bloque 7), menos que atacar. Una imagen `ubuntu` completa trae
  cientos de paquetes que no usas.
- **No root**: el contenedor corre como usuario sin privilegios (curso-docker,
  m8). Casi ninguna app necesita root; correr como root facilita el escape al
  host.
- **Escaneo**: `trivy`/`docker scout` antes de publicar (bloques 7-8) — te lista
  las CVEs de tus capas.
- **Fija por digest** (curso-docker, m1) e imágenes de origen confiable
  (supply chain, bloque 7 — una imagen base maliciosa te compromete entero).

## 2. Superficie de ataque de un contenedor

Del curso de SO (bloque 8): un contenedor es un proceso con **namespaces**
(aislamiento de lo que ve) y **cgroups** (límites). Eso NO es una frontera de
seguridad tan fuerte como una VM (comparte el kernel del host — curso-docker,
m8). Implicaciones:
- Un **escape de contenedor** (aprovechando un bug del kernel o una
  configuración laxa) da acceso al host. Por eso mínimo privilegio importa
  tanto aquí.
- **NUNCA `--privileged`** (curso-docker, m8): desactiva el aislamiento.
- **Reduce capabilities** (`--cap-drop ALL`), filesystem de solo lectura
  (`--read-only`), `no-new-privileges`, y **seccomp/AppArmor** (perfiles que
  limitan qué syscalls puede hacer el contenedor — reducen lo que un exploit
  puede intentar).
- Lo que expones: puertos (solo los necesarios, bloque 0), y no montes el socket
  de Docker (`/var/run/docker.sock`) dentro de un contenedor salvo que sepas lo
  que haces (= dar control del host).

## 3. Secretos en Kubernetes y cloud

Del bloque 6 (secretos), aplicado a la plataforma:
- **Kubernetes Secrets**: separan los secretos de los manifiestos... pero por
  defecto están solo en Base64 (no cifrado — bloque 1), y en etcd. Actívales
  cifrado en reposo y RBAC estricto de quién los lee. Considera un gestor
  externo (Vault, sealed-secrets) para secretos serios.
- **IAM roles en cloud** (AWS/GCP): en vez de meter claves de acceso en la app
  (que se filtran, bloque 6), asigna un **rol** a la máquina/pod → obtiene
  credenciales temporales automáticamente, sin secretos estáticos. Es el mínimo
  privilegio (bloque 0) hecho infraestructura: el rol da solo los permisos que
  ese servicio necesita.
- **Secrets managers gestionados**: AWS Secrets Manager, GCP Secret Manager —
  rotan, auditan, cifran.

## 4. El modelo de responsabilidad compartida

En cloud (telecom, bloque 15, desde la óptica de seguridad ahora): la seguridad
se **reparte** entre el proveedor y tú:
- **El proveedor** (AWS/GCP/Azure) asegura "la seguridad DE la nube": el
  hardware, la red física, el hipervisor, los servicios gestionados.
- **Tú** aseguras "la seguridad EN la nube": tu configuración, tus datos, tus
  permisos IAM, tu código, quién accede a qué.

El dato demoledor: **la mayoría de las brechas en cloud NO son culpa del
proveedor — son errores de configuración del cliente** (bloque 6): un bucket S3
público con datos sensibles, un security group abierto al mundo, permisos IAM
excesivos, una BD sin contraseña expuesta a internet. La nube no te hace seguro;
te da herramientas que hay que configurar bien.

Prácticas clave: **mínimo privilegio en IAM** (el error nº1 es dar permisos de
más "para que funcione ya"), cifrado en reposo y tránsito, redes privadas para
lo interno (una BD nunca accesible desde internet — VPC, telecom bloque 15), y
auditoría/logging de accesos (CloudTrail).

## 5. Práctica del bloque

1. Escanea una imagen tuya con Trivy y arréglala: base mínima + no-root +
   actualizar. Compara el número de CVEs antes/después.
2. Ejecuta un contenedor con hardening completo (`--cap-drop ALL --read-only
   --no-new-privileges --user`) y comprueba que tu app sigue funcionando (o
   descubre qué necesita de verdad).
3. Revisa un manifiesto de Kubernetes (o créalo): ¿los secretos están bien
   gestionados? ¿el pod corre como no-root (securityContext)?
4. En cloud (o conceptualmente): identifica un permiso IAM excesivo y redúcelo
   al mínimo. Revisa que no haya nada (bucket, BD) expuesto públicamente.

## Recursos

- 🌐 OWASP "Docker Security Cheat Sheet", "Kubernetes Security Cheat Sheet".
- 📄 CIS Benchmarks (Docker, Kubernetes) — checklists de hardening.
- 📚 curso-docker (m8), curso de SO (bloque 8), telecom (bloque 15), aws-architect.

## Autoevaluación

1. Da tres medidas de seguridad de imágenes de contenedor.
2. ¿Por qué un contenedor es una frontera más débil que una VM y qué hardening
   lo compensa?
3. ¿Qué problema tienen los Kubernetes Secrets por defecto? ¿Qué son los IAM
   roles y qué resuelven?
4. Explica el modelo de responsabilidad compartida. ¿De quién es la culpa de la
   mayoría de brechas cloud?
5. ¿Cuál es el error de configuración cloud más común y su principio-cura?

✅ **Checklist de salida:**
- [ ] Usas imágenes mínimas, no-root y escaneadas.
- [ ] Endureces contenedores (cap-drop, read-only) y nunca --privileged.
- [ ] Gestionas secretos con IAM roles / gestores, no claves estáticas.
- [ ] Aplicas mínimo privilegio y no expones nada interno en cloud.

**Siguiente:** [Bloque 10 — Diseñar con seguridad (shift left)](10-diseno-seguro.md)
