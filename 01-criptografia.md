# Bloque 1 — Criptografía para desarrolladores

> Objetivo: la cripto que un desarrollador necesita usar bien — sin
> matemáticas, con foco en no meter la pata. Simétrico vs asimétrico, hashing
> de contraseñas, TLS, y la regla de oro: no inventes tu propia cripto.

## 1. Los tres conceptos que se confunden

Hashing, cifrado y codificación no son lo mismo — confundirlos causa
vulnerabilidades reales:

- **Codificación** (Base64, URL-encoding): transformar formato, **reversible
  sin secreto**, NO da seguridad ninguna. Base64 no "oculta" nada — cualquiera
  lo decodifica. Es para transporte, no para proteger.
- **Cifrado** (encryption): ocultar datos de forma **reversible con una clave**.
  Cifras para guardar/enviar secretos que luego necesitas recuperar.
- **Hashing**: función **irreversible** (de una vía) que produce un resumen de
  tamaño fijo. No se "descifra". Para verificar integridad y guardar
  contraseñas (punto 3).

El error clásico: "guardo las contraseñas en Base64" (no protege nada) o
"cifro las contraseñas" (no deberías poder descifrarlas — se hashean).

## 2. Simétrico vs asimétrico

- **Cifrado simétrico** (AES): **una** clave cifra y descifra. Rápido, para
  grandes volúmenes de datos. Problema: ¿cómo compartes la clave de forma
  segura con la otra parte?
- **Cifrado asimétrico** (RSA, curvas elípticas): un **par** de claves —
  **pública** (cifra, se comparte libremente) y **privada** (descifra, secreta).
  Resuelve el problema de compartir clave: cualquiera cifra con tu pública,
  solo tú descifras con tu privada. También permite **firmas digitales**
  (firmas con la privada, cualquiera verifica con la pública → autenticidad e
  integridad). Es lento, para datos pequeños.
- **En la práctica se combinan** (cifrado híbrido, lo que hace TLS): asimétrico
  para intercambiar una clave simétrica de sesión, luego simétrico para los
  datos (rápido). Amplía el bloque 9 de telecom.

## 3. Hashing de contraseñas: NUNCA lo hagas mal

El error de seguridad más común y más dañino. Las reglas inviolables:

- **NUNCA guardes contraseñas en texto plano.** Si tu BD se filtra (y las BDs
  se filtran), todas quedan expuestas.
- **NUNCA uses hashes rápidos** (MD5, SHA-256) para contraseñas. Son rápidos a
  propósito → un atacante prueba **miles de millones por segundo** con una GPU
  (fuerza bruta / rainbow tables).
- **USA hashes lentos y con sal, diseñados para contraseñas**: **bcrypt**,
  **scrypt** o **Argon2** (el recomendado hoy). Son **deliberadamente lentos y
  costosos en memoria** → el atacante solo puede probar unos pocos por segundo.

```python
# ✅ con una librería probada (passlib, o bcrypt directamente):
from argon2 import PasswordHasher
ph = PasswordHasher()
hash = ph.hash("contraseña-del-usuario")   # lento a propósito, con sal incluida
ph.verify(hash, "contraseña-a-comprobar")  # lanza si no coincide
```

Conceptos: la **sal** (salt) es un valor aleatorio único por contraseña que se
guarda con el hash → dos usuarios con la misma contraseña tienen hashes
distintos (mata las rainbow tables). El **factor de coste** ajusta cuán lento
es (súbelo con el tiempo, según mejora el hardware). Las librerías buenas
manejan sal y coste por ti — otra razón para no hacerlo a mano.

## 4. TLS: qué garantiza y qué no

**TLS** (lo de la "s" en HTTPS) protege los datos **en tránsito**:

- Garantiza: **confidencialidad** (nadie en el medio lee los datos),
  **integridad** (nadie los altera) y **autenticidad del servidor** (hablas con
  quien crees, verificado por el **certificado** que una CA firmó).
- NO garantiza: que el servidor sea honesto (un phishing puede tener HTTPS
  válido — el candado dice "cifrado", no "de fiar"), ni protege los datos **en
  reposo** (una vez llegan, es otro problema), ni contra vulnerabilidades de la
  app.
- **HSTS**: cabecera que fuerza al navegador a usar siempre HTTPS (evita
  ataques de degradación a HTTP).
- Práctico: usa TLS en todo (Let's Encrypt lo hace gratis), no lo mezcles con
  HTTP, y no desactives la verificación de certificados en tus clientes
  (`verify=False` es un agujero clásico).

## 5. La regla de oro: no inventes cripto

El principio más importante del bloque: **no implementes tú algoritmos
criptográficos ni protocolos.** La cripto es un campo donde los errores sutiles
(un IV reutilizado, un padding mal hecho, comparar strings de forma no
constante) rompen todo, y no se ven en los tests. Usa librerías establecidas y
auditadas (`cryptography` en Python, libsodium), en su modo de alto nivel, y no
toques los detalles. "Roll your own crypto" es el chiste recurrente de la
seguridad porque siempre acaba mal. Es el "no reinventes el consenso" de
system-design, aplicado.

## 6. Práctica del bloque

1. Demuestra por qué Base64 no es seguridad: codifica un "secreto" y
   decodifícalo trivialmente. Contrasta con cifrar el mismo dato con una clave.
2. Compara la velocidad: hashea 1000 contraseñas con SHA-256 y con Argon2/bcrypt.
   Mide la diferencia y razona por qué la lentitud es una feature de seguridad.
3. Implementa registro/login con hashing correcto (Argon2 + verify) usando una
   librería. Verifica que dos usuarios con igual contraseña tienen hashes
   distintos (la sal).
4. Inspecciona el certificado TLS de una web (DevTools o `openssl s_client`):
   quién lo emitió, para qué dominio, cuándo caduca.

## Recursos

- 🌐 OWASP "Password Storage Cheat Sheet" y "Cryptographic Storage Cheat Sheet".
- 📄 Docs de la librería `cryptography` (Python) y de Argon2.
- 📖 Telecom, bloque 9 — la base conceptual (simétrico/asimétrico/TLS).

## Autoevaluación

1. Distingue codificación, cifrado y hashing. ¿Qué error causa confundirlos con
   las contraseñas?
2. Compara simétrico y asimétrico. ¿Por qué TLS los combina?
3. ¿Por qué NUNCA se usan MD5/SHA para contraseñas y qué se usa? ¿Qué hacen la
   sal y el factor de coste?
4. ¿Qué garantiza TLS y qué NO? ¿Por qué un sitio de phishing puede tener HTTPS?
5. ¿Por qué "no inventes tu propia cripto"?

✅ **Checklist de salida:**
- [ ] No confundes codificar, cifrar y hashear.
- [ ] Hasheas contraseñas con Argon2/bcrypt + sal, nunca con hashes rápidos.
- [ ] Sabes qué protege TLS y no desactivas la verificación de certificados.
- [ ] Usas librerías criptográficas probadas, sin inventar nada.

**Siguiente:** [Bloque 2 — Inyección (SQL y amigos)](02-inyeccion.md)
