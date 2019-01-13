---
layout: post
title: "Attach API"
date: "2010-07-22T08:00:00+01:00"

comments: true
categories: 
- Java
published: false
---
# L'Attach API

Dans un précédent article, les Agents Java ont été présentés. Ceux-ci permettaient de venir d'intercaler lors du chargement d'une classe pour venir modifier sa définition.

Cependant, un Agent Java doit être activé lors du démarrage de l'application avec un paramètre  <code>-javaagent:vers/mon/agent.jar</code>.
Dans certains cas, il serait intéressant de pouvoir venir greffer un agent sur une application déja lancée. Pour cela, une solution existe, c'est l'Attach API.

Voyons un peu comment elle fonctionne.

## VirtualMachine

La classe principale de cette API est la classe [VirtualMachine](http://docs.oracle.com/javase/6/docs/jdk/api/attach/spec/com/sun/tools/attach/VirtualMachine.html).

Celle-ci propose plusieurs méthodes statiques. La première permet de lister les JVMs en cours d'éxecution sur la machine :


import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;

```java
public class AttachMe {


    public static void main(String[] args) throws Exception {

        for(VirtualMachineDescriptor vmDesc : VirtualMachine.list()){
            System.out.println("Name : " + vmDesc.displayName());
            System.out.println("ID : " + vmDesc.id());
        }
    }
}
```

La deuxième permet de venir se connecter via l'identifiant du processus (le pid).
Essayons :

```java
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;
import sun.management.VMManagement;


public class AttachMe {


    public static void main(String[] args) throws Exception {


        String pid = getPidFromRuntimeMBean();

        VirtualMachine vm = VirtualMachine.attach(pid);
        Properties targetVMProperties = vm.getSystemProperties();
        vm.loadAgent("/Users/hikage/Downloads/JavaAgent-Introduction/target/introduction-1.0.jar");

    }



}
```

Résultat :

```
Failed to find Agent-Class manifest attribute from /Users/hikage/Downloads/JavaAgent-Introduction/target/introduction-1.0.jar
Exception in thread "main" com.sun.tools.attach.AgentLoadException: Agent JAR not found or no Agent-Class attribute
	at sun.tools.attach.HotSpotVirtualMachine.loadAgent(HotSpotVirtualMachine.java:117)
	at com.sun.tools.attach.VirtualMachine.loadAgent(VirtualMachine.java:526)
	at AttachMe.main(AttachMe.java:25)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:120)

```

Raté, cela ne marche pas...

## Agent 2.0

En réalité, un simple agent ne peut pas être utilisé via l'Attach API, il faut que celui-ci soit adapté.

La première chose à faire est d'ajouter une entrée <code>Agent-Class</code> dans le manifest :

```
Manifest-Version: 1.0
Premain-Class: be.hikage.agent.AgentSimple
Agent-Class : be.hikage.agent.AgentSimple
```
Ré-essayons :

```
Exception in thread "Attach Listener" java.lang.NoSuchMethodException: be.hikage.agent.HkAgent.agentmain(java.lang.String, java.lang.instrument.Instrumentation)
	at java.lang.Class.getDeclaredMethod(Class.java:1954)
	at sun.instrument.InstrumentationImpl.loadClassAndStartAgent(InstrumentationImpl.java:323)
	at sun.instrument.InstrumentationImpl.loadClassAndCallAgentmain(InstrumentationImpl.java:407)
Agent failed to start!
Exception in thread "main" com.sun.tools.attach.AgentInitializationException: Agent JAR loaded but agent failed to initialize
	at sun.tools.attach.HotSpotVirtualMachine.loadAgent(HotSpotVirtualMachine.java:121)
	at com.sun.tools.attach.VirtualMachine.loadAgent(VirtualMachine.java:526)
	at AttachMe.main(AttachMe.java:25)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:120)
```

Raté ! Cela ne marche toujours pas car la JVM cherche une méthode <code>agentmain</code> au lieu de <code>premain</code>, mais avec les mêmes  paramètres.

```java
package be.hikage.agent;

import java.lang.instrument.Instrumentation;
import java.lang.instrument.UnmodifiableClassException;

public class AgentSimple {

    public static void premain(String agentArgument, Instrumentation instrumentation) {
        System.out.println("Hello, Agent Smith ! [ " + agentArgument + "]");

        instrumentation.addTransformer(new CountInstanceTransformer());
    }


    public static void agentmain(String agentArgument, Instrumentation instrumentation) {
        System.out.println("Hello, Agent Smith 2.0 ! [ " + agentArgument + "]");

        instrumentation.addTransformer(new CountInstanceTransformer());
    }

}
```

Essayons à nouveau :

```java
public class MaClass {

    public MaClass(){
        System.out.println("Dans le contructeur de MaClass");
    }
}


package be.hikage.agent;

import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;
import sun.jvm.hotspot.memory.SystemDictionary;
import sun.management.VMManagement;

import java.lang.management.ManagementFactory;
import java.lang.management.RuntimeMXBean;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Properties;

public class AttachMe {


    public static void main(String[] args) throws Exception {


        String pid = getPidFromRuntimeMBean();


        VirtualMachine vm = VirtualMachine.attach(pid);

        Properties targetVMProperties = vm.getSystemProperties();

        // Chargement de la classe AVANT le chargement de l'agent 
        new MaClass();

        vm.loadAgent("/Users/hikage/devel/labs/attach-demo/attach-agent/target/attach-agent-1.0.jar");
        
        // Nouvelle instanciation
        new MaClass();
        
        // Chargement d'une autre classe
        new MaClasse2();



    }


}
```

Voyons ce que donne la sortie maintenant :

```
Dans le contructeur de MaClass
Hello, Agent Smith 2.0 ! [ null]
Dans le contructeur de MaClass
Dans le contructeur de MaClasse2
be/hikage/agent/MaClasse2 : 1
java/lang/Shutdown$Lock : 1
java/lang/Shutdown$Lock : 2
java/lang/Shutdown$Lock : 3
java/lang/Shutdown$Lock : 4
```

On remarque

