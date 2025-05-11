---
title: Los drivers de NVIDIA no compilan correctamente en Arch Linux
author: pylon
date: 2025-05-11 14:10:00 +0800
categories: [Blog]
tags: [Arch Linux, Nvidia]
image: /assets/images/blog/Los drivers de NVIDIA no compilan correctamente en Arch Linux/1.png
description: En algunos casos al actualizar tu Arch Linux puede que veas una actualización del kernel. Previamente instalamos los drivers NVIDIA en nuestro sistema donde los compilamos al kernel anterior, al actualizar el kernel puede que tus drivers no sean compatibles con el nuevo kernel. Hay posibilidades de arreglarlo pero lo más fácil y factible es volver al kernel anterior donde los drivers funcionaban correctamente.
render_with_liquid: false
---

# Pacman cache

Para ver la cache de pacman podemos hacer un `ls` al directorio `/var/cache/pacman/pkg` y si tenemos suerte y no hemos borrado la cache al actualizar podemos encontrar el paquete del kernel anterior donde funcionaban los drivers NVIDIA:

```bash
❯ ls /var/cache/pacman/pkg/ | grep linux-zen
linux-zen-6.11.6.zen1-1-x86_64.pkg.tar.zst
linux-zen-6.11.6.zen1-1-x86_64.pkg.tar.zst.sig
linux-zen-6.12.1.zen1-1-x86_64.pkg.tar.zst
linux-zen-6.12.1.zen1-1-x86_64.pkg.tar.zst.sig
linux-zen-headers-6.11.6.zen1-1-x86_64.pkg.tar.zst
linux-zen-headers-6.11.6.zen1-1-x86_64.pkg.tar.zst.sig
linux-zen-headers-6.12.1.zen1-1-x86_64.pkg.tar.zst
linux-zen-headers-6.12.1.zen1-1-x86_64.pkg.tar.zst.sig
```

En mi caso donde ha funcionado es en la `6.11.6` por lo que podemos seguir los pasos de la Wiki de Arch:

- [https://wiki.archlinux.org/title/Downgrading_packages](https://wiki.archlinux.org/title/Downgrading_packages)

# Downgrade

Si no podemos encontrar el kernel anterior donde funcionaban los drivers usaremos una herramienta llamada **Downgrade**.

Comparto el repositorio:

- [https://github.com/archlinux-downgrade/downgrade](https://github.com/archlinux-downgrade/downgrade)

Si tenemos `yay`podremos instalarlo fácilmente:

```bash
yay -S downgrade
```

Si no tenemos `yay` podemos instalarlo, os comparto el repositorio donde explica su instalación:

- [https://github.com/Jguer/yay?tab=readme-ov-file#installation](https://github.com/Jguer/yay?tab=readme-ov-file#installation)

Una vez instalado el downgrade pondremos el siguiente comando buscando el kernel que usamos *(en mi caso linux-zen)*:

```bash
sudo downgrade linux-zen
```

Ahora buscaremos la versión de kernel que tuvimos antes y le daremos a Enter y empezará a instalarse.

> **RECUERDA**: Tendrás que desinstalar linux-headers o linux-zen-headers dependiendo de cual estés usando e instalarlos también con el mismo comando downgrade pero buscando linux-zen-headers.
{: .prompt-warning }
