[Current Version: 2.0.1](Downloads.md)
<br>See <a href='MigrationGuide2.md'>Migration Guide to 2.x</a> and <a href='ChangeLog#2.0.1_-_December_3rd,_2013.md'>ChangeLog</a> for more information<br>
<br>
<a href='Hidden comment: 
<font color="red">**NEW*

Unknown end tag for </font>


[Downloads December 3rd, 2013 Version 2.0.1 Released!]
'></a><br>
<br>
<br>
<br>
<b>PircBotX</b> is a powerful and flexible Java IRC library forked from the popular PircBot framework, bringing many new up-to-date features and bug fixes in an official alternative distribution.<br>
<ul><li>Robust, multi-threaded Event-Listener system with <a href='http://site.pircbotx.googlecode.com/hg-history/latest/apidocs/org/pircbotx/hooks/events/package-summary.html'>over 50</a> supported IRC events<br>
</li><li>Powerful Channel/User Model<br>
</li><li>Native SSL support using SSLSocket or STARTTLS<br>
</li><li>Standard and reverse/passive DCC Chat and Filesharing<br>
</li><li>CTCP VERSION, ACTION, PING, TIME, and FINGER support<br>
</li><li>IPv6 support<br>
</li><li>Support for op, voice, halfop, superops, and owner modes<br>
</li><li><a href='Features#3:_CAP_Support.md'>IRCv3 CAP negotiation</a> with native support for SASL, TLS, and away-notify<br>
</li><li><a href='Features#WEBIRC_Authentication.md'>WEBIRC</a> support<br>
</li><li>Built in <a href='Features#Ident_Server.md'>Ident server</a></li></ul>

<b>Checkout the <a href='WikiHome.md'>Wiki</a> for tutorials and documentation</b>

<h2>PircBotX in 3 Steps</h2>
A brief getting started guide<br>
<br>
<ol><li><a href='Downloads.md'>Download PircBotX</a>
</li><li>Create and execute the following class:<br>
<pre><code>import org.pircbotx.Configuration;<br>
import org.pircbotx.PircBotX;<br>
import org.pircbotx.hooks.ListenerAdapter;<br>
import org.pircbotx.hooks.types.GenericMessageEvent;<br>
<br>
public class MyListener extends ListenerAdapter {<br>
        @Override<br>
        public void onGenericMessage(GenericMessageEvent event) {<br>
                //When someone says ?helloworld respond with "Hello World"<br>
                if (event.getMessage().startsWith("?helloworld"))<br>
                        event.respond("Hello world!");<br>
        }<br>
<br>
        public static void main(String[] args) throws Exception {<br>
                //Configure what we want our bot to do<br>
                Configuration configuration = new Configuration.Builder()<br>
                                .setName("PircBotXUser") //Set the nick of the bot. CHANGE IN YOUR CODE<br>
                                .setServerHostname("irc.freenode.net") //Join the freenode network<br>
                                .addAutoJoinChannel("#pircbotx") //Join the official #pircbotx channel<br>
                                .addListener(new MyListener()) //Add our listener that will be called on Events<br>
                                .buildConfiguration();<br>
<br>
                //Create our bot with the configuration<br>
                PircBotX bot = new PircBotX(configuration);<br>
                //Connect to the server<br>
                bot.startBot();<br>
        }<br>
}<br>
</code></pre>
</li><li>Join the #pircbotx channel on irc.freenode.net and send <code>?helloworld</code> . Your bot will respond with <code>Hello world!</code> Since its a GenericMessageEvent, it will also respond when private messaged. Congratulations, you just wrote your first bot!</li></ol>

PircBotX can do so much more! <a href='Documentation.md'>Read the docs for more information</a>


<a href='Hidden comment: 
== #pircbotx Channel Demo Bot ==
Also on the #pircbotx channel on irc.freenode.net is the bot [http://code.google.com/p/lq-projects/wiki/TheLQPircBotXExplained TheLQ-PircBotX]. It is an example implementation of a bot that supports multiple servers, command system,and [http://thelq-pircbotx.thelq.cloudbees.net/ a jetty-powered webserver with detailed status information].

[http://code.google.com/p/lq-projects/wiki/TheLQPircBotXExplained See this wiki page for more information]

'></a><br>
<br>
<h2>License</h2>

This project is licensed under GNU GPL v3 to be compatible with the PircBot license.<br>
<br>
It is assumed that commercial users can buy the commercial license of PircBot which grants "modification of the Product's source-code and incorporation of the modified source-code into your software"<br>
<br>
The PircBot developer has ignored multiple emails asking for a less restrictive license and clarification of the commercial license. Users can show support by respectfully asking him directly at <img src='http://site.pircbotx.googlecode.com/hg/0static/pircbot-email.gif' />. More up to date information is available at in <a href='https://code.google.com/p/pircbotx/issues/detail?id=#63'>Issue #63</a>