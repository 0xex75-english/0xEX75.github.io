---
title: "Technique de la retour à la libc"
description: rop.jpg
---

![forthebadge made-with-python](https://media.giphy.com/media/xT9IgG50Fb7Mi0prBC/giphy.gif)

Prérequis :
- Avoir les bases en `pwn` de comprendre et comment attaquer un buffer overflow basique.
- Et d'un ordinateur, eh eh !

Aujourd'hui je souhaite vous présentez un article pour une nouvelle technique de `Buffer Overflow`. Une technique relativement amusante et très simple, êtes-vous intéressez ? Si c'est le cas, allons-y !

# C'est quoi la technique de la retourne à la libc ?

Ce que nous avons vue la dernière fois en rapport avec les `Buffer Overflow` pour exécuter un `shellcode`,  il fallait que la `pile/stack` soit exécutable. 

    root@0xEX75:~# readelf -lW ./testing |grep GNU_STACK
    GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RWE 0x10
    
Le `R` désigne tout simplement que la pile est en lecture et la lettre `W` correspond concrètement à l'écriture sur la pile et enfin la lettre `E` correspond à la pile exécutable, et c'est grâce à ce système que nous pouvons exécuter un shellcode car c'est le processeur qui exécute notre shellcode.

![forthebadge made-with-python](https://i.imgur.com/PTuJSy1.png)

Dans des cas assez spécifique, la pile n'est pas exécutable donc c'est presque impossible de faire exécuter un shellcode au programme pour `pop` un shell par exemple. Donc les experts ont trouver une solution qui se nomme la technique de `retourne à la libc`, qui permet d'utiliser des fonctions de la `libc`, comme par exemple la fonction `system();` pour ensuite l'utiliser contre le programme.

# Pratique exploitation !

(Pour cette partie, nous désactiverons l'`ASLR`, car la technique de retourne à la libc fonctionne uniquement si la pile n'est pas exécutable et que l'ASLR n'est pas activé.)

Voici un petit script en C qui ne fait pas grand chose :
