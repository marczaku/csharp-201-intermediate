
# Attributes

Attributes are kind of classes which are not intended to become part of your "real" code, but instead are used to decorate your code with hints and properties which you can then read using reflection. What? Let's have a look:

# Required Properties

Scenario: The user wants to create a new Profile. He needs to provide some information. Some information are required, others are optional. Our code should make sure that all required information have been set.

```cs
using System;
using System.Linq;

Profile marc = new Profile
{
    Title = "Mr.",
    Name = "Marc"
};

Console.WriteLine($"Valid Profile: {ValidateProfile(marc)}");

bool ValidateProfile(Profile profile)
{
    var requiredProperties = profile
        .GetType()
        .GetProperties()
        .Where(property => property.GetCustomAttributes(typeof(RequiredAttribute), true).Any());
    
    foreach (var property in requiredProperties)
    {
        if (string.IsNullOrWhiteSpace((string)property.GetValue(profile)))
        {
            string reason = (property.GetCustomAttributes(typeof(RequiredAttribute), true).First() as RequiredAttribute).Reason;
            Console.WriteLine($"Error: {property.Name} not set. Reason: {reason}");
            
            return false;
        }
    }
    return true;
}

public class Profile
{
    public string Title { get; set; }
    [Required] public string Name { get; set; }
    public string Gender { get; set; }
    [Required(Reason = "Without a description, your profile looks boring.")] public string Description { get; set; }
}

public class RequiredAttribute : System.Attribute
{
    public string Reason { get; set; }
}
```

So, you can define an Attribute by making it inherit from `System.Attribute`. It is common to add `Attribute` at the end of your attribute's name:

```cs
public class RequiredAttribute : System.Attribute
{
    public string Reason { get; set; }
}
```

When using your Attribute, you can actually skip the Attribute-Part of the name:
```cs
[Required] public string Name { get; set; }
```

You can put the Attribute behind a lot of things. Like: classes, fields, enums, enum values, properties, methods, parameters and many more.

Also, if your attribute has Properties, you can assign them to leave additional information:

```cs
[Required(Reason = "Without a description, your profile looks boring.")] public string Description { get; set; }
```

Now, you can use Reflection to find out, whether any Class, Property, Field, ... has the Attribute:

```cs
property.GetCustomAttributes(typeof(RequiredAttribute), true).Any()
```

And even look at the Attribute's values:

```cs
RequiredAttribute required = property.GetCustomAttributes(typeof(RequiredAttribute), true).First() as RequiredAttribute;
Console.WriteLine(required.Reason);
```

You have used Attributes in a few places already, I hope this helps you understand what they are.
