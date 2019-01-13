---
layout: post
title: "Spring MongoDB en 5 minutes"
description : Introduction à Spring MongoDB, un projet du portfolio Spring simplifiant les accès à MongoDB
date: "2012-03-17T08:00:00+01:00"

comments: true
categories: 
- Java
- Spring
---

##Créer sa base MongoDB

Bien qu'il soit assez simple d'installer MongoDB sur un poste de développement, avec l'arrivée du cloud et des SaaS, il est très simple et très rapide d'avoir une instance de Mongo. 
<a href="https://mongohq.com" target="_blank">MongoHQ</a> offre des solutions d'hébergement de Mongo à la demande, à plusieurs tarifs.
L'offre <strong>starter</strong> est gratuite, mais apporte une limite de 16 Mo. 

En plus d'héberger l'instance, <a href="https://mongohq.com" target="_blank">MongoHQ</a> offre également une interface graphique très pratique pour parcourir votre base, vos collections et leurs données, ou encore gérer les indexes.


##Utiliser Spring Data MongoDB

###Ajout des dépendances
La première chose à faire est d'ajouter la dépendance vers le projet Spring Mongo.
Avec un projet Maven, cela se fera très simplement comme ceci :

``` xml
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-mongodb</artifactId>
	<version>1.0.1.RELEASE</version>
</dependency>
```

###Configuration de Spring Data MongoDB
Il faut ensuite configurer votre application Spring afin d'y ajouter les beans nécessaires pour utiliser Spring Data MongoDB. Ce dernier fournit une classe de configuration qu'il suffit d'étendre pour configurer le stricte nécessaire :

``` java
@Configuration
public class MongoConfig extends AbstractMongoConfiguration {

    @Override
    public Mongo mongo() throws Exception {
        return new Mongo("staff.mongohq.com", 10073);
    }

    @Override
    public String getDatabaseName() {
        return "spring-data";
    }

    @Override
    public UserCredentials getUserCredentials() {
        return new UserCredentials("spring-data", "spring-data-passwd");
    }
}
```

Les deux premières méthodes sont obligatoires, la première représente la connexion vers le serveur MongoDB.
Si vous utilisez mongohq, vous trouverez les informations nécessaires dans l'onglet "Database infos"

<img src="http://www.gcuisinier.net/wp-content/uploads/2012/03/Capture-d’écran-2012-03-16-à-23.36.36-300x114.png" alt="Information de connexion à Mongohq" title="Information de connexion à Mongohq" width="300" height="114" class="aligncenter size-medium wp-image-419" />

La deuxième est le nom de la base à utiliser, celui que vous avez fournit à la création dans mongohq

La troisième quand à elle ne soit être surchargée que lorsqu'on utilise des bases sécurisées et donc pour lesquelles il faut s'authentifier.
Une fois de plus, mongohq offre la possibilité de gérer les utilisateurs et leur mot de passe via l'interface web.

###Enregistrer et lire des documents

Une fois la configuration préparée, il est possible d'injecter un bean de type <code>MongoTemplate</code>.
Celui ci propose une série de méthodes pour accéder à MongoDB.

Par exemple, prenons une classe Contact.

``` java
public class Contact {

    private String firstname;

    private String lastname;

    private String email;

    private String phoneNumber;

    // Getter, Setter & Constructors

}
```

Afin d'enregistrer une instance de cette classe, il suffit d'utiliser la méthode save(Object) du MongoTemplate :

``` java
MongoTemplate template = context.getBean(MongoTemplate.class);

Contact contact = new Contact("Gildas", "Cuisinier", "gildas.cuisinier[at]domain.com", "+42 424242");
template.save(contact);
```


