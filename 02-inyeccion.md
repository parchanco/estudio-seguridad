# Bloque 2 — Inyección (SQL y amigos)

> Objetivo: la familia de vulnerabilidades más clásica y una de las más
> peligrosas — cuando datos del usuario se interpretan como código/comandos.
> SQL injection a fondo, y la única defensa real: separar datos de código.

## 1. Qué es una inyección

Una inyección ocurre cuando **datos del usuario se mezclan con código** (SQL,
comandos del sistema, HTML...) y el intérprete ejecuta parte de esos datos
**como si fueran código**. La raíz es siempre la misma: **construir código
concatenando entrada del usuario**.

## 2. SQL injection: cómo funciona

El caso canónico. Una consulta construida concatenando:

```python
# ❌ VULNERABLE: concatena la entrada del usuario en el SQL
usuario = request.form["usuario"]
query = f"SELECT * FROM users WHERE name = '{usuario}'"
db.execute(query)
```

Si el usuario mete `usuario = "' OR '1'='1"`, la consulta se convierte en:

```sql
SELECT * FROM users WHERE name = '' OR '1'='1'    -- ¡devuelve TODOS los usuarios!
```

Peor aún, `'; DROP TABLE users; --` podría borrar la tabla, y técnicas más
avanzadas permiten **extraer toda la base de datos** (contraseñas incluidas),
saltarse el login, o leer ficheros del servidor. Es una de las vulnerabilidades
más dañinas: compromete la confidencialidad y la integridad de TODOS los datos
(bloque 0, CIA).

## 3. La ÚNICA defensa real: consultas parametrizadas

La solución no es "escapar comillas" (frágil, se te escapa un caso) ni "validar
que no haya palabras malas" (evadible). Es **separar el código de los datos**
usando **consultas parametrizadas** (prepared statements): el SQL y los valores
viajan por canales distintos, y la BD nunca interpreta los valores como código:

```python
# ✅ SEGURO: el ? es un placeholder; el valor va aparte, JAMÁS como código
db.execute("SELECT * FROM users WHERE name = ?", (usuario,))
# o con nombres: db.execute("... WHERE name = :nombre", {"nombre": usuario})
```

Con parámetros, meter `' OR '1'='1` simplemente busca un usuario llamado
literalmente `' OR '1'='1` (no encuentra ninguno) — el valor nunca cambia la
estructura de la consulta. **Esta es la regla inviolable: nunca construyas SQL
concatenando entrada; siempre parámetros.** Conecta con el curso de bases de
datos (todo el SQL) y con "parse, don't validate" (curso 01 extra).

## 4. Los ORMs: cuándo protegen y cuándo los rompes

Los ORMs (Django, SQLAlchemy — curso de BD, bloque 7) usan parámetros por
debajo, así que **por defecto te protegen**:

```python
User.objects.filter(name=usuario)      # ✅ parametrizado por el ORM
```

Pero los rompes en cuanto bajas a SQL crudo concatenando:

```python
# ❌ raw SQL concatenado: vuelves a ser vulnerable
User.objects.raw(f"SELECT * FROM users WHERE name = '{usuario}'")
# ✅ raw pero parametrizado:
User.objects.raw("SELECT * FROM users WHERE name = %s", [usuario])
```

Regla: usa el ORM normalmente; cuando necesites raw SQL (curso de BD: window
functions, etc.), pásale los valores como parámetros, nunca los concatenes.

## 5. Las otras inyecciones (misma raíz, misma cura)

El patrón se repite en muchos contextos:

- **Command injection**: `os.system(f"ping {host}")` con `host = "x; rm -rf /"`
  ejecuta comandos arbitrarios. Cura: no invoques la shell con entrada del
  usuario; usa APIs que separan comando y argumentos
  (`subprocess.run(["ping", host])` — lista, no string, no `shell=True`).
- **NoSQL injection**: Mongo y similares también son inyectables si construyes
  queries con entrada sin sanear.
- **LDAP / XML / template injection**: mismo principio.
- **XSS** es inyección de HTML/JS en el navegador — tan importante que tiene su
  bloque (bloque 3).

La cura es SIEMPRE la misma idea: **separar datos de código** (parámetros, APIs
que no interpretan, escapado por contexto) y **validar la entrada en la
frontera** (parse, don't validate — tipos, formatos, listas de valores
permitidos).

## 6. Práctica del bloque

1. En tu laboratorio (Juice Shop/WebGoat), explota una SQL injection real:
   sáltate un login con `' OR '1'='1` o extrae datos. Entiende por qué funciona.
2. Escribe una función vulnerable a SQLi (concatenando) y explótala tú mismo;
   luego arréglala con parámetros y verifica que el ataque ya no funciona.
3. Reproduce una command injection con `os.system` y entrada maliciosa, y
   arréglala con `subprocess.run([...])` sin shell.
4. Revisa un proyecto tuyo: busca cualquier construcción de SQL o comandos por
   concatenación de entrada. Si encuentras alguna, es un hallazgo real.

## Recursos

- 🌐 OWASP "SQL Injection Prevention Cheat Sheet" y la categoría A03:Injection
  del Top 10.
- 🌐 PortSwigger Academy: labs de SQL injection (de los mejores que hay).

## Autoevaluación

1. ¿Cuál es la raíz común de todas las inyecciones?
2. Explica cómo funciona una SQL injection con el ejemplo del `OR '1'='1`.
3. ¿Por qué las consultas parametrizadas son la defensa real y no el escapado?
4. ¿Cuándo te protege un ORM y cómo lo rompes?
5. Da la cura de la command injection y por qué `shell=True` es peligroso.

✅ **Checklist de salida:**
- [ ] Nunca construyes SQL ni comandos concatenando entrada del usuario.
- [ ] Usas consultas parametrizadas siempre (y raw SQL del ORM con parámetros).
- [ ] Evitas la shell con entrada; usas APIs que separan comando y argumentos.
- [ ] Has explotado y arreglado una inyección en el laboratorio.

**Siguiente:** [Bloque 3 — XSS y seguridad del navegador](03-xss.md)
