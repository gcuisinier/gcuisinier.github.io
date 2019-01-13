---
layout: post
title: Créer un namespace Spring
date: "2010-04-24T08:00:00+01:00"

categories:
- Xml
- Spring
---
Une des fonctionnalités fort pratique avec Spring est la notion de namespace. Ceux-ci permettre de simplifier et réduire de manière significative une configuration XML.

Spring en possède plusieurs de bases : jms, jee, scheduling, jdbc, mvc, ...

Cependant, il est tout à fait possible d'en créer des spécifiques à nos propres besoins, et sans trop de difficulté.
<!--more-->
## Créer le schéma xml
Lorsqu'on créer un namespace pour Spring, la toute première chose à faire est de définir les éléments de celui-ci.
Et comme on parle de namespace XML, cela se traduit par un schema XSD.

Le plus simple pour créer une XSD est de préparer un exemple de XML valide, et d'ensuite créer le XSD à partir de celui-ci.
Dans le cadre du projet de modèle, je voulais pouvoir importer un fichier et définir des variables :

```xml
<import-template location="classpath:be/hikage/template/template.xml">
       <variable name="variable1" value="valeur1" />
</import-template>
```

A partir de différents outils (Intellij Idea, XmlSpy, ..), il est possible de générer une XSD correspondant à notre exemple :

```xml
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified">
  <xs:element name="import-template">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="variable">
          <xs:complexType>
            <xs:attribute name="name" type="xs:string" use="required"/>
            <xs:attribute name="value" type="xs:string" use="required"/>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
      <xs:attribute name="location" type="xs:string" use="required"/>
    </xs:complexType>
  </xs:element>
</xs:schema>
```

Cependant, cette XSD ne comporte pas encore de notion de namespace dédié. Pour cela, il faut lui ajouter l'attribut targetNamespace avec comme valeur l'identifiant de notre namespace.
Dans mon cas, il s'agira de <strong>http://www.hikage.be/schema/import-template</strong>.

La XSD complètée sera donc :

```xml
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" targetNamespace="http://www.hikage.be/schema/import-template">
  <xs:element name="import-template">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="variable">
          <xs:complexType>
            <xs:attribute name="name" type="xs:string" use="required"/>
            <xs:attribute name="value" type="xs:string" use="required"/>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
      <xs:attribute name="location" type="xs:string" use="required"/>
    </xs:complexType>
  </xs:element>
</xs:schema>
```
On enregistre cette classe dans les sources du projets, par exemple dans le package be.hikage.namespaces.schemas.

## Créer un NamespaceHandler
Une fois le schéma défini, il faut encore dire à Spring comment l'utiliser.
Pour cela, il faut implémenter un NamespaceHandler, le plus souvent en étendant la classe NamespaceHandlerSupport :
```java
public class TemplateNamespaceHandler extends NamespaceHandlerSupport {
   private static final String IMPORT_TEMPLATE_TAG = &quot;import-template&quot;;

   public void init() {
       // Spécifie que les éléments &lt;import-template&gt; seront traités
       // par la classe ImportTemplateBeanDefinitionParser
       registerBeanDefinitionParser(IMPORT_TEMPLATE_TAG, 
          new ImportTemplateBeanDefinitionParser());

   }
}                                                                                                                                                                                              }                                   
```

Etendre NamespaceHandlerSupport nécessite d'implémenter une méthode init dans laquelle il est nécessaire de lier les éléments racines (import-template dans notre cas) à un BeanDefinitionParser.

## Créer le BeanDefinitionParser
Les BeanDefinitionParser sont certainements les classes les plus importantes dans la création d'un namespace car ce sont elles qui vont véritablement dire à Spring quel Beans devront être ajoutés dans le contexte.

Dans notre cas, la classe ImportTemplateBeanDefinitionParser va être responable de

* Parser l'élément import-template et ses éléments fils (variable)
* Lire le fichier spécifié dans l'attribut location
* Injecter dans le contexte Spring les définition définies dans le fichier en remplacant les variables présente dans celui-ci

En pratique, cela se fait dans la méthode parse, qui est définie dans l'interface BeanDefinitionParser :

```java
	public BeanDefinition parse(Element element, ParserContext parserContext) {

		// On lit l'attribut location pour connaitre le fichier modèle
		String resource = element.getAttribute(LOCATION_ATTRIBUTE);

		// On récupère les &lt;variable name=&quot;cle&quot; value=&quot;valeur&quot; sous forme de
		// Map&lt;Cle, Valeur&gt;
		Map&lt;String, String&gt; variables = prepareReplacement(element);

		// On lit les BeanDefinition (représentation de la configuration d'un
		// Bean) à partir du fichier modèle
		Map&lt;String, BeanDefinition&gt; tempDefinitions = loadTemplateBeans(location);

		// On remplace les variables dans les définitions ( nom de beans,
		// attributs, valeurs )
		Map&lt;String, BeanDefinition&gt; beansDefinition = replaceVariable(
				tempDefinitions, variables);

		for (Map.Entry&lt;String, BeanDefinition&gt; entry : templateBeansDefinitions
				.entrySet()) {
			// On ajoute chaque beans dans le contexte Spring
			parserContext.getRegistry().registerBeanDefinition(entry.getKey(),
					entry.getValue());
		}

		return null;
	}

}
```

J'ai volontairement simplié le code concernant les méthodes loadTemplateBeans() et replaceVariable() par soucis de simplicité.

Dans le cas de la méthode loadTemplateBeans, j'utilise la classe XmlBeanDefinitionReader de Spring qui va lire et créer les BeanDefinitions à ma place. Un BeanDefinition est le modèle interne à Spring pour représenter un Bean : son nom, la classe, les variables constructeurs, les propriétés à configurer, ...


## Enregistrer le NamespaceHandler et le schéma
Nous avons maintenant notre schéma ainsi que les classes nécessaire pour que Spring puisse traiter correctement un fichier de ce type  :

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:hikage="http://www.hikage.be/schema/import-template"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.hikage.be/schema/import-template http://www.hikage.be/schema/import-template/import-template-1.0.xsd">

   <hikage:import-template location="classpath:template.xml">
       <hikage:variable name="env" value="dev"/>
   </hikage:import-template>

</beans>
```


Pour cela, il est nécessaire de créer deux fichiers dans META-INF :

* <strong>spring.handlers</strong> : qui va associé un namespace à son NamespaceHandler
* <strong>spring.schemas</strong> : qui va associé l'URL utilisé dans xsd:schemaLocation à la XSD dans le classpath

Le fichier spring.handlers :
```
http://www.hikage.be/schema/import-template=be.hikage.springtemplate.TemplateNamespaceHandler
```
<ul>
	<li>Clé : Le namespace défini dans le targetNamespace de la XSD</li>
	<li>Valeur : la FQN du NamespaceHandler</li>
</ul>

Le fichier spring.schemas :
```
http://www.hikage.be/schema/import-template/import-template.xsd=be/hikage/springtemplate/import-template-1.0.xsd
```

* Clé : L'URI définie dans le schemaLocation</li>
* Valeur : Le chemin de la XSD dans le classpath</li>


## Compléments
Cela dit, cet exemple est assez basique. Les possibilités d'un namespace sont plus étendue. Il est également possible d'améliorer le schema XSD et le code du BeanDefinitionParser pour mieux s'intégrer dans les IDEs. 

En attendant, pour plus d'informations :
<ul>

</ul>
	<li>Le code complet de l'<a href="http://code.google.com/p/spring-import-template/">Import-Template</a> vous permettra de voir en détail comment créer un namespace</li>
	<li>La <a href="http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/extensible-xml.html">documentation officielle </a>sur la création d'un namespace</li>




