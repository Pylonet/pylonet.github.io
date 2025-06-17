---
title: "Mi Primer Reflected XSS en Bug Bounty"
date: 2025-06-17 9:00:00 +0800
categories: [Blog, BugBounty]
tags: [BugBounty, Reflected XSS, XSS]
description: 
render_with_liquid: false
---

Hola buenas tardes a todos!! 👋☕️ Hoy venga a contaros como encontré mi primer Reflected XSS en un programa de Bug Bounty. Este bug no espero que sea recompensado económicamente, personalmente me vale para mi aprendizaje 🙂

![](/assets/images/blog/mi-primer-xss/8z0C.gif)

# ¿Qué es un Reflected XSS?

Un Reflected XSS es cuando nuestro contenido malicioso *(Por ejemplo: `<script>alert(1)</script>`)* enviado por el navegador web se refleja fuera del servidor, como un mensaje de error, un resultado de búsqueda o cualquier otra respuesta donde nuestra entrada de usuario se refleje.

- *Información sacada de [OWASP](https://owasp.org/www-community/attacks/xss/)*

# Proof Of Content

Vamos a ver como encontre el XSS!!

![](/assets/images/blog/mi-primer-xss/gCn.gif)

En la web vi un campo de búsqueda donde vi que se tramitaba por `POST`:
![](/assets/images/blog/mi-primer-xss/1.png)

Veo que se refleja nuestro input, podremos inyectar código HTML?..:

![](/assets/images/blog/mi-primer-xss/2.png)

Vemos que se inyecta, si replicamos esta solicitud en el navegador usando antes el siguiente payload:

```html
<img src=x onerror=alert(1) />
```

Vamos a replicar la solicitud en el navegador con BurpSuite:
![](/assets/images/blog/mi-primer-xss/3.png)

Vemos que funciona, pero esto no sería tan crítico ya que es un Self XSS. Pero vamos a escalarlo a un Reflected XSS. Con mi intriga me pregunte:

- ¿Con el parámetro que se emplea por `POST` funcionará por `GET`?

Voy a probar:

![](/assets/images/blog/mi-primer-xss/4.png)

Vemos que funciona!! Ahora vamos a probar el mismo payload que probamos antes con `POST`:

```html
<img src=x onerror=alert(1) />
```

![](/assets/images/blog/mi-primer-xss/5.png)

Vemos que lo hemos podido llevar a un Reflected XSS!! Por probar probe en mandarme la URL con el payload en otro navegador:

![](/assets/images/blog/mi-primer-xss/6.png)

# Conclusión

Se que no es gran cosa pero para mi personalmente es un gran avance para mi aprendizaje en este mundillo de Bug Bounty. Así que espero que os haya gustado mi primer Reflected XSS y nos vemos en la próxima!! 👋☕

* Recordar vuestro cafecito diario jeje ☕ *

---