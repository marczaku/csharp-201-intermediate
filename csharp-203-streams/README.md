# 1 Introduction
Let's go back to streams for another second.

```cs
using System.IO;

// open a stream that can write into a file 
FileStream fileStream = File.OpenWrite("log.txt");
// open a stream writer for the filestream, which makes writing text easier
StreamWriter streamWriter = new StreamWriter(fileStream);

for (int i = 0; i < 10000; i++)
{
    streamWriter.WriteLine(i);
}
```

This is, how you can use .NET's Streaming classes to write information to a file. This is again very useful in order to make sure if you have big amount of data (imagine for example your game's log, which can have tens of thousands of logs per session), that you won't have to write it all at once as a huge 4MB-File, but can instead do so line by line.

.NET's Streams are very nicely designed. You can just nest Streams in Streams in Streams in order to get exactly the correct kind of Data into a Stream or out of a Stream. This is called the Decorator-Pattern.

Where one class looks to the outside the same like the inner class, but can add some functionality to it (decorate it).

Another example:
```cs
// this one can stream a byte[] into a Byte-Stream
MemoryStream memoryStream = new MemoryStream(bytes);
// this one can decompress GZIP from a Byte-Stream
GZipStream gzipStream = new GZipStream(memoryStream, CompressionMode.Decompress);
// this one can read text from a Byte-Stream
StreamReader streamReader = new StreamReader(gzipStream);
```

This is pretty cool. It's a bit like connecting a bunch of Adapters in order to get exactly what you want.

# 2 Flush

Back to the original source code:

```cs
using System.IO;

// open a stream that can write into a file 
FileStream fileStream = File.OpenWrite("log.txt");
// open a stream writer for the filestream, which makes writing text easier
StreamWriter streamWriter = new StreamWriter(fileStream);

for (int i = 0; i < 10000; i++)
{
    streamWriter.WriteLine(i);
}
```

My output in `log.txt` for this code is:

```
...
9231
9232
9
```

Wait! I can not remember wanting to log only the number until 9232 and then a 9? What's the problem?

In order to optimize performance, Streams contain an internal buffer. Let's say for 10.000 bytes. And only, if that buffer is full, it will Flush the data and forward it to the next stream. This is really important, because Should the Stream write each byte one by one? That would be a lot of going back and forth to the Hard Drive for writing a file. Or sending data over the internet one byte at a time, waiting until it has been received, would also take forever.

Good thing is that we can flush it manually, though:

```cs
streamWriter.Flush();
```

# 3 Finalizing

Okay, that problem is solved. Let's try something else:

```cs
using System.IO;

FileStream fileStream = File.OpenWrite("log.txt");
StreamWriter streamWriter = new StreamWriter(fileStream);

for (int i = 0; i < 100; i++)
{
    streamWriter.WriteLine(i);
}

streamWriter.Flush();

// Let's think of a reason that causes us to need to open a new stream: 

FileStream newFileStream = File.OpenWrite("log.txt");
StreamWriter newStreamWriter = new StreamWriter(newFileStream);
for (int i = 500; i < 600; i++)
{
    newStreamWriter.WriteLine(i);
}

newStreamWriter.Flush();
```

Output:
```
Unhandled exception. System.IO.IOException: The process cannot access the file 'log.txt' because it is being used by another process.
```

Okay, makes sense. Better set the old stream to null first, so it gets Destroyed, right?

```cs
using System.IO;

FileStream fileStream = File.OpenWrite("log.txt");
StreamWriter streamWriter = new StreamWriter(fileStream);

for (int i = 0; i < 100; i++)
{
    streamWriter.WriteLine(i);
}

streamWriter.Flush();

// Let's think of a reason that causes us to need to open a new stream: 

// clean up the old ones first
streamWrite = null;
fileStream = null;

// then create new ones
FileStream newFileStream = File.OpenWrite("log.txt");
StreamWriter newStreamWriter = new StreamWriter(newFileStream);
for (int i = 500; i < 600; i++)
{
    newStreamWriter.WriteLine(i);
}

newStreamWriter.Flush();
```

Output:
```
Unhandled exception. System.IO.IOException: The process cannot access the file 'log.txt' because it is being used by another process.
```

Okay, that did not help. Ah yeah, the Garbage Collector will take some time before actually cleaning up the object. Let's force it:

```cs
using System;
using System.IO;

FileStream fileStream = File.OpenWrite("log.txt");
StreamWriter streamWriter = new StreamWriter(fileStream);

for (int i = 0; i < 100; i++)
{
    streamWriter.WriteLine(i);
}

streamWriter.Flush();

// set the resources to null
fileStream = null;
streamWriter = null;
// tell the Garbage Collector to run now
GC.Collect();
// wait for the finalizers to finish
GC.WaitForPendingFinalizers();

// Let's think of a reason that causes us to need to open a new stream: 

FileStream newFileStream = File.OpenWrite("log.txt");
StreamWriter newStreamWriter = new StreamWriter(newFileStream);
for (int i = 500; i < 600; i++)
{
    newStreamWriter.WriteLine(i);
}

newStreamWriter.Flush();
```

Even this did not help in my case :( Well, relying on Finalizers to clean up our classes doesn't help apparently. Is there any other solution?

[-> Dispose](../csharp-204-disposable/)