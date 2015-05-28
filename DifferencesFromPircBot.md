PircBotX has a very different API from the original PircBot. Unfortunately this means changing from PircBot won't be a simple "replace pircbot.jar with pircbotx.jar". Conceptually however its still "start bot and override methods on`*` events", just done better.



# Why? #

PircBot was forked due to its many issues and lack of maintenance
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

# Major Changes #

  1. **[Better Event system](EventSystemExplained.md)** - PircBot's event system is a basic list of overridable methods with a couple of parameters. These parameters though only have some of the information you need and can't easily be passed to other methods. PircBotX on the other hand has a full Event-Listener system with event _objects_ instead. All the available information in a single place and can easily be passed around.
  1. **New [Channel](http://site.pircbotx.googlecode.com/hg-history/2.0/apidocs/org/pircbotx/Channel.html) and [User](http://site.pircbotx.googlecode.com/hg-history/2.0/apidocs/org/pircbotx/User.html) objects** - Features, advantages, and usages of these objects include
    * More Powerful User Class - Information such as hostmask, login, channels the user is in, channels the user is an operator in, even the number of hops to the user + much more is all available. Check the javadoc for more information
    * New Powerful Channel Class - You can now easily (and cheaply) get access to information such as the channel topic, who set the topic, mode, current users and their status in the channel, and more without having to awkwardly send the command and have a listener to handle response. All of this information is available with a simple method call without a round trip to the server
    * User-per-server instead of User-per-channel - In PircBot a new separate user was created for every user in every channel, regardless if they are in multiple channels. In PircBotX each user only gets created once.
  1. **Don't extend PircBotX class** - PircBot was centered around extending the PircBot class. In PircBotX this is not necessary and isn't recommended (although it is possible). Consider learning about PircBotX and writing utility classes before extending the main class.
  1. **Feature friendly!** - As this is still a relatively new project we are very happy to take feature requests. We do not suffer from the backwards compatibility and stagnation issues that plague PircBot.
  1. **Full support for Maven** - PircBot didn't provide any Maven support, making life difficult for Maven users and projects. PircBotX though is built entirely with Maven and is available in Maven central (see the downloading section on the Home Page)
  1. **Heavily changed method parameters** - Due to the new User and Channel classes described above they have replaced their String equivalents in most methods. Its imperative that you check existing code and javadoc for new method parameters
  1. **`final` and `private` modifiers removed from most methods and fields.** - Final and private modifiers hurt extensibility, code reuse, and usability in certain situations. This leaves only a few options: a) Submit a feature request and wait, b) Copy and paste the class, modify it, and use it instead (very bad!) or c) Give up and pick a different project. With the removal of these modifiers you are free to modify methods to fix any issues you might have. Note that the entire PircBotX project originally started because of the frustration caused by these modifiers, preventing being able to modify and access certain functionality.
  1. **Significantly more [Features](Features.md)** - Almost all of them are new features that are missing in PircBot

# Bug Fixes #

These are all issues in the original PircBot code that have been fixed in PircBotX

  1. **Missing Hostmasks causes malformed events** - This bug caused [Issue #9](https://code.google.com/p/pircbotx/issues/detail?id=#9) and is in the upstream code. Its mostly found in user mode changes and channel lists. As of March 1st, 2012 the [bug report](http://www.deaded.com/forum/viewtopic.php?showtopic=1628) on PircBot's forums hasn't received any response in 14 months.
  1. **Exception Swallowing** - PircBot swallowed (didn't throw or log, just did catch(Exception e){} ) exceptions in many methods making debugging very difficult since you have no idea why your code suddenly failed. This anti-pattern and bad practice has been removed from PircBotX. To ease the pain of unchecked exceptions, Listeners can throw any encountered Exception which can then be handled at the ListenerManager level

# Better Managed Project #

  1. **New package namespace** - Because PircBotX is a significant change from PircBot, the package namespace was changed from `org.jibble.pircbot` to `org.pircbotx`.
  1. **Set/Remove events condensed** - Events like DeOpEvent and OpEvent have been compacted into just OpEvent. Use isOp() or the applicable method to check for setting or removing of modes
  1. **[Public Source Code Repository](http://code.google.com/p/pircbotx/source/list)** - PircBot didn't provide any public source code repository which made it difficult to suggest patches, track updates, and use the latest development version. PircBotX on the other hand hosts all code in this public Google Code project. See [this help page](http://code.google.com/p/pircbotx/source/checkout) for more information
  1. **[Public Issue Tracker](http://code.google.com/p/pircbotx/issues/list)** - Find a problem? Instead of emailing the developer, notifying on a forum with miscellaneous discussion and support requests, or just fixing it in your own copy, report it in the Issue Tracker.
  1. **[Public Mailing List](http://groups.google.com/group/pircbotx)** - PircBotX provides a public mailing list for discussion, announcements, support, and anything else you want to talk about. **Note: Please do not email a developer directly, use the mailing list**