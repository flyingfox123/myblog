In this article we are going to address the common misuse of exceptions, more specifically those times when programmers fail to correctly propagate exceptions. Along the way, and most of this article, we will talk about the differences between runtime and checked exceptions and the ways in which it is appropriate to use them.

#### Runtime exceptions

There are two main differences between checked, or “normal,” exceptions, and runtime exceptions:

- Runtime exceptions don’t have to be mentioned in a method’s signature, and the compiler doesn’t warn about them.

- The compiler doesn’t require that runtime exceptions are caught.
This means that when a runtime exception is thrown it has the potential to propagate to the JVM without any prior warning, thus crashing the application. This makes runtime exceptions bad for managing errors that are recoverable, and great for failing the application for errors that are irrecoverable such as defective code.

#### Checked exceptions

Checked exceptions are different from runtime exceptions in that:
- Checked exceptions have to be mentioned in a method’s signature.
- Checked exceptions have to be caught, or the code will not compile. (Exception handling is forced by the specification and compiler.)

This means that checked exceptions never propagate up to the JVM and cannot crash your application unless you have deliberately allowed it by including them in your main method’s signature. This makes checked exceptions great for managing errors that are recoverable, and bad for errors that are irrecoverable. Who would want to keep catching exceptions that they can do absolutely nothing about? (Answer: nobody.)

#### The meaning of “recovery” from errors
“Recovery” means different things to different people (and situations, and applications).
Imagine that we are trying to connect to a server and the server is not responding. It’s possible to recover from the resulting exception by connecting to a different server, given that it has the same capabilities of the server to which we originally tried to connect.
This will achieve the original goal, thus we have recovered from the error.
This is not exactly what recovery means in this context — if it's possible to make such recovery as was mentioned in the illustration, then by all means you should do it.
However, recovery could also be displaying an alert dialog to the user that describes the incident, or perhaps sending an email to an administrator, or even simply logging the error to a log file. All of these options qualify as ‘recovery’ – taking a valid and known course of action in the event of an exception.

#### Using the correct exception type
With this information about the nature of exceptions and a workable definition of “recovery” in mind, the de facto standards in industry regarding exception handling make sense, and have evidently been practiced in the JVM and the Java runtime library itself:
- If the cause of the error is because the code is incorrect, throw a runtime exception.
- If the cause of the error is because of state while the code is correct, throw a checked exception.
-
The reason for this is that if the code is correct, the matter is very likely to be recoverable.
Examples include situations where you try to connect to a server without an internet connection — there is no need to crash the app. A gentle way to deal with the error is to display an error dialog that explains what happened, allowing the user to fix their connection, given a clear enough message.
If the error is in the code, and the program itself is defective, then writing a recovery path is irrelevant — how can you recover from a problem that you don’t even know exists yet? Or if you do know what the problem is, then why write a recovery path at all instead of fixing the problem?

#### Runtime exception examples
The following is an error for which a runtime exception is appropriate:
float nan = 1 / 0;
This will throw a division by zero exception. It is appropriate because the only means of fixing this issue is to modify the code, it is not dependent on any external state.
Here’s another example, a portion of HashMap‘s constructor:

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    // more irrelevant code
}
```

In the case presented above it is also appropriate to throw runtime exceptions, because it is not logically sound to construct a hash map with negative capacity, or a load factor that is not a positive number. This error is not due to something that was transmitted over the network, the state of a file or the disk, user input, or any other external state — it’s because a calculation is wrong, or the flow is inappropriate in that it permitted these values, or the programmer is insane. Either way — it’s the code that has to be fixed.

#### Checked exception example
The following is a rather common example of “exception handling,” often written by programmers who think that they’re following Spring‘s example:

```java
public Data dataAccessCode(){
    try {
        // ..some code that throws SQLException
    } catch(SQLException ex) {
        throw new RuntimeException(ex);
    }
}
```

Honestly, the frustration of a person who would take part in such an abomination is understandable. What can they possibly do in that method to deal with an SQL exception? It’s an exception in the database, there are no means of “recovery,” and this scope is probably incapable of accessing the UI to display an error dialog. Some more sophisticated evil-doers solve this by doing something of this sort:

```java
public Data dataAccessCode() {
    try {
        // ..some code that throws SQLException
    } catch(SQLException ex) {
        // TODO: add internationalization?
        UIManagerCommanderSorcerer.inflictErrorDialog(
          "We don't know what you're trying to do, but uhh, can't access data. Sorry.", ex);
    }
}
```

This does perform a certain effort at recovery, however it may not always be the correct recovery that is appropriate for the grander scheme, nor is it necessary evil. The correct way to solve this is to simply not handle the exception in this scope, and propagate the exception:

```java
public Data dataAccessCode() throws SQLException {
    // ..some code that throws SQLException
}
```

This way, the code is not even “uglified” and it allows for the possibility of recovery by the caller, which is more aware of the grander scheme of things:

```java
public void loadDataAndShowUiBecauseUserClickedThatButton() {
    try {
        Data data = dataAccessCode();
        showUiForData(data);
    } catch (SQLException e) {
        // This method’s scope can do UI, so we don't need sorcery to show an error dialog.
        // messages is an internationalized ResourceBundle.
        showErrorDialog(messages.getString("inaccessibleData"));
    }
}
```

#### Ending notes
Exceptions are a wonderful feature; it is worthwhile to use them. Don’t invent your own ways to handle and propagate errors; you’ll have less trouble and better results if you stick to the idiom instead of fighting with the platform that you are using.
