# Java EE - Java Enterprise Edition

## What is Java EE ?
Java EE is a collection of Abstract specifications, soli=ving some of the common problems like Persistence, web services, transactions, security, loose coupling etc.

```java
javax.* //package
```

## Java EE Application Servers
Java EE concrete implementations.
e.g. 
1. Payara Server (Glassfish) : https://payara.fish
2. IBM Open Liberty : https://openliberty.io
3. JBOSS Wildfly : http://www.wildfly.org

## JSR
JSR is Formal Request to Java Comunity Platform for new proposals, enhancement to existing APIs, grouping APIs into silos etc.

## Jakarta EE
Jakrta EE is the future of Java EE hosted by Eclipse foundation.
Java will be transafering all Java EE code base to Eclipse foundation.

## Java EE / Spring Framework
- Java EE is influenced by Spring Framework
- Spring Boot is influenced by Java EE

-------------------------------------------

# Java EE - CDI
Context and Dependency Injection

- Denpendency Injection is a software paradigm where indivual components have there dependencies supplied to them, instead of creating them themshelves.
- Dependency Injection is a specific form of inversion of control.

## CDI Features
1. TypeSafe Dependency
2. Lifecycle contexts
3. Interceptors
4. Events
5. Service Provider Interface

## Bean discovery modes
Bean discovery - The mechanism by which the dependency injection runtime analyzes and discovers beans to manage.

1. annotated - the beans that are annotated with certain CDI annotations will be eligible for management.
2. all - every single bean that is created in the application are eligible for bean discovery and management.


## CDI Container
A kind of black box that takes control of the dependency management in the JAVA-EE application.
It allows to manage lifecycle of stateful components via domain-specific lifecycle contexts and inject components in a type-safe way.T
The container is responsible for managing the lifecycle of beans.
CDI container also manages the context in the beans are created and injected.

## Bean & Contextual Instances
A bean is a source of contextual objects that define application state and/or logic.
- A Java EE component is a bean if the lifecycle of its instances may be managed by the container according to lifecycle context model defined in the CDI specification.

A Contextual instance is an instance of a bean.
- A bean can be considered a template of contextual instances.


## CDI Injection Points
The point at which the CDI container can inject the dependency.
1. **Field Injection** : Injection takes place at a field.

```java
@Web
public class ScopesBean implements serializable {
  @Inject
  private RequestScope requestScope;
}
```

2. **Constructor Injection** : dependency injection takes place at constructor of the bean
```java
@Web
public class ScopesBean implements serializable {
  
  private RequestScope requestScope;
  
  @Inject
  private ScopesBean(RequestScope requestScope) {
    this.requestScope = requestScope;
  }
}
```

3. **Method Injection** : dependency injection at method call

```java
@Web
public class ScopesBean implements serializable {
  
  private RequestScope requestScope;
  
  @Inject
  private void setSessionScope (RequestScope requestScope) {
    this.requestScope = requestScope;
  }
}
```

## CDI LifeCycle callbacks
1. **@PostConstruct** : It tells the container to invoke this method after the construction of the instance.
2. **@PreDestroy** : It tells the container to invoke this method before the bean is made available for garbage collection.

## Qualifiers
CDI Qualifier is a way to remove ambiguity and tell the container the specific type of dependency to be injected.

When CDI inspects an injection point to find a suitable bean to inject, it takes not only the class type into account, but also qualifiers.
- A qualifier is an anotation that can be applied to a bean.
- default qualifier is **@Any**

```java
@Qualifier
@Retention(RUNTIME)
@Target({TYPE, METHOD, FIELD, PARAMETER})
public @interface Informal {}

package greetings;

@Informal
public class InformalGreeting extends Greeting {
    public String greet(String name) {
        return "Hi, " + name + "!";
    }
}
```
## Context and Scopes
1. **@Dependent** : default scope. make the bean dependent on the context in which the bean is injected.
2. **@RequestScoped** : bound to HTTP request scope. every http request, a new contextual instance would be created.
3. **@SessionScoped** : bound to an http session.
4. **@ApplicationScoped** : bind contextual instanaces to the lifcycle context of the application.
5. **@ConversationScoped** : manually managed by the developer


## Producer Method
A producer method generates an object that can then be injected.
Typical usecases - 
- when you want to inject an object that is not itself a bean.
- When the concrete type of the object to be injected nay vary at runtime.
- When object requires some custom initialization that the bean constructor does not perform.

```java
@Produces
@Chosen
@RequestScoped
public Coder getCoder(@New TestCoderImpl tci,
        @New CoderImpl ci) {

    switch (coderType) {
        case TEST:
            return tci;
        case SHIFT:
            return ci;
        default:
            return null;
    }
}


// use
@Inject
@Chosen
@RequestScoped
Coder coder;
```

## Producer Field
A producer field is simpler alternative to producer method. It is field of a bean that generates an object.
- It can be used `instead of simple getter method.
- Producer fields are uselful for declaring Java EE resources such as data sources, JMS resources, and web service references.

## Disposer method
A disposer methods task is to remove an object if its task is complete.
```java
@PersistenceContext
private EntityManager em;

@Produces
@UserDatabase
public EntityManager create() {
    return em;
}

public void close(@Disposes @UserDatabase EntityManager em) {
    em.close();
}
```
