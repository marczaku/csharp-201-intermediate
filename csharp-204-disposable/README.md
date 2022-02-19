# Dispose Pattern

Of course there is! The Dispose Pattern helps out here. Whenever a class has Unmanaged Resources (any resource that won't automatically be cleaned up like File Handles, Network Sockets or also other Processes or so), it should implement `System.IDisposable`

```cs
namespace System;
public interface IDisposable
{
    void Dispose();
}
```

Well, that's not very exciting technology. But let's try whether it does the job:

```cs
using System.IO;

FileStream fileStream = File.OpenWrite("log.txt");
StreamWriter streamWriter = new StreamWriter(fileStream);

for (int i = 0; i < 100; i++)
{
    streamWriter.WriteLine(i);
}

streamWriter.Flush();

streamWriter.Dispose();
fileStream.Dispose();

FileStream newFileStream = File.OpenWrite("log.txt");
StreamWriter newStreamWriter = new StreamWriter(newFileStream);

for (int i = 500; i < 600; i++)
{
    newStreamWriter.WriteLine(i);
}

newStreamWriter.Flush();

```

This works! Nice! In other words: If we have classes which implement `System.IDisposable` we should not forget to call `Dispose()` when we're done using them. This will in the case of Streams also automatically Flush the Stream, by the way. But I think that it's quite easy to forget to call `Dispose()`, or?

# Using using

The using Keyword actually handles Disposal of `IDisposable`-Classes for us! No extra code required. You assign the resource that you want to `use` within as a Function Argument and then use the File within the `{}`-Scope. When the Scope ends, `Dispose()` is called for you on the object:

```cs
using System.IO;

using (FileStream fileStream = File.OpenWrite("log.txt"))
{
    using (StreamWriter streamWriter = new StreamWriter(fileStream))
    {
        for (int i = 0; i < 100; i++)
        {
            streamWriter.WriteLine(i);
        }
    }
}

using (FileStream fileStream = File.OpenWrite("log.txt"))
{
    using (StreamWriter streamWriter = new StreamWriter(fileStream))
    {
        for (int i = 500; i < 600; i++)
        {
            streamWriter.WriteLine(i);
        }
    }
}
```

And because Code with a lot of Indentations becomes quite unreadable, .NET even came up with a neater syntax:

```cs
using System.IO;

using FileStream fileStream = File.OpenWrite("log.txt");
using StreamWriter streamWriter = new StreamWriter(fileStream);
for (int i = 0; i < 100; i++)
{
    streamWriter.WriteLine(i);
}
```

But careful! In this case, the Scope of the Variable only ends when the Outer Scope ends (which is the Main-Method). In other words, we would not be able to create a second FileStream for the same File anymore, because the previous fileStream would not have left the scope, yet:

```cs
using System.IO;

using FileStream fileStream = File.OpenWrite("log.txt");
using StreamWriter streamWriter = new StreamWriter(fileStream);
for (int i = 0; i < 100; i++)
{
    streamWriter.WriteLine(i);
}

// This will throw an Exception again:
using FileStream newFileStream = File.OpenWrite("log.txt");
using StreamWriter newStreamWriter = new StreamWriter(newFileStream);
for (int i = 0; i < 100; i++)
{
    newStreamWriter.WriteLine(i);
}
```

# What if your class uses an IDisposable Field?

Then your class should also implement `IDisposable` and make sure that all fields are disposed of when your class is disposed:

```cs
using System;
using System.IO;
public class Logger : IDisposable
{
    private readonly FileStream fileStream;
    private readonly StreamWriter streamWriter;
    bool disposed;

    public Logger()
    {
        fileStream = new FileStream("log.txt", FileMode.Create);
        streamWriter = new StreamWriter(fileStream);
    }

    public void Log(string message)
        => this.streamWriter.Write($"[{DateTime.Now}] {message}");

    public void Dispose()
    {
        fileStream.Dispose();
        streamWriter.Dispose();
    }
}
```

# Enumerator

Fun Fact: Most Enumerator Implementations implement `IDisposable`. Well, this wasn't that funny, was it? But this has reasons of some of them being reusable. And only if they've been disposed of in time, can they be reused for another method needing an Enumerator.