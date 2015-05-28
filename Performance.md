# Introduction #

PircBotX defaults are written for the majority of users. However some users may have specialized use cases with very high load. This page lists different performance benchmarks and tips to help your bot.

**Note: This page is old statistics with an old version of PircBotX on a really old computer. Will be updated in the fture**



# Performance Bottlenecks and Fixes #

These tips are important regardless of weather your doing them for performance or not.

**Note:** Just because these are listed as issues does not mean they are issues for your bot. Fixes are risky or difficult to implement. Do not do any of the listed fixes unless they are issues for your bot.

## Heavy Multithreading ##

In PircBotX, each listener gets its thread. This allows a listener to block waiting for a response without preventing other listeners from executing.

There are four issues with this
  * A high number of listeners means a high number of threads for each event. 50 listeners means 10-20 threads are used, as some lines (IE mode changes) send multiple events.
  * Very high load means very high number of threads. With jmeter-irc testing with 500 users close to 900 threads were created and used. Issues of this include
    * You can hit the maximum number of threads your OS gives to a process, maximum number of threads allowed in the OS, or Java's max thread limit which is based on a number of factors.
    * If you have an expensive listener then hundreds of the expensive listener running at once will just slow down things for everyone. Remember that during expensive operations you can't just throw threads at the problem. The same concept applies here

The number of threads per incoming line from the server can be calculated with this simple formula.

Number of Threads = Number of listeners **number of events created**

Eg

10 listeners **2 events for mode changes = 20 threads.**

### Fix 1 ###

Use a fixed thread pool with a fixed number of threads. There are several disadvantages and risks though to do this:
  * Listeners that use any multithreaded functions of PircBotX such as `waitFor` and `cycle` may run into deadlock issues. This is because they require two threads: one to have your listener wait in, and another to sort through incoming events. This means that if you have a thread pool size of 10 and there are 10 listeners using waitFor, no other listeners will be able to run and your bot will do nothing.
  * If there is expensive commands (grabbing a website page) right next to inexpensive commands (responding with hello world), then performance of the inexpensive command will appear much slower as it has to wait for the expensive command to complete

For this reason its only recommended to do this as a last resort. If you are then its also recommended to use a fairly high fixed thread pool to give your bot room to grow.

### Fix 2 ###

In specialized cases another option to write a custom ListenerManager that uses  two thread pools: One for listeners that preform quick and inexpensive operations, another for listeners that use blocking methods or perform expensive operations.

Implementation of this could be that by default all listeners are considered expensive (this way a new expensive listener won't block everything else) and only listeners that you add to a Set are considered inexpensive. The inexpensive listeners could run in a fixed threadpool of 1 or 2 threads while the expensive and blocking listeners run in an unlimited cached thread pool.

This is the best solution but the most difficult to write and manage.

## Listener Adapter uses reflection ##

To ease coding and ensure that all ListenerAdapter methods work reflection is used to call the onEvent methods. Under very heavy load the overhead of reflection may be too much.

### Fix ###

Do not use ListenerAdapter, use the older method of listening for events: Implementing the Event class and onEvent method, seeing if the event is the instance of the Event class you want, casting to that Event class, and working from there. However there is a disadvantage: You loose the ease of use of ListenerAdapter. ListenerAdapter provides all the necessary methods, setup, and automation.

For this reason its only recommended to do this when there will be very high load but inexpensive listeners. This fix would remove the overhead of rapidly calling a listener method