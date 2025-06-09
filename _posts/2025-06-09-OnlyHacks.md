---
title: "HTB Challenges (Web): OnlyHacks"
date: 2025-06-09 14:10:00 +0800
categories: [Writeups, HackTheBox]
tags: [HackTheBox, XSS, JavaScript]
description: OnlyHacks un challenge con una dificultad Very Easy, donde encontraremos una aplicación web para buscar tu perfecto amor. Dandole like a algunos perfiles podremos hablarles por chat para conectar con ellos/as. En el chat vemos reflejado nuestro input y a nivel de front-end no vemos ningún tipo de sanitización de nuestro input pudiendo inyectar código HTML (HTML Injection) pudiendolo llevar perfectamente a un XSS donde también será afectada la misma persona del chat pudiendo robarle las cookies de su sesión para inicar como ella y encontrar la flag.
render_with_liquid: false
---

---
Vamos a desplegar el Challenge:
![](/assets/images/hackthebox/challenges/onlyhacks/1.png)

Vamos a ver la web!!

![](/assets/images/hackthebox/challenges/onlyhacks/2.png)

Vemos que tenemos un inicio de sesión y un registro, como no tenemos cuenta vamos a crearnos una:
![](/assets/images/hackthebox/challenges/onlyhacks/3.png)

Vemos que podemos darle like, vamos a darle a todos like a ver que ocurre:
![](/assets/images/hackthebox/challenges/onlyhacks/4.png)

Nada, vamos a ir a la sección `Matches` a ver si se ha interesado por mi 😅:

![](/assets/images/hackthebox/challenges/onlyhacks/5.png)

Bien!! Le interese a **Renata**, vamos a probar a chatear con ella:
![](/assets/images/hackthebox/challenges/onlyhacks/6.png)

Vemos que todo con normalidad, pero... Y si pruebo código HTML?

Vamos a probar el siguiente código HTML:

```html
<h1>Magic!!</h1>
```

![](/assets/images/hackthebox/challenges/onlyhacks/7.png)

😯, vemos que podemos usar código HTML. Vamos a probar a hacer un `alert(1)` para ver si salta:

![](/assets/images/hackthebox/challenges/onlyhacks/8.png)

Bien!! Es vulnerable a un XSS, ahora me entra la siguiente duda... ¿Esto le salta a Renata también?

Para comprobarlo vamos a usar [WebHook](https://webhook.site) para usarlo como servidor público y enviarme una petición `GET`, vamos a probar el siguiente payload:

```html
<script>fetch('tu-sitio-webhook')</script>
```

![](/assets/images/hackthebox/challenges/onlyhacks/9.png)

👀
