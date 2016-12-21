#Table of Content

*   JPA 简介
*   JPA 组成架构
*   JPA 配置
*   JPA 基础项目
    *   EntityManager
    *   Transactions
    *   Tables
*   JPA 继承类的实现
*   JPA 实体关系的实现
    *   OneToOne
    *   OneToMany
    *   ManyToMany
    *   Embedded / ElementCollection
*   Converters
*   Criteria API (条件查询)
*   Sequence
*   实例分析

##JPA 简介

*   JPA是为了解决面向对象程序设计中实例模板(类)到数据库中的条目(比如关系型数据库中的表)实现的一类关系映射API的总称
*  JPA 用JPQL 作为一种可以通用的SQL语句,i.e. 这些语句在任何一个Database 上都可以进行查询或者增删操作，从而免除重复劳动。
*  JPA 实际上是对 JDBC(Java Database Connectivity)的实例的封装，以便于用户使用
*  实际上，HAPI用到了Hibernate框架（一种JPA封装实现方式），但是却并未完全实例化Hibernate框架，而是在此基础上重载并实现了DAO(Data Access Object)，理由暂时不知道，但是这样导致拿Hibernate的框架硬套代码会发现不完全一样。


##JPA组成架构：
![Hibernate JPA 框图](leanote://file/getImage?fileId=5813a4c755c2364652000000)

*   实体（Entities）
*   对象-关系型元数据（Object-relational metadata）: 应用程序的开发者们必须正确设定Java类和它们的属性与数据库中的表和列的映射关系。有两种设定方式：通过特定的配置文件建立映射；或者使用在新版本中支持的注解。
*   Java持久化查询语句（Java Persistence Query Language - JPQL): 保持SQL的通用化(不用针对数据库写不一样的SQL)

##JPA配置：

*   依靠persistence.xml文件，一般位于 xxx/META-INF/ 文件夹下
*   找不到理论上会报错
*   一个简单的代码示例如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
 
<persistence version="2.0"
    xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence 
http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">
 
    <persistence-unit name="MyPU" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
 
        <class>page20.Person</class>
        <class>page20.Cellular</class>
        <class>page20.Call</class>
        <class>page20.Dog</class>
 
        <exclude-unlisted-classes>true</exclude-unlisted-classes>
 
        <properties>
            <property name="connection.driver_class" value="org.h2.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:h2:~/jpa"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="create"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

这段代码里面显示了一个JPA框架最基本单元 持久化实例 (persistence-unit) 的配置模式，主要需要定义:

*   provider : 实例化的类是哪个
*   class
*   properties : 和数据库通信的设置

而关于JPA的依赖可以在**maven**的pom.xml中配置

##JPA基础项目

###EntityManager
```java
public class Main {
	private static final Logger LOGGER = Logger.getLogger("JPA");

	public static void main(String[] args) {
		Main main = new Main();
		main.run();
	}

	public void run() {
		EntityManagerFactory factory = null;
		EntityManager entityManager = null;
		try {
			factory = Persistence.createEntityManagerFactory("PersistenceUnit");
			entityManager = factory.createEntityManager();
			persistPerson(entityManager);
		} catch (Exception e) {
			LOGGER.log(Level.SEVERE, e.getMessage(), e);
			e.printStackTrace();
		} finally {
			if (entityManager != null) {
				entityManager.close();
			}
			if (factory != null) {
				factory.close();
			}
		}
	}
    ...
```

*   EntityManagerFactory 为工厂，用于创建EntityMangager实例
*   EntityManager用于JPA中与DB交互
*   persistPerson是一个函数，通过EntityManager实例和**transaction**实现发送数据到DB

### Transaction

persistPerson代码如下：

```java
private void persistPerson(EntityManager entityManager) {
	EntityTransaction transaction = entityManager.getTransaction();
	try {
		transaction.begin();
		Person person = new Person();
		person.setFirstName("Homer");
		person.setLastName("Simpson");
		entityManager.persist(person);
		transaction.commit();
	} catch (Exception e) {
		if (transaction.isActive()) {
			transaction.rollback();
		}
	}
}
```

*   利用**transaction**和**persist**方法链接数据库
*   注意抛出异常和回滚，保持程序健壮性

###Tables

利用Annotation技术,通过**@Entity**将Person这个class映射到数据库中的表(下面的例子为T_PERSON)。注解方式有两种：

#### **在 getter 和 setter方法中注解**
```java
@Entity
@Table(name = "T_PERSON")
public class Person {
	private Long id;
	private String firstName;
	private String lastName;

	@Id
	@GeneratedValue
	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	@Column(name = "FIRST_NAME")
	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	@Column(name = "LAST_NAME")
	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
}
```

#### **在 变量上注解**

```java
@Entity
@Table(name = "T_PERSON")
public class Person {
	@Id
	@GeneratedValue
	private Long id;
	@Column(name = "FIRST_NAME")
	private String firstName;
	@Column(name = "LAST_NAME")
	private String lastName;
	...
```
就是把column从方法拿到了变量。上面的两段代码**在没有实例继承的情况下是等价的**，做了这样一些事：

*   注解了这是一张数据库的表，表名为 T_PERSON (@Table似乎非强制)
*   添加了主键 (@Id) 和自动生成开关(@GeneratedValue)
*   添加了其余两列 (@Column), Column 可选参数举例:
    *   name
    *   length: 限制字符串长度
    *   nullable: 是否允许NULL的出现，如果不允许在此列尝试添加NULL时会导致Transaction回滚
    *   unique: 是否唯一

##JPA 继承类的实现
##JPA 实体关系的实现
###   OneToOne
###   OneToMany
###   ManyToMany
###   Embedded / ElementCollection
##   Converters
##   Criteria API (条件查询)
##   Sequence
##   实例分析

在 **jpaserver-example**里面出现了对**jpaserver-base**的大量依赖，经过查阅源码，发现base里面才是主要的服务器架构内容。

和这份文档有关的部分主要集中在 xxx/src/main/java/ca.uhn.fhir.jpa/entity 文件夹中，以**search** class为例:

```java
@Entity
@Table(name = "HFJ_SEARCH", uniqueConstraints= {
	@UniqueConstraint(name="IDX_SEARCH_UUID", columnNames="SEARCH_UUID")
}, indexes= {
	@Index(name="JDX_SEARCH_CREATED", columnList="CREATED")
})
//@formatter:on
public class Search implements Serializable {

	private static final long serialVersionUID = 1L;

	@Temporal(TemporalType.TIMESTAMP)
	@Column(name="CREATED", nullable=false, updatable=false)
	private Date myCreated;

	@Id
	@GeneratedValue(strategy = GenerationType.AUTO, generator="SEQ_SEARCH")
	@SequenceGenerator(name="SEQ_SEARCH", sequenceName="SEQ_SEARCH")
	@Column(name = "PID")
	private Long myId;

	@OneToMany(mappedBy="mySearch")
	private Collection<SearchInclude> myIncludes;

	@Temporal(TemporalType.TIMESTAMP)
	@Column(name="LAST_UPDATED_HIGH", nullable=true, insertable=true, updatable=false)
	private Date myLastUpdatedHigh;

	@Temporal(TemporalType.TIMESTAMP)
	@Column(name="LAST_UPDATED_LOW", nullable=true, insertable=true, updatable=false)
	private Date myLastUpdatedLow;

	@Column(name="PREFERRED_PAGE_SIZE", nullable=true)
	private Integer myPreferredPageSize;
	
	@Column(name="RESOURCE_ID", nullable=true)
	private Long myResourceId;
	
	@Column(name="RESOURCE_TYPE", length=200, nullable=true)
	private String myResourceType;

	@OneToMany(mappedBy="mySearch")
	private Collection<SearchResult> myResults;

	@Enumerated(EnumType.ORDINAL)
	@Column(name="SEARCH_TYPE", nullable=false)
	private SearchTypeEnum mySearchType;

	@Column(name="TOTAL_COUNT", nullable=false)
	private Integer myTotalCount;

	@Column(name="SEARCH_UUID", length=40, nullable=false, updatable=false)
	private String myUuid;

	public Date getCreated() {
		return myCreated;
	}

	public Long getId() {
		return myId;
	}

	public Collection<SearchInclude> getIncludes() {
		if (myIncludes == null) {
			myIncludes = new ArrayList<SearchInclude>();
		}
		return myIncludes;
	}
	
	public Date getLastUpdatedHigh() {
		return myLastUpdatedHigh;
	}

	public Date getLastUpdatedLow() {
		return myLastUpdatedLow;
	}

	public DateRangeParam getLastUpdated() {
		if (myLastUpdatedLow == null && myLastUpdatedHigh == null) {
			return null;
		} else {
			return new DateRangeParam(myLastUpdatedLow, myLastUpdatedHigh);
		}
	}

	public Integer getPreferredPageSize() {
		return myPreferredPageSize;
	}

	public Long getResourceId() {
		return myResourceId;
	}
	
	public String getResourceType() {
		return myResourceType;
	}

	public SearchTypeEnum getSearchType() {
		return mySearchType;
	}


	public Integer getTotalCount() {
		return myTotalCount;
	}

	public String getUuid() {
		return myUuid;
	}

	public void setCreated(Date theCreated) {
		myCreated = theCreated;
	}
	public void setLastUpdated(Date theLowerBound, Date theUpperBound) {
		myLastUpdatedLow = theLowerBound;
		myLastUpdatedHigh = theUpperBound;
	}
	
	public void setLastUpdated(DateRangeParam theLastUpdated) {
		if (theLastUpdated == null) {
			myLastUpdatedLow = null;
			myLastUpdatedHigh = null;
		} else {
			myLastUpdatedLow = theLastUpdated.getLowerBoundAsInstant();
			myLastUpdatedHigh = theLastUpdated.getUpperBoundAsInstant();
		}
	}

	public void setPreferredPageSize(Integer thePreferredPageSize) {
		myPreferredPageSize = thePreferredPageSize;
	}
	
	public void setResourceId(Long theResourceId) {
		myResourceId = theResourceId;
	}


	public void setResourceType(String theResourceType) {
		myResourceType = theResourceType;
	}

	public void setSearchType(SearchTypeEnum theSearchType) {
		mySearchType = theSearchType;
	}

	public void setTotalCount(Integer theTotalCount) {
		myTotalCount = theTotalCount;
	}

	public void setUuid(String theUuid) {
		myUuid = theUuid;
	}

	private Set<Include> toIncList(boolean theWantReverse) {
		HashSet<Include> retVal = new HashSet<Include>();
		for (SearchInclude next : getIncludes()) {
			if (theWantReverse == next.isReverse()) {
				retVal.add(new Include(next.getInclude(), next.isRecurse()));
			}
		}
		return Collections.unmodifiableSet(retVal);
	}

	public Set<Include> toIncludesList() {
		return toIncList(false);
	}

	public Set<Include> toRevIncludesList() {
		return toIncList(true);
	}
	
}
```
