---
layout: post
title:  JPA API
categories: XAP97
parent: other-data-access-apis.html
weight: 200
---

{% compositionsetup %}{% summary %}Using JPA with GigaSpaces.{% endsummary %}

# Overview

The Java Persistency API (JPA) is a Java programming language framework managing relational data in applications using Java Platform. GigaSpaces JPA allows you to use JPA's functionality, annotations and execute JPQL queries on Space. GigaSpaces JPA implementation is based on [OpenJPA](http://openjpa.apache.org/).

{% info %}
It is highly recommended that you [get yourself familiar with JPA](http://download.oracle.com/javaee/6/tutorial/doc/bnbpz.html) before reading this page.
It is also recommended that you take the [XAP PetClinic JPA Tutorial](./your-first-jpa-application.html) which describes how a standard JPA application (the Spring PetClinic) can be adapted to XAP JPA and deployed on to the XAP runtime environment
{% endinfo %}

# GigaSpaces JPA Configuration

### OpenJPA

OpenJPA's jar file is included with the GigaSpaces ditribution (provided under `<GigaSpaces root>/lib/platform/jpa`), and the GigaSpaces-specific JPA implementation classes are part of the OpenSpaces jar (located under `<GigaSpaces root>/lib/required/gs-openspaces.jar`).
Maven users should define the following dependency in their `pom.xml` file:

{% highlight xml %}
<dependencies>
  <dependency>
    <groupId>org.apache.openjpa</groupId>
    <artifactId>openjpa</artifactId>
    <version>2.0.0</version>
  </dependency>
</dependencies>
{% endhighlight %}

![new-in-801-banner.png](/attachment_files/new-in-801-banner.png)

##### OpenJPA 2.0.1

GigaSpaces 8.0.1 uses OpenJPA version 2.0.1.
Note that it's no longer needed to set a maven dependency for OpenJPA since OpenSpaces has an OpenJPA dependency.
If from some reason one needs an OpenJPA maven dependency set, make sure to set the OpenJPA version to "2.0.1".

### The persistence.xml file

To enable the GigaSpaces JPA implementation you should specify  the following 3 mandatory properties in your `persistence.xml`:

- `BrokerFactory` should be set to `"abstractstore"` which tells OpenJPA that an alternate `StoreManager` (the layer responsible for interaction with underlying dfatabase) is going to be used.
- `abstractstore.AbstractStoreManager` should be set to `"org.openspaces.jpa.StoreManager"` which tells OpenJPA to use the OpenSpaces `StoreManager`.
- `LockManager` should be set to `"none"` since OpenJPA's default lock manager is set to `"version"` (Optimistic locking) which is currently unsupported (it will be supported in one of the future 8.0 service packs)

Your persistence.xml file should be placed in any **/META-INF folder in your classpath.

##### GigaSpaces JPA 8.0.1

In 8.0.1, it is no longer needed to set the "abstractstore.AbstractStoreManager" property.
Instead, make sure to set the "BrokerFactory" property to "org.openspaces.jpa.BrokerFactory" as shown in the example below.

The following is an example of a GigaSpaces JPA persistence.xml configuration file:

{% highlight xml %}
<persistence-unit name="gigaspaces" transaction-type="RESOURCE_LOCAL">
	<provider>org.apache.openjpa.persistence.PersistenceProviderImpl</provider>
	<properties>
            <property name="BrokerFactory" value="abstractstore"/>
            <property name="abstractstore.AbstractStoreManager" value="org.openspaces.jpa.StoreManager"/>
            <property name="LockManager" value="none"/>
	</properties>
</persistence-unit>
{% endhighlight %}

{% highlight xml %}
<persistence-unit name="gigaspaces" transaction-type="RESOURCE_LOCAL">
	<provider>org.apache.openjpa.persistence.PersistenceProviderImpl</provider>
	<properties>
            <property name="BrokerFactory" value="org.openspaces.jpa.BrokerFactory"/>
            <property name="LockManager" value="none"/>
	</properties>
</persistence-unit>
{% endhighlight %}

##### Transaction Read Lock Level

GigaSpaces JPA default read lock level is set to "read" which is equivalent to GigaSpaces' ReadModifiers.REPEATABLE_READ.In order to use ReadModifiers.EXCLUSIVE_READLOCK the "ReadLockLevel" property should be set to "write":

{% highlight xml %}
  <property name="ReadLockLevel" value="write"/>
{% endhighlight %}

### Space Connection Injection

Specifying a space connection URL or a space instance can be done in one of the following ways:

##### Referencing an Existing Space Instance through Factory Properties

Specifying a space instance is possible when creating an `EntityManagerFactory` in the following way:

{% highlight java %}
GigaSpace gigaspace = ...
Properties properties = new Properties();
properties.put("ConnectionFactory", gigaspace.getSpace());
EntityManagerFactory emf = Persistence.createEntityManagerFactory("gigaspaces", properties);
{% endhighlight %}

##### Injection using Spring

It is possible to inject either an `EntityManager` or `EntityManagerFactory` using Spring. Before reading this, it is recommend that you [make yourself familiar with Spring's JPA support](http://static.springsource.org/spring/docs/3.0.x/reference/orm.html#orm-jpa).
In the following example we'll see how to inject a space-based `EntityManagerFactory`.
The following Spring xml configuration file declares a space, an `EntityManagerFactory`, a transaction manager and a JPA service bean (this is our DAO):

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:os-core="http://www.openspaces.org/schema/core"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.openspaces.org/schema/core http://www.openspaces.org/schema/8.0/core/openspaces-core.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

	<!-- space definition -->
        <os-core:space id="space" url="/./jpaSpace" lookup-groups="test"/>

        <!-- gigaspace definition -->
        <os-core:giga-space id="gigaSpace" space="space"/>

        <!-- JPA entity manager factory definition -->
	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
                <!-- this relies on the fact that our persistence.xml file defines a persistence unit named "gigaspaces" -->
		<property name="persistenceUnitName" value="gigaspaces"/>
		<property name="jpaVendorAdapter">
			<bean class="org.openspaces.jpa.OpenSpacesJpaVendorAdapter">
				<property name="space" value="#{gigaSpace.space}"/>
			</bean>
		</property>
	</bean>

        <!-- JPA transaction manager definition -->
	<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
		<property name="entityManagerFactory" ref="entityManagerFactory" />
	</bean>

        <!-- support annotations -->
	<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor" />
	<context:annotation-config/>
	<tx:annotation-driven transaction-manager="transactionManager"/>

        <!-- JPA example service definition -->
	<bean id="jpaService" class="org.openspaces.jpa.JpaService" />
</beans>
{% endhighlight %}

Note that in our DAO, we'll have Spring inject the `EntityManager` using the `@PersistentContext` annotation. Spring will make sure to create an `EntityManager` using the `EntityManagerFactory` we have defined at the beginning of every transaction.

{% highlight java %}
@Repository
@Transactional
public class JpaService {
    @PersistenceContext
    private EntityManager em;

    public JpaService() {
    }

    @Transactional
    public void persistObject() {
        em.persist(...);
    }
}
{% endhighlight %}

Detailed information regarding persistence.xml can be found in [OpenJPA's Manual](http://openjpa.apache.org/builds/2.0.0/apache-openjpa-2.0.0/docs/manual/jpa_overview_persistence.html#jpa_overview_persistence_xml).

### Listing Your Persistent Classes

{% info %}
Information regarding Entities declaration can be found [here](./jpa-api.html#GigaSpaces JPA Entities)
{% endinfo %}

When working with persistent classes, you have a number of ways to make the JPA layer aware of them:

- When using Spring, this will be done automatically for you unless otherwise specified (see below). So Spring will scan you classpath looking for classes that have the `@Entity` annotation.
- When not using Spring, you have two options:
    - Point to an `orm.xml` file your `persistence.xml` file

{% highlight xml %}
  <persistence-unit name="gigaspaces" transaction-type="RESOURCE_LOCAL">
    <mapping-file>META-INF/orm.xml</mapping-file>
    <exclude-unlisted-classes/>
  </persistence-unit>
{% endhighlight %}

    - Use the `<class>` tag in your `pesistence.xml` file. For example, if you're going to use the classes: `Trade`, `Book` & `Author`, you should list them in your `persistence.xml` file as follows:

{% highlight xml %}
<persistence-unit name="gigaspaces" transaction-type="RESOURCE_LOCAL">
	<provider>org.apache.openjpa.persistence.PersistenceProviderImpl</provider>
        <class>org.openspaces.objects.Trade</class>
        <class>org.openspaces.objects.Book</class>
        <class>org.openspaces.objects.Author</class>
	<properties>
           <property name="BrokerFactory" value="abstractstore"/>
           <property name="abstractstore.AbstractStoreManager" value="org.openspaces.jpa.StoreManager"/>
           <property name="LockManager" value="none"/>
	</properties>
</persistence-unit>
{% endhighlight %}

### Enhancing Your Classes

JPA classes are monitored at runtime for automatic dirty detection. To be transparent to the user, this requires bytecode enhancement to take place.
OpenJPA offers 3 options to do so:

1. Enhance at build time using an Ant or a Maven script.
1. Enahnce at runtime by using OpenJPA's javaagent enhancer.
1. When using Spring, you can [specify a `LoadTimeWeaver`](http://static.springsource.org/spring/docs/3.0.x/reference/orm.html#orm-jpa-setup-lcemfb) that will enhance the classes at load time.

{% info %}
Note that the first option (build time enhancement) is the best one in terms of performance and suitability for the GigaSpaces runtime environment.
Detailed information regarding how to enhance your entities can be found on [OpenJPA's Entity Enhancement page](http://openjpa.apache.org/entity-enhancement.html).
{% endinfo %}

# GigaSpaces JPA Entities

{% info %}
An entity class must meet the following requirements:

1. The class must be annotated with the `javax.persistence.Entity` annotation.
1. The class must have a public no-argument constructor (the class may have other constructors).
1. GigaSpaces and JPA annotations can only be declared on Getters, NOT on fields.
{% endinfo %}

### Annotations

GigaSpaces JPA Entities must have both JPA and GigaSpaces annotations for the following annotations:

{: .table .table-bordered}
|GigaSpaces|JPA|
|:---------|:--|
| `@SpaceId`| `@Id/@EmbeddedId`|
| `@SpaceExclude`| `@Transient`|

As with GigaSpaces POJOs, you may use the `@SpaceIndex` & `@SpaceRouting` annotations with GigaSpaces JPA entities.

{% info %}
Please note that indexes should only be declared in the owning entity of a relationship.
Examples can be found on the [JPA Relationships](./jpa-relationships.html) page.
{% endinfo %}

Here's an example of a basic JPA Entity:

{% highlight java %}
@Entity
public class Trade {
  private Long id;
  private Double quantity;
  private List<Double> rates;
  private boolean state;

  // Public no-argument constructor
  public Trade() {
  }

  // Both SpaceId and Id should be declared on the id property
  @Id
  @SpaceId
  public Long getId() {
    return this.id;
  }

  // Persistent property, no additional GigaSpaces annotations needs to be used.
  public Double getQuantity() {
    return this.quantity;
  }

  // A persistent collection property. In this case we'll use a GigaSpaces annotation
  // for indexing its values.
  @ElementCollection
  @SpaceIndex(path = "[*]")
  public List<Double> getRates() {
    return this.rates;
  }

  // A transient property. In this case we'll use both GigaSpaces and JPA annotations
  @Transient
  @SpaceExclude
  public boolean getState() {
    return this.state;
  }

  /* Additional Getters & Setters... */

}
{% endhighlight %}

For auto generated Id declaration and complex object Id declaration refer to [JPA Entity Id](./jpa-entity-id.html).

Example of a JPA Owner entity with one to many relationship:

{% highlight java %}
@Entity
public class Owner {
    //
    private Integer id;
    private String name;
    private List<Pet> pets;
    //
    public Owner() {
    }
    public Owner(Integer id, String name, List<Pet> pets) {
        super();
        this.id = id;
        this.name = name;
        this.pets = pets;
    }
    //
    @Id
    @SpaceId
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }

    @SpaceRouting
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @OneToMany(cascade = CascadeType.ALL)
    public List<Pet> getPets() {
        return pets;
    }
    public void setPets(List<Pet> pets) {
        this.pets = pets;
    }
}
{% endhighlight %}

## Non-Indexed Fields

Non-Indexed fields that are not used for queries should be placed within a user defined class (payload object) and have their getter and setter placed within the payload class. This improves the read/write performance since these fields would not be introduced to the space class model.

# JPA Query Language (JPQL)

GigaSpaces JPA supports a subset of JPQL. Here are a few examples of the supported queries:

##### Querying on Properties of Nested Objects

{% highlight java %}
EntityManagerFactory emf = Persistence.createEntityManagerFactory("gigaspaces");
EntityManager em = emf.createEntityManager();
Query query = em.createQuery("SELECT c from org.openspaces.objects.Customer c WHERE c.address.country = 'United States'");
List<Customer> customers = (List<Customer>) query.getResultList();
em.close();
emf.close();
{% endhighlight %}

##### JOIN support for one to many relationship (Owner --> List<Pet>)

{% highlight java %}
EntityManagerFactory emf = Persistence.createEntityManagerFactory("gigaspaces");
EntityManager em = emf.createEntityManager();
Query query = em.createQuery("SELECT o FROM org.openspaces.objects.Owner o JOIN o.pets p WHERE p.name = :name");
query.setParameter("name", "Whiskey");
Owner owner = (Owner) query.getSingleResult();
em.close();
emf.close();
{% endhighlight %}

{% info %}
When specifying entity names in GigaSpaces JPQL the full class qualified name should be used as shown in the above examples.
{% endinfo %}

# Persisting Collection Properties

It's possible to make a collection property persistent by using the `@ElementCollection` annotation.
In the following example we have an entity with a collection of Integers:

{% highlight java %}
@Entity
public class Card {
  // ...

  private List<Integer> numbers;

  @ElementCollection
  @SpaceIndex(path = "[*]") // the list values will be indexed.
  public List<Integer> getNumbers() {
    return this.numbers;
  }

  public void setNumbers(List<Integer> numbers) {
    this.numbers = numbers;
  }

  // ...
}
{% endhighlight %}

In order to query the Card entity using a specific Integer in the numbers collection we use JPQL's `"MEMBER OF"`:

{% highlight java %}
EntityManagerFactory emf = Persistence.createEntityManagerFactory("gigaspaces");
EntityManager em = emf.createEntityManager();
Query query = em.createQuery("SELECT c FROM org.openspaces.objects.Card c WHERE :number MEMBER OF c.numbers");
query.setParameter("number", "10");
Card card = (Card) query.getSingleResult();
em.close();
emf.close();
{% endhighlight %}

# Persisting Enum Properties

JPA allows to persist Enum proeprties using the `@Enumerated` annotation, as shown below:

{% highlight java %}
// A Vehicle entity which has an Enum property
@Entity
public class Vehicle {
  // Enum Declaration
  public enum VehicleType { CAR, TRUCK, BIKE };

  private Integer id;
  private String name;
  private VehicleType type;

  public Vehicle() {
  }

  @Id
  @SpaceId
  public Integer getId() {
    return this.id;
  }

  public String getName() {
    return this.name;
  }

  @Enumerated
  public VehicleType getType() {
    return this.type;
  }

  /* Additional Getters & Setters */

}
{% endhighlight %}

We used the `@Enumerated` annotation for persisting an Enum property.
Please note that specifying a value for the `@Enumerated.value()` attribute has no effect since Enums are saved in GigaSpaces as is.

##### Enums In JPQL

It's possible to query according to an Enum property by setting an Enum parameter or by using the Enum's value in the query string:

{% highlight xml %}
EntityManager em = emf.createEntityManager();

// Query using an Enum parameter
Query query1 = em.createQuery("SELECT vehicle FROM com.gigaspaces.objects.Vehicle vehicle WHERE vehicle.type = :type");
query1.setParameter("type", VehicleType.CAR);
Vehicle result1 = (Vehicle) query1.getSingleResult();

// Query using an Enum in query's string
Query query2 = em.createQuery("SELECT vehicle FROM com.gigaspaces.objects.Vehicle vehicle WHERE vehicle.type = 'BIKE'");
Vehicle result2 = (Vehicle) query2.getSingleResult();
{% endhighlight %}

# Interoperability

One of the nice benefits of the GigaSpaces JPA implementation is that its fully interoperable with the GigaSpaces [native POJO API](./pojo-support.html).
For instance, we can persist a JPA entity and read it using the native POJO-driven Space API:

{% highlight java %}
@Entity
public class Author {
  private Integer id;
  private String name;
  private List<Book> books;

  public Author() {
  }

  @Id
  @SpaceId
  public Integer getId() {
    return this.id;
  }

  public String getName() {
    return this.name;
  }

  @OneToMany(cascade = CascadeType.ALL)
  @SpaceIndex(path = "[*].id")
  public List<Book> getBooks() {
    return this.books;
  }

  // Additional Getters & Setters...

}

@Entity
public class Book implements Serializable {
  private Integer id;
  private String name;

  public Book() {
  }

  @Id
  @SpaceId
  public Integer getId() {
    return this.id;
  }

  public String getName() {
    return this.name;
  }

  // Additional Getters & Setters...

}

GigaSpace gigaspace = ...
Book book1 = new Book(10, "Book Title 1");
Book book2 = new Book(20, "Book Title 2");
List<Book> books = new ArrayList<Book>();
books.add(book1);
books.add(book2);
Author author = new Author();
author.setId(1234);
author.setBooks(books);

// Persist using GigaSpaces JPA..
Properties properties = new Properties();
properties.put("ConnectionFactory", gigaspace.getSpace());
EntityManagerFactory emf = Persistence.createEntityManagerFactory("gigaspaces", properties);
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
em.persist(author);
em.getTransaction().commit();
em.close();

// Read using Space API..
Author result = gigaspace.readById(Author.class, 1234);

// Or even SQLQuery..
SQLQuery<Author> query = new SQLQuery<Author>(Author.class, "id = 1234");
result = gigaspace.read(query);

// Or by a certain book..
query = new SQLQuery<Author>(Author.class, "books[*].id = 10");
result = gigaspace.read(query);
{% endhighlight %}

# Native Query Execution

![new-in-801-banner.png](/attachment_files/new-in-801-banner.png)
GigaSpaces JPA native query execution is a powerful feature used for executing:

- SQLQuery syntax-like queries ([SQLQuery](./sqlquery.html)).
- GigaSpaces Tasks ([Task Execution over the Space](./task-execution-over-the-space.html)).
- GigaSpaces Dynamic Scripts ([Dynamic Language Tasks](./dynamic-language-tasks.html)).

### SQLQuery Execution

SQLQuery execution using JPA native query API is pretty simple and made in the following way:

{% highlight java %}
// SQLQuery execution
EntityManager em = emf.createEntityManagerFactory();
Query query = em.createNativeQuery("name = 'John Doe'", Author.class);
Author author = (Author) query.getSingleResult();

// SQLQuery execution with parameters
query = em.createNativeQuery("name = ?", Author.class);
query.setParameter(1, "John Doe");
author = (Author) query.getSingleResult();
{% endhighlight %}

For more details on the SQLQuery syntax, refer to the [SQLQuery](./sqlquery.html) page.

### Task Execution

Using GigaSpaces JPA native query API it is possible to execute tasks over the space in the following manner:

{% highlight java %}
// Task definition
public class MyTask implements Task<Integer> {

  @TaskGigaSpace
  private transient GigaSpace gigaSpace;
  private Object routing;

  public MyTask(Object routing) {
    this.routing = routing;
  }

  public Integer execute() throws Exception {
    return gigaSpace.count(new Author());
  }

  @SpaceRouting
  public Object getRouting() {
    return this.routing;
  }
}

// Task execution
Query query = em.createNativeQuery("execute ?");    // Special syntax for task execution
query.setParameter(1, new MyTask(1));               // We pass our task instance as a parameter to the query
Integer result = (Integer) query.getSingleResult(); // Task execution always returns a single result
{% endhighlight %}

{% info %}
Please note that task execution using JPA's native query API is always synchronous.
{% endinfo %}

##### Getting an EntityManagerFactory instance in a Task

Its possible to get an EntityManagerFactory instance (according to the bean definition in pu.xml) by implementing the ApplicationContextAware interface.
For example:

{% highlight java %}
public class MyTask implements Task<Integer>, ApplicationContextAware {

  private transient EntityManagerFactory emf;

  public void setApplicationContext(ApplicationContext context) throws BeansException {
    // Get the entityManagerFactory bean
    emf = (EntityManagerFactory) context.getBean("entityManagerFactory");
  }

  public Integer execute() throws Exception {

    // Create an EntityManager..
    EntityManager em = emf.createEntityManager();

    // ...

    em.close();

    return 0;
  }

}
{% endhighlight %}

Another option instead of using the ApplicationContextAware interface is to annotate your Task with the @AutowireTask annotation and annotate the EntityManagerFactory property with a @Resource annotation.

For more information about GigaSpaces tasks refer to [Task Execution over the Space](./task-execution-over-the-space.html).

### Dynamic Script Execution

In addition to Task execution, GigaSpaces JPA native query execution also offers the ability to execute dynamic scripts such as Groovy, JavaScript & JRuby over the space.
Dynamic Script execution over the space is based on Task execution & remoting and therefore its required that your PU will have a remoting scripting executor service:

{% highlight xml %}
<!-- The service exporter exposing the scripting service -->
<os-remoting:service-exporter id="serviceExporter">
     <os-remoting:service ref="scriptingExecutor"/>
</os-remoting:service-exporter>
{% endhighlight %}

The next step is using the exposed scripting service on the client side using JPA's native query API:

{% highlight java %}
// Dynamic Script execution
Script script = new StaticScript("GroovyScript", "groovy", "println 'Dynamic Script Execution using JPA'; return 0");

Query query = em.createNativeQuery("execute ?");     // Special syntax for script execution (similar to task execution)
query.setParameter(1, script);                       // We pass our script as a parameter to the query
Integer result = (Integer) query.getSingleResult();  // Script execution always returns a single result
{% endhighlight %}

For more information about dynamic script execution refer to [Dynamic Language Tasks](./dynamic-language-tasks.html).

# GigaSpaces JPA Limitations

For a list of unsupported JPA features and limitations please refer to [GigaSpaces JPA Limitations](./gigaspaces-jpa-limitations.html).

