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


### JPA - Collection Mapping of Embeddable
When an embeddable property is mapped in an entity @Embedded annotation, the owning table adds columns for all the attribute of the embeddable object
```java
@Embeddable
public class Address {
  private String street;
  private String city;
}

@Entity
public class Employee {
  @Embedded // the columns for address will be created in the employee table.
  private Address address;
}
```

but when a collection of embeddable objects is to be mapped then **@ElementCollection** attribute can be used. This will create a secondary table for storing the embeddable object.
```java
@Embeddable
public class Address {
  private String street;
  private String city;
}

@Entity
public class Employee {
  @ElementCollection // secondary table will be created to map the adress objects
  @CollectionTable(name = "ADDRESS", joinColumns=@JoinColumn(name="EMP_ID")) // customize the secondary table
  private Collection<Address> address;
}
```

### JPA - Ordering the contents of a persitable collection.
**@OrderBy** annotation can be used - it adds order by clause to the generated SQL to order the members of the retrieved collection.
```java
@Entity
public class Department {
  @OneToMany(mappedBy="department")
  @OrderBy("fullName ASC, dateOfBirth desc") // order first by fullName and then by dateOfBirth 
  private List<Employee> employees = new ArrayList<>();
}
```

**@OrderColumn** annotation - it adds an additional column in the table, containing the index of the entity in the list.
```java
@Entity
public class Department {
  @OneToMany(mappedBy="department")
  @OrderColumn(name="EMPLOYEE_POS") //persist the position in the list to the table. this will be stored as an index in a new column called EMPLOYEE_POS
  private List<Employee> employees = new ArrayList<>();
}
```

### JPA - persisting key value pairs
```java
@Entity
public class Employee {
  @ElementCollection // creates a new table for mapping the Map
  @CollectionTable(name = "PHONE_NUMBERS") // customize the secondary table
  @MapKeyColumn(name="PHONE_TYPE") // customize the column that stores the key of the map
  @Column(name="PHONE_NUM") // customize the value column
  private Map<String, String> phoneNumbers;
}


// e.g. 2
@Entity
public class Employee {
  @ElementCollection // creates a new table for mapping the Map
  @CollectionTable(name = "PHONE_NUMBERS") // customize the secondary table
  @MapKeyColumn(name="PHONE_TYPE") // customize the column that stores the key of the map
  @Column(name="PHONE_NUM") // customize the value column
  @MapKeyEnumerated(EnumType.STRING) // store enum key as string
  private Map<PhoneType, String> phoneNumbers;
}


// e.g. 3
@Entity
public class Department {

  // since the value is an entity, hence it will be relational mapping
  @OneToMany // creates the join table to map the relationship
  @MapKey(name="id") // tells the JPA that the key will be the id property of the value object.
  private Map<Long, Employee> employees = new HashMap<>();
}


// e.g. 4
@Entity
public class Department {
  @ElementCollection // since the value is a basic type
  @CollectionTable(name = "EMP_RANKS")
  @MapKeyJoinColumn(name="EMP_ID") // customize the join column in the collection table
  @Column(name="RANK")
  private Map<Employee, Integer> employeesRanks = new HashMap<>();
}

// In JPA, using Map with key as embeddable is discouraged.
```


## EJB - Enterprise Java Beans
EJB is a server side software element that summarizes business logic of an application. EJB technology was created to standardize all elements of business logic code under a single interface. EJBs use container-based management to provide features like persistence, events, messaging, RPC, security, concurrency and transaction processing.

Java EE considers following type of applications -
| Web Application | EJB Application | Enterprise Application |
| ------------- | ------------- | -------------------------- |
| composed of JSP, HTML, CSS etc. | composed of Enterprise Java Beans (stateless, statefull, message driven) | kind of wrapper for Web Application and EJB application. |
| the main purpose is representing the user interface layer | EJBs provide the tools needed for building the business logic layer | It can include many web application or EJB application |
| WebContainers such as tomcat must provide a web container to deploy this kind of application | runs over EJB containers. Tomcat an't run EJB applications. TomEE is an example EJB container. | full Java EE application server like Glasfish, JBoss, Weblogic etc. is needed for deploying enterprise applications. |
| The artifcat for deployment is a WAR archive. | The artifact for deployment is a JAR archive. | The artifact for deployment is an EAR file.|


EJB 1.0 and 2.0 were considered confusing. It had took a limiting "all or nothing" approach to implementation. Because of this lightweight alternatives to EJB-functions were created, like Spring,, Hibernate, Google Guice etc.

EJB 3.0 took a complete overhaul and mirros many features from the lightweight alternatives that had replaced it.

### Features of EJB specification framework
- Declarative Metadata
- Configuration By Exception
- Dependency Management (of EJB components). 

CDI provides dependency management in general of any class or interface. EJB also has a feature of dependency management of EJB container managed beans. EJB does not involve context but CDI is contextual.
```
// both of the below statement are valid and does the same job.
@EJB EJBService ejbService;
@Inject EJBService ejbService;
```

- Lifecycle Management (of EJB components)
- Scalability
- Transactionality
- Security
- Portability

### EJB component model
The EJB specifies a server-side component model using a set of classes and interfaces from the **javax.ejb** package.

The EJB components can be broadly categorised into -
1. Session Beans
  - Stateless
  - Stateful
2. Message Driven Beans


**Stateless EJB** : normally performs independent operation. Does not have any associated client state, but it may preserve its instance state.
EJB container normally creates a pool of few stateless bean's object and use these objects to process client's request.

```java
@Stateless
public class QueryService {
  public List<Employee> getEmployees() {
    // some implementation
  }
}
```


**Stateful EJB** : It preserves the conversational state with client. It keeps associated client state in its instance variables. A seperate stateful session bean is created for each client's request.
```java
@Stateful
public class UserSession implements Serializable {
  public String getCurrentUserName() {
  
  }
}
```
EJB container can passivate the stateful bean (to free up memory) hence stateful beans implement Serializable. When need be a passive stateful bean can be activated again by container.

**Sigleton EJB** : only one instance is created and that instance lies till the lifetime of the application.
```java
@Singleton
public class ApplicationState {
  
}
```

### EJB Component lifecycle
**@PostConstruct** : called when the bean has been instantiated and all the dependencies has been resolved.\
**@PreDestroy** : called just before the bean is available for garbage collection.

**@PrePassivate** : only for stateful beans. called just before the bean is hibernated.\
**@PostActivate** : only for stateful beans. called after the bean is activated from passivation sate.


## JPA - Transaction Management
Transaction is an abstraction that is used to group togethere a series of opeation, that must all complete together or none should be allowed.
properties of a transaction - 
1. Atomicity - a transaction must be all-or-nothing
2. Consistency - data must be in consistent state when a transaction starts and when it ends. 
3. Isolation - the intermediate state of transaction must be invisibe to other transactions. As a result, transactions that run concurrently appear to be searialized.
4. Durability - after a transaction successfully completes the changes to the data persist are not undone, even in the event of a system failure.


### Bean managed transaction & Container managed transactions
Container managed - when the task of transaction management is handeled by the container.
```java
@Stateless
public class QueryService {
  @Inject
  EntityManager entityManager;
  
  public List<Employee> getEmployees() {
  
  }
}
```


Bean managed - it means that the developer wants to take over the task of transaction managemet and wants to manually manage the transaction.
```java
@Stateless
@TransactionManagement(TransactionManagementType.BEAN)
public class QueryService {
  @Inject
  EntityManager entityManager;
  
  public List<Employee> getEmployees() {
  
  }
}
```

### Transactions Management attributes
**@TransactionAttribute** annotation
@TransactionAttribute(TransactionAttribute.NEVER) - means the method should not run in a transaction. if a transaction is there, RemoteException is thrown.
@TransactionAttribute(TransactionAttribute.MANDATORY) - a transaction is must to run the method. if a transaction is not there, a TransactionRequiredException is thrown
@TransactionAttribute(TransactionAttribute.REQUIRED) - a transaction is must to run the method. If a transaction is not there, then container create a new transaction.
@TransactionAttribute(TransactionAttribute.REQUIRES_NEW) - a new transaction is required to run the method. if there is already a transaction, then the container suspends the existing transaction and creates a new. The container resumes the existing transaction after the method is complete.
@TransactionAttribute(TransactionAttribute.SUPPORTS) - transaction is not required to run the method. But if there is a transaction then also the method executes within a transaction. No new transaction should be created.
@TransactionAttribute(TransactionAttribute.NOT_SUPPORTED) - transaction is not required to run the method. But if there is a transaction, the container suspends the transaction and executes the method without transaction. The container resumes the existing transaction after the method execution.


### Persistent Unit
The persistent unit is the main configuration of entity classes. A Java EE may have many entity classes, and there may be a need to segregate these classes in to descrete units. Every descrete unit of these entity classes is defined in a persistent unit.

### Persistence Context
A persistent context is a set of entity instances in which for any persistent entity identity there is a unique identity instances. Within the persistent context, the entity instances and their lifecycle are managed.

### Entity Manager
The entity manager is an API hat manages the lifecycle of the entity instance. An EntityManager object manages a set of entities that are defined by a persistence unit. Each entity manager is associated with a persistent context.

1. **EntityManager.persist()** - operation is used to insert a new object into the database. persist does not directly insert the object into database, it just registers the new object in the persistsence context. When the transaction is committed, or when the.persistence context is flushed, then the object will be inserted into the database.

If the object uses a generated Id, it will normally be assigned when persist is called. But when IDENTITY sequencing is used then Id is only assigned on commit or flush.

persist can only be called on entity objects and not on Embeddable objects. Embeddable objects are automatically persisted as part of their owning entity.

calling persist is not always required. If new object relates to existing object in the persistence context, and the relationship is cascade persist, then it will automatically inserted when the transaction is committed or when the persistent context is flushed.


2. **EntityManager.find()** - used to fretriev an entity defined. It returns null if entity is not found in the database. If the current persistence context contains the entity instance, it will be returned from there, otherwise a select query will be fired in the database.


3. **EntityManager.remove()** - used to delete an object from the database. remove does not directly delete the object from the database, it marks the object to be deleted in the persistence context. When the transaction is committed, or if the persistence context is flushed, then the object will be deleted from the database.

calling remove on an object will also cascade the remove operation across any relationship that is marked as cascade remove.


4. **EntityManager.merge()** - is used to merge the changes made to a detached object into the persistence context. merge does not direcly update the changes to the database, it merges the changes into the persistence context.

Normally merge is not required. To update an object you simply need to read it, then change the state through its setter methods, then commit the transaction.

merge is only required when you have a detached copy of the persistence object. A detached cobject is one which was read through a different entity manager (or in a different transaction), or one that is cloned, or serialized.

calling merge on an object will also cascade the merge operation across any relationship that is marked as cascade merge.


**Detached vs Managed**
JPA defines two main states for a given persistence context, managed and detached.
- A managed object is one that was read in the current persistence context or registered with the persistence context. The persistence context will track changes to that object and maintain its object identity. 
- A detached object is one that is not managed by the current persistence context. This could be an object read through a different persistence context, or an object that was cloned or serialized. A new object is also considered detached untill persist is called on it.


### cascading
if any operation is performed on an entity and the related entity also needs to be affected, then cascading is needed.

In JPA there are following cascading types (javax.persistence.CascadeType)
- ALL : propgates all operations from a parent to a child entity
- PERSIST : propogates the persist operation from a parent to a child entity.
- MERGE : propogates the merge operation from a parent to a child entity.
- REMOVE : propogates the remove operation from parent to a child entity.
- REFRESH : reread the value of a given instance from the database. (In some cases, we may change an instance after persisting in database, but later we nee to undo those changes. In this case when we do refresh on parent entity, the child entity also gets reloaded from the database.)
- DETACH : when the parent entity is detached, the child entity also gets detached.

Hibernate supports addition cascade types (org.hibernate.annotations.CascadeType)
- REPLICATE : the replicate option is used when we have more than one datasource and we want the data in sync. The sync operation also propogates to child entities whenever performed on the parent entity.
- SAVE_UPDATE : it's useful when we perform hibernate specific operations like save, update and saveOrUpdate.
- LOCK : reattaches the entity and its child entity with the persistence context again.
- DELETE : same as JPA CascadeType.REMOVE

## JPQL - Java Persistence Query Language
String based standard API for querying entities. Similar to SQL but query on entities.

JPQL queries are portable and can be translated into variuos SQLs using different dialects.

Another API for query on entities is **criteria-API** which provides type saftey, whereas JPQL does not provide type saftey.

JPQL queries can be of two types.
1. Named Queries - static, predefined and named. The query can be referred anywhere else with its name. Named queries can be optimized by the provider because they are compiled.
2. Dynamic Queries - defined dynamically at runtime. They have to be compiled every time.

1. Select Queries

**SELECT <select_expression>
  FROM <from_clause>
  [WHERE <conditional_expression>]
  [PRDER BY <order_by_clause>]**
 
 ```
 @NamedQuery(name="SELECT_ALL_DEPARTMENTS", query="Select d from Department d") // to select all objects of department.
 
 @NamedQuery(name="GET_DEPARTMENT_NAMES", query="select d.departmentName from Department d") // to get names of all departments
 
 // passing to entity manager
 entityManager.createNamedQuery("SELECT_ALL_DEPARTMENTS", Department.class);
 
 @NamedQuery(name="EMP_NAME_SALARY", query="select e.name, e.salary from Employee e") // to select only name and salary of employees.
 
 Collection<Object[]> empNameAndSalaries = entityManager.createNamedQuery("EMP_NAME_SALARY", Object[].class).getResultList(); // gives a list of array. each array contain name of employee and its salary.
 
 // constructor expressions
 public class EmployeeDetails {
  private String name;
  private BigDecimal salary;
  
  public EmployeeDetails(String name, BigDecimal salary) {
    this.name = name;
    this.salary = salary;
  }
 }
 
 @NamedQuery(name="EMP_NAME_SALRY2", query="select new com.demo.EmployeeDetails(e.name, e.salary) from Employee e") // JPQL with constructor expression
 // the provider will call the constructor the EmployeeDetails class with the selects to create object corresponding to each row.
 // note the object does not have to be an entity.
 
 Collection<EmployeeDetails> listEmployeeDetails = entityManager.createNamedQuery("EMP_NAME_SALRY2", EmployeeDetails.class).getResultList();
 
 
 // joining two entities
 @NamedQuery(name="SELECT_EMP_ALLOWANCES", query="select al from Employee e join e.employeeAllowances al") // select all employee allowances
 
 
 // joining maps
 @NamedQuery(name="", query="select e.name, KEY(p), VALUE(p) from Employee e join e.phoneNumbers p")
 @Entity
public class Employee {
  @ElementCollection // creates a new table for mapping the Map
  @CollectionTable(name = "PHONE_NUMBERS") // customize the secondary table
  @MapKeyColumn(name="PHONE_TYPE") // customize the column that stores the key of the map
  @Column(name="PHONE_NUM") // customize the value column
  private Map<String, String> phoneNumbers;
}


// Fetch join
@NamedQuery(name="", query="select e from Employee e join fetch e.employeeAllowances") // the use of fetch keyword makes sure to fetch the allowances along with the employee. The allowances will not be lazily loaded.

 
 ```
 
 

2. Aggregate Queries
way to summarize data

```
// sum
public Cleection<Object[]> getTotalEmployeeSalariesByDept() {
  TypedQuery<Object[]> query = entityManager.createQuery("select d.name, sum(e.basicSalary) from Department d join d.employees e group by d.name", Object[].class);
}

// average
// average of employee salaries in a department, ecluding managers
public Cleection<Object[]> getAvgEmpSalariesByDept() {
  TypedQuery<Object[]> query = entityManager.createQuery("select d.name, avg(e.basicSalary) from Department d join d.employees e where e.subordinates is empty group by d.name", Object[].class);
}

// count
public Cleection<Object[]> countEmployeesByDepartment() {
  TypedQuery<Object[]> query = entityManager.createQuery("select d.name, count(e) from Department d join d.employees e group by d.name", Object[].class);
}

// max/min
public Cleection<Object[]> getEmployeesLowestSalryByDept() {
  TypedQuery<Object[]> query = entityManager.createQuery("select d.name, max(e) from Department d join d.employees e group by d.name", Object[].class);
}
```

3. Update Queries
4. Delete Queries


## JPA - Bean Validation

These validations are done just before an entity is persisted. If validation are not successful, ConstraintViolationException is thrown.

```
@NotEmpty(message= "Department name must be set") // if name is empty, the message will be passed to the user.
private String name; 

@Past(message = "Date of birth must be in past") // ensures that the date is in past
private LocalDate dateOfBirth;

@PastOrPresent(message="Hired date must be in past or present")
private LocalDate hireDate;

@DecimalMax(value="60", message="Age cannot exceed 60")
private int age;

@DecimalMin(value="5000", message="basic salary must be equal to or exceed 5000")
private int basicSalary;

@Email(message="Email must be in the form user@domain.xyz")
private String email;

@Pattern(regexp="", message="department name must be in format DEP-XXXXXX")
private String departmentName;

@Size(max="40", message="name must be less than 40 chars")
@NotEmpty(message= "name must be set")
private String name;
```

## JPA - Entity Lifecycle callbacks
**@PrePersiste** : just before the entity is about to be persisted in the persistence context.
**@PostPersiste** : just after persist method is called. 
**@PreUpdate** : just before an entity is updated. there is no specific update method. so behaviour is dependent on persistence provider.
**@PostUpdate** : 
**@PostLoad** : 

```java
@Entity
public class Employee {
  
  private LocalDate dateOfBirth;
  
  private int age;
  
  @PrePersist
  private void init() {
    this.age = Period.between(dateOfBirth, LocalDate.now()).getYears();
  }  
}
```
