# Quick start

Mongock is set up in a few steps, which will be explained briefly in this section and more detailed in the rest of the documentation:

1. **Importing artifact dependencies\(maven, gradle, etc.\)**
   1. _Mongock's bom_
   2. _Mongock's runner_
   3. _Mongock's driver_
   4. _MongoDB driver or specific underlying  library_
2. **Configure and run Mongock: Annotation approach or traditional builder approach**

All the steps in group 1\(importing artifact dependencies\)  are common independently of the approach you use\(annotation or builder\).

{% hint style="success" %}
These are the unique steps you need to run Mongock. The rest of the documentation is about explaining them and how to extend  the "configuration step" to use/tune the different features.
{% endhint %}

## Importing artifact dependencies 

Both approaches share the first 4 steps. All of them related to your pom file.

1- **Import the last version of Mongock's bom to your pom file**

```markup
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.github.cloudyrock.mongock</groupId>
            <artifactId>mongock-bom</artifactId>
            <version>LAST_RELASE_VERSION_4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

2- **Import runner dependency.** For more information, check the [runner compatibility table](standalone.md#runners-types-and-compatibility-table).

{% tabs %}
{% tab title="Spring 5" %}
```markup
<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongock-spring-v5</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Mongock Standalone" %}
```markup
<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>>mongock-standalone</artifactId>
</dependency>
```
{% endtab %}
{% endtabs %}

3- **Import driver dependency.** For more information, check the [driver compatibility table](spring.md#driver-types-and-compatibility-table).

{% tabs %}
{% tab title="Spring data 3" %}
```markup
<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongodb-springdata-v3-driver</artifactId>
</dependency>
```
{% endtab %}

{% tab title="Spring data 2" %}
```markup
<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongodb-springdata-v2-driver</artifactId>
</dependency>
```
{% endtab %}

{% tab title="MongoDB sync driver 4" %}
```markup
<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongodb-sync-v4-driver</artifactId>
</dependency>
```
{% endtab %}

{% tab title="MongoDB driver 3" %}
```markup
<dependency>
    <groupId>com.github.cloudyrock.mongock</groupId>
    <artifactId>mongodb-v3-driver</artifactId>
</dependency>
```
{% endtab %}
{% endtabs %}

3-  **Import your MongoDB and Spring Data dependencies**. In order to avoid transitive dependency issues, Mongock doesn't import any MongoDB or Spring Data library. So you need to provide them. For more information, check the [driver compatibility table](spring.md#driver-types-and-compatibility-table).

{% tabs %}
{% tab title="Spring data 3" %}
```markup
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>${mongodb.driver-sync.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>${mongodb.spring-data.v3.version}</version>
</dependency>
```
{% endtab %}

{% tab title="Spring data 2" %}
```markup
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>${mongodb.java-driver}</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>${mongodb.spring-data.v2.version}</version>
</dependency>
```
{% endtab %}

{% tab title="MongoDB sync driver 4" %}
```markup
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>${mongodb.driver-sync.version}</version>
</dependency>
```
{% endtab %}

{% tab title="MongoDB driver 3" %}
```markup
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>${mongodb.java-driver}</version>
</dependency>
```
{% endtab %}
{% endtabs %}

## Building and running Mongock: Annotation vs Builder

Once you have successfully imported the necessary dependencies, as we briefly explained in [Main concepts](main-concepts.md#builder), there are two ways you can build and run Mongock. In most cases, when using Spring framework, the most easy and convenient way is the annotation approach. However, sometimes you are not using Spring or you need more control over your Mongock bean. In that case you should opt for the traditional builder approach.

### Annotation approach

When opting for annotation approach, all your configuration will be set in your properties file. However, for minimal configuration you only need to  specify a changeLog package,  the rest of the configuration will use default values.

Then, you only need to tell Spring to use Mongock by annotating your Spring boot application with  **@EnableMongock** annotation 

1- **Add your changeLog package path to your property file**. Minimal configuration requires at least one changeLog package\(it's an array, so you can add more than one\), but anything you can configure manually with the builder, you can do it as well with properties. However note that Mongock provides default values. Worthy noticing the lock, which is now enabled by default, unlike older versions where you need to explicitly enable it, due to backward compatibility.

```yaml
spring:
  mongock:
    change-logs-scan-package:
      - com.github.cloudyrock.mongock.integrationtests.spring5.springdata3.changelogs.client.initializer
```

2. **Annotate your SpringBootApplication with** _**@EnableMongock**_

```java
@EnableMongock
@SpringBootApplication
public class App {
  public static void main(String[] args) {
    new SpringApplicationBuilder().sources(App.class).run(args);
  }
}
```

### **Traditional** builder approach

While the annotation approach is more convenient, the traditional builder approach is useful when you need more control over your bean creation, or inevitable if you are not using Spring.

In this case you need to build the Mongock Instance yourself and , in case of Spring, provide the Mongock Bean.

{% tabs %}
{% tab title="Spring ApplicationRunner" %}
```java
public MongockApplicationRunner mongockApplicationRunner(ApplicationContext springContext,
                                                         MongoTemplate mongoTemplate) {
    return MongockSpring5.builder()
        .setDriver(SpringDataMongo3Driver.withDefaultLock(mongoTemplate))
        .addChangeLogsScanPackage("your_changeLog_package_path")
        .setSpringContext(springContext)
        .buildApplicationRunner();
  }
```
{% endtab %}

{% tab title="Spring InitializingBean" %}
```java
public MongockApplicationRunner mongockApplicationRunner(ApplicationContext springContext,
                                                         MongoTemplate mongoTemplate) {
    return MongockSpring5.builder()
        .setDriver(SpringDataMongo3Driver.withDefaultLock(mongoTemplate))
        .addChangeLogsScanPackage("your_changeLog_package_path")
        .setSpringContext(springContext)
        .buildInitializingBeanRunner();
  }

```
{% endtab %}

{% tab title="Standalone" %}
```java
MongoClient mongoClient = MongoClients.create("MongoDB connection string");
MongockStandalone.builder()
        .setDriver(MongoSync4Driver.withDefaultLock(mongoCient.getDatabase("your_db"))
        .addChangeLogsScanPackage("your_changeLog_package_path")
        .buildRunner()
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Note that for standalone example we have used MongoSync4Driver as it's a more common scenario. Please take a look to our driver [version compatibility table](spring.md#driver-types-and-compatibility-table).
{% endhint %}
