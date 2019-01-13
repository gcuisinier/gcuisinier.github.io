---
layout: post
title: Projet de modèle de configuration Spring
description : Présentation d'un projet qui permet d'avoir des modèles de configuration XML pour Spring Framework
categories:
- Spring
- OpenSource
type: post
date: "2010-04-08T08:00:00+01:00"

---
##Problème
Sur certains projets, j'ai eu l'occasion de voir des fichiers de configuration Spring de ce type :
```xml
<import resource="monitoring-environnement1.xml"/>
<import resource="monitoring-environnement2.xml"/>
<import resource="monitoring-environnement3.xml"/>
```

Et bien évidemment, chacun des fichiers de configurations importés était tous semblables, avec comme seule différence les noms de beans Spring ou des valeurs de propriétés.
Si demain, un nouvel environnement devait être ajouté, je vous le donne dans le mille : un copier / coller, un s/environnement1/nouvel-environnement/g  !

Même si cela fonctionne bien, ce n'est pas la solution la plus propre : Si le système de monitoring devait être modifié, il faudrait éditer X fichiers, avec le risque d'oublier un fichier, ou un valeur...
<!--more-->
##Solution 1 : Créer un namepace dédié au monitoring
Une première solution possible serait de remplacer ces imports par un namespace dédié au monitoring. Il suffirait dès lors d'utiliser une configuration de ce type :
```xml
<monitoring:environnement name="environnement1"/>
<monitoring:environnement name="environnement2"/>
```
C'est déjà beaucoup plus propre, mais cette solution n'est pas des plus pratiques :

* La configuration "générique" sera réalisée via une API spécifique à Spring, que peu de développeurs connaissent ( BeanDefinitionParser, ParserContext, BeanDefinitionRegistry, ..), ce qui rends toute modification assez complexe
* Le namespace sera dédié au monitoring ! Si le domaine des fichiers était tout autre, il faudrait développer un nouveau namespace.
* Le namespace va cacher aux utilisateurs les beans réellement instanciés

Bref, c'est déjà mieux mais pas encore suffisamment claire et simple.
##Solution 2 : Créer un namepace d'importation de modèles de configuration
Afin de répondre à ces problématique, il est possible de créer un namespace beaucoup plus générique. Celui qui permettrait d'importer une configuration classique (un modèle), mais en remplaçant certaines variables par des valeurs.

Le modèle serait un fichier de configuration Spring tout à fait compréhensible par des habitués de Spring, mais dans laquelle des variables seraient définies :
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"> 

    <bean name="scheduledTimeTask.${environnement}" class="org.springframework.scheduling.timer.ScheduledTimerTask">
        <property name="timerTask" ref="monitoringTask.${environnement}"/> 

        <property name="delay" value="1000"/>
        <property name="period" value="1000"/>
    </bean>
    <bean name="timer.${environnement}" class="org.springframework.scheduling.timer.TimerFactoryBean">
        <property name="scheduledTimerTasks">
            <list>
                <ref bean="scheduledTimeTask.${environnement}"/>
            </list>
        </property>
    </bean>
    <bean name="monitoringTask.${environnement}" class="be.hikage.springtemplate.MonitoringTimerTask">
           <property name="url" value="${${environnement}.url}"/>
    </bean> 

</beans>
```

Ici, <span style="font-family: Consolas, Monaco, 'Courier New', Courier, monospace; line-height: 18px; font-size: 12px; white-space: pre;">${environnement}</span>, est une variable qui sera définie lors de l'importation du modèle :
```xml
	    <hikage:import-template location="template-monitoring.xml">
        <hikage:variable name="environnement" value="environnement1"/>
    </hikage:import-template>
```
##Conclusion
Cette solution possède donc plusieurs avantages :
<ul>
	<li>En cas de modification du modèle, un seul fichier devra être modifié.</li>
	<li>Le namespace pourra être utilisé pour différent domaine ( monotoring, etc .. )</li>
	<li>Le modèle sera modifiable par n'importe quel développeur connaissant Spring, et tout à fait lisible</li>
</ul>

##Informations
Ce projet est disponible sous licence Apache 2 sur [http://code.google.com/p/spring-import-template/](http://code.google.com/p/spring-import-template)

