# Java EE - Java Enterprise Edition

## What is Java EE ?
Java EE is a collection of Abstract specifications, solving some of the common problems like Persistence, web services, transactions, security, loose coupling etc.

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
- The disposer method is called automatically when the context ends.

## CDI Interceptors
A interceptor is a class used to interpose in **method invocations** or **lifecycle events** (like postConstruct, preDestory) that occur in an association **target** class.
Use cases such as logging or auditing, **cross-cutting** tasks that are separate from business logic and are repeated often.
- Interceptors allow to specify the code for cross-cutting tasks in one place for easy maintenance.
- An interceptor can be defined within a target class as an **interceptor method**, or in associated class called **interceptor class**
- An interceptor class can contain more than one method, but it must not have more than one method for each type
- An interceptor class must have a public no-arg constructor
- The target class can have any number of interceptor method annotated with it. The order in which the interceptor methods are invoked is determined by the order in which the interceptor classed are defined in the @Interceptors annotation. e.g @Interceptors({PrimaryInterceptor.class, SecondaryInterceptor.class})

```java
@Stateless
@Interceptors({PrimaryInterceptor.class, SecondaryInterceptor.class})
public class TimerBean {
...
    @Schedule(minute="*/1", hour="*")
    public void automaticTimerMethod() { ... }

    @AroundTimeout
    public void timeoutInterceptorMethod(InvocationContext ctx) { ... }
...
}
```

Interceptor Metadata annotations

| Annotation | Description |
| ----- | ------ |
| javax.interceptor.AroundInvoke | Designates the method as an interceptor method |
| javax.interceptor.AroundTimeout | method as a timeout interceptor, for interposing on timeout methods of enterprise bean timers |
| javax.annotation.PostConstruct | interceptor method for post construct lifecycle events |
| javax.annotation.PreDestory | interceptor method for pre destory lifecycle events |

- Along with interceptor, an application defines one or more **interceptor biniding** (annotations that associate interceptor with target bean or methods.
```java
@Inherited
@InterceptorBinding
@Retention(RUNTIME)
@Target({METHOD, TYPE})
public @interface Logged {
}

@Logged
@SessionScoped
public class PaymentHandler implements Serializable {...}
```

## CDI Events
One of the best feature of CDI is event mechanism.
With event mechansim we can fire events, configure single/multiple event observers(listeners) which can perform some after task for the event. before CDI 2.0 events were synchronus, but with CDI 2.0 event can be asynchronus too.

This allows the application components to be more loosely coupled.

- Event interface is in package javax.enterprise.event.

### Event Types
1. SimpleEvent - When a simple evnt is fired, all the subscribers/listeners will recieve a notification for the event. In short, all the listeners will be fired.
```java
@Named
public class EventSource {

    @Inject
    private Event<String> simpleMessageEvent;  // the CDI compiler takes care of injecting an event implementation.

    public void fireEvent(){
       simpleMessageEvent.fire("Hello");  // this statement publishes a new event with a String payload
    }

}

```
2. QualifyingEvent - Event qualifying event, Event observers can decide, ehich event they want to listen to. This is achieved by CDI Qualifier.
```java

// defining a qualifier
@Qualifier
@Retention(RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface SpecialEvent {

}


// firing an event with Qualifier
@Named
public class SelectedEventSource {

    @Inject
    @SpecialEvent
    private Event<String> specialEvent;

    public void fireEvent(){
      specialEvent.fire("Hello");
    }

}


// listener which only wants to listen to special event
@Named
public class SpecialEventObserver {

    public void observeImportantMessage(@Observes @SpecialEvent String message){
      System.out.println(message);
    }

}
```


### Transactions & Events
Normally events are executed in the same transactions in which the event was fired. But if any observer wants to receive the event only if the transaction ended up in certain state, it can also be achieved in CDI.

the @Observes annotation has a field named during. This field accepts **javax.enterprise.event.TransactionPhase** enum.

```java
@Named
public class TransactionEventObserver {

    // this observer will be notified only if the transaction ends successfully.
    public void observeImportantMessage(@Observes(during = TransactionPhase.AFTER_SUCCESS) String message){
      System.out.println(message);
    }
}
```

The TransactionPhase enums - IN_PROGRESS, BEFORE_COMPLETION, AFTER_COMPLETION, AFTER_FAILURE, AFTER_SUCCESS

Default is IN_PROGRESS


### Event Reception
In a situation when an event is fired, but the observer resides in a bean with no living instance, there are two scenarios possible.
1. CDI created the instances and delivers the event to the observer (Reception.ALWAYS)
2. The event is not delivered (Reception.IF_EXISTS)

By default Reception.ALWAYS is active.

This can be configured for each observer by means of a **notifyObserver** attribute of @Observes annotation.
```java
Named
public class EventObserver {
    public void observeEvent(@Observes(notifyObserver = Reception.IF_EXISTS) String message){
      System.out.println(message);
    }
}
```


### Async Events
CDI 2.0 allows to fire asynchronus events. Asynchronus observer methods can then handle these asynchronus events in different threads.

To fire an asynchonus event - fireAsync method is to be used
```java
public class ExampleEventSource {
    
    @Inject
    Event<ExampleEvent> exampleEvent;
    
    public void someMethod() {
        exampleEvent.fireAsync(new ExampleEvent("Hello"));
    }   
}
```

to create an asynchronus observer - @ObservesAsync annotation is to be used
```java
public class AsynchronousExampleEventObserver {

    public void onEvent(@ObservesAsync ExampleEvent event) {
        // ... implementation
    }
}
```

### Observer Method Ordering
CDI 2.0 allows ordering or prioritizing observers. We can define order in which the observer method will be called by defining **@Priority** annotation after @Observes.

```java
public String onEvent(@Observes @Priority(1) ExampleEvent event, TextService textService) {
    // ... implementation
}

public String onEvent(@Observes @Priority(2) ExampleEvent event) {
    // ... implementation
}
```

Priority levels follow a natural ordering, therefore CDI will call first the observer method with priority level 1 and then priority level 2


## Java Persistent API (JPA)
The Java Persistence API provides Java developers with an object/relational mapping facility for managing relational data in Java applications. 
It is essentially a standard frmaework for object relattional mapping.
Java Persistence consists of four areas:
- The Java Persistence API
- The query language
- The Java Persistence Criteria API
- Object/relational mapping metadata

### JPA Entity
the most atomic unit of JPA framework.
A POJO can be annotated by @Entity annotation to make it a JPA entity. Every signle instance of an entity object maps to a row in the database.

A JPA entity must have a unique identifier.

- By default a JPA entity is mapped to a table with the class name. to cutomize the table mapping, @Table annotation can be used. e.g. @Table(name = "TABLE_NAME")

- JPA Entity Using super class
```
@Entity
@AttributeOverride(name="id", column = @Column(name="tax_id"))  // if the subclass wants to override any field of superclass
public class Tax extends AbstractEntity {
  
  private BigDecimal taxRate;
  
  @Basic
  private String name;
  
}


// super class
@MappedSuperClass 
public abstract class AbstractEntity implements Serializable {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  protected Long id;
  
  protected String userEmail;
  
}
```

**@Basic** annotation on a field or property signifies that it is a basic type and JPA should use the standard mapping for it's persistance. It's an optional annotation.

### JPA @Column annotation
The @Column annotation is used to customize column mapping in database. The various attributes of @Column attributes are - 
1. @Column (name = "name_of_column") - customize the column name in the database
2. @Column (length=”varchar(255)/Integer 23) - adds column size of the particular column in the table.
3. @Column(nullable=false) - adds the not-null constraint to the column of the table.

### JPA Transient fields
**@Transient** annotation on a field or property can be used to avoid mapping of the field to the database. That particular field will not be persisted in the database.

### JPA Entity Field access types
JPA defines two types of access.
1. Field access - when the field value is directly used for persistance. Runtime access the field directly through reflection. In this case the field is annotated directly with JPA annotations @Column or @Id
2. Getter access - with this the persistance container will use the value returned from the getter method for persistance. In this case the getter method is annotated with JPA annotations.


### JPA MApping Enum Types
By default enum are mapped to a column in the table and the ordinal value of the enum is stored in the database.
```java
public enum EmployementType {
  FULL_TTIME, // 0
  CONTRACT // 1
}
```
With ordinal value being stored in the database, the problem is that we cannnot change the order of the enum constants. If the order is changed or a new constant is introduced in between, the the oridinal mapping in the database will become wrong.

To avoid this we can use **@Enumerated(EnumType.String)** - by using this the actual constant will be stored in the database.

### JPA Mapping Large Objects
**@Lob** annotation is used to store large objets such as images.
```
@Lob
@Basic(fetch = FetchType.LAZY). // do not fetch the field immediately, but will be fetched when the getter for that field is called.
private byte[] image;
```

### JPA - Mapping Embeddable classes
```java
@Embeddable
public class Address {
  private String streetAddress;
  private String city;
  privtae String country;
}

@Entity
public class Employee {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Long id;
  
  @Embedded
  private Address address;
}
```
@Embeddable annotation is used to mark a class that do not has an idetity on its own, but can be embedded into other classes.
@Embedded annotion is used mark the field to refer the embeddable class.


### JPA - Mapping Primary Keys
1. Auto primary key generation - @GeneratedValue(strategy = GenerationType.AUTO) - persistent provider dependent
2. Identity primary key generation - @GeneratedValue(strategy = GenerationType.IDENTITY) - underlying databse identity strategy
3. sequence primary key generation - @SequenceGenerator(name="some_seq", sequenceName="some_seq_name") @GeneratedValue(generator="some_seq")
4. Table primary key generation - @TableGenerator(name="Emp_Gen", table="ID_GEN". pkColumnName="GEN_NAME", valueColumnName="pk_value") @GeneratedValue(generator="Emp_Gen")


### JPA Entity Relationaship Mapping
**Directionality** : A direction in a relationship can be either bidirectional or unidirectional. A bidirectional relationship has both an owning side and an inverse side. A unidirectional relationship has only an owning side.

The owning side of a relationship determines how the persistence runtime makes updates to the relationship in database.

**Cardinality** : Cardinality refers to the maximum number of times an instance in one entity can relate to instances of another entity.

**Ordinality** : Ordinality is the minimum number of times an instance in one entity can be associated with an instance in the related entity.

@ManyToOne - to map many to one relationship.
so in database the one table will have a reference (foreignKey) to primary key of the other table. The ownership of the relationship is determined by which table has the foreignKey column. In JPA this foreignKey column is called the joinColumn.

```java
@Entity
public class Employee {
  @ManyToOne
  @JoinColumn(name="DEP_ID") // customize the foreign key column
  private Department department;
}
```

@OneToOne - to map one to one relationship

e.g of bidirectional one to one mapping
```java
@Entity
public class Employee {
  @OneToOne(mappedBy = "employee") // tells jpa that Payslip is the owning side of the relationship
  private Payslip payslip;
}

@Entity
public class Payslip {
  @OneToOne
  @JoinColumn(name="EMP_ID")
  private Employee employee;
}
```

@ManyToMany - to map many to many relationship
```java
@Entity
public class Project {
  @ManyToMany // jpa creates a join table to manage the relatioship, so ownership is on the join table
  @JoinTable(name="PROJ_EMPLOYEES", joinColumns = @JoinColumn(name="PROJ_ID"), inverseJoinColumns = @JoinColumn(name="EMP_ID") // customize the join table
  private Collection<Employee> employees;
}


@Entity
public class Employee {
  @ManyToMany
  private Collection<Project> projects;
}
```
