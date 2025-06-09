---
title: "HTB Challenges (Web): Flag Command"
date: 2025-06-08 14:10:00 +0800
categories: [Writeups, HackTheBox]
tags: [HackTheBox, API, JavaScript]
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

```js
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

No vemos nada interesante en este archivo, vamos a `main.js`:
```js
import { START, INFO, INITIAL_OPTIONS, HELP } from "./commands.js";
import { playerLost, playerWon } from "./game.js";

let availableOptions;

let currentStep = 1;
// SELECT HTML ELEMENTS
// ---------------------------------------
export const beforeDiv = document.getElementById("before-div"),
    currentCommandLine = document.getElementById("current-command-line"),
    commandText = document.getElementById("commad-written-text"),
    userTextInput = document.getElementById("user-text-input");

const typingSound = new Audio();
typingSound.src = document.getElementById("typing-sound").src;
typingSound.loop = true;

// COMMANDER VARIABLES
// ---------------------------------------
let currentCommand = 0,
    commandHistory = [],
    typingSpeed = 10,
    typing = true,
    playAudio = true,
    fetchingResponse = false,
    gameStarted = false,
    gameEnded = false;

export const startCommander = async () => {
    await fetchOptions();
    userTextInput.value = "";
    commandText.innerHTML = userTextInput.value;

    await displayLinesInTerminal({ lines: INFO });

    userTextInput.focus();
};

// HTTP REQUESTS
// ---------------------------------------
async function CheckMessage() {
    fetchingResponse = true;
    currentCommand = commandHistory[commandHistory.length - 1];

    if (availableOptions[currentStep].includes(currentCommand) || availableOptions['secret'].includes(currentCommand)) {
        await fetch('/api/monitor', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ 'command': currentCommand })
        })
            .then((res) => res.json())
            .then(async (data) => {
                console.log(data)
                await displayLineInTerminal({ text: data.message });

                if(data.message.includes('Game over')) {
                    playerLost();
                    fetchingResponse = false;
                    return;
                }

                if(data.message.includes('HTB{')) {
                    playerWon();
                    fetchingResponse = false;

                    return;
                }

                if (currentCommand == 'HEAD NORTH') {
                    currentStep = '2';
                }
                else if (currentCommand == 'FOLLOW A MYSTERIOUS PATH') {
                    currentStep = '3'
                }
                else if (currentCommand == 'SET UP CAMP') {
                    currentStep = '4'
                }

                let lineBreak = document.createElement("br");


                beforeDiv.parentNode.insertBefore(lineBreak, beforeDiv);
                displayLineInTerminal({ text: '<span class="command">You have 4 options!</span>' })
                displayLinesInTerminal({ lines: availableOptions[currentStep] })
                fetchingResponse = false;
            });


    }
    else {
        displayLineInTerminal({ text: "You do realise its not a park where you can just play around and move around pick from options how are hard it is for you????" });
        fetchingResponse = false;
    }
}

// TEXT FUNCTIONS
// ---------------------------------------
const typeText = async (element, text) => {
    if (playAudio && typingSound.paused) {
        typingSound.play();
    }

    for (let i = 0; i < text.length; i++) {
        if (text.charAt(i) === " " && text.charAt(i + 1) === " ") {
            element.innerHTML += "&nbsp;&nbsp;";
            i++;
        } else {
            element.innerHTML += text.charAt(i);
        }
        await new Promise((resolve) => setTimeout(resolve, typingSpeed));
    }

    if (playAudio) {
        typingSound.pause();
        typingSound.currentTime = 0;
    }
};

const createNewLineElement = ({ style = "", addPadding = false }) => {
    // remove the current command line until new line is displayed
    currentCommandLine.classList.remove("visible");
    currentCommandLine.style.opacity = 0;

    const nextLine = document.createElement("p");

    // add style depending on the type of line
    nextLine.className = style + (addPadding ? " spaced-line" : "");

    beforeDiv.parentNode.insertBefore(nextLine, beforeDiv);
    window.scrollTo(0, document.body.offsetHeight);

    return nextLine;
};

// process remaining text with styled and unstyled parts and apply typing effect
const processTextWithTypingEffect = async (nextLine, text) => {
    let remainingText = text;

    // process remaining text with styled and unstyled parts
    while (remainingText) {
        const styledElementMatch = remainingText.match(/<(\w+)(?:\s+class=['"]([^'"]*)['"])?>([^<]*)<\/\1>/);
        const unstyledText = styledElementMatch ? remainingText.slice(0, styledElementMatch.index) : remainingText;

        // handle unstyled text
        if (unstyledText) {
            await typeText(nextLine, unstyledText);
        }

        // handle styled text
        if (styledElementMatch) {
            const [, tagName, className, innerText] = styledElementMatch;
            const styledElement = document.createElement(tagName);
            if (className) {
                styledElement.className = className;
            }
            nextLine.appendChild(styledElement);
            await typeText(styledElement, innerText);
            remainingText = remainingText.slice(styledElementMatch.index + styledElementMatch[0].length);
        } else {
            remainingText = null;
        }
    }
};

// display a line in the terminal with optional styling and typing effect
export const displayLineInTerminal = async ({ text = "", style = "", useTypingEffect = true, addPadding = false }) => {
    typing = true;

    // create and style a new line element
    const nextLine = createNewLineElement({ style, addPadding });

    // use typing effect if enabled
    await processTextWithTypingEffect(nextLine, text);


    // reset typing flag and make the current command line visible
    typing = false;
    currentCommandLine.style.opacity = 1;
    currentCommandLine.classList.add("visible");
};

// display multiple lines in the terminal with optional styling and typing effect
export const displayLinesInTerminal = async ({ lines, style = "", useTypingEffect = true }) => {
    for (let i = 0; i < lines.length; i++) {
        await new Promise((resolve) => setTimeout(resolve, 0));

        await displayLineInTerminal({ text: lines[i], style: style });
    }
};

// EVENT LISTENERS
// ---------------------------------------
// user input keydown event listener
const keyBindings = {
    Enter: () => {
        // if a response is being fetched, do nothing on Enter
        if (fetchingResponse) {
            return;
        } else {
            commandHistory.push(commandText.innerHTML);
            currentCommand = commandHistory.length;
            displayLineInTerminal({ text: `>> ${commandText.innerHTML}`, useTypingEffect: true, addPadding: true });
            commander(commandText.innerHTML.toLowerCase());
            commandText.innerHTML = "";
            userTextInput.value = "";
        }
    },

    ArrowUp: () => {
        if (currentCommand > 0) {
            currentCommand -= 1;
            commandText.innerHTML = commandHistory[currentCommand];
            userTextInput.value = commandHistory[currentCommand];
        }
    },

    ArrowDown: () => {
        if (currentCommand < commandHistory.length) {
            currentCommand += 1;
            if (commandHistory[currentCommand] === undefined) {
                userTextInput.value = "";
            } else {
                userTextInput.value = commandHistory[currentCommand];
            }
            commandText.innerHTML = userTextInput.value;
        }
    },
};

// available user commands
export const commandBindings = {
    help: () => {
        displayLinesInTerminal({ lines: HELP });
    },

    start: async () => {
        await displayLineInTerminal({ text: START });
        let lineBreak = document.createElement("br");

        beforeDiv.parentNode.insertBefore(lineBreak, beforeDiv);
        await displayLinesInTerminal({ lines: INITIAL_OPTIONS });
        gameStarted = true;
    },
    clear: () => {
        while (beforeDiv.previousSibling) {
            beforeDiv.previousSibling.remove();
        }
    },

    audio: () => {
        if (playAudio) {
            playAudio = false;
            displayLineInTerminal({ text: "Audio turned off" });
        } else {
            playAudio = true;
            displayLineInTerminal({ text: "Audio turned on" });
        }
    },

    restart: () => {
        let count = 6;

        function updateCounter() {
            count--;

            if (count <= 0) {
                clearInterval(counter);
                return location.reload();
            }

            displayLineInTerminal({
                text: `Game will restart in ${count}...`,
                style: status,
                useTypingEffect: true,
                addPadding: false,
            });
        }

        // execute the code block immediately before starting the interval
        updateCounter();
        currentStep = 1

        let counter = setInterval(updateCounter, 1000);
    },

    info: () => {
        displayLinesInTerminal({ lines: INFO });
    },
};

// keyup event listener
export const enterKey = (event) => {
    if (!typing) {
        if (event.key in keyBindings) {
            keyBindings[event.key]();
            event.preventDefault();
        } else {
            commandText.innerHTML = userTextInput.value;
        }
    }
};

// command handler
const commander = (commandText) => {
    const cleanCommand = commandText.toLowerCase().trim();

    // Possible states:
    // 1. game has not started (gameStarted = false)
    // 2. game is in progress (gameStarted = true, gameEnded = false)
    // 3. game has ended (gameStarted = true, gameEnded = true)

    if (cleanCommand in commandBindings) {
        if (!gameStarted) {
            // game has not started
            commandBindings[cleanCommand]();
        } else if (gameStarted && !gameEnded) {
            // game is in progress
            commandBindings[cleanCommand]();
        } else {
            // game has ended
            if (cleanCommand === "restart" || cleanCommand !== "start") {
                commandBindings[cleanCommand]();
            } else {
                displayEndGameMessage();
            }
        }
    } else {
        if (gameStarted && !gameEnded) {
            CheckMessage();
        } else if (gameEnded) {
            displayEndGameMessage();
        } else {
            displayLineInTerminal({
                text: `'${cleanCommand}' command not found. For a list of commands, type '<span class="command">help</span>'`,
                useTypingEffect: true,
            });
        }
    }
};

const displayEndGameMessage = () => {
    displayLineInTerminal({
        text: "The game has ended. Please type <span class='command'>restart</span> to start a new game or <span class='command'>help</span> for a list of commands.",
        useTypingEffect: true,
    });
};

const fetchOptions = () => {
    fetch('/api/options')
        .then((data) => data.json())
        .then((res) => {
            availableOptions = res.allPossibleCommands;

        })
        .catch(() => {
            availableOptions = undefined;
        })
}
```

Se que toca un momento aburrido pero es lo que hay, toca... REVISIÓN DE CÓDIGO!!
![](/assets/images/hackthebox/challenges/flag-command/jVo.gif)

No vamos a ir parte por parte, os voy a poner las zonas importantes para entender este Challenge y resolverlo 🙂:


```js
[...]
    if (availableOptions[currentStep].includes(currentCommand) || availableOptions['secret'].includes(currentCommand)) {
        await fetch('/api/monitor', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ 'command': currentCommand })
        })
[...]
```

Podemos ver que esta enviando una solicitud `POST` con `fetch` a `/api/monitor` con la cabecera `Content-Type: application/json`, vamos a replicarla con `curl` a ver que nos dice:

```bash
❯ curl -s -X POST 'http://94.237.59.174:46361/api/monitor' -H 'Content-Type: application/json' | jq
{
  "message": "400 Bad Request"
}
```

Oh... Vemos que nos comenta `Bad Request`. Vamos a seguir leyendo! 

```js
[...]
                if(data.message.includes('Game over')) {
                    playerLost();
                    fetchingResponse = false;
                    return;
                }

                if(data.message.includes('HTB{')) {
                    playerWon();
                    fetchingResponse = false;

                    return;
                }
[...]
```

Esta zona es donde me ha llamado mucho ya que vemos 2 condicionales donde:
1. Si el mensaje incluye `Game Over` ejecuta la función `playerLost()`
2. Si el mensaje incluye `HTB{` ejecuta la función `playerWon()`

Leyendo esto pienso... Y si pongo en la web `HTB{` ganaré?..:

![](/assets/images/hackthebox/challenges/flag-command/6.png)

![](/assets/images/hackthebox/challenges/flag-command/fA1.gif)

Vamos a seguir leyenedo el código:

```js
[...]
const fetchOptions = () => {
    fetch('/api/options')
        .then((data) => data.json())
        .then((res) => {
            availableOptions = res.allPossibleCommands;

        })
        .catch(() => {
            availableOptions = undefined;
        })
}
```
En las últimas líneas podemos ver esto, esta enviando una petición `GET` al endpoint `/api/options`, vamos a probarlo con `curl`:

```bash
❯ curl -s -X GET 'http://94.237.59.174:46361//api/options' -H 'Content-Type: application/json' | jq
{
  "allPossibleCommands": {
    "1": [
      "HEAD NORTH",
      "HEAD WEST",
      "HEAD EAST",
      "HEAD SOUTH"
    ],
    "2": [
      "GO DEEPER INTO THE FOREST",
      "FOLLOW A MYSTERIOUS PATH",
      "CLIMB A TREE",
      "TURN BACK"
    ],
    "3": [
      "EXPLORE A CAVE",
      "CROSS A RICKETY BRIDGE",
      "FOLLOW A GLOWING BUTTERFLY",
      "SET UP CAMP"
    ],
    "4": [
      "ENTER A MAGICAL PORTAL",
      "SWIM ACROSS A MYSTERIOUS LAKE",
      "FOLLOW A SINGING SQUIRREL",
      "BUILD A RAFT AND SAIL DOWNSTREAM"
    ],
    "secret": [
      "Blip-blop, in a pickle with a hiccup! Shmiggity-shmack"
    ]
  }
}
```

![](/assets/images/hackthebox/challenges/flag-command/3dxZ.gif)

Podemos ver `secret`, vamos a poner el mensaje que nos da en la web, pero antes de ponerlo pondremos `start` para iniciar el juego y ya si poner el mensaje secreto:

![](/assets/images/hackthebox/challenges/flag-command/7.png)

---