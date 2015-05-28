If you are migrating from PircBot, its important to read [DifferencesFromPircBot](DifferencesFromPircBot.md) as well



# Configuring and Starting Your Bot #

## Configuration Basics ##

PircBotX 2.0 introduced the powerful immutable Configuration class. To create these, use [Configuration.Builder](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/Configuration.Builder.html).

For example, to connect to the Freenode IRC network, changing the nick when its already in use and automatically joining the #pircbotx channel
```
Configuration<PircBotX> config = new Configuration.Builder()
    .setName("MyExampleBot") //Nick of the bot. CHANGE IN YOUR CODE
    .setLogin("PircBotXUser") //Login part of hostmask, eg name:login@host
    .setAutoNickChange(true) //Automatically change nick when the current one is in use
    .setServer("irc.freenode.net") //The server were connecting to
    .addAutoJoinChannel("#pircbotx") //Join #pircbotx channel on connect
    .buildConfiguration(); //Create an immutable configuration from this builder

PircBotX myBot = new PircBotX(config);
```

For a full list of options, see the [Configuration.Builder Javadoc](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/Configuration.Builder.html)

### Templates ###

Configuration.Builder easily allows you to create a template configuration that is shared by all bots.

Lets say you want all bots to share the same listeners, handle nick already in use automatically, same login, and same nick

```
Configuration.Builder<PircBotX> templateConfig = new Configuration.Builder()
    .setName("MyMultiServerBot")
    .setLogin("PircBotXUser")
    .setAutoNickChange(true)

PircBotX freenodeBot = new PircBotX(templateConfig.buildForServer("irc.freenode.net"));
PircBotX mibbitBot = new PircBotX(templateConfig.buildForServer("irc.mibbit.net"));
```

### Copying ###

Configuration.Builder also allows you to copy any existing Configuration or Configuration.Builder (.buildForServer() is just a pretty wrapper around this)

```
//Copy an existing configuration from a running bot
Configuration.Builder<PircBotX> newConfig = new Configuration.Builder(someBot.getConfiguration());

//Add to an existing builder
Configuration.Builder<PircBotX> newBuilder = new Configuration.Builder(otherBuilder);
```

## Connection Options ##

PircBotX defaults will work on normal IRC servers on port 6667 but also supports more advanced setups

### Standard IRC ###

The following will connect to irc.freenode.net on port 6667. Usually this is all that you will need.
```
configBuilder.addServer("irc.freenode.net")
```

### SSL Support - SSL-only port ###

To connect to an IRC SSL server at startup, change the port to the servers SSL port and provide a SSLSocketFactory.
```
configBuilder.addServer("irc.freenode.net", 7000)
    .setSocketFactory(SSLSocketFactory.getDefault())
    //Other options...
```

For convenience in non-standard situations, PircBotX provides [UtilSSLSocketFactory](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/UtilSSLSocketFactory.html)

To accept any certificate,
```
configBuilder.setServer("irc.freenode.net", 7000)
    .setSocketFactory(new UtilSSLSocketFactory().disableDiffieHellman())
    //Other options...
```

To disable Difie Hellman key exchange due to JDK [bug #6521495](https://code.google.com/p/pircbotx/issues/detail?id=6521495) which can't accept prime sizes above 1024 bits (see [Issue #34](https://code.google.com/p/pircbotx/issues/detail?id=#34)),
```
configBuilder.setServer("irc.freenode.net", 7000)
    .setSocketFactory(new UtilSSLSocketFactory().trustAllCertificates())
    //Other options...
```

### SSL Support - IRCv3 tls capability ###

PircBotX supports [IRCv3 tls](http://ircv3.atheme.org/extensions/tls-3.1). Note that as of 2.0.1 your `NICK` and `USER` lines are sent in plaintext, before it learns about the servers tls capability.

```
configBuilder.addServer("irc.mozilla.org")
    .addCapHandler(new TLSCapHandler(SSLSocketFactory.getDefault(), true))
    //Other options
```

### Ident Server ###

PircBotX contains a build in [ident server](http://en.wikipedia.org/wiki/Ident_protocol) to respond to servers that require ident identification.

Since the server is separate, you must start the server manually before it can be used. Then you just need to enable ident in the configuration:

```
//Before anything else
IdentServer.startServer();
		
Configuration<PircBotX> botConfig = new Configuration.Builder()
	.setIdentServerEnabled(true)
	//...other configuration...
	.buildConfiguration();
```

## IPv6 Support ##

PircBotX fully supports IPv6 IRC servers, DCC connections, and ident. The required code to connect to IPv6 is identical to IPv4.

## WEBIRC Authentication ##

PircBotX supports [WEBIRC](http://wiki.mibbit.com/index.php/WebIRC) authentication. The Mibbit wiki explains:

> IRC is a client to server protocol. Multiple clients connect to one server, called the IRC Server, and the server handles communications between the clients. Each client announces who it is through its hostmask. When a user connects through a client that connects directly to the server, the hostmask shows their nick and their IP, though the IP is usually obfuscated before sending to other users (default mode +x). When a user connects through an indirect method, such as through Mibbit.com, the indirect client sends its own IP instead of sending the normal user unless WebIRC is implemented by both the client and the server. For IRC, IPs are extremely useful for making sure users do not disbehave. Without WebIRC implemented, the users of the indirect client will be banned more often for the actions of others or might not be allowed to join the server because too many users of that IP are connected at once.

To fully configure WEBIRC in PircBotX
```
configBuilder.setWebIrcEnabled(true)
    .setWebIrcAddress(clientsAddress) //The address of the user that's connecting
    .setWebIrcHostname(clientsHostname) //The resolved hostname of the user that's connecting
    .setWebIrcUsername("serverUsername") //Agreed upon WEBIRC username given by IRCd operator
    .setWebIrcPassword("serverPassword"); //Agreed upon WEBIRC password given by IRCd operator
```

## Starting PircBotX ##

```
Configuration config = ...

PircBotX bot = new PircBotX(config);
bot.startBot();
```

Create a new PircBotX object and call startBot()
  * This method blocks until disconnecting from the IRC server or, if configured setAutoReconnect(true), stopBot() is called.
  * If connecting fails with an Exception it is thrown immediately (and also generates a ConnectAttemptFailedEvent)
  * (2.1 Placeholder) To ignore exceptions, call...

Your bot should connect to the IRC server and join the configured channels. Congratulations, you've just wrote and ran your first bot!

### Manage Bots on Multiple Servers ###

Connecting to a single server is straightforward. Connecting to multiple servers is slightly more complex due to threading.

To make the process simple, PircBotX provides [MultiBotManager](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/MultiBotManager.html). It neatly handles creating, starting, managing, and stopping any number of bots.

Expanding on the Configuration Templates code above, this will start 2 bots on Freenode and Mibbit
```
Configuration.Builder<PircBotX> templateConfig = new Configuration.Builder()
        .setName("MyMultiServerBot")
        .setLogin("PircBotXUser")
        .setAutoNickChange(true);

MultiBotManager<PircBotX> manager = new MultiBotManager();
manager.addBot(templateConfig.buildForServer("irc.freenode.net"));
manager.addBot(templateConfig.buildForServer("irc.mibbit.net"));
manager.start();
```


# Listener/Event System #

PircBotX has an extensive and flexible Event and Listener system with over 50 different events from various mode changes (op, deop and other statuses, channel key, moderated, channel limit, etc) to DCC information to all user actions (message, private message, action, notice, etc).

[PircBotXExample.java](https://code.google.com/p/pircbotx/source/browse/src/test/java/org/pircbotx/impl/PircBotXExample.java?name=latest) contains an example class that implements 2 listeners

## Basics ##

Various [Events](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/hooks/Event.html) are dispatched to [Listeners](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/hooks/Listener.html) by the [ListenerManager](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/hooks/managers/ListenerManager.html).
  * Every line from the server is parsed by PircBotX and generates an event that contains what the line said. Sometimes multiple events are dispatched, e.g. `MODE +o user` generates an OpEvent and a ModeEvent
  * There are also generic events which are interfaces for multiple Events, e.g. GenericMessageEvent which ActionEvent, MessageEvent, NoticeEvent, and PrivateMessageEvent implement
  * Since the Listener interface only has one method for the abstract Event class, there is the easy utility class [ListenerAdapter](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/ListenerAdapter.html) which provides individual overridable methods for each event.
  * The default ListenerManager is [ThreadedListenerManager](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/managers/ThreadedListenerManager.html)
  * Because events are executed co-currently, in certain cases you might have to add synchronization code to prevent thread races. If you are unfamiliar with Java Thread synchronization practices, its recommended you read the [Synchronization](http://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html) and [Liveness](http://docs.oracle.com/javase/tutorial/essential/concurrency/liveness.html) sections of the Oracle Java Tutorials.

[Full list of Events](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/package-summary.html)

[Full list of Generic Events](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/types/package-summary.html)

## ListenerManagers Explained ##

In general, events are dispatched on multiple threads to all the added listeners. This covers most use cases and prevents 1 listeners from
deadlocking the entire bot.

### ThreadedListenerManager ###

This is the default ListenerManager that PircBotX uses. Using a cached thread pool (which you can replace with your own), it creates Threads to run each event and each Listener. IE if 2 events come in at roughly the same time and you have 3 Listeners, it creates 6 threads to run everything at the same time.

### BackgroundListenerManager ###

BackgroundListenerManager expands ThreadedListenerManager by running certain specified Listener's their own single background Thread. For example, logging listeners generally need to process events one at a time to log them in order and not on top of each other.

Usage is very similar
```
BackgroundListenerManager myListenerManager = new BackgroundListenerManager();
myListenerManager.addListener(myLoggingListener, true);
configBuilder.setListenerManager(myListenerManager)
		.addListener(myStandardListener);
```

### Writing your own ###

If none of these ListenerManagers fit your use cases you can write your own by extending the abstract [ListenerManager](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/managers/ListenerManager.html) class.

To make things easier its recommended to extend ThreadedListenerManager since it already implements most features that a standard ListenerManager has. It also has been designed to be extendable with several overridable callbacks. Look at the [source code](https://code.google.com/p/pircbotx/source/browse/src/main/java/org/pircbotx/hooks/managers/ThreadedListenerManager.java?name=2.0) to see if it provides a good starting point

# User and Channel objects #

[User](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/User.html) and [Channel](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/Channel.html) objects are mapped together by [UserChannelDao](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/UserChannelDao.html).

Most relevant information can be obtained from the Channel or User that comes with an Event. If you need more information, you can query the UserChannelDao directly, eg
```
event.getBot().getUserChannelDao().containsChannel("#somechannel")
```

See the above JavaDoc pages for User-Channel data that is available.

# DCC Chat and File Sharing #

DCC is a very powerful tool in IRC allowing direct communication with other clients without going through the IRC server. It allows for chat as well as sending files.

## Connectivity Notes ##

DCC works by having users directly connect to each other. That means that for users to directly connect to your bot, your bot needs to be accessible.

IPv4 NAT makes this process more complex (IPv6 users can skip this section). If your bot is running behind a NAT (which most home networks are) then it has a "private" IP address, an address that nobody else can connect to. PircBotX needs your public IP address. Unfortunately PircBotX cannot do this itself, you must provide the address and set it using [setDccInetAddress() in Configuration](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/Configuration.Builder.html#setDccLocalAddress(java.net.InetAddress)). Note that if the server your bot is running from already has a public IP address, this step is not necessary.

Next you need to decide which ports to use. By default PircBotX picks a random free port. If you only wish to use a set amount of ports, put them in the[DCC ports list in Configuration](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/Configuration.Builder.html#setDccPorts%28java.util.List%29). Unless your server has no firewall and is publicly accessible on all ports (not recommended for security reasons), you will need to set a specific range of ports so you can configure your router if your behind a NAT and firewall. See below.

Lastly, you need to configure your firewall to allow connections from those ports and, if your behind a NAT, setup port forwarding in your router. Consult the manual or Google if you do not know how to do this

## DCC Chat ##

DCC Chat can be thought of as a raw private message without any interference from the IRC server. Its raw simply because no processing is done on the line in PircBotX; once a line is received it is directly handed to whatever is requesting it.

DCC Chat can be useful in situations where private messaging just won't work due to various reasons. For example, one use is to create a cross server "party line", where people DCC into the bot and can have their own "channel", regardless of which server they are on. It can also be used to pass large amounts of text that would be considered spam or be truncated by standard IRC servers.

### Accepting Incoming Chat Requests ###

When a user requests to have a DCC Chat with PircBotX, an [IncomingChatRequestEvent](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/hooks/events/IncomingChatRequestEvent.html) is dispatched. You must [accept()](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/hooks/events/IncomingChatRequestEvent.html#accept%28%29) the request to get the ReceiveChat object. This object is how you will be communicating with the user over DCC.  Following is some example code that accepts and responds to a user's DCC chat.

```
public class DccIncomingChatExample extends ListenerAdapter {
	@Override
	public void onIncomingChatRequest(IncomingChatRequestEvent event) throws Exception {
		//Accept the request. This actually opens the connection. Remember that 
		//the user may have a time limit on how long you have to accept the request
		ReceiveChat chat = event.accept();

		//Basic read loop. This is similar to reading a stream
		String line;
		//Keep reading lines until the line is null, signaling the end of the connection
		while ((line = chat.readLine()) != null) {
			//Just send back what they said
			String response = "You said " + line;
			chat.sendLine(response);
			System.out.println("Sent line: " + response);
		}
	}
}
```

### Creating Outgoing Chat Requests ###

When PircBotX wants to create a DCC chat with another user, you must send a request with [user.send().dccChat()](http://site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/output/OutputUser.html#dccChat%28%29).

Following is some example code on how to use this. Note that here its triggered when a user says `?chat` in a channel.

```
public class DccOutgoingChatExample extends ListenerAdapter {
	@Override
	public void onMessage(MessageEvent event) throws Exception {
		//Only trigger when the user says ?chat
		if (!event.getMessage().startsWith("?chat"))
			return;
		
		//Attempt to get a DCC chat with the user
		SendChat chat = event.getUser().send().dccChat();
		
		//Were now connected to the user
		//In this example do the same thing as the IncommingChatRequest example
		String line;
		while ((line = chat.readLine()) != null) {
			String response = "You said " + line;
			chat.sendLine(response);
			System.out.println("Sent line: " + response);
		}
	}
}
```

## DCC Filesharing ##

DCC Filesharing is another useful feature of DCC for some bots. It allows PircBotX to send and receive files directly without any intervention by the IRC server. It can be useful as a cheap way to distribute files to users or to have users submit files to your bot.

### Warnings ###

  1. DCC Filesharing is much more dangerous than chat simply because your letting another user access your hard drive. Please be careful
  1. Some IRC servers have policies against file sharing bots. Be sure to check your IRC server's rules to prevent your bot from being banned
  1. Treat accepted files cautiously. Users may attempt to upload malware or massive files to fill up your server's hard drive. Verify all accepted files. Failure to do so may cause your server to get compromised.

### Accepting Incoming Files ###

When a user is attempting to send PircBotX a file, a [IncomingFileTransferEvent](http://site.pircbotx.googlecode.com/hg-history/2.0/apidocs/org/pircbotx/hooks/events/IncomingFileTransferEvent.html) is dispatched. You simply need to give a file for PircBotX to write to.

The following example will accept _any_ file and write it to the temp directory. Note that this method blocks until the file is finished writing. From another thread you can get status information like progress, size, transfer rate, etc from [site.pircbotx.googlecode.com/hg-history/2.0.1/apidocs/org/pircbotx/dcc/ReceiveFileTransfer.html ReceiveFileTransfer].

```
public class DccOutgoingChatExample extends ListenerAdapter {
	@Override
	public void onIncomingFileTransfer(IncomingFileTransferEvent event) throws Exception {
		//Generate a file prefix
		String prefix = "pircbotxFile" + System.currentTimeMillis() + "-";
		//File suffix is the original filename plus .txt to prevent executables
		String suffix = event.getSafeFilename() + ".txt";
		//Create this file in the temp directory
		File file = File.createTempFile(prefix, suffix);
		
		//Receive the file from the user
		event.accept(file).transfer();
	}
}
```

Some important notes
  1. Watch out for executable files -  Its highly recommended to block files that end with `.exe` or `.sh`. There are very few reasons why your bot should accept executable files. On linux, make sure uploaded don't have the executable permission
  1. You can put files anywhere, not just the temp directory. For example you can put them in a `/upload` folder in your bot's folder.
  1. Watch out for duplicate file names - Sometimes you may get filenames that already exist in your directory. Its therefore recommended to add some kind of prefix or suffix, eg a file ID that's incremented each time a file is uploaded

### Sending Files ###

To send a file, you must send the The only information you need is the file to send and the target user. Simply call [PircBotX.dccSendFile()](http://site.pircbotx.googlecode.com/hg-history/2.0/apidocs/org/pircbotx/PircBotX.html#dccSendFile%28java.io.File,%20org.pircbotx.User,%20int%29).

Following is an example that sends a file when the user says `?sendFile` in the channel

```
public class DccOutgoingChatExample extends ListenerAdapter {
	@Override
	public void onMessage(MessageEvent event) throws Exception {
		//Only trigger when the user says ?sendFile
		if (event.getMessage().startsWith("?sendFile"))
			//Send the file output.txt
			event.getUser().send().dccFile(new File("C:/output.txt")).transfer();
	}
}
```

Some important notes
  1. Watch out for attacks if the user selects a file by filename - For example if the user says `?sendFile output.txt` to get the `output.txt` file, be sure the user doesn't say `?sendfile ../../somefile.txt` to get the `somefile.txt` file 2 directories up from your upload folder. Either strip out everything but the filename or do some checks on the file making sure its coming from a directory you expect it to come from
  1. Be careful distributing uploaded files - Even if you don't execute user submitted files (preventing executing viruses), storing and sharing viruses isn't much better. Make sure the files your sharing are files you really want to share.

# Support for Operators, Voice, HalfOps, SuperOps and Owners #

PircBotX fully supports [Operator (+o)](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/OpEvent.html), [Voice (+v)](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/VoiceEvent.html), [HalfOp (+h)](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/HalfOpEvent.html), [SuperOp (+a)](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/SuperOpEvent.html), and [Owner (+q)](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/OwnerEvent.html) user modes.

Note that the latter 3 (Halfop, Superop, and Owner) are non-standard and might not be supported on every IRC server. On some server the mode letter could mean something entirely different.

# CAP Support #

CAP is part of the [IRCv3 extension](http://ircv3.atheme.org) to the IRC protocol that allows you to selectively enable certain features at connect.

CAP is handled by the CapHandler interface which are added to the configuration with `configBuilder.addCapHandler()`.

By default PircBotX will attempt to enable the following CAP features
  * [multi-prefix](http://ircv3.atheme.org/extensions/multi-prefix-3.1)
  * [away-notify](http://ircv3.atheme.org/extensions/away-notify-3.1)

## SASL Authentication ##

SASL authentication allows NickServ authentication before the server accepts your connection. For example, [Freenode](http://freenode.net/sasl/) requires SASL authentication for all tor users due to abuse. This allows them to reject users who aren't registered with nickserv before they have a chance to join a channel and send messages

To authenticate against a server
```
configBuilder.addCapHandler(new SASLCapHandler("myUser", "myPassword"));
```

If authentication fails or the server doesn't support SASL a CAPException will be thrown. If you do not care about these cases, you can turn off throwing exceptions with the other constructor `new SASLCapHandler("yourUsername", "yourPassword", true)`

## Other CAP Features ##

To enable most other CAP features you generally just need to tell the server you want to use them with EnableCapHandler.

```
configBuilder.addCapHandler(new EnableCapHandler("aCapFeature"));
```

If the server doesn't support the requested CAP feature, a CAPException is thrown. Again, if you do not care if the server supports it you can use the other constructor `new EnableCapHandler("aCapFeature", true)`



# Notable Features #

## Easy Multi-Threading Support ##

A common issue with PircBot stems from the lack of multithreading. People write bots that use [Thread.sleep()](http://download.oracle.com/javase/1.5.0/docs/api/java/lang/Thread.html#sleep%28long%29) to wait a few seconds for something else to happen only to realize that they've frozen the entire bot or that the bot simply doesn't work. This and other multithreading issues making it difficult to write more complex bots or to handle lots of load.

By default all Listeners run in their own separate threads. This means that you can run Thread.sleep(), doLongOperation(), and doLongExpensiveOperation() as much as you want without affecting other Listeners or normal bot operation.

## Easily Handle Multi-Line Situations ##

Lets say that you want to have a conversation with a user or want to get multiple lines from someone or something in one batch (NickServ help for example). In PircBot there isn't a clear, easy way to accomplish this (create a field that holds the content, onMessage adds to it, somehow figures out that its finished + figuring out how to prevent other unrelated onMessage from adding)

PircBotX 1.9 introduces a new class called [WaitForQueue](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/WaitForQueue.html) to solve this problem, which queues up received events for later processing.

Following is a simple listener to demonstrate this (This is also in the [PircBotXExample class](http://code.google.com/p/pircbotx/source/browse/src/main/java/org/pircbotx/impl/PircBotXExample.java?name=1.9)):
```
public void onMessage(MessageEvent event) throws Exception {
	//If this isn't a waittest, ignore
	//This way to handle commands is useful for listers that only listen for one command
	if (!event.getMessage().startsWith("?waitTest start"))
		return;

	//WaitTest has started
	event.respond("Started...");
	WaitForQueue queue = new WaitForQueue(event.getBot());
	//Infinate loop since we might recieve messages that aren't WaitTest's.
	while (true) {
		//Use the waitFor() method to wait for a MessageEvent.
		//This will block (wait) until a message event comes in, ignoring
		//everything else
		MessageEvent currentEvent = queue.waitFor(MessageEvent.class);
		//Check if this message is the "ping" command
		if (currentEvent.getMessage().startsWith("?waitTest ping"))
			event.respond("pong");
		//Check if this message is the "end" command
		else if (currentEvent.getMessage().startsWith("?waitTest end")) {
			event.respond("Stopping");
			queue.close();
			//Very important that we end the infinate loop or else the test
			//will continue forever!
			return;
		}
	}
}
```

While this is only an example it can be extended to other senarios as well. You can, for example, parse output from someone or something (eg Nickserv help) very easily by just adding to a variable. The possibilities are endless

## Current Bot information in Slf4j's MDC ##

Mainly useful for loggers, information about the current bot is stored in Slf4j's MDC. This can be useful in debugging situations or even to clean up logs by separating by server, port, and bot id
```
int botId = Integer.valueOf(MDC.get("pircbotx.id"));
String botServer = MDC.get("pircbotx.server");
int botPort = Integer.valueOf(MDC.get("pircbotx.port"));
```

## Framework Friendly ##
With the Listener and Event system you now have the power to easily and cleanly integrate into a higher level framework. You can wrap functionality of events and even do most of your work by just implementing the [ListenerManager interface](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/managers/ListenerManager.html) or by extending one of the existing ListenerManagers.

## Support for Maven ##
PircBot didn't provide any Maven support, making life difficult for Maven users and projects. PircBotX though is built entirely with Maven and is available in Maven central (see the downloading section on the Home Page)

Using Maven has the added benefit of automatically getting the latest and greatest version by using SNAPSHOT builds. See [DevVersion](DevVersion.md) for more information.

## Simple event.respond() ##
There are multiple ways to send messages to a channel
```
bot.sendMessage(channel, "Some Message"); //Sends "Some Message" to channel"
bot.sendMessage(channel, user, "Some Message"); //Sends "User: Some Message" to channel
//Don't do this, completely unnecessary
bot.sendRawLine("PRIVMSG #channel :Some Message");
```

However an even easier way is provided by PircBotX: [event.respond()](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/Event.html#respond%28java.lang.String%29).

Consider the following Listener
```
public void onMessage(MessageEvent event) {
	if (event.getMessage().equals(".time"))
		bot.sendMessage(event.getChannel(), event.getUser(), "The current time is " + new Date());
}
```

The [bot.sendMessage()](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/PircBotX.html#sendMessage%28org.pircbotx.Channel,%20org.pircbotx.User,%20java.lang.String%29) line looks very ugly. And it can get uglier with different parameters, different ways to get a user, and longer messages.

The fix? Simply replace that line with this
```
event.respond("The current time is " + new Date());
```

Here [MessageEvent](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/MessageEvent.html) already knows where most people are going to send a message when they receive a MessageEvent: To the channel that the message came from in "Username: Message" format. The respond() method prvoides a simple quick shortcut to that long line. And when the event doesn't send it to where you want the message to go (perhaps you want to private message a user for using bad language in the channel), you can call the longer version instead

## Unified Exception Handling ##
Exception handling is something that needs to be treated very specially. When PircBot encounters a problem, it just prints to the console without giving much room for handling. When PircBotX encounters a problem, it gives you a chance to catch and handle the Exception _before_ printing it to the console (the default action, which is easily changeable)

By extending an existing [ListenerManager](http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/managers/ListenerManager.html) or creating your own you can capture all the Exceptions that are thrown by your Listeners. Then you queue them up to be sent via a PM to you, an email, or some other system. You can also respond to the user telling them that the bot isn't working right now. This allows you to get and handle Exceptions immediately instead of discovering them when going over your logs after many user complaints.

# PircBotX Lifecycle #

(This section is on PircBotX internals and can be skipped)

This is a rough overview of the lifecycle as of 2.0.1
  1. IdentServer
    * Optional IDENT complaint server. If ident is enabled in the configuration Ident.startServer() must be called
  1. PircBotX.startBot()
    1. Continuusly calls connect if autoReconnect is true. If autoReconnect is false, call once
  1. PircBotX.connect()
    1. Verify and setup state
    1. Query DNS for all IP records for the server hostname
    1. Attempt to connect to the first IP. If it fails, dispatch a ConnectAttemptFailedEvent and try the next IP
    1. If all IPs fail stop execution and throw an IOException
    1. If were connected, dispatch a SocketConnectEvent
    1. Adds ident entry if configured
    1. Send client information to server: NICK, USER, PASS, WEBIRC
    1. called startLineProcessing()
  1. PircBotX.startLineProcessing()
    1. Runs all the following in an infinate loop. When it breaks, calls shutdown()
    1. Listen for lines from the IRC server
    1. Sometimes when waiting for lines the socket times out. Sind a PING request to see if the connecting is still alive
    1. call InpurParser
  1. InputParser
    * The "core" of PircBotX that parses raw data sent from the IRC server into nice, easy to handle events
    * In broad terms, IRC lines fall under
      * User commands - NICK, JOIN, PART, QUIT, etc
      * Server information - MOTD, server stats and options. Also esults of WHOIS, WHO, MODE, etc
    * A broad overview of the IRC protocol and how PircBotX interacts with the IRC server
      1. On connect immediately send NICK, USER, CAP LS, etc
      1. Server will respond with its looking up our IP and (usually) attempt to connect to our IdentServer
      1. If the server supports IRCv3 CAP it will respond with enabled capabilities. PircBotX will call any added CapHandler's which talk to the server on their own. When all CapHandler's report being done, sends CAP END
      1. Server starts sending us 001-005 reponses with name, server header and info, and protocol information. At 001 PircBotX considers itself fully connected to the IRC server and dispatches a ConnectEvent. Any autoJoinChannel commands are send
      1. Server sends various server statistics, MOTD, and other information
      1. Server sends us our usermode
      1. Server registers our Join commands we sent earlier. PircBotX sends WHO and MODE command to insert into PircBotX
      1. Server sends channel statistics, WHO list, mode, topic, etc
      1. We have now fully joined a channel(s). At this point we've made the client's precense known on the network. Users start sending messages, dcc requests (DCC CHAT, DCC FILE), ctcp requests ("/me likes PircBotX"), and other IRC commands
      1. Eventually, we have to quit the server with QUIT, be kicked out with ERROR.
    * Once a line is fully understood the relevant Event is dispatched to the ListenerManager
  1. DccHandler
    * An implementation of the DCC protocol. Due to the protocols complexity, it is an independant package
    * The DCC protocol itself is split into 2 parts, and has 2 ways of connecting: us to the user (standard), or the user to us (passive/reverse)
      * Standard DCC File
        * User sends DCC SEND request with filename, size, their ip address and open port
        * Dispatch IncommingFileTransferEvent
        * When accepted, reply with
      * Passive DCC File
        * User sends DCC SEND request with filename, size, and 0 for the port, indicating passive request
        * Dispatch IncommingFileTransferEvent
        * When accepted, reply with DCC SEND with our ip and open port the user connects to
        * Wait for user to connect
        * User sends file contents, PircBotX writes output to a file and closes connection when finishe
      * ...
    * ...
  1. ListenerManager
    * Recieves an Event and calls all added Listeners
    * Very important to be Multi-threaded. Some internal listeners rely on being able to block until they get the event their looking for
  1. Listener
    * Recieved an Event and does user's work
    * Usually use ListenerAdapter for its nicer API
  1. PircBotX.shutdown()
    1. Called anytime we disconnect from the server
    1. Closes socket, removes IdentEntry if configured
    1. Resets internal state
    1. Dispatches DisconnectEvent