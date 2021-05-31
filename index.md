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

  public List<ClientInvoice> processAllClientInfoForInvoices(ClientInfoRepository clientInfoRepository){
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
public List<ClientInvoice> processAllClientInfoForInvoices(ClientInfoRepository clientInfoRepository){
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

The refactoring we did before, helped improving the readibility of that particular method....

#### Leverage your tools: using functional programming in Java

TODO

#### Naming is important

TODO

#### Codebase-wide consistency 

TODO

### Testing for modern web apps

TODO

#### Docker

TODO

#### Testcontainers

TODO

#### Meaningful test data

TODO

### Importance of CI/CD

TODO

### Closing remarks

TODO



### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
