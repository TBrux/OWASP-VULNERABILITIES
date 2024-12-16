# Cross-Site Scripting (XSS)

## ¿Qué es Cross-Site Scripting (XSS)?

**Cross-Site Scripting (XSS)** es una vulnerabilidad que permite a un atacante inyectar código malicioso (como JavaScript) en una aplicación web. Este código se ejecuta en el navegador de los usuarios que visitan la aplicación comprometida, permitiendo al atacante robar datos, manipular la interfaz o realizar acciones en nombre del usuario.

Existen tres tipos principales de XSS:

1. **XSS Reflejado (Reflected XSS)**
2. **XSS Persistente (Stored XSS)**
3. **XSS Basado en DOM (DOM-Based XSS)**

---

## 1. XSS Reflejado (Reflected XSS)

El XSS reflejado ocurre cuando el servidor refleja datos proporcionados por el usuario en la respuesta HTTP sin validarlos ni sanitizarlos. Este tipo de ataque suele explotarse mediante URLs maliciosas.

### Ejemplo:

Una aplicación tiene una URL como esta:

```
https://example.com/buscar?q=usuario
```

El servidor genera una respuesta:

```html
<h1>Resultados para: usuario</h1>
```

Un atacante puede inyectar código malicioso manipulando la URL:

```
https://example.com/buscar?q=<script>alert('XSS')</script>
```

El servidor responde:

```html
<h1>Resultados para: <script>alert('XSS')</script></h1>
```

Cuando el navegador interpreta esta respuesta, ejecuta el código JavaScript.

### Impacto:
- Robo de cookies de sesión.
- Redirecciones maliciosas.
- Ejecución de acciones en nombre del usuario.

### Prevención:
- Escapar caracteres especiales en las respuestas HTML (`<`, `>`, `"`, `&`, etc.).
- Usar funciones como `htmlspecialchars()` en PHP o `HTMLEncode()` en .NET.
- Validar y sanitizar entradas del usuario.

---

## 2. XSS Persistente (Stored XSS)

El XSS persistente ocurre cuando los datos maliciosos inyectados se almacenan en la base de datos o en el sistema de la aplicación y se muestran a otros usuarios.

### Ejemplo:

Un foro permite a los usuarios publicar comentarios. El atacante publica:

```html
<script>alert('Tu sesión ha sido robada');</script>
```

El código malicioso se almacena en la base de datos y aparece cuando otros usuarios leen el comentario:

```html
<div class="comentario">
    <script>alert('Tu sesión ha sido robada');</script>
</div>
```

El navegador de cada usuario ejecuta el código cuando carga la página del comentario.

### Impacto:
- Afecta a múltiples usuarios, no solo a uno.
- Robo masivo de datos o cookies.
- Alteración de contenido visible para todos.

### Prevención:
- Escapar y sanitizar datos antes de almacenarlos.
- Usar frameworks que prevengan XSS por defecto (como React o Angular).
- Implementar Content Security Policy (CSP) para bloquear código no autorizado.

---

## 3. XSS Basado en DOM (DOM-Based XSS)

El XSS basado en DOM ocurre cuando el código JavaScript del cliente manipula el DOM directamente con datos maliciosos proporcionados por el usuario, sin que haya interacción con el servidor.

### Ejemplo:

El siguiente código HTML usa el parámetro `q` de la URL para mostrar un mensaje:

```html
<p id="resultado"></p>
<script>
    const params = new URLSearchParams(window.location.search);
    const q = params.get('q');
    document.getElementById('resultado').innerHTML = `Buscaste: ${q}`;
</script>
```

Un atacante usa la URL:

```
https://example.com/?q=<script>alert('XSS')</script>
```

Esto genera:

```html
<p id="resultado">Buscaste: <script>alert('XSS')</script></p>
```

El navegador ejecuta el código malicioso.

### Impacto:
- Robo de información del usuario.
- Alteración de la interfaz.

### Prevención:
- Nunca usar `innerHTML` para insertar datos. Usa `textContent` o `innerText`.
- Validar y sanitizar datos en el cliente y en el servidor.
- Implementar CSP para evitar la ejecución de código no autorizado.

---

## Ejemplo Seguro

En lugar de usar:

```javascript
const params = new URLSearchParams(window.location.search);
const q = params.get('q');
document.getElementById('resultado').innerHTML = `Buscaste: ${q}`;
```

Usa:

```javascript
const params = new URLSearchParams(window.location.search);
const q = params.get('q');
document.getElementById('resultado').textContent = `Buscaste: ${q}`;
```

Esto evita la ejecución de código malicioso.

---

## Herramientas para Detectar XSS

- [Burp Suite](https://portswigger.net/burp): Excelente para pruebas de seguridad manuales y automáticas.
- [OWASP ZAP](https://owasp.org/www-project-zap/): Escáner gratuito para detectar vulnerabilidades como XSS.
- [DOMPurify](https://github.com/cure53/DOMPurify): Librería para sanitizar datos y prevenir XSS en aplicaciones web.

---

## Resumen

1. **XSS** permite a atacantes ejecutar código malicioso en navegadores de usuarios.
2. Los tres tipos principales son **Reflejado**, **Persistente** y **Basado en DOM**.
3. **Validar y sanitizar entradas** es la mejor defensa contra XSS.
4. Usar herramientas y librerías modernas como **CSP** y **DOMPurify** aumenta la protección.
