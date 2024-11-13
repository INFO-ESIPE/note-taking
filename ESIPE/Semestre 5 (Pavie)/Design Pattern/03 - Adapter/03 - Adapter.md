[[Design Pattern]]
****
**Note:** The following repository provides good details on various design patterns implementation in Java (some are mentioned in this class, others aren't but are still worth checking)
https://github.com/forax/design-pattern-reloaded
****

This pattern allows to use two libraries doing the same thing but without the same API
The idea behind it is the following: We create a new API and implement each class per library on it. Those classes are the **adapters**


****
## Example 1: SMS Libraries

We have two libraries that allows to send SMS, but their API is different (even though they do the same thing):
```java
public class GoogleSMS {
    public void sendSMS(String message,String phoneNumber);
    public void sendSMSBulk(String message, List<String> phoneNumbers);
}

public class AmazonSMS {
    public void currentPhoneNumber(String phoneNumber);
    public void sendSMS(String message);
    public void spamSMS(List<String> message);
}
```

We create a new API (as we can see, it is different from the two others, and it doesn't really matter):
```java
public interface SMSSender {
       public void sendSMS(String message,String phoneNumber);
       
       default public void sendSMSBulk(String message, List<String> phoneNumbers){
            for(var phoneNumber : phoneNumbers){
                sendSMS(message,phoneNumber);
            }
       }
  
       default public void sendSMSBulk(List<String> messages, String phoneNumbers){
            for(var message : messages){
                sendSMS(message,phoneNumber);
            }
       }
   }
```

We now make our two adapters (one per library):
```java
public class GoogleSMSAdapter implements SMSSender{
    private final GoogleSMS googleSMS = new GoogleSMS();

    @Override
    public void sendSMS(String message,String phoneNumber) {
        googleSMS.sendSMS(message,phoneNumber);
    }

    @Override
    public void sendSMSBulk(String message, List<String> phoneNumbers) {
        googleSMS.sendSMSBulk(String message, List<String> phoneNumbers);
    }
}
```


****
## Example 2: Logger
*We'll see another way of doing it here*

We begin with this simple logging interface:
```java
interface Logger {
  void log(String message);
}
```

But as our code evolves, another logger is introduced, coming from another library:
```java
enum Level { WARNING, ERROR }

interface Logger2 {
  void log(Level level, String message);
}
```


What if we do not have control over `Logger2` ?
> We go through a dedicated methods that takes both the logger and the level and adapt it
```java
public static Logger adapt(Logger2 logger2, Level level) {
  return msg -> logger2.log(level, msg);
}

// ...

Logger logger = adapt(logger2, Level.WARNING);
logger.log("blabla");
```

In this case, the `Adapter` acts as a **vue that transfers the data from one method of the new interface to a method call to the old interface**.


What if we have control over `Logger2` ?
>  Then, our `adapt()` method can directly be an instance method of the `Logger2`
```java
interface Logger2 {
  void log(Level level, String message);

  default Logger adapt(Level level) {
    return msg -> log(level, msg);
  }
}

// ...

Logger logger = logger2.adapt(Level.WARNING);
logger.log("blabla");
```
