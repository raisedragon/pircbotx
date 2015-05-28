This is an attempt to figure out what is still under the GPL from PircBot for potential rewrite. See [Issue #63](https://code.google.com/p/pircbotx/issues/detail?id=#63)

Updated for PircBotX 2.0.1

  * [BAD](BAD.md) Colors
    * Very few changes since fork
    * Unsure how to make my own version (wrapper method?)
  * [MAYBE](MAYBE.md) InputThread/InputParser
    * Tweaked over the years but core handleLine() and further code is basically the same
    * Proposed rewrites are less performant, more convoluted, and generally use existing code
  * [MAYBE](MAYBE.md) ReplyConstants
    * Arguable that this is copied from the IRC RFC, and therefore uncopyrightable.
    * Is this even used outside of PircBotX? It does make searching for what code handles server code XXX more complicated. Could be removed
  * [MAYBE](MAYBE.md) PircBotX
    * Javadocs are mostly copied
    * Most of the line processing code is copied but modified
    * Anything with a hand written Getter is copied
    * connect() is very similar, just modified
  * [GOOD](GOOD.md) User
    * This class bares almost no resemblance to original User, so should be good
    * Some javadoc needs to be rewritten
  * [GOOD](GOOD.md) Everything DCC (DccManager, DccFileTransfer/`*`FileTransfer, DccChat/`*`Chat)
    * Significantly rewritten in 2.0
    * Some Javadocs needs to be rewritten
    * Remaining socket code is the standard way to read and write from sockets in Java
  * [GOOD](GOOD.md) IdentServer
    * Basically a brand new implementation in 2.0
    * Class javadoc needs to be rewritten
  * [GOOD](GOOD.md) Exception classes (NickAlreadyInUseException, IrcException)
    * Merged NickAlreadyInUseException into IrcException
    * IrcException uses an enum and is used in more places
  * [GOOD](GOOD.md) OutputThread
    * Eliminated in 2.0
    * Remaining pieces are standard socket programming in Java