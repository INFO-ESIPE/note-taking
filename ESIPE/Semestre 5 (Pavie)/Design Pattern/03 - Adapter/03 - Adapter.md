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

We begin with this simple logging interface:
```java
interface Logger {
  void log(String message);
}
```

But as our code evolves, another library logger is introduced:
```java
enum Level { WARNING, ERROR }

interface Logger2 {
  void log(Level level, String message);
}
```

