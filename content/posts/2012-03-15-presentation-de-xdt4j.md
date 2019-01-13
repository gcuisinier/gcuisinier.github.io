---
layout: post
date : "2012-03-15T08:00:00+01:00"
title: Présentation de xdt4j
categories :
- Java
- OpenSource
- Xml
---

## Introduction

Chez mon client, nous avons eu un besoin particulier : celui de pouvoir étendre des archetypes Maven.
Autrement dit, pouvoir créer un archetype qui se baserait sur un autre, et qui apporterait uniquement le delta de différence.

Pour certain fichier, nous n'avons pas eu le choix, nous avons été obliger de fournir une nouvelle version qui remplacerait la version initiale, par exemple les fichiers binaires tels que les images.

Par contre, pour le fichiers XML, et en particulier les fichiers pom.xml ou encore le fichier archetype-metadata.xml, c'était plus problématique. En effet, en cas de modification de l'un des ces fichiers dans l'archetype de base, nous aurions été obligé de copier ceux-ci, et ré-introduire les delta à la main... ce qui ne serait pas très productifs.

Partant du fait que ces fichiers sont du XML, il me semblait plus logique de partir du fichier de base et de ne décrire que les transformations à lui apporter. Quand on parle de transformation en XML, on pense tout de suite à XSLT. 
Mais il faut avouer que s'il est très puissant, il n'est pas forcément le plus simple à appréhender.

J'ai donc cherché des alternatives plus simples. En discutant avec un consultant .Net, celui-ci m'a présenté le système qui est utilisé par Microsoft pour gérer les fichiers de configuration de IIS : XML Document Transform.
J'ai tout de suite trouvé cette solution élégante pour répondre à notre problème.

Par chance, quelqu'un a eu la bonne idée de faire une première implémentation OpenSource (<a href="http://code.google.com/p/xdt/" title="XDT sur GoogleCode" target="_blank"></a> en se basant sur tout les exemples qu'il à trouvé dans la documentation et sur internet.
Son implémentation se basait sur des spécificités .Net, donc impossible de le porter tel quel en Java.
Cependant, son jeu de test était assez complet.

Grâce à celui-ci tests, j'ai pu développer en TDD une implémentation en Java : <a href="https://github.com/hikage/xdt4j" target="_blank">xdt4j</a>.
<!--more-->

## Utilisation

Le fonctionnement de XDT pour ajouter/modifier/supprimer des éléments dans un fichier XML est en réalité assez simple.
Le document XML décrivant les transformations est en réalité un document XML qui se conforme à la structure du document de base, mais sur lequel des attributs XDT ont été ajoutés pour décrire la modification voulue.

Prenons par exemple un fichier pom.xml qui servira de base :

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>be.hikage</groupId>
    <artifactId>xdt4j</artifactId>
    <version>1.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

	<dependencies>
		<dependency>
		    <groupId>log4j</groupId>
		    <artifactId>log4j</artifactId>
		    <version>1.2.16</version>
		    <packaging>bundle</packaging>
		</dependency>
	</dependencies>

</project>
```

La première modification que l'on souhaite apporter : Modifier le nom de l'artefact.
Pour cela, il suffit de créer un fichier XML ayant une structure identique, qui représentera la modification que l'on souhaite apporter.

Ici par exemple :

```xml
/tmp/temp_textmate.JcbHjN:2: warning: variable $KCODE is no longer effective; ignored
<project xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
    <artifactId xdt:Transform="Replace">xdt4j-children</artifactId>
</project>
```

Vous pouvez voir que pour exprimer les modifications, il suffit d'ajouter un attribut de type <code>xdt:Transform</code>.
Ici la valeur "Replace" signifie que l'on souhaite simplement remplacer l'élement du document de base par celui-ci.

Une fois les deux documents XML prêts, l'API Java de xdt4j est des plus simple :

```java
// Chargement des fichiers XML sous forme de Document dom4j
Document xmlBase = loadDocument(&quot;base.xml&quot;);
Document xmlTransform = loadDocument(&quot;transform.xml&quot;);

// Utilisation de xdt4j
XdtTransformer transformer = new XdtTransformer();
Document xmlResult = transformer.transform(xmlBase, xmlTransform);
```

Le résultat de la méthode XdtTransformer.transform() est une copie du document original, sur lequel les transformations ont été traités. Le document original n'est pas modifié.


## Transformations et Locators
D'autres transformations existent.
Avec la transformation "Insert", il serait possible d'ajouter une dépendance de cette manière:

```xml
<project xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
    <dependencies>
		<dependency xdt:Transform="Insert">
		    <groupId>log4j</groupId>
		    <artifactId>log4j</artifactId>
		    <version>1.2.16</version>
		    <packaging>bundle</packaging>
		</dependency>
	</dependencies>
</project>
```

Il est également possible de supprimer des élements avec la transformation "Remove".
Pour supprimer la dépendance sur log4j par exemple :

```xml
<project xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
    <dependencies>
		<dependency xdt:Transform="Remove" xdt:Locator="Condition(groupId/text()='log4j')">		  
		</dependency>
	</dependencies>
</project>
```

Vous remarquerez dans ce dernier exemple l'attribut xdt:Locator. Celui-ci permet de spécifier plus précisement sur quel élement l'on souhaite appliquer la modification.

En effet, la transformation "Remove" par défaut va supprimer le premier élément du document de base qui va correspondre à l'expression XPath de l'élement sur lequel la transformation est placée.
L'attribut Locator permet d'effectuer un filtre plus précis via les valeurs possibles <b>Condition</b>, <b>Match</b> ou <b>XPath</b>.

Dans ce cas-ci, <b>Condition</b> prends comme paramètre une condition XPath relative à l'élement courant.


<h3>Conclusion</h3>

Dans notre cas, xdt4j est une excellente réponse à notre besoin : l'extension d'archetype.

Il n'est pas forcément complet pour tout les cas de figures, mais il est ouvert et disponible sur GitHub : [https://github.com/hikage/xdt4j](https://github.com/hikage/xdt4j).

Toute participation est donc la bienvenue, que ce soit des patch, des avis, des besoins (ouvrez une issue sur GitHub).

Et bonne nouvelle, les releases de xdt4j sont disponible sur Maven Central :

```xml
<dependency>
  <groupId>be.hikage</groupId>
  <artifactId>xdt4j</artifactId>
  <version>1.0.2</version>
</dependency>
```

Les versions Snapshots de leurs coté sont disponibles sur le répository OpenSource de Sonatype : [https://oss.sonatype.org/](https://oss.sonatype.org/)



