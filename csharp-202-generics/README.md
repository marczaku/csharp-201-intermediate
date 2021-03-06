Generic Classes and Methods allow us to create one Class or Method that can be used for different Types. Imagine, you had two Classes: `YuGiOhCardCollection` and `MagicCardCollection` which are very similar, but one works with `YuGiOhCards` and the other with `MagicCards`. You would end up with two identical classes, where only the CardType is replaced. In C#, you can instead define a Generic Class `CardCollection<TCardType>` and use it with different Card Types, like `CardCollection<YuGiOhCard>` and `CardCollection<MagicCard>` without rewriting the whole class again. We'll see, how that works exactly:

# 1 The Problem

Often, we want to specialize classes for a certain Type.

```cs
public class Monster { }
```

```cs
public class Dragon { }
```
For example a trap that can only trap one kind of Monster.
```cs
public class DragonTrap {
  public Dragon TrappedObject { get; set; }
}
```

```cs
Dragon dragon = new Dragon();
Monster monster = new Monster();
DragonTrap dragonTrap = new DragonTrap();
// this works
dragonTrap.TrappedObject = dragon;
// ERROR: the dragon trap can only trap dragons
dragonTrap.TrappedObject = monster;
```

Now, if we want to have another Trap that can trap Monsters, we'd have to create a new class again:
```cs
public class MonsterTrap {
  public Monster TrappedObject { get; set; }
}
```

```cs
Dragon dragon = new Dragon();
Monster monster = new Monster();
MonsterTrap monsterTrap = new MonsterTrap();
// this works
dragonTrap.TrappedObject = monster;
// ERROR: the monster trap can only trap monsters
monsterTrap.TrappedObject = dragon;
```

These classes work fine, but they look kind of the same and still, we need two classes.\
Imagine, it would take 300 Lines of Code for each of these traps...\
And there's 200 different kinds of Monsters...

---

# 2 Using a Generic Class

We can solve this problem using a Generic Class.\
- You define a Generic class like this: `public class ClassName<GenericTypeName>`\
- Usually, T is used as the `GenericTypeName`\
A Generic Class is a Class that defines a replaceable Type (usually, `T` is used) between angle brackets `<>` behind the class name.\
This Type `T` is called "Type Parameter" of the Generic Class.\
This Type `T` is unknown to the class, but it can be used as if it's known throughout the Class, for example as the Type of a Property:

```cs
public class Trap<T> {
  public T TrappedObject { get; set; }
}
```

Now, when using the `Trap<T>`-Type for example for a local variable, we need to provide a real Type as a Type Argument to replace `T`:

```cs
Trap<Dragon> dragonTrap = new Trap<Dragon>();
```

If we do this, then all Instances of the Type `T` in the `Trap<T>`-Class are replaced by `Dragon`.\
The class `Trap<T>:
```cs
public class Trap<T> {
  public T TrappedObject { get; set; }
}
```
Looks like this as `Trap<Dragon>`:
```cs
public class Trap<Dragon> {
  public Dragon TrappedObject { get; set; }
}
```

So, just as the `DragonTrap` before! We can validate this:

```cs
Trap<Dragon> dragonTrap = new Trap<Dragon>();
// this works
dragonTrap.TrappedObject = dragon;
// ERROR: the dragon trap can only trap dragons
dragonTrap.TrappedObject = monster;
```

And of course, we can also easily pass other Type-Arguments to the Generic Class:

```cs
Trap<Monster> monsterTrap = new Trap<Monster>();
monsterTrap.TrappedObject = dragon;
```

---

# 3 Generic Type Constraints

Imagine the following classes:

A Monster that can be entrapped:
```cs
public class Monster { 
  public void Entrap() {}
}
```

A Zombie that is a Monster:
```cs
public class Zombie : Monster{}
```

A Vampire that is a Monster:
```cs
public class Vampire : Monster{
  public void SealVampireTeeth();
}
```

A Dragon that is no Monster:
```cs
public class Dragon { }
```

Now, we want to have different kind of Traps for the Player:
- A ZombieTrap for entrapping Zombies.
- A VampireTrap for entrapping Vampires.
- A Trap that can entrap all sorts of Monsters.

The Trap must call `Entrap()` on the Monster to entrap it:

```cs
public class Trap<T> {
  public void Activate(T trappedMonster) {
    // ERROR: Type T has no Method `Entrap()`
    // Because we cannot know, what Type T will have in the end...
    trappedMonster.Entrap();
  }
}
```

For this reason, we can use Type-Constraints
- Use the `where` Keyword behind your Generic Class
- With the Syntax `where TypeParameterName : TypeParameterConstraintType`
  - TypeParameterName: usually `T`
  - TypeParameterConstraintType: The Type that you want to restrict `T` to inherit from

```cs
public class Trap<T> where T : Monster {
  public void Activate(T trappedMonster) {
    // COOL: This works now! Since `T` has the Constraint of inheriting from Monster...
    // We can be sure, that whatever the Type, it will have an `Entrap()`-Method (which is defined on Monster)
    trappedMonster.Entrap();
  }
}
```

It is a good habit to name your Generic Type Parameter the same way as its Constraints, but with a `T` before.\
If the type is constrained to `Monster`, the Parameter should be called `TMonster` (read: Type Of Monster)
```cs
public class Trap<TMonster> where TMonster : Monster {
  public void Activate(TMonster trappedMonster) {
    // COOL: This works now! Since `T` has the Constraint of inheriting from Monster...
    // We can be sure, that whatever the Type, it will have an `Entrap()`-Method (which is defined on Monster)
    trappedMonster.Entrap();
  }
}
```

Let's see this class in action:

```cs
Zombie zombie = new Zombie();
Trap<Zombie> zombieTrap = new Trap<Zombie>();
zombieTrap.Activate(zombie);
```

We cannot pass a `Vampire` into the ZombieTrap, though:

```cs
Vampire vampire = new Vampire();
Trap<Zombie> zombieTrap = new Trap<Zombie>();
zombieTrap.Activate(vampire);
```

Why? because for the Type `Zombie`, the `Trap`-Class looks like this:

```cs
public class Trap<Zombie> where Zombie : Monster {
  public void Activate(Zombie trappedMonster) {
    trappedMonster.Entrap();
  }
}
```

We can create a Trap that can trap all kinds of monsters, too:

```cs
Zombie zombie = new Zombie();
Vampire vampire = new Vampire();
Trap<Monster> monsterTrap = new Trap<Monster>();
monsterTrap.Activate(zombie);
monsterTrap.Activate(vampire);
```

We cannot create a `Dragon`-`Trap`, though, because `Dragon` does not inherit from `Monster`:

```cs
// ERROR: Dragon does not inherit from Monster, which is a Type-Constraint for TMonster of class Trap<TMonster>
Trap<Dragon> dragonTrap = new Trap<Dragon>();
```

For obvious reasons, we also cannot call any methods that are only defined in the `Vampire`-Class within the Trap (since the Type-Constraint only requires `Monsters`, which could also be `Zombies`). This follows the same Rules as Polymorphism:

```cs
public class Trap<TMonster> where TMonster : Monster {
  public void Activate(TMonster trappedMonster) {
    trappedMonster.Entrap();
    // ERROR: SealVampireTeeth does not exist on `Monster`:
    trappedMonster.SealVampireTeeth();
  }
}
```

You can still Type-Cast, though:

```cs
public class Trap<TMonster> where TMonster : Monster {
  public void Activate(TMonster trappedMonster) {
    trappedMonster.Entrap();
    if(trappedMonster is Vampire vampire) {
      vampire.SealVampireTeeth();
    }
  }
}
```

---

# 4 Multiple Type Parameter


Generic Classes can have Two Type Parameters, too.\
Just like multiple Method Parameters (`void Add(int a, int b){}`)...\
Multiple Type Parameters are also just separated by Comma:\
`public class ClassName<TypeParameter1, TypeParameter2> {}`\
Usually, you need to come up with better TypeParameter-Names than `T` and `U`, though, or your code becomes unreadable.

```cs
public class Connection<TFrom, TTo> {
  public TFrom From { get; set; }
  public TTo To { get; set; }
}
```

For example, using two different classes...

```cs
public class House{}
```
```cs
public class TrainStation{}
```

We could create a `Connection` between a `House` and a `TrainStation`:

```cs
House home = new House();
TrainStation slussen = new TrainStation();
// all type arguments need to be passed when using this generic class:
Connection<House, TrainStation> connection = new Connection<House, TrainStation>();
connection.From = home;
connection.To = slussen;
```

---

# 5 Parameter-less Constructor Type-Constraint:

What you can not do, is this:

```cs
public Factory<T> {
  public T Create() {
    return new T(); // ERROR!
  }
}
```

Why is that? Well, because the Type `T` might not have a Parameter-less constructor...

```cs
public class Monster {
  public Monster(int id){

  }
}
```

And in this case, this code here would also not be possible:
```cs
public Factory<Monster> {
  public Monster Create() {
    return new Monster(); // ERROR! No Parameter-Less Constructor exists!
  }
}
```

The `new()` Type-Constraint, allows you to specify, that the passed Type Argument must have a parameter-less constructor. Which again allows you to use it:

```cs
public Factory<T> : where T : new() {
  public T Create() {
    return new T(); // Thanks to the Type-Constraint, this is possible!
  }
}
```

---

# 6 Generic Methods

Not only classes can be Generic, Methods can also be Generic:

```cs
public class Monster {
  public int id;
}
```

```cs
public TMonster CreateNewMonster<TMonster>() where TMonster : Monster, new() {
  TMonster monster = new TMonster();
  monster.id = nextId++;
  return monster;
}
```

Usage:

```cs
Zombie zombie = CreateNewMonster<Zombie>();
```

A cool feature here: If you have a method that requires an Argument of a Generic Type:

```cs
public void Equip<TWeapon>(TWeapon weapon) where TWeapon : Weapon {
  // Use your imagination for the code here... :)
}
```

Then, you can obviously use it like this:
```cs
Dagger dagger = new Dagger();
Equip<Dagger>(dagger);
```

But since C# already knows, that `dagger` is a `Dagger`...\
It can infer the type automatically:

```cs
Dagger dagger = new Dagger();
Equip(dagger); // Works because C# knows, that it needs to call Equip<Dagger>(dagger)
```

Of course, this method works for other classes, too:
```cs
Mace mace = new Mace();
Equip(mace);
```

But only, if they fulfill the Type Constraints...

```cs
Zombie zombie = new Zombie();
Equip(zombie); // ERROR: Zombie does not inherit from `Weapon`, which is Type Constraint of `Equip<TWeapon>`
```
