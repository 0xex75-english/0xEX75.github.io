---
title: "Attaque par bit flipping"
description: sam.png
tags: ["Nous allons ici expliquer ce qui se cache derrière la notion de bit flipping. Une attaque par retournement de bits est une attaque sur un chiffrement cryptographique dans laquelle l'attaquant peut modifier le texte chiffré de manière à entraîner un changement prévisible du texte en clair, bien que l'attaquant ne soit pas en mesure d'apprendre le texte en clair lui-même.
"]
---

Nous allons ici expliquer ce qui se cache derrière la notion de bit flipping. Une attaque par retournement de bits est une attaque sur un chiffrement cryptographique dans laquelle l'attaquant peut modifier le texte chiffré de manière à entraîner un changement prévisible du texte en clair, bien que l'attaquant ne soit pas en mesure d'apprendre le texte en clair lui-même.

J'étais confontré à un retournement de bits sur HTB dans la boxe Lazy, sur une vulnérabilité de type `Padding Oracle` et j'ai réussi à m'introduire dans le compte administrateur en changement seulement les bits du `Cookie`, vous comprendrez mieux par la suite.

# Théorie

Chaque fois que vous vous connectez sur un `site web`, le serveur vous donnent un `Cookie`, pour la simple et bonne raison que cela permet tout simplement de maintenir une session entre le client et le serveur, si une personne récupère vos cookies, il serait capable de se connecter à votre compte sans mettre les identifiants d'identifications, c'est pour cela que la sécurité est important !

# Fonctionnement du CBC

Si votre message que vous souhaitez chiffrer est « hello », chaque fois que vous chiffrez le mot «hello», il en résultera toujours la même sortie chiffrée. Cela pose un risque de sécurité grave car un attaquant peut procéder à une attaque en chiffrant simplement une liste de mots, puis en les comparant aux valeurs chiffrées, révélant ainsi le jeton. L'attaquant peut alors créer son propre token, le crypter et l'utiliser pour se connecter en tant qu'autre utilisateur. CBC est un moyen de randomiser la sortie de la valeur chiffrée.

![forthebadge made-with-python](https://www.researchgate.net/profile/Mousa_Farajallah/publication/308826472/figure/fig1/AS:391837119467524@1470432657367/AES-encryption-system-in-CFB-mode.png)

Le système est simple, le chiffrement `CBC` fonctionne par bloc, c'est-à-dire que pour qu'un bloc soit `XORED`, il a besoin du bloc précédent pour qu'il soit `XORED`.

    C¹ = E(P¹ ⊕ IV)
    Cⁿ = E(Pⁿ ⊕ Cⁿ - 1) — si n > 1

Vous me poserez la question, comment la première valeur du bloc peut être chiffré, si il n'a pas de précédent ?
C'est là que le système `IV` (Initialization vector ou Vecteur d'initialisation) rentre en jeu, il randomise une donnée aléatoire pour que il soit XORED avec le premier bloc et ainsi de suite jusqu'au dernier bloc, la formule ci-dessus résume la finalité.

Donc, l'attaque est relativement simple, supposons que nous avons un utilisateur qui se nomme `admin` et le chiffrement du `Cookie` est `21232f297a57a5a743894a0e4a801fc3`, notre but concrètement est de changer la valeur `admin` en changeant seulement les bits du `Cookie`, par exemple `vb232f297a57a5a743894a0e4a801fc3` qui deviendra `bdmin`, l'idée est là, c'est de changer le comportement du `Cookie` est de lui afficher quelque chose d'autre pour accéder à un compte.

# Pratique


