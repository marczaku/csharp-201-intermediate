# 1 Structs

Structs are used in C# to resemble Value-Types. Their Use-Cases usually involve some Mathematical Context or some Performance-Optimization reasons.

## 1.1 Like Classes

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

Only difference here: `struct` Keyword instead of `class`

## 1.2 They can implement Interfaces

```cs
public interface IUnit {
  public int Health {get;}
}
public struct Unit : IUnit {
  public int Health {get; private set;}
}
```

They cannot inherit from other classes, but they can implement Interfaces.
- (Detail: It is often not recommended, as casting structs to interfaces leads to boxing.)

## 1.3 Constructor Limitations

They have limitations on their Constructors:

- You can not define a parameter-less constructor.
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

## 1.4 Not nullable

Variables and Fields of Struct-Types can never be Null!

```cs
public struct Unit {
  public string name;
}

public class Game {
  public Unit unit;
  
  public Game() {
    // This will not throw a NullReferenceException:
    unit.name = "Hey!";
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
public struct Unit {
  public int age;
  public int size;
}
```

```cs
static void Main() {
  Unit adult = new Unit();
  unit.age = 15;
  ChangeUnitAge(adult);
  Console.WriteLine(adult.age); // OUTPUT: 15
}
```

```cs
void ChangeUnitAge(Unit unit) {
  Console.WriteLine(unit.age); // OUTPUT: 15
  unit.age = 2;
  Console.WriteLine(unit.age); // OUTPUT: 2
}
```

---

What happens internally?

```cs
static void Main() {
```

First, enough Memory for the `Main`-Method is reserved. The `Main` Method has no parameters, but one local variable of Type `Unit`. The `Unit` has two field of type `int`.

On 64-bit Systems,
- `int` refers to `Int32` and is 4 bytes in size.

Therefore, `Unit` is 8 bytes in size and this is the Memory Layout:

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 00 00 00 00 00 | adult(age:0,size:0)|

---

```cs
Unit adult = new Unit();
```

This doesn't really so anything in this case.

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 00 00 00 00 00 | adult(age:0,size:0)|

---

```cs
unit.age = 15;
```

Here, `15` is assigned to the first Field of `unit`. All 8 bytes of the `Main`-Method belong to the `unit`, but the first four of those belong to the `unit`'s `age`, the other four to `unit`'s `size`:

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|

(F is 15 in Hexadecimal)

---

```cs
ChangeUnitAge(adult);

void ChangeUnitAge(Unit unit) {
```

Now, when we call `ChangeUnitAge`, something interesting happens:
- Enough Memory for the new Method is allocated on the Stack.
- `adult` gets copied to `unit`

`ChangeUnitAge` only requires enough Memory for the `unit` Parameter:

|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|
| ChangeUnitAge | 00 00 00 00 00 00 00 00 | unit(age:0,size:0)|

And `unit` consist of all 8 bytes of `Main`, so they are all copied to `ChangeUnitAge`:

|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|
| ChangeUnitAge | 00 00 00 0F 00 00 00 00 | unit(age:15,size:0)|

---

```cs
Console.WriteLine(unit.age); // OUTPUT: 15
```

Now, when we want to print `unit`'s age, we receive the number of the first four bytes, which is `15`.

---

```cs
unit.age = 2;
```

Now, we assign 2 to `unit`'s `age`. Do you see how `adult` was not affected by this?

|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|
| ChangeUnitAge | 00 00 00 02 00 00 00 00 | unit(age:2,size:0)|

---

```cs
Console.WriteLine(unit.age); // OUTPUT: 2
```

This line will not print Unit's updated `age` of `2`.

|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|
| ChangeUnitAge | 00 00 00 02 00 00 00 00 | unit(age:2,size:0)|

---

```cs
}
```

Now, something interesting happen again: The Method `ChangeUnitAge` ends, which leads to all of it Memory being popped off the Stack:

|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|

There is no more information about any `unit` with an `age` of `2` in the Memory now. That was all local information.

---

```cs
Console.WriteLine(adult.age); // OUTPUT: 15
```

Time for the last line of code. Here, `adult`'s `age` is printed. Which is still unchanged and `15`.

|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 0F 00 00 00 00 | adult(age:15,size:0)|



The same goes for variables:

```cs
Unit unit = new Unit();
unit.age = 15;
Console.WriteLine(unit.age); // 15
Unit unit2 = unit;
Console.WriteLine(unit2.age); // 15
unit2.age = 2;
Console.WriteLine(unit.name); // 15
Console.WriteLine(unit2.name); // 2
```

Here, `Main` needs enough space for 2 `Unit`'s:

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 | unit(age:0,size:0),unit2(age:0,size:0)|

---

```cs
unit.age = 15;
```

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 0F 00 00 00 00 00 00 00 00 00 00 00 00 | unit(age:15,size:0),unit2(age:0,size:0)|

---

```cs
Unit unit2 = unit;
```

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 0F 00 00 00 00 00 00 00 0F 00 00 00 00 | unit(age:15,size:0),unit2(age:15,size:0)|

---

```cs
unit2.age = 2;
```

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 0F 00 00 00 00 00 00 00 02 00 00 00 00 | unit(age:15,size:0),unit2(age:2,size:0)|



## 2.2 Overview of Value-Types

Value-Types
- Numeric Types `int`, `double`, `float`, ...
- Basic Types `char`, `bool`
- Enum Types e.g. `public Enum WeaponTypes { Gun, Bow }`
- Structs e.g. `public struct Vector2 { public float x, y; }`

## 2.3 Value Types cannot be null

Again, we cannot assign `null` to Value-Types, and we cannot compare them to `null`:

```cs
int a = null; // Not possible
if(a == null){} // Nope
```

## 2.4 Nullable Value Types

```cs
public class User {
  public int? age;
}
```

You can make value type fields or variables nullable, though, by using the `?` operator.


```cs
User user = new User();
Console.WriteLine("Do you want to name your age?");
if(Console.ReadLine() == "y") {
  Console.WriteLine("What's your age?");
  user.age = Convert.ToInt32(Console.ReadLine());
} else {
  user.age = null;
}
```

In this example, we assign `null` to the age field, if the user decides, not to give his age.\
This is a more elegant solution, than just assigning `0` or `-1` to his age:

```cs
if(user.age != null) {
  Console.WriteLine("Next year, you will be " + (((int)user.age)+1) + " years old.");
}
```

Now, if you want to get the actual value of the field, you need to make sure, that it is not null and then cast it:

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
public class Unit {
  public int age;
  public int size;
}
```

### 3.1.1 Sample 1: Copy Reference into Method

```cs
Unit vampire = new Unit();
vampire.age = 15;
ChangeUnitAge(vampire);
Console.WriteLine(vampire.age); // OUTPUT: 2
```

```cs
void ChangeUnitAge(Unit unit) {
  Console.WriteLine(unit.age); // OUTPUT: 15
  unit.age = 2;
  Console.WriteLine(unit.age); // OUTPUT: 2
}
```

What happens internally?

```cs
static void Main() {
```

First, enough Memory for the `Main`-Method is reserved. The `Main` Method has no parameters, but one local variable of Type `Unit`. The `Unit` has two field of type `int`. But locally, only the address of the Reference-Type is stored. And it is `0` per default.

On 32-bit Systems,
- `int` refers to `Int32` and is 4 bytes in size.
- `class` references are stored as `IntPtr` and also 4 bytes in size.

Therefore, `Unit` is 4 bytes in size:

|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 00 | vampire(0x00`null`)|

---

```cs
Unit vampire = new Unit();
```
Now, a new `Unit` is created on the HEAP and its address is stored in `vampire`:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 00 00 00 00 00 | Unit(age:0,size:0)|

STACK:
|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 01 | vampire(0x01)|

---

```cs
vampire.age = 15;
```

We look up the `Unit` at its address (0x01) and then change the `age` there to `15`:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 0F 00 00 00 00 | Unit(age:15,size:0)|

STACK:
|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 01 | vampire(0x01)|

You can see, that on the stack, nothing has changed.

---

```cs
ChangeUnitAge(vampire);

void ChangeUnitAge(Unit unit) {
```

Now, when we call `ChangeUnitAge`, again a new Method Scope is pushed on the stack:
- Enough Memory for the new Method is allocated on the Stack.
- `vampire`, which is only a Pointer to the actual `Unit`, gets copied to `unit`

`ChangeUnitAge` only requires enough Memory for the `unit` Parameter:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 0F 00 00 00 00 | Unit(age:15,size:0)|

STACK:
|Method|Memory|Data|
|------|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 00 | unit(0x00`null`)|

And `unit` consist of all 4 bytes of the address, so they are all copied to `ChangeUnitAge`:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 0F 00 00 00 00 | Unit(age:15,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x01)|

---

```cs
Console.WriteLine(unit.age); // OUTPUT: 15
```

Now, when we want to print `unit`'s age, we receive the number of the first four bytes at address `0x01`, which is `15`:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 0F 00 00 00 00 | Unit(age:15,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x01)|

- `unit` contains the address `0x01`
- `age` is 
  - the first field of the class `Unit`
  - of type `Int32` and therefore 4 bytes in size
- therefore, the four bytes at address `0x01` are read and passed into `Console.WriteLine`

---

```cs
  unit.age = 2;
```

The same as described above applies when assigning the value `2` to `unit`'s age:
- The value at address `0x01` is changed to two.

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x01)|

---

```cs
Console.WriteLine(unit.age); // OUTPUT: 2
```

You know what's happening here already:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x01)|

---

```cs
}
```

The method ends and the Method Scope is popped off the stack:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|

---

```cs
Console.WriteLine(vampire.age); // OUTPUT: 2
```

And this is the big difference to value types now:
- The change that was applied to `unit`
- Also has an effect on `vampire`
- Because both reference the same object instance
- at the same address `0x01`

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|


---

### 3.1.2 Sample 2: Copy Reference to Local Variable

```cs
Unit unit = new Unit();
unit.age = 15;
Console.WriteLine(unit.age); // OUTPUT: 15
Unit unit2 = unit;
Console.WriteLine(unit2.age); // OUTPUT: 15
unit2.age = 2;
Console.WriteLine(unit.age); // // OUTPUT: 2
Console.WriteLine(unit2.age); // // OUTPUT: 2
```

This is the final Memory State:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 00 00 00 01 | unit(0x01),unit2(0x01)|

The address still gets copied. But instead of the whole Unit being copied / cloned, it is only the reference to the Unit.

### 3.1.3 Sample 3: Creating new Object Instances

We can break the link by creating a `new Unit()` and assigning that address to the variable:

```cs
Unit vampire = new Unit();
vampire.age = 15;
ChangeUnitName(vampire);
Console.WriteLine(vampire.age); // OUTPUT: 2
```

```cs
void ChangeUnitName(Unit unit) {
  Console.WriteLine(unit.age); // OUTPUT: 15
  unit.age = 2;
  Console.WriteLine(unit.age); // OUTPUT: 2
  unit = new Unit(); // !!!
  unit.age = 7; // !!!
  Console.WriteLine(unit.age); // OUTPUT: 7
}
```

---

The first interesting change here:

```cs
  unit = new Unit(); // !!!
```

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|
| 0x09 | 00 00 00 00 00 00 00 00 | Unit(age:0,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x09)|

A new `Unit` instance is created on the HEAP. Since the previous one requires 8 bytes in size, we can assume that the new one will be allocated at least 8 bytes further away (reality is more complex, unfortunately). So, here, we get the address of the new `Unit` returned: `0x09` and assign that to the local `unit` variable.

---

```cs
  unit.age = 7; // !!!
```

Here, `7` gets assigned to the `age` of the `Unit` at the address that is stored in the `unit` variable (`0x09`):

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|
| 0x09 | 00 00 00 07 00 00 00 00 | Unit(age:7,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x09)|

---

```cs
  Console.WriteLine(unit.age); // OUTPUT: 7
```

This prints the `age` of the `Unit` stored at address `0x09`, because that's the value stored in the `unit` variable:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|
| 0x09 | 00 00 00 07 00 00 00 00 | Unit(age:7,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|
| ChangeUnitAge | 00 00 00 01 | unit(0x09)|

---

```cs
}
```

You know, what happens when the Method ends? The Method Memory gets popped from the Stack:

HEAP:
|Address|Memory|Data|
|------|------|----|
| 0x01 | 00 00 00 02 00 00 00 00 | Unit(age:2,size:0)|
| 0x09 | 00 00 00 07 00 00 00 00 | Unit(age:7,size:0)|

STACK:
|Method|Memory|Data|
|:------:|------|----|
| Main | 00 00 00 01 | vampire(0x01)|

Now, this is an interesting state, because nobody has the address `0x09` anymore. Which means, that this `Unit` can never be referenced again (we can not "make up" addresses, like in C++).

The Garbage Collector will notice this the next time it gets active and then `~Finalize()` the `Unit` and make the Memory on th Heap available for other objects again,

---

```cs
Console.WriteLine(vampire.age); // OUTPUT: 2
```

For the sake of completeness, let's look at this line as well, but it should be any big surprise :)

---

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
  - This has to do with Classes bing stored on the Heap, which is dynamic object memory
  - Data stored on the Heap is usually slower in access
  - And generates Garbage, which needs to be cleaned up by the Garbage Collector
- Structs cause an Overhead when being Copied, though
  - Which is only relevant for large `structs` ( > 16 bytes, which is e.g. 4 `floats`) 
- Structs are exceptionally strong in combination with Arrays
  - The Array is a reference type, but an Array of `structs` is all stored in one Place
  - While an Array of classes has References to many different Objects all stored somewhere on the Heap
- Structs can never be `null`
  - No `NullReferenceException`, yay!

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
