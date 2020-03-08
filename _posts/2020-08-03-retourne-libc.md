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

![forthebadge made-with-python](https://2.bp.blogspot.com/-UPzV6M_ZsK8/W3B5kWiwYII/AAAAAAAAAeE/L1izLVAJGbwfh52XG4HjMtPDDMXC-bLqACLcBGAs/s1600/ret2libc.png)

Dans des cas assez spécifique, la pile n'est pas exécutable donc c'est presque impossible de faire exécuter un shellcode au programme pour `pop` un shell par exemple. Donc les experts ont trouver une solution qui se nomme la technique de `retourne à la libc`, qui permet d'utiliser des fonctions de la `libc`, comme par exemple la fonction `system();` pour ensuite l'utiliser contre le programme.

# Pratique exploitation !

(Pour cette partie, nous désactiverons l'`ASLR`, car la technique de retourne à la libc fonctionne uniquement si la pile n'est pas exécutable et que l'ASLR n'est pas activé.)

Voici un petit script en C qui ne fait pas grand chose :

    #include <stdio.h>
    #include <string.h>

    void foo(char* string);

    int main(int argc, char** argv) {
              if (argc > 1)
                foo(argv[1]);
        return 0;
    }

    void foo(char* string) {
              char buffer[256];
              strcpy(buffer, string);
    }
    
Un programme basique qui ne fait pas grand chose, mais la vulnérabilité se trouve au niveau de la fonction `strcpy();`. Je suppose que vous savez que les fonctions comme `strcpy();`, `strcat();` etc.. ne sont pas du tout sécurisées donc il existe un système qui se nomme `FORTIFY_SOURCE` qui permet de remplacer les fonctions par des fonctions beaucoup plus sécurisées.

Ensuite, une petite compilation est nécessaire :

    root@0xEX75:~/libc# gcc -m32 -fno-stack-protector libc.c -o libc
    root@0xEX75:~/libc# readelf -lW libc|grep GNU_STACK
    GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
    
(Le flag `E` n'est pas là, donc la pile n'est plus du tout exécutable.). Si nous essayons d'exécuter le programme après la compilation, le programme ne retournera strictement rien du tout, mais dans la mémoire il se passe des choses.

    root@0xEX75:~/libc# ./libc $(python -c 'print "A"*263')
    root@0xEX75:~/libc# ./libc $(python -c 'print "A"*264')
    segmentation fault (core dumped)
    
Nous pouvons aperçevoir que le programme plante après 263 caractères, donc l'`OFFSET` correspond exactement à 263 caractères, si nous effectuons un dépassement, la sauvegarde `sEIP` sera complètement écrasé et le programme plantera automatiquement.
