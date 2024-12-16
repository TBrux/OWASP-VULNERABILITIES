# SQL Injection (SQLi)

## ¿Qué es SQL Injection?

**SQL Injection (SQLi)** es una vulnerabilidad de seguridad en aplicaciones que interactúan con bases de datos. Ocurre cuando un atacante puede inyectar comandos SQL maliciosos en una consulta de base de datos, debido a una validación o sanitización insuficiente de las entradas del usuario.

Con un ataque exitoso, un atacante podría:
- Acceder a datos sensibles.
- Modificar datos en la base de datos.
- Eliminar registros.
- Escalar privilegios y, en casos extremos, comprometer el servidor.

---

## Ejemplo básico: Consulta vulnerable

Una aplicación web podría tener un formulario de inicio de sesión con este código en el backend:

```sql
SELECT * FROM usuarios WHERE usuario = '$usuario' AND contraseña = '$contraseña';
```

Si las entradas del usuario no están validadas, un atacante podría inyectar SQL malicioso.

### Ejemplo de inyección
Si el atacante introduce lo siguiente como nombre de usuario:

```
admin' OR '1'='1
```

Y deja la contraseña vacía, la consulta resultante sería:

```sql
SELECT * FROM usuarios WHERE usuario = 'admin' OR '1'='1' AND contraseña = '';
```

En este caso, la condición `'1'='1'` siempre es verdadera, lo que da acceso al atacante.

---

## Tipos de SQL Injection

### 1. **Inyección Clásica**
Consiste en manipular directamente las entradas para alterar consultas SQL.

#### Ejemplo:
```sql
SELECT * FROM productos WHERE id = '$id';
```
Entrada maliciosa:
```
1 OR 1=1
```
Consulta resultante:
```sql
SELECT * FROM productos WHERE id = '1 OR 1=1';
```
Esto devuelve todos los productos porque la condición siempre se cumple.

---

### 2. **Union-based SQLi**
Usa la cláusula `UNION` para combinar resultados de múltiples consultas.

#### Ejemplo:
Consulta original:
```sql
SELECT nombre, precio FROM productos WHERE id = '$id';
```
Entrada maliciosa:
```
1 UNION SELECT usuario, contraseña FROM usuarios
```
Consulta resultante:
```sql
SELECT nombre, precio FROM productos WHERE id = '1 UNION SELECT usuario, contraseña FROM usuarios';
```
Esto devuelve los nombres y precios de productos junto con los usuarios y contraseñas.

---

### 3. **Blind SQL Injection**
Se explotan aplicaciones que no muestran directamente los resultados de las consultas SQL, pero sí reaccionan de manera diferente dependiendo de si la consulta es verdadera o falsa.

#### Ejemplo:
Consulta original:
```sql
SELECT * FROM usuarios WHERE id = '$id';
```
Entrada maliciosa:
```
1 AND 1=1
```
Esto devuelve un resultado válido.

Entrada maliciosa:
```
1 AND 1=2
```
Esto no devuelve resultados. Con base en estas respuestas, el atacante puede inferir información.

---

### 4. **Boolean-based SQLi**
El atacante utiliza valores booleanos para confirmar si una condición es verdadera o falsa.

#### Ejemplo:
Entrada:
```
1' AND EXISTS (SELECT * FROM usuarios WHERE usuario = 'admin') --
```
Si la consulta es verdadera, significa que el usuario "admin" existe.

---

### 5. **Time-based SQLi**
El atacante usa funciones de tiempo como `SLEEP()` para medir la respuesta del servidor y extraer información.

#### Ejemplo:
Entrada:
```
1' AND IF(1=1, SLEEP(5), 0) --
```
El servidor se demora 5 segundos en responder, confirmando que la condición es verdadera.

---

## Prevención de SQL Injection

1. **Usa consultas preparadas (Prepared Statements)**
   - Las consultas preparadas separan los datos de la lógica de la consulta.
   - Ejemplo en PHP:

   ```php
   $stmt = $pdo->prepare('SELECT * FROM usuarios WHERE usuario = ? AND contraseña = ?');
   $stmt->execute([$usuario, $contraseña]);
   ```

2. **Usa ORM (Object-Relational Mapping)**
   - Herramientas como `SQLAlchemy` (Python) o `Hibernate` (Java) gestionan las consultas de forma segura.

3. **Valida y sanitiza entradas**
   - Asegúrate de que los datos sean del tipo esperado (números, cadenas, etc.).
   - Escapa caracteres especiales usando funciones específicas como `mysqli_real_escape_string()` en PHP.

4. **Principio de mínimo privilegio**
   - Asegúrate de que las cuentas de la base de datos usadas por la aplicación tengan acceso solo a lo estrictamente necesario.

5. **Usa un WAF (Web Application Firewall)**
   - Los WAF pueden detectar y bloquear patrones de ataques comunes de SQLi.

6. **Auditorías y pruebas periódicas**
   - Realiza análisis estático y dinámico de tu código.
   - Usa herramientas como `sqlmap` para detectar vulnerabilidades.

---

## Herramientas para pruebas de SQLi

- [SQLMap](https://sqlmap.org/): Una herramienta automática para detectar y explotar SQLi.
- [Burp Suite](https://portswigger.net/burp): Un proxy de seguridad para realizar pruebas manuales.
- [OWASP ZAP](https://owasp.org/www-project-zap/): Herramienta gratuita para pruebas de aplicaciones web.

---

## Ejemplo seguro con Prepared Statements

En lugar de este código vulnerable:

```php
$consulta = "SELECT * FROM usuarios WHERE usuario = '" . $_POST['usuario'] . "' AND contraseña = '" . $_POST['contraseña'] . "';";
```

Usa Prepared Statements:

```php
$stmt = $pdo->prepare('SELECT * FROM usuarios WHERE usuario = ? AND contraseña = ?');
$stmt->execute([$_POST['usuario'], $_POST['contraseña']]);
$result = $stmt->fetch();
```

Esto asegura que las entradas sean tratadas como datos y no como código.

---

## Resumen

1. **SQL Injection** ocurre cuando los datos del usuario no se validan correctamente.
2. Puede permitir acceso no autorizado, robo o manipulación de datos.
3. Las **consultas preparadas** son la mejor defensa.
4. Usa herramientas de análisis para identificar y corregir vulnerabilidades.

---

¡Recuerda siempre seguir buenas prácticas de seguridad en el desarrollo de tus aplicaciones!
