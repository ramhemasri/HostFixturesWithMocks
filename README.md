# HostFixturesWithMocks
How-to override DI registered services with mocks in ASP.NET Core TestServer fixtures
Replacing DI container registrations for ASP.NET Core scenario testing
Integration testing an ASP.NET Core application is made easy using the Microsoft.AspNetCore.Testing NuGet package. It allows for setting up an in memory test host and running requests against it.
```csharp
var webHost = new WebHostBuilder().UseStartup<Host.Startup>();
var testserver = new TestServer(webHost); 
var client = testserver.CreateClient();
var response = await client.GetStringAsync(“/”);
Assert.Equal(“My API response”, response);
```
This makes it easy to test the whole pipeline works, including bits like authentication, container registrations and so forth. You are free to set it up as close to production as you like. I like to test my app running, but mocking all external dependencies like database calls / network calls. The easiest way I’ve found for doing this is by use of a new feature coming in Microsoft.AspNetCore.Testing version 2.1. It lets us override existing registrations with mocks — via the .ConfigureTestServices(..) method.
```csharp
var webHost = new WebHostBuilder()
     .ConfigureTestServices(s => s.AddSingleton<IFoo, MockedFoo>())              
     .UseStartup<Host.Startup>();
var testserver = new TestServer(webHost); 
var client = testserver.CreateClient();
var response = await client.GetStringAsync(“/”);
Assert.Equal(“My mocked response”, response);
```
Voila, and we’re overriding whatever was registered for IFoo in our startup with a mock class instead for our test.

NB! Since 2.1 not yet has reached Nuget.org, you need to fetch it from their pre-release NuGet feed :

https://dotnet.myget.org/F/aspnetcore-dev/api/v3/index.json

