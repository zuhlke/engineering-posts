---
title: Source Code Generators and regular expressions
domain: campzulu.hashnode.dev
tags: architecture, .NET, C#, Blazor
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1672652289997/68d840b3-1d4d-483d-91cd-4a92a638db85.jpeg
publishAs: linkdotnet
ignorePost: true
hideFromHashnodeCommunity: false
---


## Source Code Generators and Regular Expressions
Since .NET 7 we have the possibility to use source code generators for regular expressions. The first question is: *What is a source code generator?*
I can quote here [the official Microsoft page](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview):
> ... Source Generators let C# developers inspect user code as it is being compiled. The generator can create new C# source files on the fly that are added to the user's compilation. In this way, you have code that runs during compilation. It inspects your program to produce additional source files that are compiled together with the rest of your code.

In simple terms: Source code generators hook into the build process and can create additional stuff. And that is where our journey starts with regular expressions. If we define a regular expression, that regular expression can be expressed as code. This happens when you hit the build button.

The advantage over the traditional approach is that we can get the same performance as `new Regex("...", RegexOptions.Compiled)` and the startup benefit of `Regex.CompileToAssembly`, but without the complexity of `CompileToAssembly`. As the code is generated it can be viewed and debugged.

So instead of this code:
```csharp
private static readonly Regex HelloOrWorldCompiled =
        new("Hello|World", RegexOptions.Compiled | RegexOptions.IgnoreCase | RegexOptions.CultureInvariant);
```

We can write it like this:
```csharp
[GeneratedRegex("Hello|World", RegexOptions.IgnoreCase | RegexOptions.CultureInvariant)]
private static partial Regex HelloOrWorldGenerator();
```

Two things are essential here:
 1. We have to use the `GeneratedRegexAttribute` to tell the source code generator where it should hook in.
 2. Instead of a field, we now have a `partial` method. The reason is that the source code generator has to extend something. You can not extend fields, but you can extend `partial` methods.

Now you might have noticed that I omitted the `RegexOptions.Compiled` flag from the generated version. Why is that? Well, that flag tells the .NET runtime to compile a regular expression when your application starts. It is now part of your assembly, with the new source code generators, that will always be the case. There is no "on the fly" anymore.

The usage is almost the same:
```csharp
HelloOrWorldCompiled.Match(SomeText);

HelloOrWorldGenerator().Match(SomeText);
```

They behave exactly the same and offer the same functions. One cool thing is that the source code generator also creates XML documentation that explains exactly what it is doing.

![xml doc](https://cdn.hashnode.com/res/hashnode/image/upload/v1672652289997/68d840b3-1d4d-483d-91cd-4a92a638db85.jpeg)

## Conclusion

The new regular expression source code generators are an excellent and convenient addition to the .NET ecosystem. 