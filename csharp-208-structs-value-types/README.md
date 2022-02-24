# 1 Structs

Structs are used in C# to resemble Value-Types. Their Use-Cases usually involve some Mathematical Context or some Performance-Optimization reasons.

Structs are similar to classes:

```cs
public struct Unit {
  public string name;
  private int maxHealth;
  public int Health {get; set;}
  
  
  public void SetMaxHealth(int value) {
    this.maxHealth = value;
  }
}
```

They cannot inherit from other classes, but they can implement Interfaces.
- (Detail: It is often not recommended, as casting structs to interfaces leads to boxing.)
```cs
public interface IUnit {
  public int Health {get;}
}
public struct Unit : IUnit {
  public int Health {get; private set;}
}
```

They have limitations on their Constructors:

- You can not define a parameterless constructor.
- Your constructors must assign a value to all Fields and Auto-Properties.

```cs
public struct Unit {
  public string name;
  public int Health {get; set;}
  public string Name { set => this.name = value; }
  
  public Unit(string name) {
    this.name = name; // Assign a value to the field
    this.Health = 100; // Assign a value to the Property
    // Name does not need to be assigned, because it is not an Auto-Property
  }
```

Struct Variables and Fields can never be Null!

```cs
public struct Unit {
  public string name;
}

public class Game {
  public Unit unit;
  
  public Game() {
    // This will not throw a NullReferenceException:
    unit.name = "Hey!";
    unit = null;
  }
}
```

Therefore, you cannot assign null or compare a struct variable to null:

```cs
Unit unit;
unit = null; // ERROR: Can not assign `null`
if(unit == null) // ERROR: Operator `==` can not be applied to `Unit` and `null`
```

# 2 Value-Types

We have heard about Value-Types a few times now, but what does being a Value-Type Mean, really?

## 2.1 Call-by-value

It means, that when invoking Methods, or assigning a value to a variable, the value gets copied.

```cs
public struct unit {
  public string name;
}
```

```cs

void Main() {
  Unit vampire = new Unit();
  vampire.name = "Vampire";
  
  // When passing the unit as a method argument, a copy of the unit is created
  // And assigned to the method parameter
  ChangeUnitName(vampire);
  Console.WriteLine(vampire.name); // OUTPUT: Vampire
}

void ChangeUnitName(Unit unit) {
  Console.WriteLine(unit.name); // OUTPUT: Vampire
  unit.name = "Zombie";
  Console.WriteLine(unit.name); // OUTPUT: Zombie
}
```

Output:
```
Vampire
Zombie
Vampire
```

Here, the following happens:

Code-Line Executed | Variables
------------------ | -------------
Unit vampire = new Unit(); | vampire:Unit{name:null}
vampire.name = "Vampire"; | vampire:Unit{name:Vampire}
ChangeUnitName(vampire); | vampire:Unit{name:Vampire}, unit:Unit{name:Vampire}
Console.WriteLine(unit.name); | vampire:Unit{name:Vampire}, unit:Unit{name:Vampire}
unit.name = "Zombie"; | vampire:Unit{name:Vampire}, unit:Unit{name:Zombie}
Console.WriteLine(unit.name); | vampire:Unit{name:Vampire}, unit:Unit{name:Zombie}
{{ChangeUnitName end}} | vampire:Unit{name:Vampire}
Console.WriteLine(vampire.name); | vampire:Unit{name:Vampire}

The same goes for variables:

```cs
Unit unit = new Unit();
unit.name = "Vampire";
Console.WriteLine(unit.name); // Vampire
Unit unit2 = unit;
Console.WriteLine(unit2.name); // Vampire
unit2.name = "Zombie";
Console.WriteLine(unit.name); // Vampire
Console.WriteLine(unit2.name); // Zombie
```

## 2.2 Overview of Value-Types

Value-Types
- Numeric Types `int`, `double`, `float`, ...
- Basic Types `char`, `bool`
- Enum Types e.g. `public Enum WeaponTypes { Gun, Bow }`
- Structs e.g. `public struct Vector2 { public float x, y; }`

## 2.3 Value Types cannot be null

Again, we cannot assign null to Value-Types, and we cannot compare them to null:

```cs
int a = null; // Not possible
bool b = null; // Nope
```

## 2.4 Nullable Value Types

You can make value type fields or variables nullable, though, by using the `?` operator.

```cs
public class User {
  public int? age;
}
```

In this example, we assign `null` to the age field, if the user decides, not to give his age.\
This is a more elegant solution, than just assigning `0` or `-1` to his age:

```
User user = new User();
Console.WriteLine("Do you want to name your age?");
if(Console.ReadLine() == "y") {
  Console.WriteLine("What's your age?");
  user.age = Convert.ToInt32(Console.ReadLine());
} else {
  user.age = null;
}
```

Now, if you want to get the actual value of the field, you need to make sure, that it is not null and then cast it:

```cs
if(user.age != null) {
  Console.WriteLine("Next year, you will be " + (((int)user.age)+1) + " years old.");
}
```

Since this is ugly, C# has a more pretty solution again:

```cs
if(user.age.HasValue) {
  Console.WriteLine("Next year, you will be " + (user.Value + 1) + " years old.");
}
```

# 3 Reference Types

On the opposite side of Value-Types, we have Reference Types. What's the difference?

## 3.1 Call-by-Reference

Reference Types are passed as references rather than copies to variable assignments and as method arguments.\
Which means, that a reference to the same Unit gets copied.\
The best way of demonstrating this, is by demonstrating the same code as before.\
With only one change: The `Unit` this time is a `class` instead of a `struct`:

```cs
public class unit {
  public string name;
}
```

```cs

void Main() {
  Unit vampire = new Unit();
  vampire.name = "Vampire";
  
  // When passing the unit as a method argument, the unit is passed a reference.
  // Therefore, changes to the unit within that method, will also affect the unit
  // Here, outside the method:
  ChangeUnitName(vampire);
  Console.WriteLine(vampire.name); // OUTPUT: Zombie
}

void ChangeUnitName(Unit unit) {
  Console.WriteLine(unit.name); // OUTPUT: Vampire
  unit.name = "Zombie";
  Console.WriteLine(unit.name); // OUTPUT: Zombie
}
```

Output:
```
Vampire
Zombie
Zombie
```

The same goes for variables:

```cs
Unit unit = new Unit();
unit.name = "Vampire";
Console.WriteLine(unit.name); // Vampire
Unit unit2 = unit;
Console.WriteLine(unit2.name); // Vampire
unit2.name = "Zombie";
Console.WriteLine(unit.name); // Zombie
Console.WriteLine(unit2.name); // Zombie
```

However, the reference still gets copied. But instead of the whole Unit being copied / cloned, only a reference to the Unit is copied.\
We can break it, by assigning a reference to a new Unit to the parameter:

```cs

void Main() {
  Unit vampire = new Unit();
  vampire.name = "Vampire";
  
  // When passing the unit as a method argument, the unit is passed a reference.
  // Therefore, changes to the unit within that method, will also affect the unit
  // Here, outside the method:
  ChangeUnitName(vampire);
  Console.WriteLine(vampire.name); // OUTPUT: Zombie
}

void ChangeUnitName(Unit unit) {
  Console.WriteLine(unit.name); // OUTPUT: Vampire
  unit.name = "Zombie";
  Console.WriteLine(unit.name); // OUTPUT: Zombie
  unit = new Unit();
  unit.name = "Skeleton";
  Console.WriteLine(unit.name); // OUTPUT: Skeleton
}
```

Output:
```
Vampire
Zombie
Skeleton
Zombie
```

Another way of presenting what's happening in the program:

Code-Line Executed | Variables | Objects in Memory
------------------ | --------- | --------------------
Unit vampire = new Unit(); | vampire -> Unit#1 | Unit#1{name: null}
vampire.name = "Vampire"; | vampire -> Unit#1 | Unit#1{name: Vampire}
ChangeUnitName(vampire); | vampire -> Unit#1, unit -> Unit#1 | Unit#1{name: Vampire}
Console.WriteLine(unit.name); | vampire -> Unit#1, unit -> Unit#1 | Unit#1{name: Vampire}
unit.name = "Zombie"; | vampire -> Unit#1, unit -> Unit#1 | Unit#1{name: Zombie}
Console.WriteLine(unit.name); | vampire -> Unit#1, unit -> Unit#1 | Unit#1{name: Zombie}
unit = new Unit(); | vampire -> Unit#1, unit -> Unit#2 | Unit#1{name: Zombie}, Unit#2{name: null}
unit.name = "Skeleton"; | vampire -> Unit#1, unit -> Unit#2 | Unit#1{name: Zombie}, Unit#2{name: Skeleton}
Console.WriteLine(unit.name); | vampire -> Unit#1, unit -> Unit#2 | Unit#1{name: Zombie}, Unit#2{name: Skeleton}
{{ChangeUnitName end}} | vampire -> Unit#1 | Unit#1{name: Zombie}
Console.WriteLine(vampire.name); | vampire -> Unit#1 | Unit#1{name: Zombie}


## 3.2 Overview of Reference-Types

Value-Types
- Basic Types `string`, `object`
- Arrays e.g. `int[] numbers`
- Classes e.g. `public class House { public Color color; }`
- Delegates e.g. `public delegate void HealthDelegate(int newHealth);`

## 3.3 Reference Types can be null

Reference Types have the default value `null` and can have `null` assigned to them and can therefore also be compared to `null`:

```cs
string name = null;
int[] numbers = null;
```

# 4 Null

`null` is a bit of a Curse in C# and other Languages that have Nullable Types: If you try accessing any value that is `null`, your program will Crash.

```cs
public class Unit {
  public string name;
}
```

```cs
Unit unit = new Unit();
Console.WriteLine(unit.name.Length);
```

Boom, Crash! `System.NullReferenceException` - why?

## 4.1 Checking for Null

If you are not sure, whether a value is `null`, you should check for `null` first:

```cs
if(unit != null) {
  Attack(unit);
}
```

However, do not start adding `null`-checks everywhere in your code. If you do expect a value to not be `null`, don't just say "If it is null, do nothing." - This will lead to extremely hard to find bugs! Crash Early. Dead Programs Tell no Lies.

```cs
// Very often used, often wrongly:
if(unit == null) {
  return; // Please, only do this, if you really think that it's acceptable, that the Unit is `null`. Don't hide Errors.
}
```

There is a few Handy Operators for Null-Checks:

With the `??`-Operator, you can do null checks and create a new Value, if it is `null`:

```cs
return unit ?? new Unit();
```

Translates to:

```cs
if(unit != null) {
  return unit;
} else {
  return new Unit();
}
```

You can use `??=` very similarly:

```cs
Weapon weapon;
public void Attack() {
  weapon ??= new Hammer();
  weapon.Use();
}
```

Is the same as:

```cs
Weapon weapon;
public void Attack() {
  if(weapon == null) {
    weapon = new Hammer();
  }
  weapon.Use();
}
```

And now, to my favorite, the "Elvis-Operator" `?.`:

```cs
unit?.Attack();
```

Does the same as:

```cs
if(unit != null) {
  unit.Attack();
}
```

This can be combined in crazy ways, e.g.:

```cs
void UpdateHealthDisplay() {
  Player player = FindObjectOfType<Player>();
  int health = player?.health ?? 0;
}
```

Does basically the same as:

```cs
void UpdateHealthDisplay() {
  Player player = FindObjectOfType<Player>();
  int health;
  if(player != null) {
    health = player.health;
  } else {
    health = 0;
  }
}
```


# 5 Structs vs. Classes

So, why do both exist? And which one should you use?

- Structs have much better Performance.
  - This has to do with that Classes are stored on the Heap, which is dynamic object memory
  - Data stored on the Heap is usually slower in access
  - And generates Garbage, which needs to be cleaned up by the Garbage Collector
- Structs cause an Overhead when being Copied, though
  - Which is only relevant for large structs ( > 16 bytes, which is 4 floats) 
- Structs are exceptionally strong in combination with Arrays
  - The Array is a reference type, but an Array of structs is all stored in one Place
  - While an Array of classes has References to many different Objects all stored somewhere on the Heap
- Structs can never be null
  - No NullReferenceExceptions, yay!

What does Microsoft have to say about this?

<img width="521" alt="image" src="https://user-images.githubusercontent.com/7360266/139752194-f2280bc4-5c22-46b7-aa01-a3837981b4ee.png">


# 6 Default Values

If you use Types as Fields in Classes or Structs, then they have a default value, but what is this Default Value?

Type | Default Value
---- | --------------
`string` | `null`
Arrays | `null`
`class` | `null`
Numeric Types | `0`
`bool` | `false`
Enum | `0`
Char | `\0`
`structs` | A struct in which all fields and properties have default Values

## 6.1 The Default Keyword

You can use the `default` Keyword to get the default Value of any Type:

```cs
Unit unit = default(Unit);
```

Or short:

```cs
Unit unit = default;
```

You can also use it in Methods:

```cs
Level GetPlayerLevel() {
  if(player == null) {
    return default;
  }
}
```

And in Comparisons:

```cs
if(level == default) {
  // it's a new Player
}
```
