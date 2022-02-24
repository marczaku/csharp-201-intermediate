Imagine you were using this class from a Library:

```cs
public class Person {
    public DateTime BirthDate;
}
```

What you'd like to know is a Person's age. You could write a Method for this and use it like this:

```cs
int GetAgeForPerson(Person person){
    // this is by the way not exact. Why?
    return (DateTime.Now-person.BirthDate).Days/365;
}

Person person = GetSomePerson();
Console.WriteLine("Age: " + GetAgeForPerson(person));
```

Okay, this looks just fine. But now, you need this method in another place of your code as well. Maybe, above sample was in the Shop to verify the User's Age and the other place is in the User Profile, to display the User's Age. It's about time to avoid Code Duplication and put it into a static class:

```cs
public static class PersonMethods {
    public static int GetAgeForPerson(Person person){
        // this is by the way not exact. Why?
        return (DateTime.Now-person.BirthDate)/365;
    }
}
```

And now in the other place:
```cs
Person person = GetSomePerson();
Console.WriteLine("Age: " + PersonMethods.GetAgeForPerson(person));
```

If you want this code a bit more pretty, you can use `using static` to make the `static` Methods available as if they were local Methods:

```cs
using static PersonMethods;
Person person = GetSomePerson();
Console.WriteLine("Age: " + GetAgeForPerson(person));
```

Okay, this works. But the Problem is: It's hard to miss that this method exists. Other people might not know that it exists. Also, it would've been great, if the author just would've thought of adding the Method to the `Person` class right away, so you'd be able to write code like this:

```cs
Person person = GetSomePerson();
Console.WriteLine("Age: " + person.GetAge());
```

Sometimes, you might be able to inherit from the class and add that functionality, but that's often not possible. Luckily, you can extend even existing classes from other people. Restrictions: You can only access `public` Fields and Methods. It works like this:

```cs
// The class containing extension methods must be static:
public static class PersonExtensions{
    // the extension method must be static and use the `this` keyword:
    public static GetAge(this Person person){
        return (DateTime.Now-person.BirthDate)/365;
    }
}
```

Now, to all other classes. It will look as if the Method is part of the Person:

```cs
Person person = GetSomePerson();
Console.WriteLine("Age: " + person.GetAge());
```

Again, this is only syntactic sugar. The compiler will change this code to:

```cs
Person person = GetSomePerson();
Console.WriteLine("Age: " + PersonExtensions.GetAge(person));
```