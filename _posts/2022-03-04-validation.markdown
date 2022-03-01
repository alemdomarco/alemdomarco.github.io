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

In our case, we will stick with the former.


******** Review stopped *******


### Fantasy name 
{% highlight csharp %}
public interface FantasyName {
    string value();
}
{% endhighlight %}

You can notice that I'm returning just a plain string and that's ok, bear with me as we are doing small steps here. Now I will create our first implementation `CompanyFantasyName`.

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


Not that fancy for now but we will get there, now let's add our new interface to the company service and update the tests to receive the new implementation. I will not paste the entire test class every time but I will keep the project available on Github.


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


Just give an idea of the updated test.


{% highlight csharp %}
    [Test]
    public void savingValidCompany() {
        var companyService = new CompanyService(new SomeKindOfCompanyRepository());
        var result = companyService.save(new CompanyFantasyName("Fancy company"), "120");
        Assert.True(result);
    }
{% endhighlight %}


All green so let's move on.
In the company service, we are doing 2 validation regarding `FantasyName`, checking if the name is not null or whitespace and the size as we now have a specific class representing our fantasy name we can move the verifications to the class but first let me write some tests.


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


They are broken now so let's update the class.


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


It's all green now, notice that I created different exception messages for each error just to illustrate what can be done, but at the end of the day in your class, you can implement the way you see fit for your context and team. So let's update the `CompanyService`.


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


Notice how we remove all the fantasy name validation from the service, now we need to update the `CompanyServiceTest` tests to expect an Exception instead of false and if the test passes we should be good to remove all fantasy names tests from the company service test class.

We could use the same approach on our `numberOfEmployees` parameter, so let's fast forward to that.


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


It's is looking good and all the tests are green, let's do a quick recap on what we achieved so far. We provided more meaningful abstraction to both parameters creating specific interfaces, the new classes are responsible for just one single job so they can be easily tested and reused.

When I look back to the UserService class I realize we could use the same approach.


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


We should start creating the naming interface and its first implementation.


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


I noticed the code is very similar in both `CompanyFantasyName` and `UserName` as they both check string validation and size. I could use one interface and class for both, passing some attributes for the length of the characters however as they have different business meaning some changes can add some unnecessary complexity for example if I need to check if the user already exists in the database. I could also create a utility class but that would not have any meaning for the business too.

One thing we could do is create another abstraction that could be reused in both places, for this particular example I will use one interface called `Text` handling the validation inside the implementations just like in EO, just keep in mind you always can do and should do what fits your context the best.


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


Now we could use inject `Text` as a dependency inside our `UserName` and `CompanyFantasyName` validating both classes without repeating code but I think we could do better and I will explain why.

Both validations are fine inside their respective classes, `UserName` should validate a user name and `CompanyFantasyName` should validate a company fantasy name on the other hand `Text` should validate only text I know it sounds weird but bears with me. If I have another class in the system that wants to validate only if the string is valid and not the length or even do not want to validate at all.

This means `Text` class has more than one responsibility different from their counterparts as it's in charge of validating if the string is not null and also checking if it is in a specific range of characters.

So I will use the decorator pattern to add some composition in our `Text` implementations, one for each check and add the tests for it.

Let's change simple text and create the null check implementation first.


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


This class is just a base value holder in case we don't want to validate anything for example when retrieving a text from the database.


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


This is the `NotNullText` that does what the name suggests, validating if the text is valid or not, so how can we use this new class?

We just need to inject the dependency.


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


There are some things to unpack here, first, we create 2 constructors one for text and one for the string as you can see the string one is a little more complicated as it needs to decorate the `SimpleText` class with the `NotNullText` class and also you can notice the latter call the former. This is happening because we have one primary constructor and one secondary constructor and the primary constructor should always be called to create the instance.

One observation is that you can have as many secondary constructors as you want but in the end, you need to call the primary.


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


I think it's good enough for now so let's go back and do the same thing with the character length.


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


Here we go, now we have a class that the main reason to exist is to validate the string length. If you take a closer look at the secondary constructor you will notice that I'm already creating a `NotNullText` instance as would be great to check this before the length.

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


All the tests are passing for `CompanyFantasyName` so we can delete them as these validations are already being tested in the `Text` implementations classes that I did not show like `NotNullTextTest`. Let's do some refactor and add some constructors to get rid of excess of `new` in the secondary.

Here are the results.


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


Now let's see what needs to be done in our UserName class.


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


We reused the classes so was an effortless change and after running the tests everything is still green.

Moving on to the `CompanyNumberOfEmployees` class we need to parse to int and also check the number of employees on the company, we already learned a couple of things so let's try to go a little faster now and create a `Number` abstraction and a class to parse the string.


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


It's looking good pretty good but something is a little off, that would work in case we need to use this class to parse a string from an entry point like json/events but if we want to parse some `text` or compose a more rich validation we would have to break encapsulation and would be hard to inject this kind of dependency, let me show in the code below.


{% highlight csharp %}
    var value = limitedText.value();            // Breaking encapsulation
    var parsedNumber = new ParsedNumber(value)  // This is high coupling
{% endhighlight %}


Let me explain:
- We are breaking encapsulation because the class that is making the call for value needs to get the data from the abstraction and put it in another abstraction but we should be able to keep the data hidden most of the time.
- This is high coupling because the class that instantiates the ParsedNumber will have to use the real implementation difficulting tests and maintainability, that's why we instantiate inside the constructor to provide room for different implementations.

So how can we convert some string to int without breaking the abstraction?

We already have a class in place that represents a text so instead of passing a raw string we should pass the text (mind explosion sound). One more thing `ParsedNumber` is not a very helpful name so I will change to a more familiar name at least to me `TextToNumber`.


{% highlight csharp %}
public class TextToNumber : Number {
    private readonly Text text;

    public TextToNumber(string text) : this(new NotNullText(text)) { }

    public TextToNumber(Text text) {
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


Now it's looking good, so let's change our `CompanyNumberOfEmployees` class and run the tests.


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


I think we are done with the company stuff. Looking back to `UserService` there is a lot we can change.


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


Now we need to follow the same process and create an Age abstraction.


{% highlight csharp %}
public interface Age {
    int value();
}
{% endhighlight %}

{% highlight csharp %}
public class UserAge : Age {
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


Then go back to the service


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


Now we have a lot of classes from the initial one in the other hand we have a lot of reusability and meaning for each class, we can still add more classes for the under-aged user and max number of employees but as this code is written just once we can delay this refactor until we need it.


## Conclusion section

Firstly I want to address a couple of things, this post and refactoring are aiming for code reuse and maintainability. The former is easy to understand but the latter has a lot of nuances as maintainable code could mean different things for different people.

Code maintainability for me is the ratio between cost/time when adding new features, correcting bugs, refactoring and understanding the business. We want people to read the code and know what's going on without overloading the brain (cognitive load), if we need to understand 1k lines of code to change it we will very likely introduce bugs or left branches uncovered by tests taking more time than usual even for small changes.

The services classes are small with just one dependency but they will grow eventually becoming more complex which could lead to complicated tests, duplicated and hard to understand code.

To give a simple example how many classes need to verify if a string is not null/empty or has some specific length? How did you test these classes?

Having classes abstracting and encapsulating data could have a huge impact on your understanding of the code and the amount of code you need to change to add a new feature or fix a bug. The class constructor should not be underestimated and must be used to enforce what the class needs to exist as is the data and dependency entry point.

A lot of companies use DI frameworks so creating a class using the `new` keyword could look really weird but that is what the framework is doing, the only difference is the framework limit any chance of using OOP. This is a conversation for another post but let me briefly explain.

DI is all about passing an abstraction (interface) hiding implementation details (class) usually when you use DI frameworks you have one interface and one implementation which is kinda pointless because, in theory, your abstraction can have an infinite number of implementations but the framework by default cap that at 1. It would be very complicated to create `UserAge` class using a framework but I can easily do using the already known `new` keyword, doing that you bring the responsibility to yourself so you need to be mindful about how to avoid coupling and when its ok to create/compose new classes.

Everything we do in software is context dependant and there is no silver bullet as expected this approach has its own problems that I will tackle in later posts like exception handling and memory (many classes). My main objective is to give some light to a different perspective and see if there is a better way to achieve maintainability than we do today. 

This post has a lot of different things mixed up and I will name it here for a quick search:
- Elegant Objects
- Micro Objects
- Domain Driven Design
- SOLID principles
- Layer Architecture (Service/Repository)
- Decorator (Design pattern)

The code is on my GitHub so feel free to check it out and if you have any thoughts about it just let me know.












<figure class="articleimg">
    <img src="{{page.image}}" alt="Prank callers">
    <figcaption>
    Regular Show - Caffeinated Concert Tickets, by J.G. Quintel
    </figcaption>
</figure>
