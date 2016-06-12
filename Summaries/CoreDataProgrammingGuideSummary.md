# Core Data Programming Guide

[文档地址](https://developer.apple.com/library/watchos/documentation/Cocoa/Conceptual/CoreData)

## Managing Object Life Cycle

### Creating Managed Object Relationships

`NSEntityDescription` 和 `NSManagedObjectContext` 的作用

> A managed object is associated with an entity description (an instance of `NSEntityDescription`) that provides metadata about the object and with a managed object context that tracks changes to the object graph. The object metadata includes the name of the entity that the object represents and the names of its attributes and relationships.

store 中的 record 和 `NSManagedObject` 是 **一对多** 的关系（在多个 context 中），引出“关系”这一概念

> In a given managed object context, a managed object provides a representation of a record in a persistent store. In a given context, for a given record in a persistent store, there can be only one corresponding managed object, but there may be multiple contexts, each containing a separate managed object representing that record. Put another way, there is a **to-one** relationship between a managed object and the data record it represents, but a **to-many** relationship between the record and corresponding managed objects.

不能夸库建立关系

> Core Data does not let you create relationships that cross stores. If you need to create a relationship from objects in one store to objects in another, consider using [Weak Relationships (Fetched Properties)](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/HowManagedObjectsarerelated.html#//apple_ref/doc/uid/TP40001075-CH17-SW13).

#### Relationship Definitions in the Managed Object Model

创建关系前需要考虑的

> There are a number of things you have to decide when you create a relationship. What is the destination entity? Is the relationship a to-one or a to-many? Is the relationship optional? If it’s a to-many, are there maximum or minimum numbers of objects that can be in the relationship? What should happen when the source object is deleted?

例子

> In an object model relationship, you have a source entity (for example, Department) and a destination entity (for example, Employee). You define a relationship between a source entity and a destination entity in the Relationship panel of the Data Model inspector for the source entity.

![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Art/Tomany_relationship_2x.png)

##### Relationship Fundamentals

关系的作用

> A relationship specifies the entity, or the parent entity, of the objects at the destination. 

可以是自己和自己的关系

> This entity can be the same as the entity at the source (a reflexive relationship).

> Relationships do not have to reference a single entity type. If the Employee entity has two subentities, say Manager and Assistant, and Employee is not abstract, then a given department's employees may be made up of Employees (the primary entity), Managers (subentity of Employee), Assistants (subentity of Employee), or any combination thereof.

一对一、一对多、多对多

> In the Type field of the Relationship pane, you can specify a relationship as being to-one or to-many, which is known as its cardinality. To-one relationships are represented by a reference to the destination object and to-many relationships are represented by mutable sets. Implicitly, to-one and to-many typically refer to one-to-one and one-to-many relationships respectively. A many-to-many relationship is one where both a relationship and its inverse are to-many. How you model a many-to-many relationship depends on the semantics of your schema. For details on this type of relationship, see Many-to-Many Relationships.

数量限制

> You can also put upper and lower limits on the number of objects at the destination of a to-many relationship. The lower limit does not have to be zero. You can specify that the number of employees in a department must be between 3 and 40.

optional

> You also specify a relationship as either optional or not optional. If a relationship is not optional, then for it to be valid there must be an object or objects at the destination of the relationship.

数量限制和 optional 可以共存

> Cardinality and optionality are orthogonal properties of a relationship. You can specify that a relationship is optional, even if you have specified upper and/or lower bounds. This means that there do not have to be any objects at the destination, but if there are, then the number of objects must lie within the bounds specified.

##### Creating Relationships Does Not Create Objects

声明关系和生命类的实例变量类似，针对有关系的两个 managed object，它们的创建并没有直接联系

> It is important to note that simply defining a relationship does not cause a destination object to be created when a new source object is created. In this respect, defining a relationship is akin to declaring an instance variable in a standard Objective-C class.

##### Inverse Relationships

关系都是双向的

> Most object relationships are inherently bidirectional. If a Department has a to-many relationship to the Employees who work in a Department, there is an inverse relationship from an Employee to the Department that is to-one. The major exception is a fetched property, which represents a weak one-way relationship—there is no relationship from the destination to the source. 

最好指定双向关系

> It is highly recommended that you model relationships in both directions, and specify the inverse relationships appropriately. Core Data uses this information to ensure the consistency of the object graph if a change is made

##### Relationship Delete Rules

删除 source object 时的删除规则

> A relationship's delete rule specifies what should happen if an attempt is made to delete the source object. 

删除规则说明

> Deny<br>
If there is at least one object at the relationship destination (employees), do not delete the source object (department).

> For example, if you want to remove a department, you must ensure that all the employees in that department are first transferred elsewhere (or fired!); otherwise, the department cannot be deleted.

> Nullify<br>
Remove the relationship between the objects but do not delete either object.

> This only makes sense if the department relationship for an employee is optional, or if you ensure that you set a new department for each of the employees before the next save operation.

> Cascade<br>
Delete the objects at the destination of the relationship when you delete the source.

> For example, if you delete a department, fire all the employees in that department at the same time.

> No Action<br>
Do nothing to the object at the destination of the relationship.

> For example, if you delete a department, leave all the employees as they are, even if they still believe they belong to that department.

尽量按业务要求使用前三种删除规则

> It should be clear that the first three of these rules are useful in different circumstances. For any given relationship, it is up to you to choose which is most appropriate, depending on the business logic. It is less obvious why the No Action rule might be of use, because if you use it, it is possible to leave the object graph in an inconsistent state (employees having a relationship to a deleted department).

如果使用 No Action，需要自己维护 object graph 一致性

> If you use the No Action rule, it is up to you to ensure that the consistency of the object graph is maintained. You are responsible for setting any inverse relationship to a meaningful value. This may be of benefit in a situation where you have a to-many relationship and there may be a large number of objects at the destination.

##### Manipulating Relationships and Object Graph Integrity

修改对象关系图时，需要注意的

> When you modify an object graph, it is important to maintain referential integrity. 

Core Data 做了很多事

> Core Data makes it easy for you to alter relationships between managed objects without causing referential integrity errors. 

relationship descriptions 对象

> Much of this behavior derives from the relationship descriptions specified in the managed object model.

只需要关注关系中的一端，其他事情 Core Data 帮你做

> When you need to change a relationship, Core Data takes care of the object graph consistency maintenance for you, so you need to change only one end of a relationship. This feature applies to to-one, to-many, and many-to-many relationships. 

如果没有 Core Data，你需要做很多工作来维护关系的正确性（一致性、完整性）

> Without the Core Data framework, you must write several lines of code to ensure that the consistency of the object graph is maintained. Moreover you must be familiar with the implementation of the Department class to know whether or not the inverse relationship should be set from the employee to the new department. This may change as the application evolves. 

使用 Core Data 只需要一行代码：

~~~swift
anEmployee.department = newDepartment
~~~

or

~~~swift
newDepartment.mutableSetValueForKey("employees").addObject(employee)
~~~

以上代码的作用：建立或销毁相关的关系

> By referencing the application’s managed object model, the framework automatically determines from the current state of the object graph which relationships must be established and which must be broken.

#### Many-to-Many Relationships

如何创建

> You define a many-to-many relationship using two to-many relationships. 

必须双向设置多对多关系

> You must define many-to-many relationships in both directions—that is, you must specify two relationships, each being the inverse of the other. You can’t just define a to-many relationship in one direction and try to use it as a many-to-many. If you do, you will end up with referential integrity problems.

*多对多*对反身关系也有效

> This relationship configuration works even for relationships where an entity has a relationship back to itself (often called reflexive relationships). For example, if an employee can have more than one manager (and a manager can have more than one direct report), then you can define a to-many relationship directReports from the Employee entity to itself that is the inverse of another to-many relationship, managers, again from the Employee entity back to itself. 

![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Art/reciprocalToMany_2x.png)

#### Modeling a Relationship Based on Its Semantics

基于语义对关系进行建模

> Consider the semantics of the relationship and how it should be modeled. 

以 “friends” 关系进行说明什么样的关系适合建模

> A common example of a relationship that is initially modeled as a many-to-many relationship that’s the inverse of itself is “friends”. 

你不能是你表姐的表姐，但是你可以是你朋友的朋友

> Although you are your cousin’s cousin whether the cousins like it or not, it’s not necessarily the case that you are your friend’s friend. For this sort of relationship, use an intermediate (join) entity.

对关系进行建模的好处

> An advantage of the intermediate entity is that you can also use it to add more information to the relationship. For example, a FriendInfo entity might include some indication of the strength of the friendship with a ranking attribute

举例说明

![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Art/friendsRelationship_2x_2x.png)

对上图的说明

> In this example, Person has two to-many relationships to FriendInfo: friends represents the source person’s friends（我的朋友）, and befriendedBy represents those who count the source as their friend（把我当朋友的人）. FriendInfo represents information about one friendship, in one direction（用一个实例表示单向关系）. A given instance notes who the source is, and one person they consider to be their friend. If the feeling is mutual, then there will be a corresponding instance where source and friend are swapped（若是双向的关系，则需要两个 source 和 friend 对调的实例）.

对例子中的关系对象的操作说明

> There are several other considerations when dealing with this sort of model:

> * To establish a friendship from one person to another, you have to create an instance of FriendInfo. If both people like each other, you have to create two instances of FriendInfo.

> * To break a friendship, you must delete the appropriate instance of FriendInfo.

> * The delete rule from Person to FriendInfo should be Cascade. That is, if a person is removed from the store, then the FriendInfo instance becomes invalid, so it must also be removed.

> * As a corollary, the relationships from FriendInfo to Person must not be optional—an instance of FriendInfo is invalid if the source or friend is null.

> * To find out who one person’s friends are, you have to aggregate all the friend destinations of the friends relationship, for example:

> 
~~~swift
let personsFriends = aPerson.valueForKeyPath("friends.friend")
~~~

> * To find out who considers a given person to be their friend, you have to aggregate all the source destinations of the befriendedBy relationship, for example:

> 
~~~swift
let befriendedByPerson = aPerson.valueForKeyPath("befriendedBy.source")
~~~

#### Cross-Store Relationships Not Supported

不支持夸库关系，用 fetched properties 实现相同需求

> Be careful not to create relationships from instances in one persistent store to instances in another persistent store, as this is not supported by Core Data. If you need to create a relationship between entities in different stores, you typically use fetched properties.

#### Weak Relationships (Fetched Properties)

弱的、单向的关系

> Fetched properties represent weak, one-way relationships. In the employees and departments domain, a fetched property of a department might be Recent Hires. Employees do not have an inverse to the Recent Hires relationship.

适合做跨库、松耦合、短暂临时分组关系建模

> In general, fetched properties are best suited to modeling cross-store relationships, loosely coupled relationships, and similar transient groupings.

与关系的不同之处

> * Rather than being a direct relationship, a fetched property's value is **calculated** using a fetch request. (The fetch request typically uses a predicate to constrain the result.)
> * A fetched property is represented by an **array** (NSArray), not a set (NSSet). The fetch request associated with the property can have a sort ordering, and thus the fetched property may be **ordered**.
> * A fetched property is evaluated **lazily**, and is subsequently **cached**.

![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Art/Fetched_Property_2x.png)

> In some respects you can think of a fetched property as being similar to a smart playlist, but with the important constraint that it is **not dynamic**. If objects in the destination entity are changed, you must **reevaluate** the fetched property to ensure it is up to date. You use `refreshObject:mergeChanges:` to manually refresh the properties—this causes the fetch request associated with this property to be executed again when the object fault is next fired.

在 predicate 中用到的两个参数

> You can use two special variables in the predicate of a fetched property, `$FETCH_SOURCE` and `$FETCHED_PROPERTY`. 

$FETCH_SOURCE

The source refers to the specific managed object that has this property, and you can create key paths that originate with that source, for example, `university.name LIKE [c] $FETCH_SOURCE.searchTerm`. 

$FETCHED_PROPERTY

> The `$FETCHED_PROPERTY` is the entity's fetched property description. The property description has a userInfo dictionary that you can populate with whatever key-value pairs you want. You can therefore change some expressions within a fetched property's predicate or any object to which that object is related.

这两个参数如何工作

> To understand how the variables work, consider a fetched property with a destination entity **Author** and a predicate of the form, **(university.name LIKE [c] $FETCH\_SOURCE.searchTerm) AND (favoriteColor LIKE [c] $FETCHED\_PROPERTY.userInfo.color)**. If the source object had an attribute searchTerm equal to Cambridge, and the fetched property had a user info dictionary with a key color and value Green, then the resulting predicate would be **(university.name LIKE [c] "Cambridge") AND (favoriteColor LIKE [c] "Green")**. The fetched property would match any Authors at Cambridge whose favorite color is green. If you changed the value of searchTerm in the source object to Durham, then the predicate would be **(university.name LIKE [c] "Durham") AND (favoriteColor LIKE [c] "Green")**.

fetched properties 的限制

> The most significant constraint for fetched properties is that you cannot use substitutions to change the structure of the predicate — for example, you cannot change a LIKE predicate to a compound predicate, nor can you change the operator (in this example, LIKE [c]).