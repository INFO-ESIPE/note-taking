[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## Strategy

Introduced by the Gang of Four, it allows to **varies the algorithm used by a class**
	*It's what we have done [[02 - SOLID#**O**pen-closed principle|here]]
	Also, this is pretty much what we do each time a method takes a lambda as parameter*

For instance, the sorting method `List<T>.sort(Comparator<? super T>)` takes as a parameter the code that allows to compare elements.

Conceptually, it looks like this:
![[strategy.png]]


****
## Example: Credential policy

We start with the following code:
```java
class Credential{
    private final String login;
    private final String password;

    // getters and setters
}

class CredentialValidator {
    // ...
    
    private boolean isSecured(Crendential cred){
         if (!Pattern.compile("[a-z]+").matcher(cred.getLogin()).matches()){
             return false;
         }
         return cred.getPassword().length() >= 8;
    }
}
```


The following code gives us more flexibility, as we can provide and change our policy easily:
```java
@FunctionalInterface
interface PasswordValidator {
    boolean isValid(String s)
}

class CredentialValidator {
	/* We set a default validator */
    private PasswordValidator validator = s -> s.length()>=8;

	/* We can change it by calling this method */
    public void setPasswordValidator(PasswordValidator validator){
         this.validator = Objects.requireNonNull(validator);
    }     
    // ...
    private boolean isSecured(Crendential cred){
         if (!Pattern.compile("[a-z]+").matcher(cred.getLogin()).matches()){
             return false;
         }
         return validator.isValid(cred.getPassword()); // validator used here
    }
}
```


Note that we have made a functional interface for more readability, but this acts exactly as a `Predicate<String>`. We could actually limit ourselves to this:
```java
class CredentialValidator {
    private Predicate<String> validator = s -> s.length() >= 8;

    public void setPasswordValidator(Predicate<String> validator) {
        this.validator = Objects.requireNonNull(validator);
    }

    private boolean isSecured(Credential cred) {
        if (!Pattern.compile("[a-z]+").matcher(cred.getLogin()).matches()) {
            return false;
        }
        // "test()" instead of "isValid()", but that's exactly the same...
        return validator.test(cred.getPassword());
    }
}
```
