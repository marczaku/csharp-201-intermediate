# 1 Type-Introspection

Type-Introspection is an extremely powerful tool that allows your code to work with types that it doesn't even know of. It comes automatically with .NET, is called `System.Reflection` and it provides Type-Information for every single Type and Object Instance of that Type.

# 2 Usage
- Most obvious: To find out, whether a class has a certain type or inherits from a certain type or implements a certain interface. Keywords like `is` use Type-Introspection under the hood.
- Just as obvious: Safely casting the Type. Keywords like `as` use Type-Introspection under the hood.
- Showing Editors, like in Unity. This is how Unity knows what fields to Display in the Inspector. You could also build a Remote Inspector this way.
- Serializing or Deserializing any class.
- Scanning the codebase for all classes implementing a certain Interface, to automatically load them or show them in the UI.
- Building a Visual Scripting Editor that can call your methods or change you Properties.
- Dangerous: To access private fields and methods. This is of course not recommended, but sometimes you find a class that's almost perfect. But there's also this one method that is `private` that you'd like to use.
- To call methods or constructors of unknown Types. e.g. for an In-Game Console or for using Dependency Injection.

# 3 GetType()

The Method `System.Type GetType();` is defined on the `object`-Baseclass and therefor usable on any object in C#:

```cs
using System;

int a = 5;
string name = "Hi!";

Console.WriteLine(a.GetType());
Console.WriteLine(name.GetType());
```

Output:
```
System.Int32
System.String
```

What do you think is the Result here?

```cs
using System;

Dog dog = new Dog();
Animal animal = new Dog();
object obj = new Dog();

Console.WriteLine(dog.GetType());
Console.WriteLine(animal.GetType());
Console.WriteLine(obj.GetType());

class Animal{}
class Dog : Animal{}
```

Output:
```
Dog
Dog
Dog
```

Okay cool, so this way, we can get the REAL Type of an object. Not only the base-Type that we know of.

# 4 typeof

`typeof()` is a useful Keyword which can give you the `System.Type` of a Type that's known during compile-time. Look at this example:

```cs
Animal animal = new Dog();
Console.WriteLine(animal.GetType() == typeof(Dog));
Console.WriteLine(animal.GetType() == typeof(Animal));

class Animal{}
class Dog : Animal{}
```

Output:
```
True
False
```

Two things learned here: Into `typeof`, we can pass the name of a Type and it will return the `System.Type` of that Type.

And, we can compare `System.Type` and it returns `true` only, if it's exactly the same Type. It's not enough if there's a common base class. In comparison, if you check the Type with `is`, it returns `true` both times:

```cs
Animal animal = new Dog();
Console.WriteLine(animal is Dog);
Console.WriteLine(animal is Animal);

class Animal{}
class Dog : Animal{}
```

Output:
```
True
True
```

This is by the way, how Methods were made Generic before Generic Methods existed:

```cs
gameObject.GetComponent(typeof(Health));
```

Instead of:

```cs
gameObject.GetComponent<Health>();
```

# 5 nameof

Another very useful feature is `nameof`, especially when using Reflection or displaying output to the user. Look at this example:

```cs
void Attack(Enemy enemy) {
    if(enemy == null) {
        Console.WriteLine($"Error: '{nameof(enemy)}' can not be null!");
    }
}
```

Output:
```
Error: 'enemy' can not be null!
```

Well, this was a very complicated way of writing this:

```cs
void Attack(Enemy enemy) {
    if(enemy == null) {
        Console.WriteLine($"Error: 'enemy' can not be null!");
    }
}
```

Why go the long way, then? Because the long way will cause errors, if the parameter was to be removed or renamed. This makes sure, that you don't change the code at some point and make it look like this:

```cs
void Attack(Unit unit) {
    if(unit == null) {
        Console.WriteLine($"Error: 'enemy' can not be null!");
    }
}
```

Now, the Log shows wrong information. There is no parameter named `enemy` anymore. If you use `nameof`, IDEs will actually automatically rename `enemy` to `unit` for you. This is such a useful feature, that I'll always strike you in Code Review, if I see you put the name of a class or field or parameter into a string without using `nameof`.

# 6 Let's Find all Types that Implement `IEnumerable`

```cs
using System;
using System.Collections;
using System.Linq;

var allEnumerableTypes = 
    // All Assemblies that are currently loaded
    AppDomain.CurrentDomain.GetAssemblies()
    // Select from each assembly all types
    .SelectMany(assembly => assembly.GetTypes())
    // Filter them by types which can be assigned to `IEnumerable`
    .Where(type => type.IsAssignableTo(typeof(IEnumerable)))
    // Filter them by types which are not abstract (so we don't get interfaces or abstract base classes)
    .Where(type => !type.IsAbstract);

Console.WriteLine($"Total Amount of Enumerables: {allEnumerableTypes.Count()}");
foreach (var type in allEnumerableTypes)
{
    Console.WriteLine(type);
}
```

Output:
```
Total Amount of Enumerables: 106
System.String
System.ArraySegment`1[T]
```

Usage:
You can for example find all classes implementing `IGame` and then have a Main Menu that automatically scans the solution for all classes implementing `IGame`. Or you could scan for all classes implementing `IDifficulty` or so.

# 7 Let's find out all the fields that a Type has

```cs
using System.Linq;
using System.Reflection;

var propertyInfos = 
    gameObject
    .GetType()
    .GetProperties();

Debug.Log($"Found a total of {propertyInfos.Count()} {nameof(propertyInfos)}");
Debug.Log(string.Join("\n", propertyInfos.Select(GetDescription)));

string GetDescription(PropertyInfo propertyInfo)
{
    return $"Found {nameof(propertyInfo)} {propertyInfo.Name} of Type {propertyInfo.PropertyType.FullName}";
}
```

Output:
```
Found a total of 25 propertyInfos:
Found propertyInfo rigidbody of Type UnityEngine.Component
Found propertyInfo rigidbody2D of Type UnityEngine.Component
Found propertyInfo camera of Type UnityEngine.Component
...
```

Usage:
You can write tools, editors, inspectors, serializers or deserializes using this Method. For example convert any C# class to a text-file and load it from a text-file again.

# 8 Let's get sneaky and access something private

```cs
using System;
using System.Reflection;

Unit unit = new Unit();
unit.Health = 200;
Console.WriteLine(unit.Health); // 100
// Getting the Field Info of a private instance member:
var fieldInfo = unit.GetType().GetField("health", BindingFlags.NonPublic | BindingFlags.Instance);
// Using the Field Info to set a value on a unit:
fieldInfo.SetValue(unit, 200);
Console.WriteLine(unit.Health); // 200
// Using the Field Info to get the value from a unit:
int health = (int)fieldInfo.GetValue(unit);

class Unit
{
    private int health;

    public int Health
    {
        get => health;
        set => health = Math.Clamp(value, 0, 100);
    }
}
```

Output:
```
100
200
```

Well, here, we've used it for pure evil. And also in general, it's not a good idea to set private values of a class. But sometimes it might look like this is the easiest way of solving a problem. If you use a really good Physics Engine that for some reason has not made a really important value accessible to you, you might as well force access instead of switching the whole Physics Engine :)

# 9 More

There is many other use-cases, like finding a constructor and using it to construct an object, or putting objects in a dictionary, sorted by their types or many, many more.
