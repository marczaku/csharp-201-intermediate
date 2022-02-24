# 1 Ogre Weapon Sample

```cs
class Weapon {}
class Hand {
  public Weapon weapon;
}
class Unit {
  public string Name {get;}
  public Hand leftHand;
  public Hand rightHand;

  public Unit(string name, Weapon weapon) {
    this.Name = name;
    this.leftHand.weapon = weapon;
  }
}

static class Program {
  static void Main() {
    Weapon club = new Weapon();
    Unit ogre = new Unit("Ogre", club);
    Console.WriteLine($"You meet {ogre.Name}.");
    Console.WriteLine($"He carries a {ogre.leftHand.weapon}");
  }
}
```

What do you think is the Output, when this program is executed?

## 1.1 The Output

```
Unhandled Exception. System.NullReferenceException: Object reference is not set to an intance of an object.
    at Unit..ctor(String name, Weapon weapon) in /Users/marczaku/Projects/OgreGame/Program.cs:line 14
    at Program.Main() in /Users/marczaku/Projects/OgreGame/Program.cs:line 19

Process finished with exit code 6.
```

Can you explain, why this is happening?

## 1.2 Unhandled Exception

```
Unhandled Exception.
```

This means, that an Exception has happened during program execution. And that it has not been handled.
- This results in your application terminating, as we can see in the last line of the Output:

## 1.3 Exit Code

```
Process finished with exit code 6.
```

Each process can pass an Exit Code when exiting. If another application started this process, it can then read the Exit Code after termination and thereby get some idea of what happened.

There is no standards to these Exit Codes, unfortunately, but anything other than 0 usually means trouble, unless otherwise documented.

## 1.3 Exception Type

```
System.NullReferenceException:
```

Unhandled Exceptions usually print their Type to the Output to give you a more clear idea of what sort of Exceptions caused your program to crash. In this case, it was a `NullReferenceException`. The most common reason of applications crashing or malfunctioning.

## 1.4 Exception Message

```
Object reference is not set to an instance of an object.
```

Exceptions also usually provide some information that may help you understand, what went wrong. Great Exception Messages even come with instructions on how to fix it. Here, the message is rather technical, but in combination with the Type, we get somewhat of an idea that something is `null`.

## 1.5 The Call Stack

Next, you are provided with a printed Call Stack that shows you, where exactly the Exception happened:

```
    at Unit..ctor(String name, Weapon weapon) in /Users/marczaku/Projects/OgreGame/Program.cs:line 14
    at Program.Main() in /Users/marczaku/Projects/OgreGame/Program.cs:line 19
```

### 1.5.1 First Line of the Call Stack

The first line directly points at the Line of Code where the Exception Happened:

```
    at Unit..ctor(String name, Weapon weapon) in /Users/marczaku/Projects/OgreGame/Program.cs:line 14
```

- `at Unit..ctor(String name, Weapon weapon)` tells us, that the Exception happened in the `Unit`'s constructor (`ctor`)
- `in /Users/marczaku/Projects/OgreGame/Program.cs` points directly at the file that contained the code. (`Program.cs`)
- `:line 14` even tells us the Line Number: (`14`)

Well, that's something. our `Program.cs` File at Line 14 looks like this:

```cs
class Unit {
  public Unit(string name, Weapon weapon) {
    // ...
    this.leftHand.weapon = weapon; // Line 14
    // ...
  }
}
```

Okay, so somewhere here, an object reference must be accessed (using `reference.`), but that reference points to `null`.
- `this.` never can be `null`
- `leftHand.` is the only other reference that is being accessed here, so this can be the only source.
- `.weapon` is used here, but it is only assigned to, not accessed. So, this can not be the problem.
- ` = weapon;` can also not be the culprit, because again it is not being accessed (using `weapon.`)

Nice, apparently, `leftHand` is `null`, when we're trying to assign a Weapon to it's `weapon`-Field.

### 1.5.2 Second Line of the Call Stack

The second line points at the Line of Code that called the first line:

```
    at Program.Main() in /Users/marczaku/Projects/OgreGame/Program.cs:line 19
```

- `at Program.Main()` tells us, that the Exception happened in the `Program`'s `Main()`-Method.
- `in /Users/marczaku/Projects/OgreGame/Program.cs` points directly at the file that contained the code. (`Program.cs`)
- `:line 19` even tells us the Line Number: (`19`)

Let's confirm that:

```cs
static class Program {
  static void Main() {
    // ...
    Unit ogre = new Unit("Ogre", club); // Line 19
    // ...
  }
}
```

Looks correct! This is the line of code that called the `Unit`-Constructor. Wow, so much information!

## 1.6 NullReferenceException

Again, this is the most common exception. It always means, that in the printed line of code, there must be some code like:

```cs
foo.bar = ...
foo.Bar();
... = foo.bar;
another.foo.bar = ...
```

Where 
- `foo` is a Field or Variable
- its Type is a Reference Type
- its value is `null`

Which makes trying to access any of its non-static Members illegal.

## 1.7 Debugging

Here, we have been able to quite easily determine, what exactly has been `null`, but sometimes code might look like this:

```cs
Console.WriteLine(enemy.leftHand.weapon.stats.health);
```

> This kind of code is called a Train Wreck and a violation of the Law of Demeter

Anyways, it's not easy to determine, what's `null` here, just by looking at things. We ned more information. Let's Debug.

- Start the Program using the `Debug`-Button.
- The Program will pause automatically when an `Exception` is thrown.
- You can now use the `Locals` Window or hover Fields and Variables to see their Value.

This way, you can find out, exactly what's `null`.

## 1.8 Fixing the Bug

We need to assign an object instance to the `leftHand`-Field before trying to add a weapon to the `leftHand`'s `weapon`-Field.
- and while we're at it, we might as well give our Unit a `rightHand`, too.

```cs
class Unit {
  public string Name {get;}
  public Hand leftHand;
  public Hand rightHand;

  public Unit(string name, Weapon weapon) {
    this.Name = name;
    this.leftHand = new Hand(); // NEW
    this.rightHand = new Hand(); // NEW
    this.leftHand.weapon = weapon;
  }
}
```

# 2 Many Ogres

```cs
static void Main() {
  // Spawn 10 Ogres
  for(int i = 10; i >= 0; i++){
    Spawn(new Unit("Ogre", new Weapon()));
  }
}
```

If you run this program, a lot of nothing will happen.

# 2.1 Debugging

In this case, a very useful Tool is the `Pause` Button. It pauses the execution and it gives us a good indicator.

The yellow line shows, where the execution has stopped.

The Call Stack Window shows us even the whole Call Stack (just `Program.Main()`)

Also, the Locals Window shows us the values of all local variables:
- `unit = Program.Unit`
- `i = 1497120137`

Oh, wait, why is `i` so large? Should it not only go from 10 to 0?

Oh, I see, we wrote `i++` instead of `i--`. Must have been overseen while Refactoring.

# 2.2 Fixing the Bug

Just replace `i++` with `i--`:

```cs
static void Main() {
  // Spawn 10 Ogres
  for(int i = 10; i >= 0; i--){ // CHANGE
    Spawn(new Unit("Ogre", new Weapon()));
  }
}
```

# 3 Critical Ogre

```cs
int criticalBonusDamagePercent = 30;
float originalDamage = 22f;
float criticalDamage = originalDamage * (1f + criticalBonusDamagePercent / 100);
Console.WriteLine("Attack Damage: " + criticalDamage);
```

Output:
```
Attack Damage: 22

Process finished with exit code 0.
```

What's the problem?

## 3.1 Semantic Bugs
Here, we have a special kind of problem:
- No Exception
  - No Call Stack
- The Application is not stuck
- The Application actually pretends that everything's fine

But not everything's fine.

```cs
float criticalDamage = originalDamage * (1f + criticalBonusDamagePercent / 100);
```

This line is the same as 

```cs
float criticalDamage = 22f * (1f + 30 / 100);
```

Which can be evaluated:
```
22 * (1 + 30 / 100)
= 22 * (1 + 0.3)
= 22 * 1.3
= 28.6
```

So, why is the output:
```
Attack Damage: 22
```

Let's debug!

## 3.2 Pause the Application

I dare you to try to pause the application in that one nano-second that it takes to run :D

This obviously won't work.

## 3.3 Breakpoint
- Hover over the line, where you want to have a breakpoint
- Then, click on the red circle.
- You just created a breakpoint.
- When debugging, the application takes a break here.

## 3.3 Application taking a Break

Now, the application is paused again and you can take a look at the state of all variables.

- We kind of now the value of all variables already :/ - nothing new here

But, you can also hover expressions by hovering the operator.

```cs
float criticalDamage = 22f * (1f + 30 / 100);
```

If you hover
- the `*`, it will show you: `{float} 22`
- the `+`, it will show you: `{float} 1`
- the `/`, it will show you: `{int} 0`

Wait, what? `30/100=0`?

The problem is the numeric typ `int`, it can only store integers. Without decimals.

Great Debugging Work!

## 3.4 Fixing the Bug
```cs
float criticalDamage = originalDamage * (1f + bonusDamagePercent / 100f);
```
This will already fix the Bug. By dividing `30` by `100f`, we force the compiler to convert the `int` `30` to the `float` `30f` before the division in order to execute `float` division.

In Detail:
- An Implicit Conversion from `int` to `float` exists.
- The Operator `/` only exist for:
  - `/(int a, int b)`
  - `/(float a, float b)`

And since `b` is a `float` here, the compiler automatically concludes that it needs to implicitly cast the first parameter from `int` to `float`. Fascinating.

# 4 Debugging Tools

Here is another overview of great Debugging Tools:

## 4.1 Breakpoints

- Hover over the line, where you want to have a breakpoint
- Then, click on the red circle.
- You just created a breakpoint.
- When debugging, the application takes a break here.

### 4.1.2 Conditional Breakpoint

```cs
for(int i = 0; i < 10000; i++){
  CreateEnemy(i); // Conditional Breakpoint: `i == 437`
}
```

You can right-click a breakpoint in order to put a condition on it.
- e.g. if you know that the 437th spawned enemy always causes trouble.
- or to break when a certain value is `null` or `> 100`

## 4.2 Call Stack

- Whenever paused at a breakpoint
- You can check the Call Stack
- To see, what Method Calls brought you here
  - Who's calling whom?

## 4.3 Locals

Local Variables' Values are displayed both
- in the Debug Window
- As well as the Code Editor Window

Also, you can select another Method from the Call Stack
- and then see its Local Variables' Values

You can also Use the Fold-Arrow to look at each Value's internal Data

## 4.4 Evaluate Expression

You can add your own Expressions as a Watch.

This is useful to take a look at
- a nested value, e.g. `unit.leftHand.weapon.damage`
- a calculated result, e.g. `unit.damage * unit.criticalDamageMultiplier`
- a processed result, e.g. `units.Select(unit => unit.Weapon).ToArray()`

Also, you can actually execute code that changes your Game State!
- `grid.Reset()`
- `Destroy(player)`

## 4.5 Watches

You can add your own Expressions as a Watch.

This is useful to repeatedly take a look at an Evaluated Expression.

## 4.6 Step through Code

Sometimes, when debugging, you want to look at how a value develops over time. You can use Code Stepping to execute your code line by line. All of these Methods don't actually change how your code works. It only changes, where the Code pauses next:

```cs
static void Main(){
  StartGame();
  RunGame(); // Step Out
}

static void StartGame(){
/*>>*/  SpawnPlayer(); // Breakpoint Position
  SpawnEnemy(); // Step Over
}

static void SpawnPlayer() {
  player = new Player(); // Step Into
}
```

- Step Over: Steps over the next Method call (stays on current level of the Call Stack). The called Method will still be fully executed. You just don't go through it Step by Step.
- Step Into: Steps into the next Method call (goes deeper into the Call Stack). So you can go through it Step by Step.
- Step Out: Steps out of the current Method, returning to the current Method's Caller's next code line. The current Method will still be fully executed. You just won't see it Step by Step.

### 4.6.1 Force Step

Sometimes, when you try to Step Over, your Program is still halted by another Breakpoint within that method. You can prevent that by using Force Stepping.

### 4.6.2 Skip Step

When using Skip To, your code will actually be Skipped and not be Executed. Useful, if you need to change your currently executed code while debugging. Use with caution.

### 4.6.3 Skip To Cursor

This kind of creates a temporary Breakpoint and runs the code until it is hit and then removes it again.