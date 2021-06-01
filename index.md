## Writing quality code

There are endless books and blog posts, articles and videos of lots of respected people in the industry telling you all about code quality and what it takes to write good code.

While all of these are extremely useful in its own right, I feel that sometimes it's all too much on the realm of the theoretical, it's all about arbitrarily large projects, customers demanding features that seem to be a perfect fit to illustrate certain design patterns or refactoring approaches. 

This is rarely true in the real world, where good code is tightly linked to good practices within a development team, besides the single individual refactoring the code to patterns.

I will present what I've gathered after around ~5 years of professional software development on several levels. There won't be a specific order to any of the sections and they can all be read independently of one another. This will be focused mostly on the standard Java, Spring Boot, Docker stack. I will try to otherwise keep all of these as generic as possible, to be useful for the widest possible audience.

### Process and company culture 

Process and company culture can affect code quality much more than any single individual contributions. There is a law, called Conway's law, that states that organizations design systems and structure their code in a way that mirrors their own organizational structure, so, this immediately implies that the culture and company that you work for, will affect the way technical problems are handled by you, as an active part of such culture.

It can be very hard to change such a culture from the inside-out, and, sometimes, it may even be impossible, so, it's important to be aware that there is only so much you can do as an individual.

A good way to approach this is to tap on your previous experiences and knowledge, and, trying to leverage it near where there are higher chances it will be listened to and acted upon: near senior developers who can be naturally more in touch with managers, so, your own ideas can ripple up in the organizational structure by first being "seeded" near where they can have the most impact: people who write code and are very aware of current limitations, but, at the same time, can exert some pressure near management. This is typically senior developers who know the company business well, have spent countless hours on the codebase, and, can have influence over management to slightly tip the scales towards better approaches.

### Write maintainable code

There are many ways to write maintainable code, and, while each codebase is a different beast, there are some general underlying principles that can help you write better code. Let's see some of them.

#### Keep classes and methods short

Classes and methods need to be kept short. What this means in practice, is, classes should have a single responsibility, and their behavior should serve one single purpose.

Methods should be kept as short as possible while not hurting readibility or code flow. A good rule of thumb, actually highlighted by IntelliJ IDEA, is that, if a method is longer than 15 lines, it can or should be considered a candidate for a split. Exceptions to the rule apply, of course. Let's see some examples:

```java
@Service
public class InvoiceManagementService {
    private final ClientInfoRepository clientInfoRepository;
    private final InvoiceRepository invoiceRepository;

   public InvoiceManagementService(ClientInfoRepository clientInfoRepository, InvoiceRepository invoiceRepository){
      this.clientInfoRepository = clientInfoRepository;
      this.invoiceRepository = invoiceRepository;
  }

  public List<ClientInvoice> processAllClientInfoForInvoices(){
      List<ClientData> clientData = clientInfoRepository.retrieveClientDataFromDatabase();
      
      List<ClientDetails> details = new ArrayList<>();
      for(ClientData individualClientData: clientData) {
         var clientDetails = individualClientData.getDetails();
         // Long processing logic over the details....
         details.add(clientDetails);
      }

      List<Long> accountNumbers = new ArrayList<>();
      for(ClientDetails cd: details) {
         Long an = cd.getClientAccountNumber();
         accountNumbers.add(an);
      }

      List<ClientInvoice> invoices = new ArrayList<>();
      for(Long accNumber: accountNumbers) {
        Optional<ClientInvoice> invoice = invoiceRepository.findInvoiceByAccountNumber(accNumber);
        if(invoice.isPresent()){
          invoices.add(invoice.get());
        }
      }

      return invoices;
}
```

We can use this method `processAllClientInfoForInvoices` as an example.

The method is long, and it clearly has many different responsibilities, so, there are a couple of approaches we can take here. Note that this is an intentional example where several things can be improved, but, it's not a toy one, code in production really does look similar.

The two important takeaways from this example are:

1. When a method is too long, split it into logical sub-units:

    Note how we can identify four logical sub-units inside our method:

    1 - Database retrieval
```java
      List<ClientData> clientData = clientInfoRepository.retrieveClientDataFromDatabase();
```

   2 - Processing client details
```java
      List<ClientDetails> details = new ArrayList<>();
      for(ClientData individualClientData: clientData) {
         var clientDetails = individualClientData.getDetails();
         // Long processing logic over the details....
         details.add(clientDetails);
      }
```

   3 - Getting account numbers
```java
      List<Long> accountNumbers = new ArrayList<>();
      for(ClientDetails cd: details) {
         Long an = cd.getClientAccountNumber();
         accountNumbers.add(an);
      }
```

   4 - Retrieving invoices
```java
      List<ClientInvoice> invoices = new ArrayList<>();
      for(Long accNumber: accountNumbers) {
        Optional<ClientInvoice> invoice = invoiceRepository.findInvoiceByAccountNumber(accNumber);
        if(invoice.isPresent()){
          invoices.add(invoice.get());
        }
      }
```

Note that this is already offering us a great opportunity for refactoring. Since these sub-units all seem to have their own individual responsibilities, we now can look at what makes sense to extract as isolated units into separate, single-purpose methods, to make the code more readable. If we choose to extract the processing of the client details together with the retrieval of the account numbers, as a separate method (note that it makes sense to extract this as a single sub-unit since the methods are closely related) we end up with the following original method:

```java
public List<ClientInvoice> processAllClientInfoForInvoices(){
      List<ClientData> clientData = clientInfoRepository.retrieveClientDataFromDatabase();
      
     extractAccountNumbersFromClientData(clientData);

      List<ClientInvoice> invoices = new ArrayList<>();
      for(Long accNumber: accountNumbers) {
        Optional<ClientInvoice> invoice = invoiceRepository.findInvoiceByAccountNumber(accNumber);
        if(invoice.isPresent()){
          invoices.add(invoice.get());
        }
      }

      return invoices;
}

public List<AccountNumber> extractAccountNumbersFromClientData(List<ClientData> clientData){
 List<ClientDetails> details = new ArrayList<>();
      for(ClientData individualClientData: clientData) {
         var clientDetails = individualClientData.getDetails();
         // Long processing logic over the details....
         details.add(clientDetails);
      }

      List<Long> accountNumbers = new ArrayList<>();
      for(ClientDetails cd: details) {
         Long an = cd.getClientAccountNumber();
         accountNumbers.add(an);
      }

    return accountNumbers;
}

```

Note how now we have managed to isolate the behavior of the account number extraction into a separate method, that is much more "narrow" in its responsibilities: it simply extracts the account numbers from the client details while at the same time serving the purpose of reducing the total method length that spanned the `processAllClientInfoForInvoices` method.

Added benefits include:

* The responsibilities of the original method are now reduced, and, the overall size and amount of code that is necessary to be read to understand the original method is simplified;

* With better isolation in terms of responsibilities, we can more easily debug the code when any problems arise, since we have "laser focus" onto what needs to be looked at;

2. When a class seems to be pulling in many different responsibilities, extract parts of the original code into separate, collaborating services

The refactoring we did before, helped improving the readibility of that particular method, but, sometimes, it is useful to try and extend these foundational refactoring ideas to the class level: "Is this the right place to add this dependency or this method? Does it make sense?". When doing this, we are forced to look more at our class and package structure for our services and confront exactly if what we are doing at one place can be generalized or placed in a different place.

For this simple example, we can look at it from a business perspective: if we have a service that is handling invoice management, does it make sense to couple it with business logic responsible for handling client information? Maybe if it's the only case, it's fine, but, let's assume for a second that retrieving extra data from the client information is a central workflow within the business, with many more applications besides just using the data concerning invoice management.

In that scenario, it makes sense to use composition to create a new service, dedicated to the retrieval of client data, and, this service can then be injected into the other collaborating services that would need this information. This would turn our original class into something like:

```java
@Service
public class InvoiceManagementService {
    private final ClientInformationRetrievingService clientInfoService;
    private final InvoiceRepository invoiceRepository;

   public InvoiceManagementService(ClientInformationRetrievingService clientInfoService, InvoiceRepository invoiceRepository){
      this.clientInfoService = clientInfoService;
      this.invoiceRepository = invoiceRepository;
  }

  public List<ClientInvoice> processAllClientInfoForInvoices(){
     List<Long> accountNumbers = clientInfoService.retrieveClientAccountNumbers();
     List<ClientInvoice> invoices = new ArrayList<>();
     for(Long accNumber: accountNumbers) {
       Optional<ClientInvoice> invoice = invoiceRepository.findInvoiceByAccountNumber(accNumber);
       if(invoice.isPresent()){
         invoices.add(invoice.get());
       }
     }
     return invoices;
}
```

Note how all the logic concerning the client data is completely abstracted away into a separate service.

Now, the code itself is better encapsulated and we have communication between services where each service has its own "domain actors", both the client and the invoices, and each service handles each entity separately, which makes it easier to test, easier to compose and easier even to reuse the newly extracted service into contexts that may not yet even exist (for example, requiring client info when interacting with new target platforms that haven't been yet developed, like mobile apps, or pages containing some report data, etc, the possibilities are endless, and all of them are enabled thanks to this service composition we have just created.

_Aim for composition at every possible level, and leverage your domain entities names and roles to guide you in creating this composition_

#### Leverage your tools: using functional programming in Java

Java has evolved a lot over the course of the last year or year and a half... The core team working on the stack now releases a new major version every 6 months, resulting in Java 17 being already available. With every major release there is a lot of new features being packed, from allowing type inference, to methods in interfaces, to modules, and to a lot of background work on garbage collection and on hiding the JVM internals from the programmers, the Java of today is a very different beast from the Java of simply a year ago.

Java 8 arguably was the biggest release in terms of impact for the developers, thanks to the introduction of functional programming concepts like `streams`, `Optional` and functional interfaces.

Streams support a wide range of functional operations that simplify common operations over lists or spring repositories considerably.

1. Whenever you need to apply a certain operation over each element of a list, be it filtering or transforming or aggregating, streams are the modern way of doing it:

```java
     //getting even numbers out of a list
    listOfNumbers.stream().filter(element -> element % 2 == 0).collect(toList());

    //Transforming a list of database entities into business domain entities
    List<BusinessLogicEntity> transformed = listOfDbEntities.stream().map(dbEntity -> transformEntityToDoBusinessLogic(dbEntity)).collect(toList());

    //Sum all values of balances of clients who have an account registered in your products. Assume `Balance` contains client info about registration 
    Double total = balancesList.stream().filter(entry -> entry.clientIsRegistered()).map(client -> client.retrieveBalance()).summingDouble());
```

Simple ifs and loop constructs can be replaced with these type of one-liners, that are both easier to read and write, contribute to adpoting a more modern Java style, and, you should, especially because staying outdated can become a real burden for a company and for yourself as a developer: if everyone is using these features and you are still stuck writing loops, maybe consider updating your own knowledge. Why? Because staying on top of the technologies with which you have to work daily will help you become a better developer and give you chances to introduce improvements in your company's codebase.

2. Optionals are meant to be used as containers that we assume have an object in it, if not, we can just "do nothing"

```java
    //Dont do this
    Optional<Invoice> invoice = invoiceRepository.findInvoiceById(1L);
    
    if(invoice.isPresent()){
        Invoice inv = invoice.get();
        return processInvoice(inv);
    }
    else{
        return Optional.empty();
    }

    //Do this instead
    Optional<Invoice> invoice = invoiceRepository.findInvoiceById(1L);
    
    return invoice.map(invoice -> processInvoice(invoice));
```

Read about the new features in your language, and, read about leveraging functional programming constructs specifically, and you will become a better programmer.

These ideas we have just seen of both functional programming idioms as well as composition of services, can actually be combined into a very nice principle for architectural design:

_Imagine your codebase is structured in a layered design, where the DB is one layer, business logics and services is another and external input is yet another: Aim for sanitizing and controlling any external input, push mutable and stateful operations where they belong: the persistence and database layer and maybe dedicated entry points in your business layer, and keep all behavior that is left as immutable as possible: leverage functional composition to build your business services and you will find your code easier to test and maintain._

#### Naming is important

Naming is one of the hardest things to do well in programming, for many reasons, but, for me, the killer one is context. While we are in the flow and coding, usually names are almost an afterthought and we just use whatever comes to mind so we don't break our own flow, or, sometimes, well...we simply can't come up with anything truly meaningful for the current context, and it's easy to find lots of generic or incomplete names, like: `getId()`, `computeResult(p)`, etc...

These names only _seem_ helpful while we are working, because we have all the surrounding context in our heads: "ofc it's the invoice order id, or of course the result is the profit of all the clients for the portfolio argument, etc".

Since naming is so important, the best advice I can give here is:

Every time you think in these terms (implying your surrounding code context into your naming style), take a step back and make it **explicit** in the code:

```java
getId() --> getInvoiceOrderId()

computeResult(p) --> calculateProfitFromAllClientForPortfolio(p)

...
```

Your colleagues will thank you, and, so will your future self, when you have to revisit your own code for that new killer feature or for the refactoring we all know it's coming at some point down the line. Following the. [Zen of Python](https://www.python.org/dev/peps/pep-0020/): "Explicit is better than implicit."

#### Codebase-wide consistency 

Codebase-wide consistency is a very small aspect that is largely covered by leveraging modern frameworks for your tech stack and/or area of domain: using frameworks like Flask (Python), Yesod or IHP (for Haskell) and of course Springboot for Java/Kotlin, largely ends up defining the main building blocks of how your apps will be structured.

These building blocks, however, are only a base, a scaffold, if you will, and, on top of that scaffold, concerns like: what frameworks to use to support testing, how to integrate the code with pipelines, and more importantly, how to USE the framework itself within your company, still need to be addressed and these are the things which will end up defining your application code and how you structure it.

Not all "Java shops" use Spring, and, of those that do use it, each of them will have a different, opinionated way of leveraging the stack to better adjust it to company culture or needs. Be on the look out for this, and, as soon as possible, if you aren't yet doing it, immediately propose to codify your own "stack flavour" into code, i.e. use any available library (or roll your own, really, it is worth it and invaluable!!) to ENFORCE ARCHITECTURAL GUIDELINES IN CODE. What does this mean?

Well, let's see:

- You can't directly inject repository methods inside resource classes, because you think they rightfully should be inside services? Make it an architectural test.

- All services need to be placed into a `service` package? Make it an architectural test.

- Native queries are only allowed on classes that your company annotates with a custom Spring annotation called `@NativeUsed`? Make it an architectural test.

- ...

You get the idea, right? [ArchUnit](https://www.archunit.org/) is a Java specific tool that allows you to "unit test your architecture". An absolutely amazing tool for enforcing code consistency, "codifying" best practices and also great as "live documentation" for onboarding new developers. I cannot recommend this enough!

### Testing for modern web apps

Testing has always been important. I remember being in university and having exam assignments where I needed to write tests in a standalone fashion by hand on paper, mimicking the junit setup. It was a daunting, terrible experience, simply because I could never appreciate it enough when I was younger, how important tests are for writing good, reliable code. I also never used it at my first two jobs, so, it definitely all takes a long time and effort to sink in.
However, testing is extremely important in the sense that:

- it serves as documentation of its associated production code, both for new and more seasoned developers. Tests exercise the public methods of the classes they aim to test, so, their public contract is exposed in an isolated, easy to grasp setting;

- ensures code correctness of old and new features, and, when coupled with CI/CD, provides a safe way for new developers to experiment without fear of breaking production;

- Supports automatic docs generation, which is a big plus!

Additionally, not all code can be tested, meaning that for code to be "testable", a certain discipline and code organization is needed. This relates to points discussed above about code consistency and keeping things "short and neat".

Note that testing needs to be done at least at three levels for a typical backend web app:

1. Unit tests: testing method results in isolation. These tests will comprise the bulk of any test suite, for the simple reason that they are self-contained, small, easy to write, and provide valuable feedback when testing complicated methods. For e.g.: when writing a service to check the status of a book requisition in a library, unit tests would be testing concerns like: when no books, an exception will be thrown. A fine will be paid if the date for return is after day X, etc.

2. Integration tests: These tests are the ones doing the heavylifting on most modern apps. They usually rely on a prepared set of test data, and they can interact with databases and simulate calls over the wire using libraries like Spring's `MockMVC` to assert on endpoint responses, checking authorization of endpoints, etc. It's not uncommon to have these tests setting up and tearing down entire application contexts for testing purposes. They test an entire service flow in integration with a set of test data that approaches what would be real world usage.

3. Performance tests: Used as the means to do e2e testing, leveraging API calling tools like Postman, these usually assert response codes and measure response times to catch inneficient implementations in a pipeline on early stages of development.

For modern web apps, usually, Rest APIs using Springboot coupled with a relational database (in the Java world), using Docker for testing has become not only a common practice, but actually, the standard way of writing tests. We will see Docker's place in aiding with application testing at the integration level and we will see why it's so useful.

Final and very important remark about testing:

_If you have taken the time to write and setup tests, you need to make the time to automate them on every commit and run them in a CI/CD pipeline. Testing benefits are only as good as their automation. You need to write them, but, they should run automatically on every commit to offer real value_

#### Docker

Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files together.

The main advantage of Docker is that it removes configuration and local setup pains away from developers, because all the necessary dependencies are bundled as custom images that, once started, will be available as a container, i.e. _a container is a running instance of an image, and, that image can be any pre-baked one available from public registries, or, more importantly, it can be our own code packaged as an image, usually on private container registries managed by your company._

Using `docker-compose`, one can define a configuration of containers that will be started together, and, dependencies between containers can be added to ensure they start up in the correct order, and, like this, a full test environment can be spun up via `docker-compose up --build` and torn down with `docker-compose down`.

An example of a compose file could be:

```yaml
version: '3'

services:
  authenticationMock:
    image: <some_private_company_registry>.com/authenticationAPI/authenticationMock:latest
    ports:
      - 7575:7575

  main-app-api:
    build: .
    depends_on:
      - authenticationMock
    ports:
      - 8080:8080
    entrypoint: '/bin/bash /lab/MainAPI.jar'
    environment:
      spring_datasource_url: ...
      some_db_user: ...
      some_db_user_pass: ....

networks:
  default:
    ipam:
      driver: default
      ...
```

#### Testcontainers

TODO

#### Meaningful test data

TODO

### Importance of CI/CD

TODO

### Closing remarks

Code quality is important, but, can be hard to get right, and, following these tips has helped me improving myself over time as well as helping others do the same!

If you've managed to read all the way until here: thank you so much, and if you feel like chatting about it or have suggestions, you can always drop me an email at olivbruno8 at gmail dot com, or open a PR, I guess :)

All of these aspects stem from a combination of all my experiences together with the countless blog posts, books and articles I have read and keep reading on a daily basis, these are my own opinions of course, and if you can relate and liked it, then I'm glad! If you feel I should add more to it, let me know! 
