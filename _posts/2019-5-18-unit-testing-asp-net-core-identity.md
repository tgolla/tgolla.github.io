---
layout: single
title:  "Unit Testing ASP.NET Core Identity"
date: 2019-5-18 2:58:00 -0600
category: .NET
tags: Unit-Testing ASP.NET-Core-Identity ASP.NET-Core SQLite-In\-Memory CreateWebHostBuilder ServiceCollection UserManager RoleManager IdentityUser IdentityRole
header:
  image: /assets/images/headers/20170821_131105-1280x320.jpg
---

I had the opportunity to work on an authentication/account management Web API in which we use the new ASP.NET Core Identity.  ASP.NET Core Identity is easy to work with and customize.  Our challenge came when we began to look at how we would write unit test for our custom classes that inherited from ASP.NET Core Identity classed like UserManager, extension classes built on ASP.NET Core Identity classes and the controllers that used ASP.NET Core Identity classes.

In this article I will outline a rather elegant way to test code that utilizes ASP.NET Core Identity.  What I will not be doing in this article is outlining how to use ASP.NET Core Identity.  There are plenty of articles on this and so I’m going to assume that if you are at the point where you need to understand how to write unit test for code using ASP.NET Core Identity, you have a reasonable understanding of how it works.

## The Concept

As I searched the internet looking for examples of doing unit testing for code that utilized ASP.NET Core Identity I found very few and those I did find had relatively simple needs which relied on mocking.  Don’t get me wrong, mocking is great, but is this case there are two arguments against using it.

The first is that the code base for ASP.NET Core Identity is massive containing a large number of interfaces and classes.  An example of this is the constructor for UserManager which requires nine complex objects.  This presented a huge effort in properly mocking method calls for each of these objects. 

The second issue is that ASP.NET Core Identity is built such that it plays well in the dependency injection service configuration environment found in an ASP.NET Core web applications making it difficult to configure and build outside of this environment.

I can’t say I had a sudden epiphany in finding this solution. It was more of a gradual understanding of how I might access ASP.NET Core Identity from my unit test.  Instead of mocking the different objects needed for testing the idea came to me to build the exact ASP.NET Core Identity configuration I would have in my application using the ASP.NET Core service collection class responsible for building a web application’s dependency injection services and then extract the services needed to unit test my controllers and methods.

I’m not going to get into a fight with the purest that will insist we are not properly unit testing our code because we have not separating the code being tested from its dependencies on other code.  I will however repeat something I once heard from another developer, which is that we don’t remove dependencies to the .NET framework when we unit test.  Why, because we trust the framework has been well tested.  What I’m offering here is a simpler and faster way to test code which uses the ASP.NET Core Identity, which can also be trusted.

## The Premise for Testing Code Utilizing ASP.NET Core Identity

1. Ceate a service collection in the same manner as an ASP.NET Core web application.
2. Add the needed services in the exact same manner as you would in the ConfigureServices method call found in Startup.cs of your web application.  In this case we will need a database context and ASP.NET Core Identity.  The example also configures access to the appsettings.json file to pull application settings.
3. Extract those services need to test our controllers and methods.  Then run our test.

## Look at the Code Needed

The example code I created to demonstrate these three steps is very simple and provides some examples of unit testing both an MVC Web API controller and some UserManager extension methods.  The actual project that prompted the need for a unit testing solution also included custom UserManager, RoleManager , UserStore and RoleStore ASP.NET Core Identity classes.  You can pull the entire code example from GitHub at the following link. 

[https://github.com/tgolla/ASP.NETCoreIdentityUnitTestingDemo](https://github.com/tgolla/ASP.NETCoreIdentityUnitTestingDemo){:target="_blank"}

The following is the code need to create an ASP.NET Core Identity configuration for unit testing.  Let’s take a line by line look at what is happening.

{% highlight csharp %}
private SqliteConnection sqliteConnection;
private ApplicationIdentityDbContext identityDbContext;
private IOptions<TokenValidation> tokenValidation;
private UserManager<ApplicationUser> userManager;
private RoleManager<ApplicationRole> roleManager;

[SetUp]
public void Setup()
{
    // Build service colection to create identity UserManager and RoleManager. 
    IServiceCollection serviceCollection = new ServiceCollection();

    // Add ASP.NET Core Identity database in memory.
    sqliteConnection = new SqliteConnection("DataSource=:memory:");
    serviceCollection.AddDbContext<ApplicationIdentityDbContext>(options => options.UseSqlite(sqliteConnection));

    identityDbContext = serviceCollection.BuildServiceProvider().GetService<ApplicationIdentityDbContext>();
    identityDbContext.Database.OpenConnection();
    identityDbContext.Database.EnsureCreated();

    // Add Identity using in memory database to create UserManager and RoleManager.
    serviceCollection.AddApplicationIdentity();

    // Get UserManager and RoleManager.
    userManager = serviceCollection.BuildServiceProvider().GetService<UserManager<ApplicationUser>>();
    roleManager = serviceCollection.BuildServiceProvider().GetService<RoleManager<ApplicationRole>>();

    // Get token validation settings.
    var builder = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true);

    IConfigurationRoot configuration = builder.Build();

    var tokenValidationSection = configuration.GetSection("TokenValidation");
    serviceCollection.Configure<TokenValidation>(tokenValidationSection);

    tokenValidation = serviceCollection.BuildServiceProvider().GetService<IOptions<TokenValidation>>();
}

[TearDown]
public void TearDown()
{
    identityDbContext.Database.EnsureDeleted();
    identityDbContext.Dispose();
    sqliteConnection.Close();
}
{% endhighlight %}

The code above is from AuthenticationControllerUnitTest.cs.  The same code can also be found in UserManagerExtensionsUnitTest.cs but varies slightly in the way data from appsetting.json is presented (tokenValidation vs. tokenValidationSettings).  This code is responsible for building the ASP.NET Core Identity configuration need to run a unit test.

The example is using the NUnit unit-testing framework and in the code snippet above the method have attribute setting of [Setup] and [TearDown].  These attributes allow for the related Setup() and TearDown() methods to be executed before and after each unit test.

The Setup() method starts by creating a service collection.  This is normally done under the covers in your web application by CreateWebHostBuilder.

{% highlight csharp %}
    IServiceCollection serviceCollection = new ServiceCollection();
{% endhighlight %}

The next thing we need to do is add a DBContext for the database used by ASP.NET Core Identity.  Yes, this is where we start treading on the issue of separation of dependencies.  The compromise is that like the .NET framework we also trust that Entity Framework is well tested and for testing we will use a light weight in-memory database solution.

While exploring the use of an in-memory database I looked at both the Entity Framework in-memory database provider and SQLite in-memory solution which I found to work best in this scenario.  One thing to note is that you need to create the SQLite connection object outside of the AddDbContext call.  Not doing so creates an odd behavior which appears to open and close the connection each time a call was made to a UserManager method causing data in the database to be cleared each time.

{% highlight csharp %}
    sqliteConnection = new SqliteConnection("DataSource=:memory:");
    serviceCollection.AddDbContext<ApplicationIdentityDbContext>(options => options.UseSqlite(sqliteConnection));
{% endhighlight %}

The next step is to configure the ASP.NET Core Identity service in the same manner as you plan to use it in your application. 

{% highlight csharp %}
    serviceCollection.AddApplicationIdentity();
{% endhighlight %}

AddApplicationIdentity is an extension method which can be found in the WebApplication.Identity project.  Placing the ASP.NET Core Identity service configuration into an extension method guaranteed that if any changes were made to the configuration it would be reflected in both the application and unit test.

{% highlight csharp %}
    public static IServiceCollection AddApplicationIdentity(this IServiceCollection services)
    {
        services.AddIdentity<ApplicationUser, ApplicationRole>()
            .AddUserManager<UserManager<ApplicationUser>>()
            .AddEntityFrameworkStores<ApplicationIdentityDbContext>()
            .AddDefaultTokenProviders(); 

        return services;
    }
{% endhighlight %}

Finally, with the ASP.NET Core Identity service configured we can get both the UserManager and RoleManager services for use in a unit test.

{% highlight csharp %}
    userManager = serviceCollection.BuildServiceProvider().GetService<UserManager<ApplicationUser>>();
    roleManager = serviceCollection.BuildServiceProvider().GetService<RoleManager<ApplicationRole>>();
{% endhighlight %}

The last part of the code in the Setup() method is used to pull data from the appSettings.json file.

{% highlight csharp %}
    var builder = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true); 

    IConfigurationRoot configuration = builder.Build();

    var tokenValidationSection = configuration.GetSection("TokenValidation");
    serviceCollection.Configure<TokenValidation>(tokenValidationSection);

    tokenValidation = serviceCollection.BuildServiceProvider().GetService<IOptions<TokenValidation>>();
{% endhighlight %}

The code varies on last line between the controller and the user manager extension unit test in how data from the application settings file is retrieved.

{% highlight csharp %}
    tokenValidationSettings = serviceCollection.BuildServiceProvider().GetService<IOptions<TokenValidation>>().Value;
{% endhighlight %}

The code in the TearDown() method simply makes sure the database is deleted, disposed of and the connection is closed.

{% highlight csharp %}
    identityDbContext.Database.EnsureDeleted();
    identityDbContext.Dispose();
    sqliteConnection.Close();
{% endhighlight %}

Now that we have a valid UserManager and RoleManager instance we can proceed to using then in our unit test.

{% highlight csharp %}
    [Test]
    public async Task ChangePasswordAsync_ConfirmChange()
    {
        // Seed database with user.
        string userName = "Test";
        string password = "Abc!23";
        ApplicationUser user = new ApplicationUser() { UserName = userName, Email = "mail@domain.com" };
        await userManager.CreateAsync(user, password);
        string passwordHash = user.PasswordHash;

        // Run test.
        IdentityResult result = await userManager.ChangePasswordAsync(user, "!23Abc"); 

        Assert.That(user.PasswordHash, Is.Not.EqualTo(passwordHash));
    }
{% endhighlight %}

In the unit test ChangePasswordAsync_ConfirmChange in UserManagerExtensionsUnitTest.cs you can see how we use the UserManager instances to create a user, get the created password hash and then test.

## Summary

So, yes it is this simple.  Create a service collection, instantiate the ASP.NET Core Identity and consume the instance in your unit test.