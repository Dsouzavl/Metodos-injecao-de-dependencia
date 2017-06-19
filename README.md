# Maneiras de realizar a _Injeção de Dependência_

### Escrito por Victor Souza A.k.a _vhost_

Injeção de dependência, é uma prática que tem a finalidade de remover o alto acoplamento entre as classes causados por dependências que são resolvidas de forma _hard-coded_ na classe. Por exemplo, pense que você tem uma classe assim:

```csharp
public class MyClass
{
    public int PropertyOne { get; set; }

    public OtherClass OtherClass = new OtherClass;

    public void MyMethod(string someParameter)
    {
        OtherClass.DoSomethingWith(someParameter);
    }
}
```
Podemos dizer que **MyClass** tem um alto acoplamento a **OtherClass**, pois ele resolve a dependência que ele tem da **OtherClass** diretamente dentro da sua classe, pense que ocorreu uma alteração no método **DoSomethingWith()**, teríamos que alterar nessa classe também, ou e se quiséssemos utilizar outro objeto que tem o método **DoSomethingWith()**, mas também tem o método **DoOtherThingWith()** que eu também preciso na minha classe, eu teria que alterar na **MyClass**, pois ela está diretamente acoplada ao objeto específico, e isso viola o **_Open/Closed principle_** ( Para saber mais sobre os princípios de SOLID leia meu [**Artigo no GitHub**](https://github.com/Dsouzavl/SOLID-o-que-significa)). 

Precisamos resolver esse problema, e resolvemos ele através da _Injeção de Dependência_, há 3 maneiras de realizar a _Injeção de Dependência_ que vou apresentar nesse artigo: 

1) Injetando a dependência através do construtor da Classe.
2) Injetando a dependência através dos _Getters_ e _Setters_ da Classe.
3) Definindo interfaces para injetar a dependência.


## Injeção de dependência através do construtor.

Uma maneira de resolvermos a dependência é criarmos uma interface, que vai definir uma separação entre o contrato e a implementação concreta do método que eu preciso, e após fazer isso, passamos o tipo essa interface como parâmetro do construtor da nossa classe, ao fazer isso, podemos criar _N_ elementos que a implementam, e ficamos livres para passar, qualquer objeto que implemente a interface para o construtor da nossa classe, ao invés de termos a classe apontando para o objeto específico, desse modo conseguimos resolver a nossa dependência, mas com um acoplamento reduzido. Observe:

```csharp
public interface IObjThatIWantToUse
{
    void DoSomethingWith(string someParameter);

    void DoOtherThingWith(string someOtherParameter);
}
```
Criamos a interface que os objetos que temos dependência vão utilizar, e agora criamos as outras classes.

```csharp
public class OtherClass : IObjThatIWantToUse
{
    public void DoSomethingWith(string someParameter)
    {
        //Do something absolutely awesome
    }

    public void DoOtherThingWith(string someOtherParameter)
    {
        //Do other thing absolutely awesome
    };
}
```

```csharp
public class OtherClassButMoreCool : IObjThatIWantToUse
{
    public void DoSomethingWith(string someParameter)
    {
        //Do something absolutely awesome, better than OtherClass.
    }

    public void DoOtherThingWith(string someOtherParameter)
    {
        //Do other thing absolutely awesome, better than OtherClass.
    };
}
```
Para podermos de fato injetar a dependência, temos que fazer as seguintes alterações na **MyClass**:

```csharp
public class MyClass
{
    public int PropertyOne { get; set; }

    //Criamos uma propriedade privada, que guardará o valor do nosso Obj.
    private IObjThatIWantToUse ObjThatIWantToUse;

    //Criamos um construtor que recebe IObjThatIWantToUse como parâmetro.
    public MyClass(IObjThatIWantToUse ObjThatIWantToUse)
    {
        //Setamos o valor da propriedade para o obj que recebemos por parâmetro.
        this.ObjThatIWantToUse = ObjThatIWantToUse;
    }

    public void MyMethod(string someParameter)
    {
        ObjThatIWantToUse.DoSomethingWith(someParameter);
    }
}
```
Perceba que dessa forma, podemos passar qualquer objeto que implementa a interface **IObjThatIWantToUse** para nossa classe, resolvemos a nossa dependência, pois continuamos a realizar o método **DoSomethingWith()**, mas não estamos presos a implementação concreta do obj, agora somos livres para fazer isso:
```csharp
class Program
    {
        static void Main(string[] args)
        {
            IObjThatIWantToUse objOne = new OtherClass();
            
            IObjThatIWantToUse objTwo = new OtherClassButMoreCool();
            
            MyClass classThatIWillUse = new MyClass(objTwo /* Or objOne*/);

            classThatIWillUse.MyMethod("Awesome string");
        }
    }
```
## Injeção de dependência através de _Getters_ e _Setters_.

A injeção ocorre seguindo parte do conceito da injeção através do construtor, pois ela mantém firme a idéia de utilizar uma interface para separar o contrato da implementação concreta, mas ao invés de passarmos o objeto para o construtor, passaremos ele como valor de uma propriedade da classe. Observe: 

```csharp
public class MyClass
{
    public int PropertyOne { get; set; }

    public OtherClass OtherClass = new OtherClass;

    public void MyMethod(string someParameter)
    {
        OtherClass.DoSomethingWith(someParameter);
    }
}
```
Aqui temos nossa classe altamente acoplada, resolvendo suas dependências _hard-coded_, o que teríamos que fazer é isso:


```csharp
public class MyClass
{
    public int PropertyOne { get; set; }

    private IObjThatIWantToUse _objThatIWantToUse;

    public IObjThatIWantToUse ObjThatIWantToUse
    {
        get{
            return _objThatIWantToUse;
        };

        set{
            this._objThatIWantToUse = value;
        };
    }

    public void MyMethod(string someParameter)
    {
        _objThatIWantToUse.DoSomethingWith(someParameter);
    }
}
```

```csharp
class Program
    {
        static void Main(string[] args)
        {
            IObjThatIWantToUse objOne = new OtherClass();
            
            IObjThatIWantToUse objTwo = new OtherClassButMoreCool();
            
            MyClass classThatIWillUse = new MyClass();

            classThatIWillUse.ObjThatIWantToUse = objOne //Or objTwo.

            classThatIWillUse.MyMethod("Awesome string");
        }
    }
```

Continuamos a poder passar qualquer objeto que implemente a nossa interface, porém ao invés resolvermos a dependência passando o objeto pelo construtor, passamos como valor de uma propriedade, que será manipulada dentro da classe. 

## Definindo interfaces para injetar a dependência.

Neste método, o conceito geral é: Criamos uma interface que contém um método para injetar a dependência, onde nossa classe que tem as dependências, vai implementar a interface e definir concretamente o método de injetar a dependência. Observe: 

```csharp
public class MyClass
{
    public int PropertyOne { get; set; }

    public OtherClass OtherClass = new OtherClass;

    public void MyMethod(string someParameter)
    {
        OtherClass.DoSomethingWith(someParameter);
    }
}
```
Novamente nossa classe altamente acoplada e com dependências resolvidadas de forma _hard-coded_.

```csharp
public interface DependencyResolver
{
    void ResolveDependency(IObjThatIWantToUse objThatIWantToUse);
}
```
Criamos uma interface com o método **InjectDependency()** que vamos implementar na nossa classe.

```csharp
public class MyClass : DependencyResolver
{
    public int PropertyOne { get; set; }

    private ObjThatIWantToUse _objThatIWantToUse;

    public void ResolveDependency(IObjThatIWantToUse objThatIWantToUse)
    {
        this._objThatIWantToUse = objThatIWantToUse
    }

    public void MyMethod(string someParameter)
    {
        _objThatIWantToUse.DoSomethingWith(someParameter);
    }
}
```
Desse modo, para resolver a dependência, o método **ResolveDependendy()** terá que ser chamado antes de **MyMethod()** para que possa ser realizado.

```csharp
class Program
    {
        static void Main(string[] args)
        {
            IObjThatIWantToUse objOne = new OtherClass();
            
            IObjThatIWantToUse objTwo = new OtherClassButMoreCool();
            
            MyClass classThatIWillUse = new MyClass();

            classThatIWillUse.ResolveDependency(objOne /*Or objTwo.*/)

            classThatIWillUse.MyMethod("Awesome string");
        }
    }
```
Espero que esse artigo lhe ajude a compreender um pouco mais sobre _Injeção de Dependência_, e que venha a calhar para que você possa  escrever um código melhor. 

Obrigado a todos os leitores!


