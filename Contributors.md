# How to help PircBotX #

PircBotX is closing in on version 2.1 but still has a long list of tasks before it can be released



## Fix Reported bugs ##
The number one priority is [fix reported bugs in the Issues tab](https://code.google.com/p/pircbotx/issues/list)

## TODO List ##
I maintain a [public PircBotX TODO list at Trello](https://trello.com/b/PXQkOUhO/pircbotx). This is lower priority but has ideas on the future of PircBotX. There is also some low hanging fruit I notice but don’t have time to fix at that moment.

## General TODO List ##
  1. _**Unit Tests!**_ - Lots of the events and even the original PircBot code itself are untested. This is leading to lots of bugs as things don't work as they seem like they would.
    1. The most important thing now is Event testing which is primarily taking place in [org.pircbotx.PircBotXProcessingTest](http://code.google.com/p/pircbotx/source/browse/src/test/java/org/pircbotx/PircBotXProcessingTest.java).
    1. Next should be important parts of the API. This consists of primarily PircBotX, User, and Channel classes.
    1. Lastly would be anything left over. There are many more classes that should have testing but are of a lower priority
  1. **Documentation** - There are still multiple places that are completely undocumented. Explanations, example code, wiki pages, and any other types of documentation would immensely help out
  1. **Code cleanup/review/refactoring** - _Only after unit testing is mostly complete_ Most of the origional code is unchanged simply because it works. However there are many parts that just don't look right, use outdated code, or follow bad practices. A code review, code cleanup, or general refactoring would help to clean the codebase.

## Code Style Reports ##
The PircBotX is run through multiple source code analyzers to find potential bugs and documentation issues. Reports are run after every build
  * **Difficulty: easy to medium** [Findbugs Report](http://site.pircbotx.googlecode.com/hg/findbugs.html) – Analyzes source code with [FindBugs](http://findbugs.sourceforge.net/) for potential bugs. Most items are easy fixes, however some warnings should be ignored.
  * **Difficulty: medium** [PMD CPD Report](http://site.pircbotx.googlecode.com/hg/cpd.html) – Analyzes source code with [PMD Copy/Paste Detector](http://pmd.sourceforge.net/pmd-4.3.0/cpd.html) for duplicate code.
  * **Difficulty: easy** [PMD Report](http://site.pircbotx.googlecode.com/hg/pmd.html) – Analyzes source code with [PMD](http://pmd.sourceforge.net/) for source cleanliness and potential bugs
  * **Difficulty: easy** [Checkstyle Report](http://site.pircbotx.googlecode.com/hg/checkstyle.html) – Analyzes source code with [Checkstyle](http://checkstyle.sourceforge.net/) for source code formatting, cleanliness, and Javadoc problems. There is a lot of documentation issues that would make the Javadoc look better and be more useful.

# Source code #
## Style Guidelines ##
Source code is run through the Netbeans formatter (although any IDE’s formatter will work for submitting patches) with the following rules
  * Tabs instead of spaces
  * Braces are eliminated whenever possible
  * No extra new line after class header

## Environment ##
  * Project Management is done through [Apache Maven](http://maven.apache.org). Due to substantial changes in Maven 3.x, you will need Maven 3.x or above.
  * PircBotX uses [Project Lombok](http://projectlombok.org/) to generate Java boilerplate. If you open the PircBotX and get lots of “getXXX not found” errors visit [Lombok’s downloads](http://projectlombok.org/download.html) page for instructions on how to configure your IDE
  * While PircBotX does work on Java 1.5, developing or extending it requires Java 1.6 due to use of Lombok

## Download Source ##
Clone Mercurial repository:
> hg clone https://code.google.com/p/pircbotx/
or download a ZIP archive of the latest version by visiting the [Changes page](https://code.google.com/p/pircbotx/source/list) and clicking "Download ZIP".

Occasionally there will be some large breaking changes that aren't stable enough for the `default`/master branch and therefore happens in the `dev` branch.

# Source Code Overview #

## /pom.xml ##

Maven configuration

## /src/etc ##

Additional build configuration files for license headers and scanners like findbugs, pmd, and checkstyle.

## /src/main/java ##

Actual main source code

**org.pircbotx.`*`** - Core classes (utilities, parsers, managers, etc)

**org.pircbotx.cap.`*`** - CAP negotiation interface + built in implementations.

**org.pircbotx.dcc.`*`** - DCC parser + frontends for chats and file transfers.

**org.pircbotx.exception.`*`** - Exception classes

**org.pircbotx.hooks.`*`** - Everything dealing with the event listener system goes under this package. Contains listener and event interfaces, core hook implementations, and utilities.

**org.pircbotx.hooks.events.`*`** - All supported events

**org.pircbotx.hooks.managers.`*`** - Dispatches events to listeners. Contains manager interface + core implementations and exception handler interface + core implantation.

**org.pircbotx.hooks.types.`*`** - Contains interfaces for events that are similar. Eg channel message, private message, actions, and notice events all contain a message (GenericMessageEvent) from a user (GenericUserEvent).

**org.pircbotx.output.`*`** - Classes that generates lines to send to the IRC server, sorted by destination.

**org.pircbotx.snapshot.`*`** - Immutable versions of info classes for snapshots.

## /src/test/java ##

Test classes, mirroring organization in the main folder just named `<MainClass>Test`. Other test classes should be self explanatory by name.


## /target (after building) ##

This folder will appear when you've built PircBotX. If successful, it should contain

  * **`pircbotx-<version>`.jar** - Final built library jar

  * **pircbotx-unified.jar** - For users who don't want to download PircBotX's dependencies this uber-jar is built containing all dependencies inside of it.

Also if you use the complete\_build profile you'll see

  * **delombok/** - "De-lombok"ed sources, meaning after lombok has transformed them. This is useful for tools that cannot understand lombok annotations (eg javadoc).