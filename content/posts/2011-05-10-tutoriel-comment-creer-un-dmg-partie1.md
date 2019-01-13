---
layout: post
title: ! 'Tutoriel : Comment créer un DMG'
description : Comment créer un DMG, les archives spécifiques à Mac OS X
tags:
- OSX
- Packaging
date : "2011-05-11T08:00:00+01:00"
---
Les utilisateurs de Mac utilisent souvent de type de fichier, qui sont en fait des images disques (comme des .iso) qui peuvent être montées et qui apparaitront comme un nouveau disque sous Mac OS X.

Ce type de fichier est très souvent utilisé sous Mac comme moyen d'installer des logiciels.
Afin de fournir un Bundle pour OpenJDK pour Mac OS X, j'ai voulu regarder de plus près comment faire pour créer un tel fichier.
<!--more-->

##Créer le dmg
La première étape consiste à créer un fichier .dmg via l'utilitaire de disque, via le bouton "Nouvelle Image" :

<a href="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.01.57.png"><img src="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.01.57-300x251.png" alt="" title="Utilitaire de disque" width="300" height="251" class="aligncenter size-medium wp-image-242" /></a>

La nouvelle fenêtre permet de spécifier le type d'image que l'on souhaite :

<a href="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.02.25.png"><img src="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.02.25-300x260.png" alt="" title="Création d&#039;une image" width="300" height="260" class="aligncenter size-medium wp-image-244" /></a>

* Nom : Le nom de l'image une fois montée
* Taille : La taille maximum du conteneur, il est nécessaire de mettre suffisamment de place, voir trop. L'image finale sera réduite plus tard
* Format : Mac OS X Etendu
* Chiffrement : Aucun
* Partition : Partition unique - Table de partition Apple
* Format d'image : Image disque en lecture / écriture

Le format d'image en lecture / écriture sera modifié plus tard afin de ne pas laisser au "utilisateur" de modifier une image .dmg fournie.

##Remplir le DMG
Une fois le DMG créé, il est monté et visible sur le bureau :

<a href="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.25.15.png"><img src="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.25.15-300x234.png" alt="" title="Image disque montée" width="300" height="234" class="aligncenter size-medium wp-image-247" /></a>

Il suffit de l'ouvrir pour afficher son contenu, vide pour l'instant, et de le modifier comme n'importe quel répertoire


##Modifier son icone
Comme pour n'importe quel fichier sous Mac, il est possible de modifier l'icone du volume une fois monté.
Pour cela, il faut tout d'abord ouvrir l'image que l'on souhaite utilisé comme icone avec l'<strong>Aperçu</strong>.

Dans celui-ci, faire <i>Command+A</i> pour sélectionner l'image, puis <i>Command+C</i> pour copier celle-ci.

Une fois cela fait, aller sur le bureau, click-droit sur l'icone de votre image montée et sélectionner "Lire les informations"

<a href="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.47.31.png"><img src="http://blog.hikage.be/wp-content/uploads/2011/04/Capture-d’écran-2011-04-25-à-21.47.31-288x300.png" alt="" title="Lire les informations" width="288" height="300" class="aligncenter size-medium wp-image-249" /></a>

Cliquer ensuite sur l'icone tout en haut à gauche de la fenêtre qui vient de s'ouvrir, et faire <i>Command+V</i>
Cette dernière devrait être devenue l'image que vous avez choisie.


##Modifier l'arrière plan
Il est également possible de modifier l'arrière plan de la fenêtre qui sera ouverte lors de l'accès au volume.
Voici par exemple à quoi ressemble l'image OpenJDK :

<a href="http://blog.hikage.be/wp-content/uploads/2011/05/openjdk.png"><img src="http://blog.hikage.be/wp-content/uploads/2011/05/openjdk-296x300.png" alt="" title="openjdk" width="296" height="300" class="aligncenter size-medium wp-image-253" /></a>

Pour cela, il faut que l'image soit elle même présente dans le DMG, mais que le fichier lui même ne soit pas visible.
Pour cela, il suffit de créer un répertoire <strong>.background</strong> dans lequel nous viendront mettre l'image.

Ensuite, à la racine du volume monté, il suffit de faire un click-droit et de séléctionner "Afficher les options de présentation" :
<a href="http://blog.hikage.be/wp-content/uploads/2011/05/option-de-presentation.png"><img src="http://blog.hikage.be/wp-content/uploads/2011/05/option-de-presentation-147x300.png" alt="" title="option de presentation" width="147" height="300" class="aligncenter size-medium wp-image-254" /></a>

Dans la section "Arrière plan" il suffira de faire glisser l'image du répertoire .background dans l'espace "Faire glisser ici".

##Finaliser l'image
Une fois que le Volume contient les fichiers que vous souhaitez publier et que la présentation vous convient, il faut finaliser l'image.

Pour cela, nous retournons dans l'utilitaire de disque, dans la frame de gaucher, selectionnez le dmg que vous venez de créer, et cliquez sur "Convertir" dans le menu du haut.

Sélectionner le format "Compressé", et cliquez sur OK.
Le nouveau DMG sera réduit à la taille que vous utilisez rééllement, et ne pourra plus être modifié.

Il est donc prêt à être livrer !

##Amélioration
Si il est facile de créer un DMG de cette manière, cela demande de nombreuse manipulation par le développeur.
Dans un prochain article, j'expliquerai comment il est possible de scripter la création de ce DMG.





 
