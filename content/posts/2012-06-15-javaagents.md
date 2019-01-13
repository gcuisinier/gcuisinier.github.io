---

tags : ["french","java"]
date : "2012-06-15T08:00:00+01:00"
draft : false

title : "Les agents Java"
lang : "fr"
shortTitle : "javaagents"
---


Les Agents Java ? Pas la moindre idée de ce que c'est ?
Mais si, vous en avez certainement déja vu, ils se cachent dans la ligne de commande Java via ce paramètre <code>-javaagent:vers/mon/agent.jar</code>. 

Ceux-ci interviennent lors du chargement des classes par un classloader, et ont la possiblité de venir modifier la classe en cours de chargement. Ce mécanisme est utilisé par divers outils et frameworks :

- AspectJ pour faire du tissage d'Aspect au chargement
- Par des outils de Profiling pour venir ajouter du code permettant de tracer les appels
- ... et plein d'autres

<!--more-->

## Getting started !

Pour mieux comprendre comment fonctionne un agent, le plus simple est de tenter d'en faire un simple.

### Créer la classe de l'agent

```java
package be.hikage.agent;

import java.lang.instrument.Instrumentation;

public class AgentSimple {

	public static void premain(String agentArgument, Instrumentation instrumentation){		
		System.out.println("Hello, Agent Smith ! [ " + agentArgument + "]");
	}
}
```

La méthode <code>premain(String agentArgument, Instrumentation instrumentation)</code> est pour un agent ce qu'est la méthode <code>main(String ... arguments)</code> est pour une application Java.

Cette méthode appelée lors du chargement de l'agent, et possède deux arguments :

- <code>String agentArguments</code> : Ce sont les options passée en paramètre dans la ligne de commande pour l'agent. <code>-javaagent:jarpath=parametre</code>
- <code>Instrumentation instrumentation</code>, est un service qui va fournir des méthodes pour modifier les classes.

### Préparer le packaging

Mais avoir la classe ne suffit pas pour avoir un agent utilisable. En effet, un agent doit obligatoirement un jar, avec un manifest correctement configuré.

Celui-ci doit contenir une entrée <code>Premain-Class</code>

```
Manifest-Version: 1.0
Premain-Class: be.hikage.agent.AgentSimple
```

### Tester l'Agent

Une fois le <strong>Jar</strong> prêt, il faut le tester. Pour cela créer un petit programme simple :

```java
public class TestMain {

	public static void main(String[] args) {
		System.out.println("Mon Programme");
	}
}
```

Et lançons celui-ci en fournissant l'agent :

```sh
$ java -javaagent:AgentSimple.jar="Mes paramètres" be.hikage.agent.TestMain
Hello, Agent Smith ! [ Mes paramètres]
Mon Programme
```

On remarque que l'agent est bien lancé avant l'application, et qu'il a bien reçu les paramètres que l'on lui a fourni.


## Interception des chargements de classes

Voyons maintenant à quoi sert le deuxième paramètre fourni à la méthode *Premain* : Instrumentation.
La principale fonctionnalité de celui-ci est de permettre d'enregistrer un *ClassFileTransformer*. Cette interface oblige à implémenter une unique méthode :

```java
byte[]
    transform(  ClassLoader         loader,
                String              className,
                Class            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte[]              classfileBuffer)
        throws IllegalClassFormatException;
```

Le premier paramètre étant le classloader impliqué pour le chargement de la classe. Les deux suivant, le nom de la classe et l'instance de Class représentant la classe en train d'être chargée.
Le dernier est le bytecode brut de la classe, sous la forme d'un tableau de bytes.


Modifions maintenant notre agent pour afficher les classes en cours de chargement :

```java
package be.hikage.agent;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.lang.instrument.Instrumentation;
import java.security.ProtectionDomain;

public class AgentSimple {

	public static void premain(String agentArgument, Instrumentation instrumentation){		
		System.out.println("Hello, Agent Smith ! [ " + agentArgument + "]");

        instrumentation.addTransformer( new ClassFileTransformer() {
            @Override
            public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
                System.out.println("Tu es en train de charger la classe :" + className);
                return classfileBuffer;
            }
        });
	}
}
```

Le résultat sera le suivant :
```
Hello, Agent Smith ! [ Mes paramètres]
Tu es en train de charger la classe : sun/launcher/LauncherHelper
Tu es en train de charger la classe : java/lang/Enum
Tu es en train de charger la classe : be/hikage/agent/TestMain
Tu es en train de charger la classe : java/lang/Void
Mon Programme
Tu es en train de charger la classe : java/lang/Shutdown
Tu es en train de charger la classe : java/lang/Shutdown$Lock
```

## Modification de classe

Passons maintenant la seconde, et tentons de venir modifier les classes en cours de chargement afin d'avoir des statistiques sur le nombre d'instance.
Pour cela, il suffit d'ajouter un champ statique qui servira de compteur, et de modifier les constructeurs pour y ajouter une incrémentation de celui-ci ainsi que de l'afficher dans les logs.

Pour cela, il faut utiliser un outil de manipulation de bytecode tels que [Javassist](http://www.jboss.org/javassist).
Voici une implémentation d'un ClassFileTransformer possible :

```java
package be.hikage.agent;

import javassist.ClassPool;
import javassist.CtBehavior;
import javassist.CtClass;
import javassist.CtField;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

public class CountInstanceTransformer implements ClassFileTransformer {


    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        return visitClass(className, classBeingRedefined, classfileBuffer);

    }

    private byte[] visitClass(String className, Class<?> classBeingRedefined, byte[] classfileBuffer) {
        ClassPool pool = ClassPool.getDefault();
        CtClass cl = null;
        try {
            cl = pool.makeClass(new java.io.ByteArrayInputStream(classfileBuffer));
            if (cl.isInterface() == false) {

                // Ajout d'un champ static dans la classe
                CtField field = CtField.make("private static long _instanceCount;", cl);

                cl.addField(field);

                CtBehavior[] constructors = cl.getDeclaredConstructors();
                for (int i = 0; i < constructors.length; i++) {
                    // On incrémente le compteur et on l'affiche
                    constructors[i].insertAfter("_instanceCount++;");
                    constructors[i].insertAfter("System.out.println(\"" + className + " : \" + _instanceCount);");


                }

                // Génération du bytecode modifié
                classfileBuffer = cl.toBytecode();
            }
        } catch (Exception e) {
            e.printStackTrace();

        } finally {
            if (cl != null) {
                cl.detach();
            }
        }
        return classfileBuffer;
    }
}
```

Et un petit programme de tests :

```java
public class TestMain {

	public static void main(String[] args) {
		System.out.println("Mon Programme");
        List test = new ArrayList();
        test.add(new MonObject()) ;
        test.add(new MonObject()) ;
        test.add(new MonObject()) ;
        test.add(new MonObject()) ;
	}
}
```

Le lancement de l'agent se fait de la même manière qu'auparavant. Cependant, celui-ci apporte une dépendance supplémentaire, Javassist, au runtime.
Il faut donc que celui-ci soit disponible dans le classpath :

```
$ java -classpath "path/to/javassist-3.14.0-GA.jar:./" -javaagent:AgentSimple.jar="Mes paramètres" be.hikage.agent.TestMain
Hello, Agent Smith ! [ Mes paramètres]
java/lang/Enum : 1
sun/launcher/LauncherHelper : 1
Mon Programme
be/hikage/agent/MonObject : 1
be/hikage/agent/MonObject : 2
be/hikage/agent/MonObject : 3
be/hikage/agent/MonObject : 4
java/lang/Shutdown$Lock : 1
java/lang/Shutdown$Lock : 2
java/lang/Shutdown$Lock : 3
java/lang/Shutdown$Lock : 4
```

# Conclusion

Les Agents Java sont assez simple à mettre en place, et les possibilités sont assez nombreuses.
Cependant, cela nécessite d'avoir déclarer celui-ci au démarrage de l'application.

Et parfois, on aimerait avoir la possibilité d'activer un agent plus tard, au besoin.
Pour cela, il existe une API, l'Attach API. 
Celle-ci fera l'objet d'un prochain article.


