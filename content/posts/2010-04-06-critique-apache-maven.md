---
layout: post
title: ! 'Critique : Apache Maven'
date: "2010-04-06T08:00:00+01:00"

description :  Critique du livre Apache Maven de Nicolas de Loof et Arnaud Héritier
categories:
- Livres
- Java
published: true
---
<a href="http://blog.hikage.be/wp-content/uploads/2010/03/ApacheMaven.gif"><img class="size-medium wp-image-28" title="Apache Maven" src="http://blog.hikage.be/wp-content/uploads/2010/03/ApacheMaven-243x300.gif" alt="Apache Maven" width="243" height="300" /></a>

#Informations

- Public : Débutants / Initiés
- ISBN : <a href="http://www.amazon.fr/Apache-Maven-Nicolas-loof/dp/274402337X/">274402337X</a> / <a href="http://www.amazon.fr/Apache-Maven-Nicolas-loof/dp/274402337X/">978-2744023378</a>
- Prix Amazon : 30,39 €





<!--more-->
#Sommaire
* Premiers pas avec Maven
 	* Introduction
 	* Au-delà de java.lang
 	* Un peu plus que compiler
 	* Mettre en place des tests unitaires
 	* Mettre en place des tests d’intégration
* Maven en entreprise
 	* Gestion avancée des dépendances
 	* Quand le projet devient trop lourd
 	* Maven et JEE
 	* Maven et les IDE
 	* Le jour J : la livraison
* Encore un peu plus loin avec Maven
 	* Utiliser un outil non supporté
 	* L’assurance qualité
 	* Respecter un format de distribution
 	* Un nouveau projet démarre
 	* Avons-nous fait le bon choix
 	* Nos recommandations
 	* Épilogue
 	* Lexique



#Résumé de l'éditeur
Maven, l'outil open-source de gestion et d'automatisation de développement Java, a le vent en poupe. Les raisons : il systématise, rationalise et simplifie le développement collaboratif de projets Java, faisant gagner aux entreprises comme aux développeurs du temps et de l'argent !

Les auteurs, membres de l'équipe de développement Maven, aidés par toute la communauté francophone, ont imaginé de présenter Maven 2 sous un angle original et didactique, à travers un projet fictif, inspiré de leurs expériences sur le terrain, dont ils détaillent toutes les phases successives. Ce projet évolue au fil des besoins et de la contribution de développeurs aux profils différents, vous familiarisant avec les concepts fondamentaux de Maven et leur mise en œuvre pratique, mais aussi avec les fonctionnalités plus avancées. Vous profitez également des recommandations et bonnes pratiques pour optimiser votre utilisation de Maven.
Vous découvrez ainsi de manière ludique et grâce à des exemples concrets le potentiel de Maven, et tous les avantages qu'il peut apporter à vos propres projets.
#Ma critique
##La forme
e livre est une vraie merveille au niveau de la présentation. Il se démarque clairement de la plupart des livres d'informatique que j'ai pu lire. Contrairement à ceux-ci (qui sont beaucoup plus formels, présentant de manière tout à fait neutre leur sujet) Apache Maven est beaucoup plus vivant et dynamique.

Ici, le livre est écrit comme une retranscription de la vie d'une équipe qui a choisi Maven. Nous avons droit à une dose d'humour, mais aussi à des débats où chacun défend son point de vue.
Et le résultat est étonnant, car il nous donne l'impression de vivre cette aventure avec eux, nous donnant envie de continuer d'avancer dans le livre sans attendre.
##Le contenu
Du point de vue du contenu, Apache Maven couvre les fonctionnalités les plus importantes et/ou les plus intéressantes de Maven, le tout réparti en trois parties :
###Premiers pas avec Maven
Dans cette partie, c'est tout d'abord le choix de Maven face à d'autres outils similaires qui est introduit. Sa force principale est les conventions apportées car elles permettent à n'importe quel développeur connaissant Maven de retrouver ses repères rapidement sur n'importe quel projet basé sur Maven.

Dans cette partie on découvre également la gestion de dépendances, les premiers plug-ins et comment générer des livrables.
###Maven en entreprise
Dans cette seconde partie, l'intégration de Maven dans les trois plus importants IDE (Eclipse, Netbeans et Idea)
est présentée, le tout sous la forme d'un débat, dont la conclusion est particulièrement intéressante ! La notion de dépôt d'entreprise est également introduite, permettant de réduire considérablement la bande passante vers Internet, mais également d'avoir une gestion plus fine de ce qui est permis ou non au sein des projets.

Et pour finir, les auteurs présentent un plug-in permettant de simplifier considérablement la livraison d'un livrable (deux commandes !)
###Encore plus loin avec Maven
Maven, c'est aussi tous ses plug-ins.
Mais comment faire quand il n'en existe aucun qui correspond à nos besoins ? Les auteurs nous montrent qu'il n'est pas sorcier de créer nos propres plug-ins Maven.

C'est ensuite la notion d'archétype qui est présentée : ceux-ci sont une espèce de template de projet préconfiguré. Exit le copier-coller d'un projet et de renommer à la main les fichiers qui vont bien. Les archétypes proposent une solution beaucoup plus élégante !

Les auteurs terminent finalement par le chapitre certainement le plus intéressant : des best practices basées sur leurs propres expériences
#Conclusion
Au final, j'ai réellement dévoré ce livre. Je connaissais et utilisais déjà Maven mais sans avoir pris le temps de regarder
toutes ses possibilités. À la fin de ce livre, j'avais qu'une seule envie : tester par moi-même ! Je conseille donc vivement ce livre à toute
personne ne connaissant pas (ou mal) Maven. Par contre pour les très bons connaisseurs, je doute que ce livre soit très intéressant,
sauf pour convaincre leur management d'utiliser Maven.
