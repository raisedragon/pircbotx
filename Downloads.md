**Current Version: 2.0.1**

# Maven #

Using Maven is recommended as PircBotX and its dependencies will be downloaded automatically.

Add the following to the `<dependencies>` section in your pom.xml. [Click here for Ivy, Gradle, and other configs](http://search.maven.org/#artifactdetails|org.pircbotx|pircbotx|2.0.1|jar).

```
<dependency>
        <groupId>org.pircbotx</groupId>
        <artifactId>pircbotx</artifactId>
        <version>2.0.1</version>
</dependency>
```

See Logging section below

# JARs #

[Download PircBotX 2.0.1 JAR](http://repo1.maven.org/maven2/org/pircbotx/pircbotx/2.0.1/pircbotx-2.0.1.jar)

PircBotX depends on the following libraries. You need to download the latest version from their site:
  * [Apache Commons Lang 3](https://commons.apache.org/proper/commons-lang/)
  * [Google Guava](https://code.google.com/p/guava-libraries/)
  * [Apache Commons Codec](https://commons.apache.org/proper/commons-codec/) - Only required for SASL authentication support.
  * [Slf4j](http://www.slf4j.org/) - See the [Logging](#Logging.md) section below

# Logging #

PircBotX uses Slf4j, an abstract logging framework allowing users to plug in their own desired logging framework. You must provide one in your project.

Beginners are recommended to use [LogBack](http://logback.qos.ch/) or slf4j-simple-x.x.x.jar

Please read the [Slf4j site](http://www.slf4j.org/) for more details and how to use other frameworks like log4j or Apache Commons