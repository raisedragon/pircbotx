**Warning:** This guide is still being written

2.0 is the next major version of PircBotX, changing the direction of PircBotX from "PircBot with some improvements" to "**A Modern Java IRC Library**".

Up until now PircBotX has been keeping roughly the same API and same way of doing things as PircBot did. Lately however this restriction has meant more work to do things you want in a less clear way. 2.0 aims to solve this with a significantly cleaner and better API. Read below for an explanation on why things changed and how you can move your bot to 2.0



# New Configuration class #

## Why was this created? ##

The PircBotX class originally had many configuration setters mixed in with unrelated methods like sendMessage or getUser. Confusingly, changing most of these after connect didn't do anything to the current connection; they only applied to the next one. This added a ton of methods to the already large method list and just wasn't a good way to handle configuration

In PircBotX 2.0 a new Configuration class was created to handle all configuration options in a single location. This has other advantages: You can now save a bot configuration in a file or a database. With the Configuration Builder, you can also have a template configuration with global options and then copy the template into other builders.

Several new configuration options were also added
  * Automatically join channels on connect with `addAutoJoinChannel`
  * Multiple DCC options (see DCC section for more information)
  * Automatically identify to nickserv with `setNickservPassword`
  * Use the ident server with `setUseIdentServer` (see IdentServer section for more information)
  * Use your own classes instead (IE custom Channel or User classes) with `setBotFactory`
  * WebIRC is explicitly enabled or disabled with `setWebIrcEnabled`
  * Change the default locale with `setLocale`
  * Change potential channel prefixes with `setChannelPrefixes`

## How will this change my code? ##

Configuration is passed to the PircBotX constructor instead of the `connect` method. You also obviously need to move where you configure your bot to the Configuration class

Old code
```
//Create a new bot
PircBotX bot = new PircBotX();

//Setup this bot
bot.setName("PircBotX"); //Set the nick of the bot. CHANGE IN YOUR CODE
bot.setLogin("LQ"); //login part of hostmask, eg name:login@host
bot.setVerbose(true); //Print everything, which is what you want to do 90% of the time
bot.setAutoNickChange(true); //Automatically change nick when the current one is in use
bot.setCapEnabled(true); //Enable CAP features

//This class is a listener, so add it to the bots known listeners
bot.getListenerManager().addListener(new PircBotXExample());

//bot.connect throws various exceptions for failures
try {
         //Connect to the freenode IRC network
         bot.connect("irc.freenode.net");
         //Join the official #pircbotx channel
         bot.joinChannel("#pircbotx");
} catch (Exception ex) {
         ex.printStackTrace();
}
```

New code
```
//Setup this bot
Configuration configuration = new Configuration.Builder()
	.setName("PircBotX") //Set the nick of the bot. CHANGE IN YOUR CODE
	.setLogin("LQ") //login part of hostmask, eg name:login@host
	.setAutoNickChange(true) //Automatically change nick when the current one is in use
	.setCapEnabled(true) //Enable CAP features
	.addListener(new PircBotXExample()) //This class is a listener, so add it to the bots known listeners
	.setServerHostname("irc.freenode.net")
	.addAutoJoinChannel("#pircbotx") //Join the official #pircbotx channel
	.buildConfiguration();
PircBotX bot = new PircBotX(configuration);

//Connect to server
try {
	bot.connect();
} catch (Exception ex) {
         ex.printStackTrace();
}
```

# Sending moved to Output classes #

## Why did this change? ##

Originally all sending was handled through the send`*` methods in the PircBotX class, each having multiple overloads to attempt to cover all possible options. This resulted in a huge method list and a awkward calling syntax

In 2.0 output has been moved to context-specific Output classes: OutputIRC, OutputUser, OutputChannel, OutputCAP, and lastly OutputRaw. Each class only contains send methods that would apply for that specific destination, resulting in a much cleaner API.

New methods
  * `channel.send().who()` - Ask for WHO list of channel
  * `channel.send().getMode()` - Ask for channel mode

Changed methods
  * All
  * Join a channel manually with `sendIrc().joinChannel("#channel")` (Note that Configuration.addAutoJoinChannel("#channel") will join a channel automatically on connect)

## How will this change my code? ##

Most users should of been using `event.respond(message)` as it covers most cases where you want to "respond" to an event (channel message responds with a channel message, private action responds with a private action, etc). Users who only rely on `respond(message)` are unaffected.

For other users, each Output object is meant to be called from a send() method, so the send prefix was dropped from the method names themselves. See the examples below for how older versions handled sending lines and 2.0 handles sending lines. As most users interact with the bot in a Listener, these examples use provided Event objects
```
//Tell a user hello
event.getBot().sendMessage(event.getUser(), "Hello!");
event.getUser().send().message("Hello!");

//Tell the channel we like pizza
event.getBot().sendAction(event.getChannel(), "Really likes pizza");
event.getChannel().send().action("Really likes pizza");

//Tell a user in the channel they are wrong
event.getBot().sendMessage(event.getChannel(), event.getUser(), "That is incorrect");
event.getChannel().send().message(event.getUser(), "That is incorrect");

//Start a channel list
event.getBot().listChannels();
event.getBot().sendIRC().listChannels();

//Change our nick to PircBotXUser2
event.getBot().changeNick("PircBotXUser2");
event.getBot().sendIRC().changeNick("PircBotXUser2");

//Send a raw line to the server immediately
event.getBot().sendRawLineNow("Some raw line");
event.getBot().sendRaw().rawLineNow("Some raw line");

//Join a channel (Note, Configuration.addAutoJoinChannel is most likely what your wanting to use)
event.getBot().joinChannel("#pircbotx");
event.getBot().sendIrc().joinChannel("#pircbotx");

//Part a channel
event.getBot().partChannel(event.getChannel(), "Goodbye");
event.getChannel().send().part("Goodbye");
```

# Better Connection Handling #

## Why did this change? ##

PircBotX's original connection handling was to create an InputThread and OutputThread in connect(), and on disconnect if reconnect was enabled it would try to reconnect again. This was awkward to deal with, especially exceptions in the IO threads (print to console) or reconnect (sent as an event) and with users who wanted to run all threads in a thread pool.

In 2.0 all IO threads have been eliminated. OutputThread has been replaced by a Conditional lock in OutputRaw and InputThread has been replaced by InputParser.

The active connection is now handled by the new `startBot()` method. `startBot()` will (if autoReconnect is enabled in configuration) continuously connect to the server until `stopBotReconnect()` is called or an exception is thrown during the connection.

This new process makes connection handling much easier
  1. You can run the bot in whatever thread you want
  1. Reconnecting should be significantly more stable
  1. Any exceptions from any future connection come from `startBot()`, allowing you to handle them in 1 place instead of during connect() and during reconnect
  1. Less places where an exception forces PircBotX to log to the console
  1. Fundamentally PircBotX's "core" doesn't worry about thread management, giving users much more control

## How will this change my code? ##

You will need to call `startBot()` instead of connect() to connect to the server, but remember that it blocks **until the bot disconnects from the server**. Note that `connect()` still exists do the actual connection, but it is now protected.

Reconnect is completely rewritten
  * `bot.reconnect()` was removed since `startBot()` will continually call `connect()`
  * Reconnect event was removed since its no longer necessary.
  * Like other options,  disabling (default) and enabling automatic reconnection has moved to `autoReconnect` in the configuraton class

# DCC Overhauled #

## Why did this change? ##

DCC has always been difficult to work with since its a multi-step process. The API wasn't much better, leaving a lot of guess work up to the user.

2.0 is finally overhauling DCC support with a significantly cleaner API in the new org.pircbotx.dcc package. The model has changed to "block until the user accepts or an Exception is thrown" which should significantly simplify DCC handling. The old DccChat and ReceiveChat classes have been split up into SendChat, ReceiveChat, SendFileTransfer, and ReceiveFileTransfer. These objects represent an active transfer that's already been accepted; no more wondering if the object you have is valid. Sending a DCC request has also been moved to the OutputUser class (see Output section for more information). Lastly, the IncommingFileTransferEvent and IncommingChatRequestEvent classes contain all the information about the request, instead of keeping them in a request field.

There are also several new features
  * **Passive DCC Support** (also known as reverse DCC)
    * Receiving a passive DCC request is handled automatically
    * Sending a passive DCC request by default can be enabled in Configuration with `dccPassiveRequest`
    * Individually enable passive DCC with the `passive` parameter in the send methods `dccChat` and `dccFile`
  * Chat
    * Send and received lines are now logged
    * The encoding specified in the configuration is used for the connection
  * File Transfer
    * Since a filename can have quotes or spaces, there is now a "safe" filename and a "raw" filename, which should fix problems from not knowing which one was valid

## How will this change my code? ##

Make sure your code is updated with the following concepts
  * The new dcc classes (SendChat, ReceiveChat, SendFileTransfer, and ReceiveFileTransfer) represent an _active_ transfer. If the request times out, user can't connect, or some other error happens you will receive an Exception instead
  * Sending or accepting a DCC request blocks until the transfer is active.

Following are some examples on how code has changed

```
/** Accept an incoming DCC chat **/
//Old
public void onIncomingChatRequest(IncomingChatRequestEvent event) throws Exception {
	DccChat chat = event.getChat();
	chat.accept();
	String line;
	while ((line = chat.readLine()) != null) {
		String response = "You said " + line;
		chat.sendLine(response);
		System.out.println("Sent line: " + response);
	}
}
//2.0
@Override
public void onIncomingChatRequest(IncomingChatRequestEvent event) throws Exception {
	ReceiveChat chat = event.accept();
	chat.sendLine("Hello incomming request!");
	String line;
	while ((line = chat.readLine()) != null)
		chat.sendLine("You said " + line);
	System.out.println("Chat ended");
}

/** Accept an incoming DCC file transfer. Add .txt to prevent executables **/
//Old
public void onIncomingFileTransfer(IncomingFileTransferEvent event) throws Exception {
	DccFileTransfer transfer = event.getTransfer();
	//If the filename had quotes this would throw an exception since "file".txt isn't a valid filename
	File file = File.createTempFile("pircbotx-dcc", event.getSafeFilename() + ".txt");
	transfer.receive(file, true);
}
//2.0
@Override
public void onIncomingFileTransfer(IncomingFileTransferEvent event) throws Exception {	
	File file = File.createTempFile("pircbotx-dcc", event.getSafeFilename() + ".txt");
	event.accept(file).transfer();
}

/** Chat with a user **/
//Old
DccChat chat = event.getBot().dccSendChatRequest(event.getUser(), 5000);
String line;
while ((line = chat.readLine()) != null) {
	String response = "You said " + line;
	chat.sendLine(response);
	System.out.println("Sent line: " + response);
}
//2.0
event.getUser().send().dccChat(true);
String line;
while ((line = chat.readLine()) != null)
	chat.sendLine("You said " + line);

/** Send file to a user **/
File file = new File("aFile.txt");
//Old
event.getBot().dccSendFile(file, event.getUser(), 120000);
//2.0
event.getUser().send().dccFile(file).transfer();
```

# New Dependencies on Several Libraries #

## Why did this change? ##

Ever since the original project's conception there have been no dependencies on any external library. However as PircBotX continues to evolve not using libraries has meant more work and potentially bugs.

2.0 uses the following libraries
  * [slf4j](http://www.slf4j.org/) - Modern logging facade framework to replace odd log() method and System.out.println. It allows users to use a logging framework of their own choice instead of using one dictated by us.
  * [Apache Commons Lang 3](https://commons.apache.org/proper/commons-lang/) - Extremely useful Java utility library to replace many redundant utility methods as well as access many more useful ones
  * [Google Guava](https://code.google.com/p/guava-libraries/) - Another useful utility library to provide features like better collections and easier method argument checking
  * [Apache Commons Codec](https://commons.apache.org/proper/commons-codec/) - Provides a tested and proven Base64 implementation. Technically this library is not needed if you do not use SASL CAP authentication

## How will this change my code? ##

We always recommend using a dependency manager like Maven which will handle downloading and updating the libraries automatically for you. However if you cannot, you'll have to download the jars from the projects website.

As slf4j only provides an abstract logging api, you still need to provide an actual logging implementation. If you do not have one already, we recommend logback due to its native integration with slf4j

# `MultiBotManager` Overhauled #

## Why did this change? ##

With big changes like the new Configuration class and the elimination of IO threads (see their respective sections for more information), MultiBotManager had to be mostly rewritten. Its goal however is still the same: Make handling multiple bots an easy, painless process.

MultiBotManager has a defined lifecycle now.
  1. When created, any added bots or configurations are queued
  1. When `start()` is called, all queued bots are connected. Any bots added after this point are automatically connected
  1. When `stop()` is called, `sendIRC().quitServer()` is called on all bots. No more bots can be added, the Manager is finished
    * An optional `stopAndWait()` method is provided to block until all bots have shutdown.

## How will this change my code? ##

All of the configuration setters and adding a bot methods have been simplified to 2 methods: `addBot(configuration)` and if you for some reason want to provide the bot yourself `addBot(bot)`

`connectAll()` was replaced by `start()` which connects all the bots in the thread pool. It also no longer throws any Exceptions, these are logged since they occur in other threads

`disconnectAll()` was replaced by `stop()` which sends tells all bots to `quitServer()` and shuts down the thread pool. If you want to wait for all the bots to shutdown, use `stopAndWait()`

# Standalone `IdentServer` #

## Why did this change? ##

IdentServer has been mostly untouched since PircBot. The issues were numerous
  * Only supported 1 bot. Confusingly, calling startIdentServer on another bot caused IOException's
  * All other exceptions were silently ignored, causing debug pains
  * No way to manage the created thread

Since fundamentally you can only have 1 IdentServer running at a time regardless of how many bots there are, IdentServer was rewritten as a standalone server. Bots must be told to use the running identserver with `setIdentServerEnabled` in the Configuration class

## How will this change my code? ##

As IdentServer is standalone you must start it yourself with IdentServer.startServer() . PircBotX must also be told to use the ident server with `setIdentServerEnabled` in Configuration. Everything else is handled automatically.

You can stop the IdentServer with IdentServer.stopServer() .