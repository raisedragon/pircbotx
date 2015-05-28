New project description: Modern Up to Date Java IRC Library

<font color='red'><b><code>*</code>NEW</b><code>*</code>**</font> [April 7th, 2013 Version 1.9 Available!](#Downloading.md)**

<a href='Hidden comment: 
'></a>

# Introduction #

PircBotX is a simple, easy to use, Java IRC Framework forked from the popular PircBot framework, bringing many new up-to-date features and bug fixes in an official alternative distribution.

[Features](Features.md) include
  * Fully configurable Listener system with [over 50](http://site.pircbotx.googlecode.com/hg-history/1.9/apidocs/index.html) supported IRC events
  * Extensive User and Channel info classes
  * Native SSL support using SSLSocket's or STARTTLS
  * Normal DCC and Reverse/Passive Chat and Filesharing
  * CTCP VERSION, ACTION, PING, TIME, and FINGER support
  * IPv6 support
  * Support for op, voice, halfop, superops, and owner modes
  * [CAP negotiation](http://ircv3.atheme.org/) with native support for SASL, TLS, and away-notify
  * [WEBIRC](http://wiki.mibbit.com/index.php/WebIRC) support
  * Built in [Ident server](https://en.wikipedia.org/wiki/Ident_protocol)

This project was created due to the many [limitations of PircBot](DifferencesFromPircBot.md), including
  * Attempts to work on Java **1.1**, so newer and better Java features like Collections are not used
  * The project has essentially stagnated, with little to no new features added in years
  * Very limited "event" system
  * Lack of any available information on the server or the channel except the channel topic
  * Lack of important user info like hostmasks
  * No support for HalfOp, SuperOp, and Owner statuses available on some IRC servers
  * Exceptions are hidden in many places making debugging extremely difficult
  * Any attempt to integrate into a higher-level framework takes significantly more work than should be required
  * Methods or fields that might be useful to override or use are hindered by `private` and `final` modifiers
  * Due to heavy usage of `private` and `final` modifiers, any modification requires modification of the original source code instead of overriding.
  * Most documentation relies solely on JavaDocs and self-documentation instead of an actual full guide explaining everything
  * Naming convention is not adhered to throughout the entire project, mainly when dealing with users
  * Abuse of the [God Object](http://en.wikipedia.org/wiki/God_object) pattern
  * Doesn't scale very well. The larger the bot the more unmaintainable it is.

The major goals of this project include
  * Taking advantage of the many features available in Java **1.5**
  * Cleaning up the code to meet coding standards
  * Have a very open API that welcomes extension instead of preventing it.
  * Replace the method based event system with a [new event object based event system](EventSystemExplained.md)
  * Ensure that the stability in the original PircBot project continues into this one
  * Make it easy to integrate into larger frameworks

# Downloading #

**Current Version: 1.9**

Important links
| [ChangeLog of important changes](ChangeLog#1.9_-_April_7th,_2013.md) - Please read! | **[JavaDoc Available](http://site.pircbotx.googlecode.com/hg-history/1.9/apidocs/index.html)** **Updated** | [Maven Site Available](http://site.pircbotx.googlecode.com/hg-history/1.9/index.html) |
|:------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------|
| [Major differences from PircBot](DifferencesFromPircBot.md)                         | [HowTo Guide](HowTo.md)                                                                                    | [In-Depth explanation of Event System](EventSystemExplained.md)                       |
| [Features](Features.md)                                                             | [Development Version](DevVersion.md)                                                                       | **NEW** [DCC Explained](DccExplained.md)                                              |

## Maven ##

This project uses [Apache Maven](http://maven.apache.org/) for project management. If you are going to be using Maven you can add this to your pom.xml

```
<dependencies>
        <dependency>
                <groupId>org.pircbotx</groupId>
                <artifactId>pircbotx</artifactId>
                <version>1.9</version>
        </dependency>
</dependencies>
```

## JARs ##

If you are not using Maven and want the Jar, you can download it from [this link](http://code.google.com/p/pircbotx/downloads/detail?name=pircbotx-1.9.jar).

You will also need the most recent version [Slf4j](http://www.slf4j.org/), [Commons Lang 3, [https://code.google.com/p/guava-libraries/ Google Guava](https://commons.apache.org/proper/commons-lang/), and [Apache Commons Codec](https://commons.apache.org/proper/commons-codec/)

# License #

This project is licensed under GNU GPL v3 to be compatible with the PircBot license.

It is assumed that commercial users can buy the commercial license of PircBot which grants "modification of the Product's source-code and incorporation of the modified source-code into your software"

The PircBot developer has ignored multiple emails asking for a less restrictive license and clarification of the commercial license. Users can show support by respectfully asking him directly at ![http://img35.imageshack.us/img35/8227/ly5.gif](http://img35.imageshack.us/img35/8227/ly5.gif). More up to date information is available at in [Issue #63](https://code.google.com/p/pircbotx/issues/detail?id=#63)