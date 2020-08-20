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

Donc, l'attaque est relativement simple, supposons que nous avons un utilisateur qui se nomme `admin` et le chiffrement du `Cookie` est `21232f297a57a5a743894a0e4a801fc3`, notre but concrètement est de changer la valeur `admin` en changeant seulement les bits du `Cookie`, par exemple `vb232f297a57a5a743894a0e4a801fc3` qui deviendra `bdmin`, l'idée est là, c'est de changer le comportement du `Cookie` et de lui afficher quelque chose d'autre pour accéder à un compte.

# Pratique

Dans mon cas, j'utiliserai un serveur `XAMPP` et d'installer `Mullitidae`, Mullitidae est un environnement de `pentest`, n'hésitez pas à l'[installer](https://www.owasp.org/index.php/OWASP_Mutillidae_2_Project) pour faire des testes intéréssants. Démarrons simplement le service `APACHE` et `MYSQL`

![forthebadge made-with-python](https://github.com/0xEX75/0xEX75.github.io/blob/master/Capture.PNG?raw=true)

Dans la version 2.6.10 de Mutilidae, il existe une page appelée Niveau de privilège utilisateur. Ceci est conçu pour pratiquer l'attaque de retournement de bits CBC. Il se trouve sous: OWASP 2013, Authentification interrompue et gestion de session, Échelle de privilèges, afficher les privilèges des utilisateurs. 

Comme vous pouvez le voir, le but de ce défi est de changer l'utilisateur et le groupe en `000`. La première chose dont nous avons besoin est l'`IV`. Nous devons utiliser un proxy qui se situe entre nous et le serveur pour intercepter la communation entre le `client` et le `serveur`. J'utiliserai `BurpSuite` pour cela. Burp suite est un outil utilisé pour aider au pentesting d'applications Web. Vous devez configurer votre navigateur pour passer par le proxy burp. La configuration du burp est hors de portée pour ce poste.

![forthebadge made-with-python](https://raw.githubusercontent.com/0xEX75/0xEX75.github.io/master/000.PNG)

Ensuite, interceptons la communication entre le client et serveur pour falsifier la communication entre le client et le serveur à l'aide de `BurpSuite`.
