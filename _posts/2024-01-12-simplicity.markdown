---
layout: post
title:  "Simplicity and/or easy"
date:   2024-01-12 23:35:00
tags: miscellaneous
comments: true
author: alemdomarco
preview: Simplicity
---

Linguagem é uma das ferramentas mais importantes no mundo, e sem ela, seria impossível nos comunicarmos de forma efetiva. 
Mesmo com ela, ainda enfrentamos problemas de clareza e definição. Neste post, quero discutir o significado de duas palavras: simplicidade e facilidade, palavras que são amplamente utilizadas em software de maneira não clara, o que acaba nos levando a problemas de manutenção no futuro.

Primeiramente, o que significa algo ser simples ou até mesmo fácil? Por definição, temos uma diferença:

Simples: Está relacionado a algo que não é complexo ou elaborado.
Fácil: Está relacionado à facilidade de fazer ou aprender algo sem muito esforço.
Assim como no mundo real, elas são usadas de forma similar no mundo do software, frequentemente de maneira intercambiável, mesmo tendo significados distintos. Essa prática leva à confusão e nos afasta de um software mais sustentável. Nada melhor para demonstrar o que estou escrevendo do que um exemplo:


Vamos supor que eu tenha um programa que receba por uma entrada qualquer um usuario com os seguintes dados obrigatorios
Nome
Sobrenome
Idade

Nesses dados, precisamos fazer as seguintes validações:

- Nome e sobrenome devem conter, no mínimo, 2 e, no máximo, 100 caracteres.
- Nome e sobrenome devem conter, no mínimo, 2 e, no máximo, 50 caracteres.
- O usuário precisa ser maior de 18 anos.

{% highlight csharp %}
public class UserService
{
    
    private readonly UserRepository _userRepository;
    public UserService(UserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public void Save(NewUserDto newUserDto)
    {
        if (string.IsNullOrWhiteSpace(newUserDto.Name))
        {
            throw new ArgumentException("Name is required");
        }
        if (newUserDto.Name.Length is 100 or < 2)
        {
            throw new ArgumentException("Name must be between 2 and 100 characters");
        }
        if (string.IsNullOrWhiteSpace(newUserDto.LastName))
        {
            throw new ArgumentException("Last name is required");
        }
        if (newUserDto.LastName.Length is > 50 or < 2)
        {
            throw new ArgumentException("Last name must be between 2 and 100 characters");
        }
        if (string.IsNullOrWhiteSpace(newUserDto.Age))
        {
            throw new ArgumentException("Age is required");
        }
        if (!int.TryParse(newUserDto.Age, out var age))
        {
            throw new ArgumentException("Age must be a number");
        }
        if (age < 18)
        {
            throw new ArgumentException("Age must be greater than 18");
        }
        _userRepository.Save(newUserDto.Name, newUserDto.LastName, age);
    }
}
{% endhighlight %}

Vamos começar exercitando o aspecto simples da questão: este serviço é bem simples de ser lido. Ele verifica todos os valores recebidos conforme as regras estabelecidas, e pronto, estamos com o software funcionando. Dependendo da empresa, ainda podem sugerir que você coloque esse código em uma classe de 'negócio' ou 'domínio', o que, na prática, não muda muita coisa além de remover a dependência do repositório no nosso cenário como mostrado abaixo:

{% highlight csharp %}
public class User
{
    private readonly string? _name;
    private readonly string? _lastName;
    private readonly string? _age;
    
    public User(string? name, string? lastName, string? age)
    {
        _name = name;
        _lastName = lastName;
        _age = age;
    }

    public string Name()
    {
        if (string.IsNullOrWhiteSpace(_name))
        {
            throw new ArgumentException("Name is required");
        }
        if (_name.Length is 100 or < 2)
        {
            throw new ArgumentException("Name must be between 5 and 100 characters");
        }
        return _name;
    }
    
    public string LastName()
    {
        if (string.IsNullOrWhiteSpace(_lastName))
        {
            throw new ArgumentException("Last name is required");
        }
        if (_lastName.Length is > 50 or < 2)
        {
            throw new ArgumentException("Last name must be between 5 and 100 characters");
        }
        return _lastName;
    }
    
    public int Age()
    {
        if (string.IsNullOrWhiteSpace(_age))
        {
            throw new ArgumentException("Age is required");
        }
        if (!int.TryParse(_age, out var age))
        {
            throw new ArgumentException("Age must be a number");
        }
        if (age < 18)
        {
            throw new ArgumentException("Age must be greater than 18");
        }
        return age;
    }
}
{% endhighlight %}


{% highlight csharp %}
public class UserService
{
    
    private readonly UserRepository _userRepository;
    public UserService(UserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public void Save(NewUserDto newUserDto)
    {
        var user = new User(newUserDto.Name, newUserDto.LastName, newUserDto.Age);
        _userRepository.Save(user);
    }
}
{% endhighlight %}

Já que estamos acostumados a ver código dessa maneira, ele parece ser fácil de manter, embora 'fácil' seja um termo relativo, variando de pessoa para pessoa de acordo com suas experiências. Vamos dar uma olhada nos testes com a classe de domínio.

{% highlight csharp %}
public class UserTest
{
    [Fact]
    public void ShouldThrowExceptionWhenNameIsNotProvided()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("", "lastName", "18");
            return user.Name();
        });
        Assert.Equal("Name is required", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenNameIsLessThan2Characters()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
                var user = new User("J", "lastName", "18");
                return user.Name();
        });
        Assert.Equal("Name must be between 2 and 100 characters", exception.Message);
    }

    [Fact]
    public void ShouldThrowExceptionWhenNameIsMoreThan100Characters()
    {
        var builder = new StringBuilder(101);
        builder.Append("test");
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User(builder.ToString(), "lastName", "18");
            return user.Name();
        });
        Assert.Equal("Name must be between 2 and 100 characters", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenLastNameIsNotProvided()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "", "18");
            return user.LastName();
        });
        Assert.Equal("Last name is required", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenLastNameIsLessThan2Characters()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "D", "18");
            return user.LastName();
        });
        Assert.Equal("Last name must be between 2 and 100 characters", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenLastNameIsMoreThan50Characters()
    {
        var builder = new StringBuilder(51);
        builder.Append("test");
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", builder.ToString(), "18");
            return user.LastName();
        });
        Assert.Equal("Last name must be between 2 and 100 characters", exception.Message);
    }

    [Fact]
    public void ShouldThrowExceptionWhenLastAgeIsNotProvided()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "Doe", "");
            return user.Age();
        });
        Assert.Equal("Age is required", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenAgeIsNotANumber()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "Doe", "test");
            return user.Age();
        });
        Assert.Equal("Age must be a number", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenAgeIsLessThan18()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "Doe", "17");
            return user.Age();
        });
        Assert.Equal("Age must be greater than 18", exception.Message);
    }
}
{% endhighlight %}

Não podemos negar que separar em métodos tornou tudo visualmente mais fácil, evitando toda a confusão dos condicionais.
Poderíamos ter realizado os testes antes da implementação, mas desta forma, facilita o entendimento e é importante destacar que o foco do artigo não é o TDD, pois criar os testes antecipadamente provavelmente nos levaria a uma etapa de design, o que não é o objetivo aqui. 
Ao reexaminarmos o código, notamos uma complexidade inerente a todos os valores, como a necessidade de verificar a existência do texto, tamanho dos campos, conversao para inteiro e garantia da maioridade. Esta complexidade permanecerá, independentemente das mudanças no código, pois todas essas regras são essenciais para o funcionamento adequado do software. Mesmo 'escondendo-as' na classe de domínio, elas ainda existem. Portanto, por enquanto, podemos concluir que a classe e os testes estão simplificados e faceis de serem lidos, pois, estruturalmente, seguem uma lógica imperativa sem indireções, supondo que até mesmo um desenvolvedor com pouca experiência conseguirá entender o que está acontecendo.
Continuando nossa jornada, vamos precisar adicionar um novo campo a pedido da empresa:
- Apelido e deve conter no mínimo, 5 e, no máximo, 50 caracteres.

{% highlight csharp %}
 public string Nickname()
    {
        if (string.IsNullOrWhiteSpace(_nickName))
        {
            throw new ArgumentException("Nickname is required");
        }
        if (_nickName.Length is > 50 or < 5)
        {
            throw new ArgumentException("Nickname must be between 2 and 100 characters");
        }
        return _nickName;
    }
{% endhighlight %}

{% highlight csharp %}
    [Fact]
    public void ShouldThrowExceptionWhenNicknameIsNotProvided()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "Doe", "18", "");
            return user.NickName();
        });
        Assert.Equal("Nickname is required", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenNicknameIsLessThan2Characters()
    {
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "Doe", "18", "J");
            return user.NickName();
        });
        Assert.Equal("Nickname must be between 2 and 50 characters", exception.Message);
    }
    
    [Fact]
    public void ShouldThrowExceptionWhenNicknameIsMoreThan50Characters()
    {
        var builder = new StringBuilder(51);
        builder.Append("test");
        var exception = Assert.Throws<ArgumentException>(() =>
        {
            var user = new User("John", "Doe", "18", builder.ToString());
            return user.NickName();
        });
        Assert.Equal("Nickname must be between 5 and 50 characters", exception.Message);
    }
{% endhighlight %}

Recapitulando, temos uma classe de domínio `User` com todas as regras e uma classe correspondente de testes. Aos poucos, podemos perceber como a classe pode crescer com cada novo campo adicionado, bem como as repetições de código, como a verificação da validade da string e o número de caracteres. Imagine aumentar isso para 20 classes com 5 atributos cada, interagindo entre si. Você consegue imaginar a repetição de código?

Apesar de a leitura continuar relativamente fácil, as mudanças de código vão se tornando mais difíceis, pois os impactos de uma simples mudança aumentam a cada interação.

Também quero pontuar que, sempre que abrimos nossa classe de domínio, todas as regras que vemos são carregadas em nosso cérebro antes de efetuar qualquer alteração, o que ao longo do dia pode nos levar ao que chamam de `sobrecarga cognitiva`, onde nossa capacidade de focar e tomar decisões fica comprometida devido à quantidade de informações que precisamos armazenar antes de cada tarefa.

Emprestando uma analogia de Rich Hickey, pense em um malabarista: quantas bolinhas uma pessoa, em média, consegue manter no ar? 3, 5, 7? Segundo a Wikipedia, o campeão mundial consegue 14. Existe um limite físico em manter bolinhas no ar com apenas dois braços, e o mesmo limite se aplica ao nosso cérebro ao carregar e processar informações. Portanto, quanto mais informações carregarmos no cérebro, por mais simples que sejam, ainda assim eventualmente chegaremos no limite.

Mas e agora? A complexidade nas regras ainda existe e precisa estar em algum lugar para que o software continue funcionando. Então, como podemos melhorar esse cenário?

Vamos tentar mexer um pouco na estrutura e pensar em uma solução. Podemos começar criando uma classe para resolver um dos problemas. Tentemos sentir o problema na ponta dos dedos, ahahaha. Começaremos com uma classe chamada Nome, que cuidará da validação do número de caracteres do nome e garantirá que o nome exista, realocando o código que está no User para outro lugar.

{% highlight csharp %}
public class Name
{
    private readonly string? _name;
    
    public Name(string? name)
    {
        _name = name;
    }
    public string Value()
    {
        if (string.IsNullOrWhiteSpace(_name))
        {
            throw new ArgumentException("Name is required");
        }
        if (_name.Length is 100 or < 5)
        {
            throw new ArgumentException("Name must be between 2 and 100 characters");
        }
        return _name;
    }
}
{% endhighlight %}

Agora, podemos instanciar a classe Nome na classe de domínio e, com isso, teremos uma classe dedicada exclusivamente à validação do nome. Infelizmente, agora nossa classe de domínio está fortemente acoplada à implementação da classe Nome, o que pode dificultar os testes na classe de domínio.

{% highlight csharp %}
public class User
{
    private readonly Name _name;
    private readonly string? _lastName;
    private readonly string? _age;
    private readonly string? _nickName;
    
    public User(Name name, string? lastName, string? age, string? nickName)
    {
        _name = name;
        _lastName = lastName;
        _age = age;
        _nickName = nickName;
    }

    public string Name()
    {
        return _name.Value();
    }
{% endhighlight %}

Para resolver esse problema, vamos criar uma interface para a classe Nome e passá-la pelo construtor, utilizando a boa e velha injeção de dependência.

{% highlight csharp %}
public class User
{
    private readonly IName _name;
    private readonly string? _lastName;
    private readonly string? _age;
    private readonly string? _nickName;
    
    public User(IName name, string? lastName, string? age, string? nickName)
    {
        _name = name;
        _lastName = lastName;
        _age = age;
        _nickName = nickName;
    }

    public string Name()
    {
        return _name.Value();
    }
{% endhighlight %}

Acabei de notar que o mesmo conjunto de regras é muito parecido com o do Sobrenome. Então, vamos copiar essas regras.

{% highlight csharp %}
public class LastName: ILastName
{
    private readonly string? _lastName;
    
    public LastName(string? lastName)
    {
        _lastName = lastName;
    }
    public string Value()
    {
        if (string.IsNullOrWhiteSpace(_lastName))
        {
            throw new ArgumentException("Last name is required");
        }
        if (_lastName.Length is 50 or < 5)
        {
            throw new ArgumentException("Last name must be between 2 and 100 characters");
        }
        return _lastName;
    }
}
{% endhighlight %}


Copiando a classe Nome e renomeando-a para Sobrenome conseguimos remover mais uma lógica da classe User. Reavaliando Nome e Sobrenome percebo que elas têm muitos elementos em comum (praticamente tudo), sugerindo que talvez exista outra abstração a ser descoberta.
E se criássemos uma classe responsável por validar o tamanho inicial e final de uma string? Parece uma boa ideia. Vamos tentar implementá-la.

{% highlight csharp %}
public class LimitedString
{
    private readonly string? _value;
    private readonly int _minLength;
    private readonly int _maxLength;

    public LimitedString(string? value, int minLength, int maxLength)
    {
        _value = value;
        _minLength = minLength;
        _maxLength = maxLength;
    }

    public string Value()
    {
        if (string.IsNullOrWhiteSpace(_value))
        {
            throw new ArgumentException("Value is required");
        }
        if (_value.Length > _maxLength || _value.Length < _minLength)
        {
            throw new ArgumentException($"Value must be between {_minLength} and {_maxLength} characters");
        }
        return _value;
    }
}
{% endhighlight %}

Agora que temos uma classe que limita uma string posso utiliza-la nas classes Nome e Sobrenome

{% highlight csharp %}
public class Name: IName
{
    private readonly LimitedString limitedString;
    
    public Name(string? name)
    {
        limitedString = new LimitedString(name, 5, 100);
    }
    public string Value()
    {
        return limitedString.Value();
    }
}
{% endhighlight %}

Agora, enfrentamos um problema de acoplamento, no qual as classes Nome e Sobrenome dependem de uma implementação concreta. Para resolver esse problema, precisamos criar uma interface que represente uma string.

{% highlight csharp %}
public interface IText
{
    string Value();
}
{% endhighlight %}

Agora, as nossas classes Nome e Sobrenome podem receber a interface no construtor, em vez de LimitedString, que agora implementa a interface.

public class Name: IName
{
    private readonly IText _text;
    public Name(IText text)
    {
        _text = text;
    }

    public string Value()
    {
        return _text.Value();
    }
}


Neste cenário, temos que passar uma implementação de IText, ou seja, precisamos criar uma instância de LimitedString. Para facilitar nossa vida, vamos criar um construtor secundário que sempre chame o nosso construtor principal, fornecendo alguns dados pré-definidos ('hard-coded'), o que é tranquilo, já que ninguém mais utiliza a classe Name.

{% highlight csharp %}
public class Name : IName
{
    private readonly IText _text;

    public Name(IText text) // primary constructor
        _text = text;
    }

    public Name(string? name) : this(new LimitedString(name, 2, 100)) {} //secondary constructor

    public string Value()
    {
        return _text.Value();
    }
}
{% endhighlight %}

{% highlight csharp %}
public class LastName: ILastName
{
    private readonly IText _text;
    
    public LastName(IText text)
    {
        _text = text;
    }
    public LastName(string? lastName) : this(new LimitedString(lastName, 2, 50)) {}

    public string Value()
    {
        return _text.Value();
    }
}
{% endhighlight %}

Vamos parar um pouco e refletir o que fizemos:

Vamos parar um pouco e refletir sobre o que fizemos:

A classe de domínio User ficou mais complexa, pois agora temos abstrações que introduzem indireção, o que desvia positivamente o nosso foco dos detalhes da implementação. Por exemplo, em vez de nos concentrarmos nas regras específicas, sabemos que as classes correspondentes contêm essas regras internamente, e isso é suficiente. A menos que queiramos mudar uma regra, não há razão para nos aprofundarmos em nenhuma dessas classes. Estamos, aos poucos, transitando para um código mais declarativo.

Está mais fácil? Como isso é muito subjetivo, acho difícil chegar a uma conclusão definitiva. Acredito que ficou mais fácil entender o que está acontecendo de modo geral, sem entrar nos detalhes da implementação, o que em excesso sobrecarregaria nossas mentes.

Vamos conferir como ficou o UserService.

{% highlight csharp %}
public class UserService
{
    
    private readonly UserRepository _userRepository;
    public UserService(UserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public void Save(NewUserDto newUserDto)
    {
        var user = new User(
            new Name(newUserDto.Name), 
            new LastName(newUserDto.LastName), 
            newUserDto.Age, 
            newUserDto.NickName
        );
        _userRepository.Save(user);
    }
}
{% endhighlight %}

Nada mal. Antes de concluir, ainda gostaria de explorar mais um campo, onde é necessário converter o valor de uma string para número a fim de verificar a idade mínima. Vamos criar uma classe que represente essa lógica, a qual chamaremos de Age.

{% highlight csharp %}
public class Age: IAge
{
    private readonly string? _age;

    public Age(string? age)
    {
        _age = age;
    }

    public int Value()
    {
        if (string.IsNullOrWhiteSpace(_age))
        {
            throw new ArgumentException("Age is required");
        }
        if (!int.TryParse(_age, out var age))
        {
            throw new ArgumentException("Age must be a number");
        }
        if (age < 18)
        {
            throw new ArgumentException("Age must be greater than 18");
        }
        return age;
    }
}
{% endhighlight %}

{% highlight csharp %}
public interface IAge
{
    int Value();
}
{% endhighlight %}

A primeira coisa a fazer é remover a dependência da implementação concreta em nossa classe de negócio.

{% highlight csharp %}
public class User
{
    private readonly IName _name;
    private readonly ILastName _lastName;
    private readonly IAge _age;
    private readonly string? _nickName;
    
    public User(IName name, ILastName lastName, IAge age, string? nickName)
    {
        _name = name;
        _lastName = lastName;
        _age = age;
        _nickName = nickName;
    }
    
    public User(string? name, string? lastName, string? age, string? nickName): 
        this(new Name(name), new LastName(lastName), new Age(age), nickName) {}

    public string Name()
    {
        return _name.Value();
    }
    
    public string LastName()
    {
        return _lastName.Value();
    }
    
    public int Age()
    {
        return _age.Value();
    }

    ...
}
{% endhighlight %}

{% highlight csharp %}
public class UserService
{
    
    private readonly UserRepository _userRepository;
    public UserService(UserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public void Save(NewUserDto newUserDto)
    {
        var user = new User(
            new Name(newUserDto.Name), 
            new LastName(newUserDto.LastName), 
            new Age(newUserDto.Age), 
            newUserDto.NickName
        );
        _userRepository.Save(user);
    }
}
{% endhighlight %}


É curioso observar como a classe User começa a ficar cada vez mais anêmica, ou até mesmo perder o sentido, de certa forma. Vamos dar um passo para trás e pensar: temos uma string que está sendo convertida para inteiro, seguida de uma comparação de idade. Em um projeto real, poderíamos deixar as coisas como estão, pois não há motivo para mexer na classe neste momento. Não existe uma necessidade real para uma mudança no código, como a duplicação que nos motivou a mudar o Nome e Sobrenome. Normalmente, tentar antecipar o futuro não dá muito certo, porém...

Consultando as 'cartas', percebo que, no futuro, outros números também precisarão ser convertidos de string para inteiro. Utilizando essa informação 'mística', podemos pensar em uma solução para esse problema. Primeiramente, precisamos validar se a string existe, e isso já está acontecendo em todas as classes que manipulam texto o que normalmente pede uma nova classe.

{% highlight csharp %}
public class NotNullOrEmptyText: IText
{
    private readonly string? _value;
    
    public NotNullOrEmptyText(string? value)
    {
        _value = value;
    }
    public string Value()
    {
        if (string.IsNullOrWhiteSpace(_value))
        {
            throw new ArgumentException("Value is required");
        }
        return _value;
    }
}
{% endhighlight %}

Com essa nova implementação, podemos simplificar outras classes, como a LimitedText, aplicando a mesma lógica de validação.

{% highlight csharp %}
public class LimitedText: IText
{
    private readonly IText _text;
    private readonly int _minLength;
    private readonly int _maxLength;

    public LimitedText(IText text, int minLength, int maxLength)
    {
        _text = text;
        _minLength = minLength;
        _maxLength = maxLength;
    }
    
    public LimitedText(string? value, int minLength, int maxLength) : 
        this(new NotNullOrEmptyText(value), minLength, maxLength) {}

    public string Value()
    {
        var value = _value.Value();
        if (value.Length > _maxLength || value.Length < _minLength)
        {
            throw new ArgumentException($"Value must be between {_minLength} and {_maxLength} characters");
        }
        return value;
    }
}
{% endhighlight %}

E tambem atualizar a nossa classe `Age`

{% highlight csharp %}
public class Age: IAge
{
    private readonly IText _text;

    public Age(IText text)
    {
        _text = text;
    }
    
    public Age(string? age) : this(new NotNullOrEmptyText(age)) {}

    public int Value()
    {
        var value = _text.Value();
        if (!int.TryParse(value, out var age))
        {
            throw new ArgumentException("Age must be a number");
        }
        if (age < 18)
        {
            throw new ArgumentException("Age must be greater than 18");
        }
        return age;
    }
}
{% endhighlight %}

Agora, precisamos de uma maneira de converter esse Text em algo diferente. Para facilitar nossa linha de raciocínio, vamos criar primeiro uma classe focada na conversão, que chamaremos de TextToNumber.

{% highlight csharp %}
public class TextToNumber
{
    private readonly IText _text;

    public TextToNumber(IText text)
    {
        _text = text;
    }

    public TextToNumber(string? text) : this(new NotNullOrEmptyText(text)) {}

    public int Value()
    {
        var value = _text.Value();
        if (!int.TryParse(value, out var age))
        {
            throw new ArgumentException("Age must be a number");
        }
        return age;
    }
}
{% endhighlight %}

Que, obviamente, vai precisar retornar um novo tipo que vou chamar de INumber.

{% highlight csharp %}
public interface INumber
{
    int Value();
}
{% endhighlight %}

Atualizando TextToNumber

{% highlight csharp %}
public class TextToNumber:INumber
{
    private readonly IText _text;

    ...
}
{% endhighlight %}

A classe de servico e dominio continuam as mesmas

{% highlight csharp %}
public class UserService
{
    
    private readonly UserRepository _userRepository;
    public UserService(UserRepository userRepository)
    {
        _userRepository = userRepository;
    }
    
    public void Save(NewUserDto newUserDto)
    {
        var user = new User(
            new Name(newUserDto.Name), 
            new LastName(newUserDto.LastName), 
            new Age(newUserDto.Age), 
            newUserDto.NickName
        );
        _userRepository.Save(user);
    }
}
{% endhighlight %}

{% highlight csharp %}
    public int Age()
    {
        return _age.Value();
    }
    
    public string NickName()
    {
        if (string.IsNullOrWhiteSpace(_nickName))
        {
            throw new ArgumentException("Nick name is required");
        }
        if (_nickName.Length is > 50 or < 5)
        {
            throw new ArgumentException("Nick name must be between 2 and 100 characters");
        }
        return _nickName;
    }
{% endhighlight %}

Acho que já deu para ter uma boa ideia de onde quero chegar com isso. Com essas pequenas classes, podemos reaproveitar boa parte do código, o que, no futuro, vai direcionar nosso foco para longe da implementação e mais próximo do negócio. Eu sei que essa abordagem não é comum, mas espero ter aguçado sua curiosidade para entender o que está acontecendo e como podemos utilizar a mesma estratégia para outros tipos, criando abstrações que realmente representam algo significativo. Com essas poucas classes, conseguimos validar se uma string é nula, seu tamanho, e também podemos convertê-la para inteiro sem precisar criar mais nada. Esses componentes podem ser utilizados e estendidos por todo o sistema, tornando o código, no futuro, muito fácil de ser escrito, onde a composição é a chave.

Ser simples não significa que o código será fácil de ser modificado ou suportar novas funcionalidades, e ser complicado também não significa o oposto. É importante ponderar os trade-offs e, ao criar uma abstração, perceber se ela trará mais benefícios do que a indireção produzida. Escrever código de modo automático só pode nos levar a um código com custo alto de manutenção a médio/longo prazo. A complexidade sempre vai existir, e como lidamos com ela é crucial para um software com vida longa que ira trazer alegria no dia a dia.