[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## Builder

The builder pattern is used to build an object which requires a lot of parameters, especially if they are optional
	*We call it a convenience builder, pretty self explanatory*

Let's take the following class as an example:
```java
public class Client{
    
    public Client(String firstName,
                  String lastName,
                  List<Email> email,
                  List<PhoneNumber>) {
        // ...
    }
}

var client = new Client("Djebloune", "Cino",
                           List.of("cino@esiee.fr","cino@univ-eiffel.fr"),
                           List.of());
```

This is kind of unpractical, especially the necessity to pass an empty list to indicate an optional parameter we can't fill.
	*A boilerplate solution would be to use a default value in the constructor signature, but again, this is very unpractical as we have to chain parameters in the correct order, etc*

The idea is the following: We will make a **mutable class designed to create the final object**:
```java
public class Client {
	/* 
	We place the builder inside the class itself for convenience 
	The Builder class only must be public as well (else we can't get it...)
	*/
	public static class ClientBuilder {
	    private String firstName;
	    private String lastName;
	    private ArrayList<PhoneNumber> phones = new ArrayList<>();
	    private ArrayList<Email> emails = new ArrayList<>();
	    
	    public ClientBuilder firstName(String firstName){
	        this.firstName=Objects.requireNonNull(firstName);
	        return this;
	    }    
	
	    public ClientBuilder lastName(String lastName){
	        this.lastName=Objects.requireNonNull(lastName);
	        return this;
	    }
	
	    public ClientBuilder phoneNumber(PhoneNumber phoneNumber){
	        phones.add(Objects.requireNonNull(phoneNumber));
	        return this;
	    }
	
	    public ClientBuilder email(Email email){
	        emails.add(Objects.requireNonNull(email));
	        return this;
	    }
	
	    public Client build(){
	        if (firstName == null || lastName == null){
	            throw new IllegalStateException();
	        }
	        return new Client(firstName,lastName,emails,phones);
	    }
	}
}
```

As we can see, each building method returns itself. 
The idea is that **we return `this` so we can chain calls, and the `build()` method will check consistency of the state before returning the final object**:
```java
Client client = new ClientBuilder().firstName("Djebloune")
                                   .lastName("Cino")
                                   .email("cino@esiee.fr")
                                   .email("cino@univ-eiffel.fr")
                                   .build();
```

This way, we can skip optional parameters, and obtain a way more readable code.


****
## Conventions

In general, when we design a `Builder` for an object, **the builder should be the only way to go** (no access to the constructor from outside):
```java
public class Client{
	/* 
	The constructor is private, will only be used by the Builder when the
	"build()" method is called
	*/
	private Client(ClientBuilder clientBuilder){
		this.firstName = clientBuilder.firstName;
		...
	}

	public static class ClientBuilder{
		// ...

		/* 
		Instead of returning all properties one by one like in the previous code
		we simply return "this" (the builder itself). Client's constructor will 
		be in charge of dealing with parameters accordingly, now that
		it is private  
		*/
		public Client build() { 
			if (firstName == null || lastName == null) { 
				throw new IllegalStateException(); 
			} 
			return new Client(this); 
		}
	}
}
```


This works fine, but retrieving the builder via `new Client.ClientBuilder` is kind of ugly.
	*We don't want to explicitly write down in our code that we want a builder, we would rather want to manipulate the Client class directly... it is more readable*

So, we introduce a `with()` static method that forwards the builder in a more readable way:
```java
public class Client {
	// ...
	public static ClientBuilder with() { 
		return new ClientBuilder(); 
	}

	public static class ClientBuilder {
		// ...
	]
}

/* 
"with()" static method on Client class returns a builder
*/
Client client = Client.with().firstName("Djebloune")
	                            .lastName("Cino")
                                .email("cino@esiee.fr")
                                .email("cino@univ-eiffel.fr")
                                .build();
```


Our builder is fine now!
If we want to be perfectionist, we could add a `clear()` method to our builder which sets it's fields to the default value.
	*The purpose is to make the Builder re-usable*

We obtain the final code:
```java
public class Client {
	// ...
	
	private Client(ClientBuilder clientBuilder){ // private constructor
		this.firstName = clientBuilder.firstName;
		// ...
	}
	
	public static ClientBuilder with() { // static method to get the builder 
		return new ClientBuilder(); 
	}

	public static class ClientBuilder { // static inner-class, convenient
	    private String firstName;
	    private String lastName;
	    private ArrayList<PhoneNumber> phones = new ArrayList<>();
	    private ArrayList<Email> emails = new ArrayList<>();
	
	    public ClientBuilder firstName(String firstName) {
	        this.firstName = Objects.requireNonNull(firstName);
	        return this;
	    }
	
	    public ClientBuilder lastName(String lastName) {
	        this.lastName = Objects.requireNonNull(lastName);
	        return this;
	    }
	
	    public ClientBuilder phoneNumber(PhoneNumber phoneNumber) {
	        phones.add(Objects.requireNonNull(phoneNumber));
	        return this;
	    }
	
	    public ClientBuilder email(Email email) {
	        emails.add(Objects.requireNonNull(email));
	        return this;
	    }
	
	    public Client build() {
	        if (firstName == null || lastName == null) {
	            throw new IllegalStateException();
	        }
	        var client = new Client(this);
	        clear(); // reset fields after building the Client
	        return client;
	    }

		// resets the builder, re-usable (and auto-clears!
	    public ClientBuilder clear() {
	        this.firstName = null;
	        this.lastName = null;
	        this.phones.clear();
	        this.emails.clear();
	        return this;
	    }
	}
}
```


****
## Builders in Java API

There are some builders in the wilderness already:
- `StringBuilder`: The one we are the most familiar with
- `HttpRequest.Builder`: To build HTTP requests...
- `Locale.Builder`: To build localisation parameters


****
## Gang of Four

The GoF (Gang of Four) refers to the four authors of the famous book "Design Patterns: Elements of Reusable Object-Oriented Software" published in 1994.
	*Introduced 23 new robust design patterns which became standards to this day, including the `Builder`, `Factory`, `Observer`... It's a very important book for the OOP world*

Well, their version of the `Builder` is more ambitious than ours. It's designed for a **product with several implementations where each still follows the same construction process**.
	*E.g., A report which can either be in Markdown, HTML or XML
	Well, they all follow the same construction logic (author, date, title, paragraphs...), they just have a different syntax...*


Conceptually, it looks like this:
![[gof.png]]


In practice, it looks like this:
```java
public interface ReportBuilder{
	public ReportBuilder addTitle(String title);
	public ReportBuilder addParagraph(String paragraph);
	public ReportBuilder addSignature(String signature);
	public Report build();
}

/* Implementation example for the HTML one */
public class HTMLReportBuilder implements ReportBuilder{
    private StringBuilder sb = new StringBuilder();

    @Override
    public ReportBuilder addTitle(String title){
        sb.add("<h1>"+title+"</h1>");
        return this;
    }

    @Override
    public ReportBuilder addParagraph(String paragraph){
        sb.add("<p>"+paragraph+"</p>");
        return this;
    }

    // ...

    @Override
    public ReportBuilder build(){
        return new HTMLReport(sb.toString());
    }
}
```
