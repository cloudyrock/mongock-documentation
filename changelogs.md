# ChangeLogs

In simple words **ChangeLogs** are your migration classes. They contain **ChangeSets**, that are the methods that actual perform your migration. 

To tell Mongock to run your migration, you need:

1. Annotate your changeLog classes by **@ChangeLog**
2. Annotate your changeSet methods by **@ChangeSet**
3. Tell Mongock where your changeLog classes are by providing the changeLog scan package\(you can specify more than one\)

{% hint style="warning" %}
When using Spring, you must use **MongockTemplate,** instead of Spring MongoTemplate. MongockTemplate is just a decorator/wrapper providing exactly the same API than MongoTemplate, but ensuring your changes are correctly synchronised. 
{% endhint %}

{% hint style="info" %}
Please take a look the [Best practices]() section for design decisions.  
{% endhint %}

## @ChangeLog

Class with change sets must be annotated by **@ChangeLog.** As **t**here can be more than one changeLog , Mongock will use the **order** argument to decide how to sort them:

```java
@ChangeLog(order = "001")
public class DatabaseChangelog {
  //...
}
```

## @ChangeSet

Method annotated by **@ChangeSet** is taken and applied to the database. History of applied change sets is stored in a collection in your MongoDB.   
This collection is called **mongockChangeLog** by default. You can instruct Mongock to use another name by configuration. Please refer to [Driver](spring.md) page for more information.

The **@ChangeSet** parameters are:

| Parameter | Type | Default | Description |
| :--- | :--- | :---: | :--- |
| id | String | Mandatory unique | Name of a change set, **must be unique** for all change logs in a database |
| author | String | Mandatory | Author of a change set |
| order | String | null | String for sorting change sets in one changeLog. Sorting in alphabetical order, ascending. It can be a number, a date etc. |
| runAlways | Boolean | false | If true, changeSet will always be executed |
| systemVersion | String | "0" | Defines which SystemVersion the changeSet is linked to.  See [System Version]() for more information. |

### Defining ChangeSet methods

```java
@ChangeSet(order = "001", id = "changeWithoutArgs", author = "mongock")
public void changeWithoutArgs() {
   // method without arguments can do some non-db changes
}

@ChangeSet(order = "002", id = "changeWithMongoDatabbase", author = "mongock")
public void changeWithMongoDatabbase(MongoDatabase db) {
  // type: com.mongodb.client.MongoDatabase : original MongoDB driver v. 3.x, operations allowed by driver are possible
  // example: 
  MongoCollection<Document> mycollection = db.getCollection("mycollection");
  Document doc = new Document("testName", "example").append("test", "1");
  mycollection.insertOne(doc);
}

@ChangeSet(order = "005", id = "changeWithMongockTemplate", author = "mongock")
public void changeWithMongockTemplate(MongockTemplate mongockTemplate) {
  // type: com.github.cloudyrock.mongock.driver.mongodb.springdata.[v2 | v3].decorator.impl.MongockTemplate
  // You must use MongockTemplate instead of MongoTemplate. It's just a wrapper/decorator
  // so it provides exactly the same API. You won't miss anything
  // example:
  mongockTemplate.save(myEntity);
}

@ChangeSet(order = "006", id = "changeWithCustomBean", author = "mongock")
public void changeWithCustomBean(CustomBean myean) {
  // You can use custom beans like your Spring Data repositories, as long as 
  // they are interfaces
}
```

### System version

Method annotated by **@ChangeSet** have also the possibility to contain a systemVersion. While most of the case this won't be needed, it can be a useful feature from a consultancy perspective. The more descriptive scenario is when a software provider has several customers who he provides his software to. 

The clients may be using different versions of the software at the same time. So when he installs the product in a customer, the changeSets need to be applied depending on the product version. With this solution, he can tag every changeSet with his product version and will tell Mongock which version range to apply.

```java
@ChangeSet(order = "001",  systemVersion = "1", id = "changeToVersion1", author = "mongock")
public void someChange1(MongoDatabase db) {
}

@ChangeSet(order = "002", systemVersion = "1.1", id = "changeToVersion1.1", author = "mongock")
public void someChange2(MongoDatabase db) {
}

@ChangeSet(order = "003", systemVersion = "2.5.1" id = "changeToVersion2.5.1", author = "mongock")
public void someChange3(MongoDatabase db) {
}

@ChangeSet(order = "004", systemVersion = "2.5.5", id = "changeToVersion2.5.5", author = "mongock")
public void someChange5(MongoDatabase db) {
}

@ChangeSet(order = "005", systemVersion = "2.6", id = "changeToVersion2.6", author = "mongock")
public void someChange6(MongoDatabase db) {
}
```

When specifying versions you are able to upgrade to specific versions:

{% tabs %}
{% tab title="Properties" %}
```yaml
mongock:
  start-system-version: 1
  end-system-version: 2.5.5
```
{% endtab %}

{% tab title="Builder" %}
```java
  mongockBuilder
      //...
      .setStartSystemVersion("1")
      .setEndSystemVersion("2.5.5")
      //... 
```
{% endtab %}
{% endtabs %}

This example will execute changeSet methods 1, 2 and 3, because the specified systemVersion in the changeSet should be greater equals the **startSystemVersion** and lower than **endSystemVersion**. In other words, startSystemVersion is inclusive, while endSystemVersion is not.

## Best practices

There are too many ways to approach the changeLog's design, we list the points we think best:

#### ChangeLog per migration, instead of ChangeLog per class domain

Imagine you have two domain classes, Client and Product. Developers tend to split their changeLogs in ClientChangeLog and ProductChangeLog and then, they update those ChangeLogs in every migration. While this can make sense, specially when the developer is familiar with domain driven design, it's normally not the best approach. In this context, the domain loses a bit its importance and it's given to the migration. In this terms is more important to separate which "package" of migration has been executed in....

