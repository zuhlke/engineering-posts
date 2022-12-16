---
title: Structure and order your DI container
domain: campzulu.hashnode.dev
tags: .net,csharp
cover: https://linkdotnetblogstorage.azureedge.net/blog/20221216_DIStructure/Thumbnail.jpg
publishAs: LinkDotNet
hideFromHashnodeCommunity: false
ignorePost: true
---
Does your **Dependency Injection** container is one big pile of method calls one after the other?
Are there 50 lines of just `AddScoped`, `AddTransient`, and so on? Well, let's fix this.

We can utilize extension methods to make an order to that mess!

## The problem
When your project grows, so does your dependency injection container. You might have over 100 lines of code just with registrations and configurations. So after a while, a typical DI container can look like the following example.

```csharp
builder.Services.AddSingleton(_ => AppConfigurationFactory.Create(builder.Configuration));
builder.Services.AddScoped<IServiceA, ServiceA>();
builder.Services.AddScoped<IServiceB, ServiceB>();
// 50 more lines like that
builder.Services.AddScoped<IServiceZ, ServiceZ>();
builder.Services.AddDbContext<MyDbContext>();
builder.Services.AddScoped<IRepository, Repository>();
```

That isn't really great. If I would add another thing to the container, where should I put this? Should make that mess even worse? The first option one could consider is private methods. That would help, but it has a fundamental downside to the approach I will show you later. The downside is that still all of your registrations are in one big method. But in a nice world, they are located where they belong and fit from a conceptual point of view. And here is where extension methods can come into play. The idea is to create new `public static class`es that are located close to the code that gets registered in the container and only hold those types that are necessary. The overall DI container then just is a bunch of those extension method calls that have a very descriptive naming. So if someone is interested (s)he can go into said method to see what is going on. Also, it is very clear where you would have to add another registration as everything is kind of modular.

```csharp
builder.AddConfiguration(builder);
builder.AddDomainUseCaseA();
builder.AddDomainUseCaseB();
builder.AddPersistence();

public static class ConfigurationExtensions
{
    public static void AddConfiguration(this IServiceCollection services, WebApplication builder)
    {
        services.AddSingleton(_ => AppConfigurationFactory.Create(builder.Configuration));
    }
}

public static class DomainServiceAExtensions
{
    public static void AddDomainUseCaseA(this IServiceCollection services, WebApplication builder)
    {
        services.AddScoped<IServiceA, ServiceA>();
        services.AddScoped<IServiceB, ServiceB>();
    }
}

public static class DomainServiceBExtensions
{
    public static void AddDomainUseCaseB(this IServiceCollection services, WebApplication builder)
    {
        services.AddScoped<IServiceG, ServiceG>();
        services.AddScoped<IServiceH, ServiceH>();
    }
}

public static class PeristenceExtensions
{
    public static void AddPersistence(this IServiceCollection services)
    {
        services.AddDbContext<MyDbContext>();
        services.AddScoped<IRepository, Repository>();
    }
}
```

As you can see way more structured. We directly see what is going on and if we want to know what is in the `DomainUseCaseA` package, we can just go into that extension method. And of course, all of those `public static class`es can be located wherever they want to be.

## Conclusion
With extension methods you have an easy way to structure and organize your dependency injection container.