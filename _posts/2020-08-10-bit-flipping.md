---
title: "Attaque par bit flipping"
description: sam.png
tags: ["Dans cet article je vous présente comment exploiter une vulnérabilité pour bypass le système NX avec la technique de la retourne à la libc."]
---

Nous allons ici expliquer ce qui se cache derrière la notion de bit flipping. Une attaque par retournement de bits est une attaque sur un chiffrement cryptographique dans laquelle l'attaquant peut modifier le texte chiffré de manière à entraîner un changement prévisible du texte en clair, bien que l'attaquant ne soit pas en mesure d'apprendre le texte en clair lui-même.

J'étais confontré à un retournement de bits sur HTB dans la boxe Lazy, sur une vulnérabilité de type `Padding Oracle` et j'ai réussi à m'introduire dans la machine en changement seulement les bits du `cookie`.
