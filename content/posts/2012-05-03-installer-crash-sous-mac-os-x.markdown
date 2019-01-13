---
layout: post
title: "Installer CRaSH sous Mac OS X"
description : Comment installer CraSH, LE shell pour accèder à une JVM
date : "2012-04-03T08:00:00+01:00"

comments: true
categories: 
- OSX
- Java
---
##CRaSH, c'est quoi ?
Derrière un nom qui ferait fuir n'importe qui, se cache un outil assez prometteur. CRaSH est un projet Open Source, sous licence LGPL, créé par Julien Viet. Et plus exactement un Shell qui permet de se connecter à une JVM et de la contrôler à l'aide de commande développée en Groovy.

Il est assez extensible, mais de base il vient avec des commandes pour JDBC, le logging, les Thread.
Pour en savoir plus, je vous recommande la lecture de la [documentation](http://julienviet.com/crash/), de regarder [un screencast](http://blog.julienviet.com/2012/04/13/crash-standalone/) ou de parcourir [les slides](http://www.slideshare.net/jviet/crash-12638600) de Julien lors de son quickies à Devoxx France

##Comment l'installer ?
CrAsh est disponible sous la forme d'une [archive tar.gz](http://code.google.com/p/crsh/downloads/detail?name=crsh-1.0.0.tar.gz). Mais sous Mac OS X, il est maintenant possible de l'installer facilement via Homebrew.

Si vous n'avez pas encore installer ce package manager pour Mac OS X, il vous faudra tout d'abord installer

- XCode
- ou les [Commande Line Tools for XCode](https://developer.apple.com/downloads/)
- ou une [solution alternative](https://github.com/kennethreitz/osx-gcc-installer)

Une fois fait, l'installation de Homebrew se fait via cette commande :

``` sh
/tmp/temp_textmate.AZ9yKo:2: warning: variable $KCODE is no longer effective; ignored
/usr/bin/ruby -e "$(/usr/bin/curl -fsSL https://raw.github.com/mxcl/homebrew/master/Library/Contributions/install_homebrew.rb)"
```

Une fois homebrew installé, l'installation de CRaSH est très simple :

``` sh
brew update
brew install crash
```
And that's all :-)

<a href="http://www.gcuisinier.net/wp-content/uploads/2012/05/Capture-d’écran-2012-05-03-à-21.15.14.png" class="fancybox"><img class="aligncenter " title="CRaSH" src="http://www.gcuisinier.net/wp-content/uploads/2012/05/Capture-d’écran-2012-05-03-à-21.15.14.png" alt="" width="665" height="446" /></a>
