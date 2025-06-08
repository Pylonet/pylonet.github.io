---
title: Flag Command Challenge HackTheBox
date: 2025-06-08 14:10:00 +0800
categories: [Writeups,HackTheBox, Challenges]
tags: [HackTheBox, API]
#image: /assets/images/hackthebox/challenges/
description: Flag Command un challenge con una dificultad Very Easy donde nos encontraremos una web donde se encuentra un juego, leyendo algunos códigos JavaScript nos damos cuenta que hay una API por detrás donde comunicandonos con ella encontraremos un mensaje oculto para que nos de la flag.
render_with_liquid: false
---

Vamos a iniciar el Challenge:

![](/assets/images/hackthebox/challenges/flag-command/1.png)

Perfecto vamos a ver que hay:

![](/assets/images/hackthebox/challenges/flag-command/2.png)

Vemos que la web tiene estetica de terminal, vamos a probar a poner cualquier cosa a ver que ocurre:

![](/assets/images/hackthebox/challenges/flag-command/3.png)

Vale, nos indica que para listar los comandos posibles pongamos `help`:

![](/assets/images/hackthebox/challenges/flag-command/4.png)

Hmmm, mientras veia todo esto mi instinto de la curiosidad empezaba a preguntar:
- ¿Qué esta ocurriendo por detrás? 🤔
- ¿Como me estoy comunicando con la aplicación web? 🤔

Así que decidí abrir en las `DevTools` de Firefox el `Debugger` y encontre estos 2 archivos:

![](/assets/images/hackthebox/challenges/flag-command/5.png)

Hmmm, vamos a curiosear `game.js`:

````js
import { displayLineInTerminal } from "./main.js";
import { GAME_LOST, GAME_WON } from "./commands.js";

// GAME MECHANICS
// ---------------------------------------
const timeDelay = 1000;

function displayGameResult(message, style) {
    setTimeout(() => {
        displayLineInTerminal({
            text: message,
            style: `${style} margin-right`,
            useTypingEffect: true,
            addPadding: true,
        });
    }, timeDelay);
}

export function playerLost() {
    displayGameResult(GAME_LOST, "error");
}

export function playerWon() {
    displayGameResult(GAME_WON, "success");
}
```