---
layout: post
title:  "Validation"
date:   2022-03-04 22:00:00
tags: oop
comments: true
shareable: true
author: randommancer
preview: Writing software is no simple task and comes with a lot of challenges that are worth discussing but in this post, I want to shine a light upon one of the hardest ones to address daily, `maintainability`. I want to present a common scenario before refactoring using OOP techniques to a different but not new perspective that has been around for several years however I see almost no discussion or adoption in the software community.

image: https://randommancer.com/images/caffeinated_concert_tickets.png
--- 


Writing software is no simple task and comes with a lot of challenges that are worth discussing but in this post, I want to shine a light upon one of the hardest ones to address daily, `maintainability`. I want to present a common scenario before refactoring using OOP techniques to a different but not new perspective that has been around for several years however I see almost no discussion or adoption in the software community.

Some years ago I was in charge of breaking down a monolith application that handled money exchange payment into microservices, the biggest reason for this change was because the application was not scaling well enough and started to hurt the business.

That was a good opportunity to dive deep into the business knowledge using DDD techniques clarifying domain gaps and tightening up code to business. As many people already know Domain Driven Design is not about language, patterns or any code techniques, it's about domain knowledge, ubiquitous language and bounded context, so there are no strings attached to any specific framework.

The first step was to isolate the existing code in specific contexts inside the monolith, helping the team to visualize the relationship between code and business clarifying what needs to be extracted and the impact of that change.

After some extensive research and testing, we decided to go for a classic layer architecture in the new microservices using what we thought was the best to maintain high code quality and maintainability. Fast forwarding a little the project was a huge success and we received a huge tap on the back (lol) however the code was not great as our initial vision predicted and after some iterations, we realize some clear maintainability problems with bothered me a lot.

When researching how to improve maintainability I came to cross a community that uses an approach called `Elegant Objects`, after reading some blog posts I move on as what they were claiming and showing seems wrong and was against most of the things I learned in my career, being the biggest one was SQL speaking objects.

After a year of trying to find a better way to improve maintainability for the microservice without success I went back to the `EO` community, started to pay more attention to related blogs, worked on some pet projects and also help some open source initiatives that used this technique. That was mind blowing, I learned more about `OOP` at that time than my entire life working/studying in the industry.

I know `EO` is not a silver bullet being in some cases too extreme to fit the current industry, however, this line of thinking created different perspectives on the community and implementations leading to variations like for example `micro-objects`. What I will show below is basically `OOP` and it will follow the good standards you probably already know, hopefully, this post will make you at least curious to look further about the `maintainability` and `OOP` subject. Enough of context and let's jump into the code.

There are 2 classes with their respective tests from a classic layered architecture project validating some attributes and calling the repository, both save methods are returning a boolean for educational purposes only as the main focus it's to improve maintainability in the validation step.

Keep in mind both classes are doing similar validations, however, they have different meanings to the business that can change anytime.


### Company class
{% highlight csharp %}
public class CompanyService {
    private readonly SomeKindOfCompanyRepository someKindOfCompanyRepository;

    public CompanyService(SomeKindOfCompanyRepository someKindOfCompanyRepository) {
        this.someKindOfCompanyRepository = someKindOfCompanyRepository;
    }

    public bool save(string fantasyName, string employees) {
        if (string.IsNullOrWhiteSpace(fantasyName) || fantasyName.Length > 150) {
            return false;
        }

        var tryParse = int.TryParse(employees, out var parsedNumber);
        if (!tryParse || parsedNumber > 300) {
            return false;
        }

        return someKindOfCompanyRepository.save(fantasyName, parsedNumber);
    }
}
{% endhighlight %}


### Company test class
{% highlight csharp %}
public class CompanyServiceTest {
    [Test]
    public void savingValidCompany() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save("Fancy company", "120");
        Assert.True(result);
    }

    [Test]
    public void savingNullCompanyName() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save(null, "120");
        Assert.False(result);
    }

    [Test]
    public void savingWhiteSpaceCompanyName() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save(" ", "120");
        Assert.False(result);
    }

    [Test]
    public void savingTooLongCompanyName() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var fantasyName = new string('A', 151);
        var result = companyService.save(fantasyName, "120");
        Assert.False(result);
    }

    [Test]
    public void savingExactlyLenghtCompanyName() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var fantasyName = new string('A', 150);
        var result = companyService.save(fantasyName, "120");
        Assert.True(result);
    }
    
    [Test]
    public void savingTooBigCompany() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save("Fancy company", "301");
        Assert.False(result);
    }
    
    [Test]
    public void savingExactLenghtCompany() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save("Fancy company", "300");
        Assert.True(result);
    }
}
{% endhighlight %}


### User class
{% highlight csharp %}
public class UserService {
    private readonly SomeKindOfUserRepository someKindOfUserRepository;

    public UserService(SomeKindOfUserRepository someKindOfUserRepository) {
        this.someKindOfUserRepository = someKindOfUserRepository;
    }

    public bool save(string name, string age) {
        if (string.IsNullOrWhiteSpace(name) || name.Length > 100) {
            return false;
        }

        var tryParse = int.TryParse(age, out var parsedNumber);
        if (!tryParse || parsedNumber < 18) {
            return false;
        }

        return someKindOfUserRepository.save(name, parsedNumber);
    }
}
{% endhighlight %}


### User test class
{% highlight csharp %}
public class UserServiceTest {
    [Test]
    public void savingValidUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var result = userService.save("John", "25");
        Assert.True(result);
    }

    [Test]
    public void savingNameTooLongUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var name = new string('A', 101);
        var result = userService.save(name, "25");
        Assert.False(result);
    }

    [Test]
    public void savingNameNullUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var result = userService.save(null, "25");
        Assert.False(result);
    }

    [Test]
    public void savingNameWhiteSpaceUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var result = userService.save(" ", "25");
        Assert.False(result);
    }

    [Test]
    public void savingNameExactLengthUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var name = new string('A', 100);
        var result = userService.save(name, "25");
        Assert.True(result);
    }

    [Test]
    public void savingUnderAgeUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var result = userService.save("John", "17");
        Assert.False(result);
    }

    [Test]
    public void savingInvalidAgeUser() {
        var userService = new UserService(new SomeKindOfUserRepository());
        var result = userService.save("John", "orange");
        Assert.False(result);
    }
}
{% endhighlight %}


Let's start thinking about how to refactor the company class, the first thing I notice is the attribute naming `employees` in this small class is clear that is the number of employees as we can confirm by the validation a few lines below, however, that could be easily a list of employees as the name suggests, so I will rename `numberOfEmployees` for now.

Renaming the parameter is a good start to give more meaning to the method, you also can notice that we are going for a primitive obsession in the long run as string have no meaning to the business. So my second step is to create an abstraction that expresses better the business, I will start with the first parameter creating the `FantasyName` interface.

I want to highlight some nuances here regarding the `Single responsibility principle`, `SOLID` is great but sometimes it's too broad which can make it hard to pinpoint some specific cases like this one. Bear with me on this one, you could argue the service class has too much responsibility as it needs to validate fantasy name and also the number of employees, on the other hand, you could argue the service class just has one responsibility that is validating the company which is also correct.

In our case, we will assume the former.


### Fantasy name
{% highlight csharp %}
public interface FantasyName {
    string value();
}
{% endhighlight %}

I created the interface with the same return type of the parameter and now I will create our first implementation `CompanyFantasyName`.

### Company Fantasy name
{% highlight csharp %}
public class CompanyFantasyName : FantasyName {
    private readonly string fantasyName;

    public CompanyFantasyName(string fantasyName) {
        this.fantasyName = fantasyName;
    }

    public string value() {
        return fantasyName;
    }
}
{% endhighlight %}


We created a really fancy implementation that does not do a lot but we will get there, let's add our new interface to the company service and update the tests to work with the new implementation. I will not paste the entire test class every time but you can check the project on Github.


{% highlight csharp %}
public class CompanyService {
    private readonly SomeKindOfCompanyRepository someKindOfCompanyRepository;

    public CompanyService(SomeKindOfCompanyRepository someKindOfCompanyRepository) {
        this.someKindOfCompanyRepository = someKindOfCompanyRepository;
    }

    public bool save(FantasyName fantasyName, string numberOfEmployees) {
        if (string.IsNullOrWhiteSpace(fantasyName.value()) || fantasyName.value().Length > 150) {
            return false;
        }

        var tryParse = int.TryParse(numberOfEmployees, out var parsedNumber);
        if (!tryParse || parsedNumber > 300) {
            return false;
        }

        return someKindOfCompanyRepository.save(fantasyName.value(), parsedNumber);
    }
}
{% endhighlight %}


Here is a sample of the updated test.


{% highlight csharp %}
    [Test]
    public void savingValidCompany() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save(new CompanyFantasyName("Fancy company"), "120");
        Assert.True(result);
    }
{% endhighlight %}


In the company service, we are doing 2 validation regarding `FantasyName`. First, we check if the name is not null or whitespace, then we check the size for the name. With that in mind let's move these validations to our new `CompanyFantasyName` class, but first, we will write some tests.


{% highlight csharp %}
public class CompanyFantasyNameTest {
    [Test]
    public void validCompanyName() {
        const string fantasyName = "Fancy name";
        var companyFantasyName = new CompanyFantasyName(fantasyName);
        Assert.AreEqual(fantasyName, companyFantasyName.value());
    }

    [Test]
    public void whiteSpaceCompanyName() {
        const string fantasyName = " ";
        var companyFantasyName = new CompanyFantasyName(fantasyName);
        Assert.Throws<Exception>(() => companyFantasyName.value());
    }

    [Test]
    public void tooLongCompanyName() {
        var fantasyName = new string('A', 151);
        var companyFantasyName = new CompanyFantasyName(fantasyName);
        Assert.Throws<Exception>(() => companyFantasyName.value());
    }

    [Test]
    public void exactlyLengthCompanyName() {
        var fantasyName = new string('A', 150);
        var companyFantasyName = new CompanyFantasyName(fantasyName);
        Assert.AreEqual(fantasyName, companyFantasyName.value());
    }
}
{% endhighlight %}


All the tests are failing so we need to make them pass.


{% highlight csharp %}
public class CompanyFantasyName : FantasyName {
    private readonly string fantasyName;

    public CompanyFantasyName(string fantasyName) {
        this.fantasyName = fantasyName;
    }

    public string value() {
        if (string.IsNullOrWhiteSpace(fantasyName)) {
            throw new Exception("Fantasy name was not provided");
        }

        if (fantasyName.Length > 150) {
            throw new Exception("Fantasy name is more than 150 characters");
        }

        return fantasyName;
    }
}
{% endhighlight %}


It's all green now, notice that I created different exception messages for each error to illustrate what can be done, but at the end of the day is your class and you can implement the way you see fit for your context. Now let's update the `CompanyService` and the `CompanyServiceTest` class.


{% highlight csharp %}
public class CompanyService {
    private readonly SomeKindOfCompanyRepository someKindOfCompanyRepository;

    public CompanyService(SomeKindOfCompanyRepository someKindOfCompanyRepository) {
        this.someKindOfCompanyRepository = someKindOfCompanyRepository;
    }

    public bool save(FantasyName fantasyName, string numberOfEmployees) {
        var tryParse = int.TryParse(numberOfEmployees, out var parsedNumber);
        if (!tryParse || parsedNumber > 300) {
            return false;
        }

        return someKindOfCompanyRepository.save(fantasyName.value(), parsedNumber);
    }
}
{% endhighlight %}


We remove all fantasy name validation from the service class and the `CompanyServiceTest` is expecting an exception instead of a boolean, these tests are exaclty the same we did in `CompanyFantasyNameTest` so it's safe to delete them if we want to.

We could use the same approach on our `numberOfEmployees` parameter, so let's fast forward a litte.


### Number of employees
{% highlight csharp %}
public interface NumberOfEmployees {
    int value();
}
{% endhighlight %}

### Company number of employees
{% highlight csharp %}
public class CompanyNumberOfEmployees : NumberOfEmployees {
    private readonly string numberOfEmployees;

    public CompanyNumberOfEmployees(string numberOfEmployees) {
        this.numberOfEmployees = numberOfEmployees;
    }

    public int value() {
        var tryParse = int.TryParse(numberOfEmployees, out var parsedNumber);
        if (!tryParse) {
            throw new Exception("Invalid number of employees");
        }

        if (parsedNumber > 300) {
            throw new Exception("Number of employees is too big");
        }

        return parsedNumber;
    }
}
{% endhighlight %}

### Company number of employees tests
{% highlight csharp %}
public class CompanyNumberOfEmployeesTest {
    
    [Test]
    public void invalidNumberOfEmployees() {
        var companyNumberOfEmployees = new CompanyNumberOfEmployees("John");
        Assert.Throws<Exception>(() => companyNumberOfEmployees.value());
    }
    
    [Test]
    public void tooLongNumberOfEmployees() {
        var companyNumberOfEmployees = new CompanyNumberOfEmployees("301");
        Assert.Throws<Exception>(() => companyNumberOfEmployees.value());
    }
    
    [Test]
    public void validNumberOfEmployees() {
        const string numberOfEmployees = "200";
        var companyNumberOfEmployees = new CompanyNumberOfEmployees(numberOfEmployees);
        Assert.AreEqual(200, companyNumberOfEmployees.value());
    }
    
    [Test]
    public void exactlyNumberOfEmployees() {
        const string numberOfEmployees = "300";
        var companyNumberOfEmployees = new CompanyNumberOfEmployees(numberOfEmployees);
        Assert.AreEqual(300, companyNumberOfEmployees.value());
    }
    
}
{% endhighlight %}

### Updated company service
{% highlight csharp %}
public class CompanyService {
    private readonly SomeKindOfCompanyRepository someKindOfCompanyRepository;

    public CompanyService(SomeKindOfCompanyRepository someKindOfCompanyRepository) {
        this.someKindOfCompanyRepository = someKindOfCompanyRepository;
    }

    public bool save(FantasyName fantasyName, NumberOfEmployees numberOfEmployees) {
        return someKindOfCompanyRepository.save(fantasyName.value(), numberOfEmployees.value());
    }
}
{% endhighlight %}


Let's do a quick recap on what we achieved so far. 

We provided more meaningful abstractions to both parameters creating specific interfaces `CompanyFantasyName` and `NumberOfEmployees`, the new classes are responsible for just one single job that is validating fantasy name and number of employees respectively, being more focused, reusable and easy to test.

Now back to the `UserService` class, we should be able to see similarities so we will use the same approach to refactor.


{% highlight csharp %}
public class UserService {
    private readonly SomeKindOfUserRepository someKindOfUserRepository;

    public UserService(SomeKindOfUserRepository someKindOfUserRepository) {
        this.someKindOfUserRepository = someKindOfUserRepository;
    }

    public bool save(string name, string age) {
        if (string.IsNullOrWhiteSpace(name) || name.Length > 100) {
            return false;
        }

        var tryParse = int.TryParse(age, out var parsedNumber);
        if (!tryParse || parsedNumber < 18) {
            return false;
        }

        return someKindOfUserRepository.save(name, parsedNumber);
    }
}
{% endhighlight %}


First, we will create the name interface and implementation.


### Name interface
{% highlight csharp %}
public interface Name {
    string value();
}
{% endhighlight %}

### User name class
{% highlight csharp %}
public class UserName : Name {
    private readonly string name;

    public UserName(string name) {
        this.name = name;
    }

    public string value() {
        if (string.IsNullOrWhiteSpace(name)) {
            throw new Exception("User name was not provided");
        }

        if (name.Length > 100) {
            throw new Exception("User name is more than 100 characters");
        }

        return name;
    }
}
{% endhighlight %}


This is important, we can notice the code is very similar in both `CompanyFantasyName` and `UserName` as they both check string validation and size.

{% highlight csharp %}
        if (string.IsNullOrWhiteSpace(name) || name.Length > 100) {
            return false;
        }
{% endhighlight %}

{% highlight csharp %}
        if (string.IsNullOrWhiteSpace(name) || name.Length > 150) {
            return false;
        }
{% endhighlight %}


I could merge both classes receiving the length parameter from the constructor and that would solve my problem, however, if anything changes on `CompanyFantasyName` or `UserName` it will cause some problems to our application and we might need to split the classes again. despite the fact they have a different meaning for the business, so merging them will impact our code/business language (ubiquitous language).

To get rid of this duplication without compromising the business class I will create another abstraction representing a text on the application, this new interface will be named `Text` and I will name the first implementation `SimpleText`.


{% highlight csharp %}
public interface Text {
    string value();
}
{% endhighlight %}


{% highlight csharp %}
public class SimpleText : Text {
    private readonly string name;
    private readonly int maxLength;

    public SimpleText(string name, int maxLength) {
        this.name = name;
        this.maxLength = maxLength;
    }

    public string value() {
        if (string.IsNullOrWhiteSpace(name)) {
            throw new Exception("Text was not provided");
        }

        if (name.Length > maxLength) {
            throw new Exception("Text is more than " + maxLength + " characters");
        }

        return name;
    }
}
{% endhighlight %}


Now we could inject `Text` as a dependency to our `UserName` and `CompanyFantasyName` validating  without repeating code or hurting the meaning.


{% highlight csharp %}
public class CompanyFantasyName : FantasyName {
    private readonly Text fantasyName;

    public CompanyFantasyName(Text fantasyName) {
        this.fantasyName = fantasyName;
    }

    public string value() {
        return fantasyName.value();
    }
}
{% endhighlight %}


We could end here and will be fine for now but what is the responsibility of `SimpleText`, for me this new class is doing too much, it's validating the string and checking its size. What if another class in the system wants to validate a string without checking the size? With the current implementation, this is not possible.

To solve this issue we might need more classes one for each responsibility. To achieve our goal I will use a pattern called `decorator`, so let's get to work.


{% highlight csharp %}
public class SimpleText : Text {
    private readonly string text;

    public SimpleText(string text) {
        this.text = text;
    }

    public string value() {
        return text;
    }
}
{% endhighlight %}


Now our class is just a value holder and it will serve when we need a text without validation like when retrieving text data from the database.


{% highlight csharp %}
public class NotNullText : Text {
    private readonly Text text;

    public NotNullText(Text text) {
        this.text = text;
    }

    public string value() {
        var value = text.value();
        if (string.IsNullOrWhiteSpace(value)) {
            throw new Exception("Text was not provided");
        }

        return value;
    }
}
{% endhighlight %}


This is the `NotNullText` it does what the name suggests, validate if the text is null or whitespace. 

To use this new class we add a `Text` dependency on `CompanyFantasyName`, then we created a second constructor decorating `SimpleText` with `NotNullText` and calling the primary constructor extending how the class can be created.

Now we have two ways to create `CompanyFantasyName`, we can pass directly a `Text` for testing or specific cases or we can pass a string and let the class compose the validations it needs.


{% highlight csharp %}
public class CompanyFantasyName : FantasyName {
    private readonly Text fantasyName;

    public CompanyFantasyName(string fantasyName) : this(new NotNullText(new SimpleText(fantasyName))) { }

    public CompanyFantasyName(Text fantasyName) {
        this.fantasyName = fantasyName;
    }

    public string value() {
        var value = fantasyName.value();
        if (value.Length > 150) {
            throw new Exception("Fantasy name is more than 150 characters");
        }

        return value;
    }
}
{% endhighlight %}


There are some important things to unpack here, first, we created 2 constructors but could be many this is mentioned in `Elegant Objects` as primary and secondary constructors where the primary constructor should always be called in the end by all others secondary constructors ensuring the abstraction that in our case is `Text`.


### Primary constructor
{% highlight csharp %}
public CompanyFantasyName(Text fantasyName) {
        this.fantasyName = fantasyName;
    }
{% endhighlight %}

### Secondary constructor
{% highlight csharp %}
public CompanyFantasyName(string fantasyName) : this(new NotNullText(new SimpleText(fantasyName))) { }
{% endhighlight %}


it's good enough for now so let's go back and do the same thing with the character length validation.


{% highlight csharp %}
public class LimitedText : Text {
    private readonly Text text;
    private readonly int maxLength;

    public LimitedText(string text, int maxLength) : this(new NotNullText(new SimpleText(text)), maxLength) { }

    public LimitedText(Text text, int maxLength) {
        this.text = text;
        this.maxLength = maxLength;
    }

    public string value() {
        var value = text.value();
        if (value.Length > maxLength) {
            throw new Exception("Text is more than " + maxLength + " characters");
        }

        return value;
    }
}
{% endhighlight %}


Here we go, now we have a class that the main reason to exist is to validate the string length. If you take a closer look at the secondary constructor you will notice that I'm already creating a `NotNullText` instance as would be important to check if the text is null before the length size.

Now let's change our company name class again.


{% highlight csharp %}
public class CompanyFantasyName : FantasyName {
    private readonly Text fantasyName;
    private const int MaxLentgh = 150;

    public CompanyFantasyName(string fantasyName) : this(
        new LimitedText(
            new NotNullText(
                new SimpleText(fantasyName)
            ), MaxLentgh)
    ) { }

    public CompanyFantasyName(Text fantasyName) {
        this.fantasyName = fantasyName;
    }

    public string value() {
        return fantasyName.value();
    }
}
{% endhighlight %}


All the tests are passing for `CompanyFantasyName` so we can delete them if we want as these validations are already being tested in the `Text` implementations classes like `NotNullTextTest`. Let's do some refactoring and add some constructors to get rid of excess of `new` in the secondary constructor.


{% highlight csharp %}
public class CompanyFantasyName : FantasyName {
    private readonly Text fantasyName;
    private const int MaxLentgh = 150;

    public CompanyFantasyName(string fantasyName) : this(
        new LimitedText(fantasyName, MaxLentgh)
    ) { }

    public CompanyFantasyName(Text fantasyName) {
        this.fantasyName = fantasyName;
    }

{% endhighlight %}


{% highlight csharp %}
public class NotNullText : Text {
    private readonly Text text;

    public NotNullText(string text) : this(new SimpleText(text)) { }

    public NotNullText(Text text) {
        this.text = text;
    }
{% endhighlight %}


{% highlight csharp %}
public class LimitedText : Text {
    private readonly Text text;
    private readonly int maxLength;

    public LimitedText(string text, int maxLength) : this(new NotNullText(text), maxLength) { }

    public LimitedText(Text text, int maxLength) {
        this.text = text;
        this.maxLength = maxLength;
    }
{% endhighlight %}


I want to highlight the `MaxLength` constant is part of the `CompanyFantasyName` class and it's ok to be hardcoded and not in an external enum.

Here are all the changes we need to do in the `UserName` class after our `Text` abstractions.


{% highlight csharp %}
public class UserName : Name {
    private readonly Text name;
    private const int MaxLentgh = 100;

    public UserName(string name) : this(
        new LimitedText(name, MaxLentgh)
    ) { }

    private UserName(Text name) {
        this.name = name;
    }

    public string value() {
        return name.value();
    }
}
{% endhighlight %}


We manage to reuse everything and all the classes are simple and focused.

Moving on to the `CompanyNumberOfEmployees` class we need to parse a string to int and also check the number of employees on the company, we already learned a couple of things from our previous refactor so let's try to go a little faster now creating `Number` abstraction and an implementation class to parse the string.


{% highlight csharp %}
public interface Number {
    int value();
}
{% endhighlight %}


{% highlight csharp %}
public class ParsedNumber : Number {
    private readonly string text;

    public ParsedNumber(string text) {
        this.text = text;
    }

    public int value() {
        var tryParse = int.TryParse(text, out var parsedNumber);
        if (!tryParse) {
            throw new Exception("Invalid number of employees");
        }

        return parsedNumber;
    }
}
{% endhighlight %}


Something is a little off, this code could work but we also want to make sure the string is not null before parsing, good for us we already have a class that does exactly that. So we just need to inject `Text` in the constructor like we did in the other classes.

{% highlight csharp %}
public class ParsedNumber : Number {
    private readonly Text text;

    public ParsedNumber(Text text) {
        this.text = text;
    }

    public int value() {
        var value = text.value();
        var tryParse = int.TryParse(value, out var parsedNumber);
        if (!tryParse) {
            throw new Exception("Invalid number of employees");
        }

        return parsedNumber;
    }
}
{% endhighlight %}


Something magical is happening here, we are receiving a `Text` and returning a `Number` without breaking abstraction or encapsulation, look the code below:


{% highlight csharp %}
Number parsedNumber = new ParsedNumber(
    new SimpleText(numberOfEmployees)
)
{% endhighlight %}


We are composing the application keeping everything hidden, for example, the class that will execute the number abstraction has no idea about the composition of the Number, could be like the example above or just a `SimpleNumber`.

Just to make this class more meaningful for reuse I will change its name to `TextToNumber`, I think this new name reflects better what the class is doing it, now let's update our `CompanyNumberOfEmployees` class.


{% highlight csharp %}
public class CompanyNumberOfEmployees : NumberOfEmployees {
    private readonly Number numberOfEmployees;

    public CompanyNumberOfEmployees(string numberOfEmployees) : this(new TextToNumber(numberOfEmployees)) { }

    public CompanyNumberOfEmployees(Number numberOfEmployees) {
        this.numberOfEmployees = numberOfEmployees;
    }

    public int value() {
        var parsedNumber = numberOfEmployees.value();
        if (parsedNumber > 300) {
            throw new Exception("Number of employees is too big");
        }

        return parsedNumber;
    }
}
{% endhighlight %}


I think we are done with the `CompanyNumberOfEmployees`. Now lets go look back to `UserService` and update the class.


{% highlight csharp %}
public class UserService {
    private readonly SomeKindOfUserRepository someKindOfUserRepository;

    public UserService(SomeKindOfUserRepository someKindOfUserRepository) {
        this.someKindOfUserRepository = someKindOfUserRepository;
    }

    public bool save(string name, string age) {
        if (string.IsNullOrWhiteSpace(name) || name.Length > 100) {
            return false;
        }

        var tryParse = int.TryParse(age, out var parsedNumber);
        if (!tryParse || parsedNumber < 18) {
            return false;
        }

        return someKindOfUserRepository.save(name, parsedNumber);
    }
}
{% endhighlight %}


First, we need to change `string name` to `Name name`  and remove duplicated code.


{% highlight csharp %}
public class UserService {
    private readonly SomeKindOfUserRepository someKindOfUserRepository;

    public UserService(SomeKindOfUserRepository someKindOfUserRepository) {
        this.someKindOfUserRepository = someKindOfUserRepository;
    }

    public bool save(Name name, string age) {
        var tryParse = int.TryParse(age, out var parsedNumber);
        if (!tryParse || parsedNumber < 18) {
            return false;
        }

        return someKindOfUserRepository.save(name.value(), parsedNumber);
    }
}
{% endhighlight %}


Now we need to follow the same process of `CompanyNumberOfEmployees` and create an `Age` abstraction with one implementation.


{% highlight csharp %}
public interface Age {
    int value();
}
{% endhighlight %}

{% highlight csharp %}
public class UserUserAge : Age {
    private readonly Number age;

    public UserAge(string age) : this(new TextToNumber(age)) { }

    public UserAge(Number age) {
        this.age = age;
    }

    public int value() {
        var parsedNumber = age.value();
        if (parsedNumber < 18) {
            throw new Exception("User is underage");
        }

        return parsedNumber;
    }
}
{% endhighlight %}


Now the `UserService` again.


{% highlight csharp %}
public class UserService {
    private readonly SomeKindOfUserRepository someKindOfUserRepository;

    public UserService(SomeKindOfUserRepository someKindOfUserRepository) {
        this.someKindOfUserRepository = someKindOfUserRepository;
    }

    public bool save(Name name, Age age) {
        return someKindOfUserRepository.save(name.value(), age.value());
    }
}
{% endhighlight %}


We could have the urge to refactor `UserUserAge` and `CompanyNumberOfEmployees` trying to join both funcionalities but if you pay close attention you can notice they are not the same code or funcionality so it's better to keep this way for now.

We are done, finishing this refactor with a lot of new classes compared to the initial setup and with tons of reusability/meaning for each one of them.


## Conclusion section

Firstly I want to address a couple of things, this post is aiming for code reuse and maintainability. The former is easy to understand but the latter has a lot of nuances as maintainable code could mean different things for different people/teams.

To clarify a little code maintainability for me is the ratio between cost/time when adding new features, correcting bugs, refactoring and time investment to understand the business. We want people to read the code and know what's going on without overloading the brain (cognitive load), if we need to understand 1k lines of code to change it we will very likely introduce bugs or left code uncovered by tests increasing the risk for the business.

I know the initial service classes were small with just one dependency but they will grow eventually becoming more complex, leading to complicated tests, duplicated and hard to understand code. To give a simple example how many classes in your code today need to verify if a string is not null/empty or has some specific length? How did you test these classes?

Having classes abstracting and encapsulating data could have a huge impact on your understanding of the code and the amount of code you need to change to add a new feature or fix a bug. The class constructor should not be underestimated and must show what a class needs to exist, providing ways to extend funcionality and increase reusability.

I hope you are not scared by the `new` keyword, I know a lot of companies use DI frameworks to create classes, they can facilitate some configuration with some magic but the price is limiting implementations and flexibility of `OOP` but this is a conversation for another post but let me briefly explain.

Everything we do in software is context dependant and there is no silver bullet and this approach has its own problems that I will tackle in later posts but briefly we can see exception handling and memory use (many classes). My main objective is to give some light on a different perspective and see if there is a better way to achieve a better maintainability in software than we do today.

If read until here, thank you for your time `:)` and if you have any thoughts about the subject feel free to discuss in the comments section and also check the code on github.s


<figure class="articleimg">
    <img src="{{page.image}}" alt="Prank callers">
    <figcaption>
    Regular Show - Caffeinated Concert Tickets, by J.G. Quintel
    </figcaption>
</figure>
