# 0 Introduction

Exceptions are used in .NET in order to implement Error-Handling. It allows both reporting Errors by `throw`int as well as Error-Handling by `catch`ing Exceptions.

This is essential in order to write Games that aren't completely buggy or crash all the time.

# 1 Exception

Exceptions are classes that inherit of the class `System.Exception`

Many Exceptions are already shipped with .NET and are thrown by various .NET Classes
- e.g. `System.NullReferenceException` when you try to access a Member of an object that's `null`.
- or `System.IndexOutOfRangeException` when you try to access an invalid index on an Array.

# 2 Throw

```cs
Console.WriteLine("This line will be executed.");
throw new Exception("Something bad has happened.");
Console.WriteLine("This line will never be reached.");
```

You can `throw` an Exception. This will mak your Program Crash instantaneously (unless the exception handled)

Output:
```
This line will be executed.
Unhandled exception. System.Exception: This program encountered an unknown error.
   at Program.Main() in ...Program.c:line 2

Process finished with exit code 6.
```

## 2.1 Return Value

The Compiler is smart enough to know that methods don't need to return any results, if an Exception is thrown:

```cs
public float CalculateDamage() {
  if(TryCrit() == false) {
    return baseDamage;
  }
}
```

Would yield a Compile Error: `Not all Code Paths return a value`

But:


```cs
public float CalculateDamage() {
  if(TryCrit() == false) {
    return baseDamage;
  } else {
    throw new NotImplementedException("Sorry, did not get this far, yet.");
  }
}
```

Is fine, because the Compiler knows that the Program will crash here anyways.

Okay, but making Programs crash is only one side of the story. We actually don't want our Programs to crash, right?

# 3 Catch


```cs
void Attack() {
  try {
    float damage = CalculateDamage();
    enemy.TakeDamage(damage);
  } catch (Exception e) {
    Console.WritLine($"Attack failed because of: {e}");
  }
  Console.WriteLine("I will not be skipped this time :)");
}
```

Using `try...catch`, you can catch Exceptions and prevent your Program from Crashing.

This Concept is called Error-Handling. 

## 3.1 A word of Caution

```cs
void Attack() {
  try {
    float damage = CalculateDamage();
    enemy.TakeDamage(damage);
  } catch (Exception e) {
    // ignore
  }
}
```

You should be careful when applying thi.
- Just catching all sorts of errors can lead to you just hiding bugs
- Imagine above scenario: the Player's character would just stop attacking completely, if an Exception is thrown.
  - This can be very confusing for the Player
  - And even more confusing to the developer
- Make sure to only handle Errors that make sense to Handle
  - Wrong User-Input
  - Network Errors
  - Memory Allocation Errors
  - Bugs that you can fix through Error Handling
- Do not
  - Hide Errors
  - Ignore Errors

## 3.2 Rethrow

```cs
void Attack() {
  try {
    float damage = CalculateDamage();
    enemy.TakeDamage(damage);
  } catch (Exception e) {
    if(enemy.IsDead){
      // ignore
    } else {
      throw; // this will re-throw
    }
  }
}
```

Sometimes, you can Handle an Error only under certain conditions.
- e.g. here, we can ignore the Failed Attack, if the Enemy is already dead anyways.

You can use `throw;` to re-throw a caught Exception.

This will print the original Exception Call-Stack to the Console.

If you were to write
```cs
throw e;
```
Then, the Call-Stack would point to the current line of code instead of the original one, though.

## 3.3 Handle some Errors only

```cs
try {
  float damage = CalculateDamage();
  enemy.TakeDamage(damage);
} catch (NoWeaponEquippedException e) {
  EquipWeapon();
  TryAttackAgain();
} catch (NullReferenceException e) {
  Console.WriteLine("Error: " + e + ". We had some issues with this, but were not able to fix it. Preventing a crash here.");
}
```

You can specify, what kind of Exceptions you want to handle. All other sorts of Exception would still crash your Program.

This is recommended to prevent you from unintentionally hiding new problems.

# 4 Finally

```cs
// Temporarily store password in file. lol.
File.WriteAllText("password.txt", Console.ReadLine());
Console.WriteLine("Saved password.");

try {
  LogIn();
  UpdateStatusMessage();
  LogOut();
} finally { // use finally
  // to delete password file
  File.Delete("password.txt");
  Console.WriteLine("Deleted password.");
}
Console.WriteLine("Crashed already.");
```

```cs
void LogIn(){
  throw new Exception("Example of an Error.");
}
```

`finally` ensures that the code block within the `finally`-Body gets executed, even if the Program Crashes within the `try`-Block.

This makes sure that you can clean up after yourself.

Output:
```
Saved password.
Deleted password.
```

But the Program will still crash.

## 4.1 And Altogether
```cs
try{
  // do something
} catch (Exception e) {
  // log error / try handling / rethrow
} finally {
  // clean up
}
```
You can also combine Error-Handling and Finally.

# 5 Your Own Exceptions

```cs
public class PlayerDisconnectedException : Exception {
  public PlayerDisconnectedException() {}
  public PlayerDisconnectedException(string message) : base(message){}
  public PlayerDisconnectedException(string message, Exception inner) : base(message, inner){}
}
```

In order for your program to have clear instructions, you need to define your own Exception-Types.

This allows callers - including yourself:
- to catch specific types of exceptions
- to track the source and solution to errors

## 5.1 Parameters

Here, the Exception always requires the thrower to specify, what the idea of the missing balance values is:

```cs
	public class MissingBalanceValuesException : Exception {

		static string GetExceptionString(string id) {
			return "Can not find Balance Values with id: " + id;
		}

		static string GetExceptionStringWithMessage(TemplateId templateId, string message) {
			return GetExceptionString(templateId) + " Additional Info: " + message;
		}

		public MissingBalanceValuesException(TemplateId templateId) : base(GetExceptionString(templateId)) { }

		public MissingBalanceValuesException(TemplateId templateId, string message) : base(GetExceptionStringWithMessage(templateId, message)) { }

		public MissingBalanceValuesException(TemplateId templateId, string message, Exception innerException) : base(GetExceptionStringWithMessage(templateId, message), innerException) { }
	}
```

## 5.2 Hints

It can be extremely useful to your users to provide some help with common sources of an error and possible solutions:

```cs
		static string GetExceptionString(string id) {
			return "Can not find Balance Values with id: " + id + ". Hint: Have you manually synced the Bundles? Fix: Enable Auto-Sync in Settings.;
		}
```

## 5.3 Leaking Information

But careful! Your application should not leak sensible information through exceptions! That's actually the reason, why .NET's standard Exceptions barely provide any information, e.g.
- the IndexOutRangeException mentions neither the index nor the size of the Array.
- the NullReferenceException does not mention the name of the variable that's null

Else, attackers could try to provoke Exceptions and thereby gain knowledge about your code and data and use that discover vulnerabilities and angles of attack. Even from remote!