## Hi 👋, I'm Christian Resma Helle

![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=christianhelle&count_private=true)

### 📙 Blog Posts
<!--START_SECTION:feed-->
#### [Extending Visual Studio for Mac 2022](https:&#x2F;&#x2F;christianhelle.com&#x2F;2023&#x2F;03&#x2F;extending-vsmac.html) 
*This is step by step walkthrough guide to getting started with developing extensions for Visual Studio for Mac 2022 with explanations, code examples, and a couple of links to offical documentation
The extensibility story for Visual Studio for Mac was almost non-existent for a while, and the documentation for getting started was really outdated. Visual Studio for Mac was originally a re-branding of Xamarin Studio, which was built over MonoDevelop and the extensibility SDK’s we used for the longest time was all from the old MonoDevelop Addin libraries. The original getting started guide from MonoDevelop is still somewhat correct, but the libraries referred to in the guide will no longer build.
For Visual Studio for Mac 2022 this has changed and now we can create Visual Studio for Mac extensions using the Microsoft.VisualStudioMac.Sdk library that we can install from nuget.org. To make things even better, we can now break free of our old .NET Framework 4.x shackles and start targetting .NET 7.0. and all it’s goodness
As of the time I’m writing this, there is still no File -&gt; New -&gt; Extension Project experience, but it’s not hard to get started either.
Walkthrough
In this walkthrough, we will build a simple Visual Studio for Mac extension that adds the Insert Text menu item to the Edit menu. All this can do is to insert the text &#x2F;&#x2F; Hello to the active document from the current cursor position
Step 1 - Create New Project
Let’s start with creating a new project called Sample.csproj
Here’s how a csproj file for an empty Visual Studio for Mac extension project looks like:

&lt;Project Sdk&#x3D;&quot;Microsoft.NET.Sdk&quot;&gt;
  &lt;PropertyGroup&gt;
    &lt;TargetFramework&gt;net7.0&lt;&#x2F;TargetFramework&gt;
  &lt;&#x2F;PropertyGroup&gt;
  &lt;ItemGroup&gt;
    &lt;PackageReference Include&#x3D;&quot;Microsoft.VisualStudioMac.Sdk&quot; Version&#x3D;&quot;17.0.0&quot; &#x2F;&gt;
  &lt;&#x2F;ItemGroup&gt;
&lt;&#x2F;Project&gt;


Step 2 - Addin info
A Visual Studio for Mac extension has metadata about its name, version, dependencies, etc. It also defines any number of extensions that plug into extension points defined by other extensions, and can also define extension points that other extensions can extend.
Let’s define some AddIn information in a file called AddinInfo.cs

using Mono.Addins;
using Mono.Addins.Description;

[assembly: Addin(Id &#x3D; &quot;Sample&quot;, Namespace &#x3D; &quot;Sample&quot;, Version &#x3D; &quot;1.0&quot;)]
[assembly: AddinName(&quot;My First Extension&quot;)]
[assembly: AddinCategory(&quot;IDE extensions&quot;)]
[assembly: AddinDescription(&quot;My first Visual Studio for Mac extension&quot;)]
[assembly: AddinAuthor(&quot;Christian Resma Helle&quot;)]


The combined Id and Namespace from Addin should be unique among all Visual Studio for Mac extensions. The other attributes are self-explanatory
Step 3 - Addin Manifest
Now that the Addin is defined, we can add some extensions.
We do this by defining the Manifest.addin.xml file

&lt;?xml version&#x3D;&quot;1.0&quot; encoding&#x3D;&quot;UTF-8&quot;?&gt;
&lt;ExtensionModel&gt;
    &lt;Extension path &#x3D; &quot;&#x2F;MonoDevelop&#x2F;Ide&#x2F;Commands&#x2F;Edit&quot;&gt;
        &lt;Command id &#x3D; &quot;Sample.SampleCommands.InsertText&quot;
            _label &#x3D; &quot;Insert Text&quot;
            defaultHandler &#x3D; &quot;Sample.InsertTextHandler&quot; &#x2F;&gt;
    &lt;&#x2F;Extension&gt;

    &lt;Extension path &#x3D; &quot;&#x2F;MonoDevelop&#x2F;Ide&#x2F;MainMenu&#x2F;Edit&quot;&gt;
        &lt;CommandItem id&#x3D;&quot;Sample.SampleCommands.InsertText&quot; &#x2F;&gt;
    &lt;&#x2F;Extension&gt;
&lt;&#x2F;ExtensionModel&gt;


This extension defines a command for the command system. The Command ID should correspond to an enum value. The _label attribute is the display name of the command. The defaultHandler attribute is the full type name of the CommandHandler implementation that will execute when the extension executes
The Command System provides ways to control the availability, visibility and handling of commands depending on context.
Commands can be bound to keyboard shortcuts and can be inserted into menus. In this exaple, we are going to insert the InsertText command into the main Edit menu with another extension.
Step 4 - Implement the CommandHandler
Now that the InsertText command is registered, we need to implement a command handler.  The simplest way to use it is with a default handler, which is a class that implements MonoDevelop.Components.Commands.CommandHandler. Let’s implement CommandHandler as InsertTextHandler to be only avaiable when an active document is open
We will also need to create the SampleCommands enum

using MonoDevelop.Components.Commands;
using MonoDevelop.Ide;
using MonoDevelop.Ide.Gui;
using System;

namespace Sample
{
    public class InsertTextHandler : CommandHandler
    {
        protected override void Run()
        {
            var textBuffer &#x3D; IdeApp.Workbench.ActiveDocument.GetContent&lt;ITextBuffer&gt;();
            var textView &#x3D; IdeApp.Workbench.ActiveDocument.GetContent&lt;ITextView&gt;();
            textBuffer.Insert(textView.Caret.Position.BufferPosition.Position, &quot;&#x2F;&#x2F; Hello&quot;);
        }

        protected override void Update(CommandInfo info)
        {
            var textBuffer &#x3D; IdeApp.Workbench.ActiveDocument.GetContent&lt;ITextBuffer&gt;();
            if (textBuffer !&#x3D; null &amp;&amp; textBuffer.AsTextContainer() is SourceTextContainer container)
            {
                var document &#x3D; container.GetTextBuffer();
                if (document !&#x3D; null)
                {
                    info.Enabled &#x3D; true;
                }
           }
        }
    }

    public enum SampleCommands
    {
        InsertText,
    }
}


Step 5 - Package the extension
This can be done by right clicking on the extension project from Visual Studio for Mac then selecting Pack from the context menu

You can also do it from the command line. With the new SDK, Microsoft.VisualStudioMac.Sdk, you can build the project from the command line simply by using dotnet build. Running dotnet build will ONLY build the project, it will not create the distributable .mpack package.
Let’s start with building the project in Release configuration

$ dotnet build -c Release Sample.csproj


This will produce the bin&#x2F;Release&#x2F;net7.0&#x2F;Sample.dll file
To create the .mpack package from the command line, we need to use the Visual Studio Tool Runner a.k.a. vstool. The Visual Studio Tool Runner is included in the Visual Studio for Mac installation. The Visual Studio Tool Runner is available from the following path

$ &#x2F;Applications&#x2F;Visual\ Studio.app&#x2F;Contents&#x2F;MacOS&#x2F;vstool


We need to run the Visual Studio Extension Setup Utility pack command

$ &#x2F;Applications&#x2F;Visual\ Studio.app&#x2F;Contents&#x2F;MacOS&#x2F;vstool setup pack [absolute path to main output DLL] -d:[absolute path to output folder]


A little tip for getting the absolute path is to use $PWD. So if you created your project under the ~&#x2F;projects&#x2F;my-extension folder and this is currently your working directory then you can do something like

$ &#x2F;Applications&#x2F;Visual\ Studio.app&#x2F;Contents&#x2F;MacOS&#x2F;vstool setup pack $PWD&#x2F;Sample.dll -d:$PWD


The command above will produce the output ~&#x2F;projects&#x2F;my-extension&#x2F;Sample.mpack
Step 6 - Test the extension
Debugging a Visual Studio for Mac is possible, but doesn’t come out of the box. To enable Debugging the extension from Visual Studio for Mac we need to add the following to our C# project

&lt;PropertyGroup Condition&#x3D;&quot; &#39;$(RunConfiguration)&#39; &#x3D;&#x3D; &#39;Default&#39; &quot;&gt;
  &lt;StartAction&gt;Program&lt;&#x2F;StartAction&gt;
  &lt;StartProgram&gt;\Applications\Visual Studio.app\Contents\MacOS\VisualStudio&lt;&#x2F;StartProgram&gt;
  &lt;StartArguments&gt;--no-redirect&lt;&#x2F;StartArguments&gt;
  &lt;ExternalConsole&gt;true&lt;&#x2F;ExternalConsole&gt;
&lt;&#x2F;PropertyGroup&gt;


Now our Sample project should look something like this:

&lt;Project Sdk&#x3D;&quot;Microsoft.NET.Sdk&quot;&gt;
  &lt;PropertyGroup&gt;
    &lt;TargetFramework&gt;net7.0&lt;&#x2F;TargetFramework&gt;
  &lt;&#x2F;PropertyGroup&gt;
  &lt;PropertyGroup Condition&#x3D;&quot; &#39;$(RunConfiguration)&#39; &#x3D;&#x3D; &#39;Default&#39; &quot;&gt;
    &lt;StartAction&gt;Program&lt;&#x2F;StartAction&gt;
    &lt;StartProgram&gt;\Applications\Visual Studio.app\Contents\MacOS\VisualStudio&lt;&#x2F;StartProgram&gt;
    &lt;StartArguments&gt;--no-redirect&lt;&#x2F;StartArguments&gt;
    &lt;ExternalConsole&gt;true&lt;&#x2F;ExternalConsole&gt;
  &lt;&#x2F;PropertyGroup&gt;
  &lt;ItemGroup&gt;
    &lt;PackageReference Include&#x3D;&quot;Microsoft.VisualStudioMac.Sdk&quot; Version&#x3D;&quot;17.0.0&quot; &#x2F;&gt;
  &lt;&#x2F;ItemGroup&gt;
&lt;&#x2F;Project&gt;


Debugging the extension will basically start another instance of Visual Studio for Mac where you can test your extension
Try it out and if all goes well the Edit menu should have the Insert Text item at the bottom


Step 7 - Install extension
If you followed Step 5, then you should already have a .mpack at hand. To install a Visual Studio for Mac extension, you need to follow these steps:





You need to restart Visual Studio for Mac at this point before you can see our new extension under the Edit menu

I hope you found this useful and get inspired to start building extensions of your own. If you’re interested in the full source code then you can grab it here*
#### [Generate Refit interfaces from OpenAPI specifications using Refitter](https:&#x2F;&#x2F;christianhelle.com&#x2F;2023&#x2F;03&#x2F;refitter.html) 
*Refit is an awesome tool that I only just recently discovered and started using in systems that involve a lot of HTTP API integration points. With Refit, I can keep my API client code as simple as an interface with some magic sauce behind the scenes.
For those aren’t familiar with Refit, it is an automatic type-safe REST library for .NET. Refit is heavily inspired by Square’s Retrofit library for Android and Java. Refit turns your REST API into a live interface.
Refit currently supports the following platforms and any .NET Standard 2.0 target:
.NET Framework 4.6.1 and above
.NET 5.0 and above
Xamarin.Android
Xamarin.Mac
Xamarin.iOS
Blazor
Uno Platform
UWP
Given that Refit will actually do most of the work, you still need to create the Refit interface and the contracts that are used by the HTTP API your system is calling. A Refit interface could be something as simple as:

public interface IFooApi
{
    [Get(&quot;&#x2F;foo&#x2F;{id}&quot;)]
    Task&lt;Foo&gt; GetFoo(string id);
}


Using the interface above would be the equivalent of performing GET https:&#x2F;&#x2F;foo.api.somesystem.com&#x2F;foo&#x2F;12345 which returns a deserialized Foo resource
Introducing Refitter
I’m generally a lazy person who hates repeating the same tasks more than once and recently I thought that working with Refit could be done easier, so I created Refitter.
Refitter is an open source CLI tool for generating a C# REST API Client using the Refit library. Refitter can generate the Refit interface (and contracts using NSwag) from OpenAPI specifications. Refitter is also available as a library for other IDE extension and develop tool authors to integrate with
Refitter is available in different forms as a CLI Tool and from the REST API Client Code Generator extension that supports the following IDE:
Visual Studio 2019
Visual Studio 2022
Visual Studio for Mac 2022
CLI Tool
The CLI tool is packaged as a .NET Tool and is published to nuget.org. You can install the latest version of this tool like this:

dotnet tool install --global Refitter --prerelease


Refitter provides some --help information for getting started

$ refitter --help



USAGE:
    refitter [input file] [OPTIONS]

EXAMPLES:
    refitter .&#x2F;openapi.json --namespace &quot;Your.Namespace.Of.Choice.GeneratedCode&quot; --output .&#x2F;Output.cs

ARGUMENTS:
    [input file]    Path to OpenAPI Specification file

OPTIONS:
                                      DEFAULT                                                          
    -h, --help                                         Prints help information                         
    -n, --namespace                   GeneratedCode    Default namespace to use for generated types    
    -o, --output                      Output.cs        Path to Output file                             
        --no-auto-generated-header                     Don&#39;t add &lt;auto-generated&gt; header to output file


To generate code from an OpenAPI specifications file, run the following:

$ refitter [path to OpenAPI spec file] --namespace &quot;[Your.Namespace.Of.Choice.GeneratedCode]&quot;


This will generate a file called Output.cs which contains the Refit interface and contract classes generated.
REST API Client Code Generator
Since most of us, including me, spend most of our time in IDE´s, we have a lot of tools in our toolbox. I love Swagger and OpenAPI, which was the reason as to why I built the REST API Client Code Generator. This tool allows me to right click on a solution and select Add New REST API Client and prompts me to enter the URI to where the OpenAPI specifications can be downloaded from
In Visual Studio 2019 and 2022 that looks like this:

and in Visual Studio for Mac it looks like this:

From the context menu, select Generate with Refitter and get this a prompt that looks like this:

This will result in the file being OpenAPI (Swagger) specifications file to be downloaded, included in your project, and configured to use a custom tool that generates a code behind file upon any changes on the OpenAPI specifications file

Using the generated code
Here’s an example generated output from the Swagger Petstore example

using Refit;
using System.Threading.Tasks;
using System.Collections.Generic;

namespace Your.Namespace.Of.Choice.GeneratedCode
{
    public interface ISwaggerPetstore
    {
        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Update an existing pet by Id
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Put(&quot;&#x2F;pet&quot;)]
        Task&lt;Pet&gt; UpdatePet([Body]Pet body);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Add a new pet to the store
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Post(&quot;&#x2F;pet&quot;)]
        Task&lt;Pet&gt; AddPet([Body]Pet body);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Multiple status values can be provided with comma separated strings
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Get(&quot;&#x2F;pet&#x2F;findByStatus&quot;)]
        Task&lt;ICollection&lt;Pet&gt;&gt; FindPetsByStatus();

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Multiple tags can be provided with comma separated strings. Use tag1, tag2, tag3 for testing.
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Get(&quot;&#x2F;pet&#x2F;findByTags&quot;)]
        Task&lt;ICollection&lt;Pet&gt;&gt; FindPetsByTags();

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Returns a single pet
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Get(&quot;&#x2F;pet&#x2F;{petId}&quot;)]
        Task&lt;Pet&gt; GetPetById(long? petId);

        [Post(&quot;&#x2F;pet&#x2F;{petId}&quot;)]
        Task UpdatePetWithForm(long? petId);

        [Delete(&quot;&#x2F;pet&#x2F;{petId}&quot;)]
        Task DeletePet(long? petId);

        [Post(&quot;&#x2F;pet&#x2F;{petId}&#x2F;uploadImage&quot;)]
        Task&lt;ApiResponse&gt; UploadFile(long? petId, [Body]StreamPart body);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Returns a map of status codes to quantities
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Get(&quot;&#x2F;store&#x2F;inventory&quot;)]
        Task&lt;IDictionary&lt;string, int&gt;&gt; GetInventory();

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Place a new order in the store
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Post(&quot;&#x2F;store&#x2F;order&quot;)]
        Task&lt;Order&gt; PlaceOrder([Body]Order body);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; For valid response try integer IDs with value &lt;&#x3D; 5 or &gt; 10. Other values will generated exceptions
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Get(&quot;&#x2F;store&#x2F;order&#x2F;{orderId}&quot;)]
        Task&lt;Order&gt; GetOrderById(long? orderId);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; For valid response try integer IDs with value &lt; 1000. Anything above 1000 or nonintegers will generate API errors
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Delete(&quot;&#x2F;store&#x2F;order&#x2F;{orderId}&quot;)]
        Task DeleteOrder(long? orderId);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; This can only be done by the logged in user.
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Post(&quot;&#x2F;user&quot;)]
        Task CreateUser([Body]User body);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; Creates list of users with given input array
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Post(&quot;&#x2F;user&#x2F;createWithList&quot;)]
        Task&lt;User&gt; CreateUsersWithListInput([Body]ICollection&lt;User&gt; body);

        [Get(&quot;&#x2F;user&#x2F;login&quot;)]
        Task&lt;string&gt; LoginUser();

        [Get(&quot;&#x2F;user&#x2F;logout&quot;)]
        Task LogoutUser();

        [Get(&quot;&#x2F;user&#x2F;{username}&quot;)]
        Task&lt;User&gt; GetUserByName(string username);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; This can only be done by the logged in user.
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Put(&quot;&#x2F;user&#x2F;{username}&quot;)]
        Task UpdateUser(string username, [Body]User body);

        &#x2F;&#x2F;&#x2F; &lt;summary&gt;
        &#x2F;&#x2F;&#x2F; This can only be done by the logged in user.
        &#x2F;&#x2F;&#x2F; &lt;&#x2F;summary&gt;
        [Delete(&quot;&#x2F;user&#x2F;{username}&quot;)]
        Task DeleteUser(string username);
    }
}


Refit provides two ways to use the interface:
Resolve the interface via the RestService class
Register the Refit interface with HttpClientFactory and use it through dependency injection
RestService
Here’s an example usage of the generated code above

using Refit;
using System;
using System.Threading.Tasks;

namespace Your.Namespace.Of.Choice.GeneratedCode;

internal class Program
{
    private static async Task Main(string[] args)
    {
        var client &#x3D; RestService.For&lt;ISwaggerPetstore&gt;(&quot;https:&#x2F;&#x2F;petstore3.swagger.io&#x2F;api&#x2F;v3&quot;);
        var pet &#x3D; await client.GetPetById(2);

        Console.WriteLine(<!--START_SECTION:feed-->
<!--END_SECTION:feed-->quot;Name: {pet.Name}&quot;);
        Console.WriteLine(<!--START_SECTION:feed-->
<!--END_SECTION:feed-->quot;Category: {pet.Category.Name}&quot;);
        Console.WriteLine(<!--START_SECTION:feed-->
<!--END_SECTION:feed-->quot;Status: {pet.Status}&quot;);
    }
}


The RestService class generates an implementation of ISwaggerPetstore that uses HttpClient to make its calls.
The code above when run will output something like this:

Name: Gatitotototo
Category: Chaucito
Status: Sold


ASP.NET Core and HttpClientFactory
Here’s an example Minimal API with the Refit.HttpClientFactory library:

using Refit;
using Your.Namespace.Of.Choice.GeneratedCode;

var builder &#x3D; WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services
    .AddRefitClient&lt;ISwaggerPetstore&gt;()
    .ConfigureHttpClient(c &#x3D;&gt; c.BaseAddress &#x3D; new Uri(&quot;https:&#x2F;&#x2F;petstore3.swagger.io&#x2F;api&#x2F;v3&quot;));

var app &#x3D; builder.Build();
app.MapGet(
        &quot;&#x2F;pet&#x2F;{id:long}&quot;,
        async (ISwaggerPetstore petstore, long id) &#x3D;&gt;
        {
            try
            {
                return Results.Ok(await petstore.GetPetById(id));
            }
            catch (Refit.ApiException e)
            {
                return Results.StatusCode((int)e.StatusCode);
            }
        })
    .WithName(&quot;GetPetById&quot;)
    .WithOpenApi();

app.UseHttpsRedirection();
app.UseSwaggerUI();
app.UseSwagger();
app.Run();


.NET Core supports registering the generated ISwaggerPetstore interface via HttpClientFactory
The following request to the API above

$ curl -X &#39;GET&#39; &#39;https:&#x2F;&#x2F;localhost:5001&#x2F;pet&#x2F;1&#39; -H &#39;accept: application&#x2F;json&#39;


Returns a response that looks something like this:

{
  &quot;id&quot;: 1,
  &quot;name&quot;: &quot;Special_char_owner_!@#$^&amp;()&#x60;.testing&quot;,
  &quot;photoUrls&quot;: [
    &quot;https:&#x2F;&#x2F;petstore3.swagger.io&#x2F;resources&#x2F;photos&#x2F;623389095.jpg&quot;
  ],
  &quot;tags&quot;: [],
  &quot;status&quot;: &quot;Sold&quot;
}


For those of you who never tried Refit, I think that you should definitely check it out. It’s very easy to use*
#### [Atc.Cosmos - Azure Cosmos DB with A Touch of Class](https:&#x2F;&#x2F;christianhelle.com&#x2F;2023&#x2F;02&#x2F;atc-cosmos.html) 
*For the past 6 years, I have been using Azure Cosmos DB as my go-to data store. Document databases make so much more sense for the things that I have been building over the past 6 years. The library Atc.Cosmos is the result of years of collective experience solving problems using the same patterns. Atc.Cosmos is a library for configuring containers in Azure Cosmos DB and provides easy, efficient, and convenient ways to read and write document resources.
Using Atc.Cosmos
Here’s an example usage of Atc.Cosmos in a Minimal API project targeting .NET 7.0

var builder &#x3D; WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.ConfigureCosmosDb();

var app &#x3D; builder.Build();
app.MapGet(
    &quot;&#x2F;foo&quot;,
    (
        ICosmosReader&lt;FooResource&gt; reader,
        CancellationToken cancellationToken) &#x3D;&gt;
            reader
                .ReadAllAsync(FooResource.PartitionKey, cancellationToken)
                .ToBlockingEnumerable(cancellationToken)
                .Select(c &#x3D;&gt; c.Bar))
    .WithName(&quot;ListFoo&quot;)
    .WithOpenApi();
app.MapGet(
    &quot;&#x2F;foo&#x2F;{id}&quot;,
    async (
        ICosmosReader&lt;FooResource&gt; reader,
        string id,
        CancellationToken cancellationToken) &#x3D;&gt;
        {
            var foo &#x3D; await reader.FindAsync(id, FooResource.PartitionKey, cancellationToken);
            return foo is not null ? Results.Ok(foo.Bar) : Results.NotFound(id);
        })
    .WithName(&quot;GetFoo&quot;)
    .WithOpenApi();
app.MapPost(
    &quot;&#x2F;foo&quot;,
    async (
        ICosmosWriter&lt;FooResource&gt; writer,
        [FromBody] Dictionary&lt;string, object&gt; data,
        CancellationToken cancellationToken) &#x3D;&gt;
        {
            var id &#x3D; Guid.NewGuid().ToString();
            await writer.CreateAsync(
                new FooResource
                {
                    Id &#x3D; id,
                    Bar &#x3D; data,
                },
                cancellationToken);
            return Results.CreatedAtRoute(&quot;GetFoo&quot;, new { id });
        })
    .WithName(&quot;PostFoo&quot;)
    .WithOpenApi();

app.UseHttpsRedirection();
app.UseSwaggerUI();
app.UseSwagger();
app.Run();


Let’s break that down a bit and start with the IServiceCollection extension method ConfigureCosmosDb().
To use Atc.Cosmos you need to do the following:
Implement IConfigureOptions&lt;CosmosOptions&gt; to configure the database itself
Define Cosmos resource document types by deriving from CosmosResource or implementing ICosmosResource
Implement ICosmosContainerInitialize to define a CosmosDb container for every Cosmos resource document type




public static class ServiceCollectionExtensions
{
    public static void ConfigureCosmosDb(this IServiceCollection services)
    {
        services.ConfigureOptions&lt;ConfigureCosmosOptions&gt;();
        services.ConfigureCosmos(
            cosmosBuilder &#x3D;&gt;
            {
                cosmosBuilder.AddContainer&lt;FooContainerInitializer, FooResource&gt;(&quot;foo&quot;);
                cosmosBuilder.UseHostedService();
            });
    }
}


Here’s an example implementation of IConfigureOptions&lt;CosmosOptions&gt;

public class ConfigureCosmosOptions : IConfigureOptions&lt;CosmosOptions&gt;
{
    public void Configure(CosmosOptions options)
    {
        options.UseCosmosEmulator();
        options.DatabaseName &#x3D; &quot;SampleApi&quot;;
        options.SerializerOptions.PropertyNamingPolicy &#x3D; JsonNamingPolicy.CamelCase;
    }
}


Here’s an example implementation of the ICosmosContainerInitializer interface for creating a container called foo:

public class FooContainerInitializer : ICosmosContainerInitializer
{
    public Task InitializeAsync(
        Database database,
        CancellationToken cancellationToken) &#x3D;&gt;
        database.CreateContainerIfNotExistsAsync(
            new ContainerProperties
            {
                PartitionKeyPath &#x3D; &quot;&#x2F;pk&quot;,
                Id &#x3D; &quot;foo&quot;,
            },
            cancellationToken: cancellationToken);
}


Here’s an example Cosmos resource document type called FooResource that derives from CosmosResource

public class FooResource : CosmosResource
{
    public const string PartitionKey &#x3D; &quot;foo&quot;;
    public string Id { get; set; } &#x3D; null!;
    public string Pk &#x3D;&gt; PartitionKey;
    public Dictionary&lt;string, object&gt; Bar { get; set; } &#x3D; new Dictionary&lt;string, object&gt;();
    protected override string GetDocumentId() &#x3D;&gt; Id;
    protected override string GetPartitionKey() &#x3D;&gt; Pk;
}


ICosmosReader&lt; T &gt;
Cosmos DB is very good at point-read operations, and this is cheap to do. The ICosmosReader&lt;T&gt; interface provides the following methods for point read operations:

Task&lt;T&gt; ReadAsync(
    string documentId, 
    string partitionKey, 
    CancellationToken cancellationToken &#x3D; default);

Task&lt;T?&gt; FindAsync(
    string documentId, 
    string partitionKey, 
    CancellationToken cancellationToken &#x3D; default);


ReadAsync() does a point read look-up on the document within the specified partition and throws a CosmosException with the Status code NotFound if the resource could not be found. FindAsync() on the other hand will return a null instance of T if the resource count not be found
You will notice that the majority of methods exposed in ICosmosReader&lt;T&gt; require the partition key to be specified. this is because read operations on Azure Cosmos DB are very cheap and efficient as long as you stay within a single partition.
ICosmosReader&lt;T&gt; provides methods for reading multiple documents out. This can be done by reading all the documents within a partition or running a query against the partition. Here are some methods that do exactly that:

IAsyncEnumerable&lt;T&gt; ReadAllAsync(
    string partitionKey, 
    CancellationToken cancellationToken &#x3D; default);

IAsyncEnumerable&lt;T&gt; QueryAsync(
    QueryDefinition query, 
    string partitionKey, 
    CancellationToken cancellationToken &#x3D; default);


As the name states, ReadAllAsync() reads all documents from the specified partition and returns an asynchronous stream of individual documents. QueryAsync() executes a QueryDefinition against the specified partition.
When working with large partitions, you will most likely want to use paging to read out data so that you can return a response to the consumer of your system as fast as possible. ICosmosReader&lt;T&gt; provides the following methods for paged queries:

Task&lt;PagedResult&lt;T&gt;&gt; PagedQueryAsync(
    QueryDefinition query,
    string partitionKey, 
    int? pageSize,
    string? continuationToken &#x3D; default,
    CancellationToken cancellationToken &#x3D; default);


When working with very large partitions, you might want to parallelize the processing of the documents you read from Cosmos DB, and this can be done by streaming a collection of documents instead of individual ones. ICosmosReader&lt;T&gt; provides the following methods for batch queries

IAsyncEnumerable&lt;IEnumerable&lt;T&gt;&gt; BatchReadAllAsync(
    string partitionKey,
    CancellationToken cancellationToken &#x3D; default);

IAsyncEnumerable&lt;IEnumerable&lt;T&gt;&gt; BatchQueryAsync(
    QueryDefinition query,
    string partitionKey,
    CancellationToken cancellationToken &#x3D; default);


Cross-partition queries are normally very inefficient, expensive, and slow. Regardless of these facts, there will be times when you will still need them. ICosmosReader&lt;T&gt; provides the following methods for performing cross-partition read operations. ICosmosReader&lt;T&gt; provides methods for executing a query, a paged query, or a batch query across multiple partitions

IAsyncEnumerable&lt;T&gt; CrossPartitionQueryAsync(
    QueryDefinition query,
    CancellationToken cancellationToken &#x3D; default);

Task&lt;PagedResult&lt;T&gt;&gt; CrossPartitionPagedQueryAsync(
    QueryDefinition query,
    int? pageSize,
    string? continuationToken &#x3D; default,
    CancellationToken cancellationToken &#x3D; default);

IAsyncEnumerable&lt;IEnumerable&lt;T&gt;&gt; BatchCrossPartitionQueryAsync(
    QueryDefinition query,
    CancellationToken cancellationToken &#x3D; default);


ICosmosWriter &lt; T &gt;
There are multiple ways to write to Cosmos DB and my preferred way is to do upserts. This is to create when not exist, otherwise, update. ICosmosWriter&lt;T&gt; provides methods for simple upsert operations and methods that includes retry attempts.

Task&lt;T&gt; WriteAsync(
    T document,
    CancellationToken cancellationToken &#x3D; default);

Task&lt;T&gt; UpdateOrCreateAsync(
    Func&lt;T&gt; getDefaultDocument,
    Action&lt;T&gt; updateDocument,
    int retries &#x3D; 0,
    CancellationToken cancellationToken &#x3D; default);


Deleting a resource will usually involve knowing what resource to delete. ICosmosWriter&lt;T&gt; provides methods for deleting a resource that MUST exists and another method that returns true if the resource was successfully deleted, otherwise false

Task DeleteAsync(
    string documentId,
    string partitionKey,
    CancellationToken cancellationToken &#x3D; default);

Task&lt;bool&gt; TryDeleteAsync(
    string documentId,
    string partitionKey,
    CancellationToken cancellationToken &#x3D; default);


Unit Testing
The ICosmosReader&lt;T&gt; and ICosmosWriter&lt;T&gt; interfaces can easily be mocked, but there might be cases where you would want to fake it instead. For this purpose, you can use the FakeCosmosReader&lt;T&gt; or FakeCosmosWriter&lt;T&gt; classes from the Atc.Cosmos.Testing namespace contains the following fakes. For convenience, Atc.Cosmos.Testing provides the FakeCosmos&lt;T&gt; class which fakes both the reader and writer
Based on the example in the beginning of this post, let’s say we have a component called FooService which can do CRUD operations over the FooResource

public class FooService
{
    private readonly ICosmosReader&lt;FooResource&gt; reader;
    private readonly ICosmosWriter&lt;FooResource&gt; writer;

    public FooService(
        ICosmosReader&lt;FooResource&gt; reader,
        ICosmosWriter&lt;FooResource&gt; writer)
    {
        this.reader &#x3D; reader;
        this.writer &#x3D; writer;
    }

    public Task&lt;FooResource?&gt; FindAsync(
        string id,
        CancellationToken cancellationToken &#x3D; default) &#x3D;&gt;
        reader.FindAsync(id, FooResource.PartitionKey, cancellationToken);

    public Task UpsertAsync(
        string? id &#x3D; null,
        Dictionary&lt;string, object&gt;? data &#x3D; null,
        CancellationToken cancellationToken &#x3D; default) &#x3D;&gt;
        writer.UpdateOrCreateAsync(
            () &#x3D;&gt; new FooResource { Id &#x3D; id ?? Guid.NewGuid().ToString() },
            resource &#x3D;&gt; resource.Data &#x3D; data ?? new Dictionary&lt;string, object&gt;(),
            retries: 5,
            cancellationToken);
}


Using a combination of Atc.Cosmos.Testing and the Atc.Test library, unit tests using the fakes could look like this:

public class FooServiceTests
{
    [Theory]
    [AutoNSubstituteData]
    public async Task Should_Get_Existing_Data(
        [Frozen(Matching.ImplementedInterfaces)] FakeCosmos&lt;FooResource&gt; fakeCosmos,
        FooService sut,
        FooResource resource)
    {
        fakeCosmos.Documents.Add(resource);
        (await sut.FindAsync(resource.Id)).Should().NotBeNull();
    }

    [Theory]
    [AutoNSubstituteData]
    public async Task Should_Create_New_Data(
        [Frozen(Matching.ImplementedInterfaces)] FakeCosmos&lt;FooResource&gt; fakeCosmos,
        FooService sut,
        Dictionary&lt;string, object&gt; data)
    {
        var count &#x3D; fakeCosmos.Documents.Count;
        await sut.UpsertAsync(data: data);
        fakeCosmos.Documents.Should().HaveCount(count + 1);
    }

    [Theory]
    [AutoNSubstituteData]
    public async Task Should_Update_Existing_Data(
        [Frozen(Matching.ImplementedInterfaces)] FakeCosmos&lt;FooResource&gt; fakeCosmos,
        FooService sut,
        FooResource resource,
        Dictionary&lt;string, object&gt; data)
    {
        fakeCosmos.Documents.Add(resource);
        await sut.UpsertAsync(resource.Id, data);

        fakeCosmos
            .Documents
            .First(c &#x3D;&gt; c.Id &#x3D;&#x3D; resource.Id)
            .Data
            .Should()
            .BeEquivalentTo(data);
    }
}


If you’re interested in the full source code then you can grab it here.*
#### [Generate REST API Clients using Visual Studio and Microsoft Kiota](https:&#x2F;&#x2F;christianhelle.com&#x2F;2023&#x2F;02&#x2F;visual-studio-kiota.html) 
*A week ago while I was browsing around what’s trending on Github, I stumbled upon something called Microsoft Kiota. Kiota is a command line tool for generating an API client to call any OpenAPI described API you are interested in. Getting started was quite a good experience as documentation from project Kiota is quite decent, especially for Building SDK’s in .NET.
Since Kiota is a .NET Tool and distributed on nuget.org, installation is as simple as

dotnet tool install --global --prerelease Microsoft.OpenApi.Kiota


The command line arguments for Kiota is very straight forward and looks something like this:

kiota generate -d [relative path to OpenAPI spec file] -n [default namespace] -o [relative output path]


The code generated by Microsoft Kiota depends on the following NuGet packages:
Microsoft.Kiota.Abstractions
Microsoft.Kiota.Http.HttpClientLibrary
Microsoft.Kiota.Serialization.Form
Microsoft.Kiota.Serialization.Json
Microsoft.Kiota.Serialization.Text
Microsoft.Kiota.Authentication.Azure
Azure.Identity
Kiota is built to target .NET 7.0 so this is required to run Kiota. The code C# generated by Kiota builds on the following .NET versions:
.NET 7.0 and 6.0
.NET Standard 2.1 and 2.0
.NET Framework 4.8.1, 4.8, 4.7.2, 4.6.2
All this perfectly fits the requirements I have in my Visual Studio extension, REST API Client Code Generator, so I immediately got started with integrating Kiota in my extension

After generating code, the extension will install the required NuGet packages and configure the project to use a Custom Tool on the OpenAPI Swagger file

Currently my tool just installs and runs the Kiota CLI tool, so there is a slight pause in Visual Studio while the custom tool is running, because Visual Studio needs to start an external process and wait for the results
Here’s an example of how to use the Kiota generated code from the Swagger Petstore OpenAPI specifications example.

using Microsoft.Kiota.Http.HttpClientLibrary;
using Microsoft.Kiota.Abstractions.Authentication;

var authProvider &#x3D; new AnonymousAuthenticationProvider();
var requestAdapter &#x3D; new HttpClientRequestAdapter(authProvider);
requestAdapter.BaseUrl &#x3D; &quot;https:&#x2F;&#x2F;petstore3.swagger.io&#x2F;api&#x2F;v3&quot;;

var client &#x3D; new ApiClient(requestAdapter);
var pet &#x3D; await client.Pet[&quot;0&quot;].GetAsync(c &#x3D;&gt; c.Headers.Add(&quot;accept&quot;, &quot;application&#x2F;json&quot;));

Console.WriteLine(<!--START_SECTION:feed-->
<!--END_SECTION:feed-->quot;Name: {pet.Name}&quot;);
Console.WriteLine(<!--START_SECTION:feed-->
<!--END_SECTION:feed-->quot;Category: {pet.Category.Name}&quot;);
Console.WriteLine(<!--START_SECTION:feed-->
<!--END_SECTION:feed-->quot;Status: {pet.Status}&quot;);


The code above outputs the following:

Name: doggie
Category: Dogs
Status: Available


It is the equivalent of calling the Swagger Petstore API using cURL

curl -X &#39;GET&#39; &#39;https:&#x2F;&#x2F;petstore3.swagger.io&#x2F;api&#x2F;v3&#x2F;pet&#x2F;0&#39; -H &#39;accept: application&#x2F;json&#39;


which responds with

{
   &quot;id&quot;:0,
   &quot;category&quot;:{
      &quot;id&quot;:1,
      &quot;name&quot;:&quot;Dogs&quot;
   },
   &quot;name&quot;:&quot;doggie&quot;,
   &quot;photoUrls&quot;:[
      &quot;string&quot;
   ],
   &quot;tags&quot;:[
      {
         &quot;id&quot;:0,
         &quot;name&quot;:&quot;string&quot;
      }
   ],
   &quot;status&quot;:&quot;available&quot;
}


I think Kiota is really promising and I think that you should also give it a shot. You can use it via my Visual Studio extension REST API Client Code Generator, my CLI tool Rapicgen, or via the Kiota CLI Tool directly*
#### [AutoFaker - A Python library to minimize unit testing ceremony](https:&#x2F;&#x2F;christianhelle.com&#x2F;2022&#x2F;10&#x2F;autofaker.html) 
*Around a year ago, I found myself working full time in Python. Coming from a C# background, I really missed the unit testing tools you have in .NET. To be more specific, I missed things like xUnit, AutoFixture, and Fluent Assertions. This immediately got me started working on a project I called AutoFaker
AutoFaker is a Python library designed to minimize the setup&#x2F;arrange phase of your unit tests by removing the need to manually 
write code to create anonymous variables as part of a test cases setup&#x2F;arrange phase.
This library is heavily inspired by AutoFixture and was initially created 
for simplifying how to write unit tests for ETL (Extract-Transform-Load) code running from a python library on an 
Apache Spark cluster in Big Data solutions.
When writing unit tests you normally start with creating objects that represent the initial state of the test.
This phase is called the arrange or setup phase of the test.
In most cases, the system you want to test will force you to specify much more information than you really care about, 
so you frequently end up creating objects with no influence on the test itself just simply to satisfy the compiler&#x2F;interpreter
AutoFaker is available from PyPI and should be installed using pip

pip install autofaker


AutoFaker can help by creating such anonymous variables for you. Here’s a simple example:

import unittest
from autofaker import Autodata

class Calculator:
  def add(self, number1: int, number2: int):
    return number1 + number2

class CalculatorTests(unittest.TestCase):
    def test_can_add_two_numbers(self):      
        # arrange
        numbers &#x3D; Autodata.create_many(int, 2)
        sut &#x3D; Autodata.create(Calculator)        
        # act
        result &#x3D; sut.add(numbers[0], numbers[1])        
        # assert
        self.assertEqual(numbers[0] + numbers[1], result)


Since the point of this library is to simplify the arrange step of writing unit tests, we can use the
@autodata and @fakedata are available to explicitly state 
whether to use anonymous variables or fake data and construct our system under test.
To use this you can either define the types or the arguments as function arguments to the decorator, or specify 
argument annotations

import unittest
from autofaker import autodata

class Calculator:
  def add(self, number1: int, number2: int):
    return number1 + number2

class CalculatorTests(unittest.TestCase):
    @autodata(Calculator, int, int)
    def test_can_add_two_numbers_using_test_arguments(self, sut, number1, number2):
        result &#x3D; sut.add(number1, number2)
        self.assertEqual(number1 + number2, result)

    @autodata()
    def test_can_add_two_numbers_using_annotated_arguments(self, 
                                                           sut: Calculator, 
                                                           number1: int, 
                                                           number2: int):
        result &#x3D; sut.add(number1, number2)
        self.assertEqual(number1 + number2, result)


There are times when completely anonymous variables don’t make much sense, especially in data centric scenarios. 
For these use cases this library uses Faker for generating fake data. This option 
is enabled by setting use_fake_data to True when calling the Autodata.create() function

from dataclasses import dataclass
from autofaker import Autodata

@dataclass
class DataClass:
    id: int
    name: str
    job: str

data &#x3D; Autodata.create(DataClass, use_fake_data&#x3D;True)

print(f&#39;id:     {data.id}&#39;)
print(f&#39;name:   {data.name}&#39;)
print(f&#39;job:    {data.job}\n&#39;)


The following code above might output something like:

id:     8952
name:   Justin Wise
job:    Chief Operating Officer


Supported data types
Currently autofaker supports creating anonymous variables for the following data types:
Built-in types:
int
float
str
complex
range
bytes
bytearray
Datetime types:
datetime
date
Classes:
Simple classes
@dataclass
Nested classes (and recursion)
Classes containing lists of other types
Dataframes:
Pandas dataframe
Example usages
Create anonymous built-in types like int, float, str and datetime types like datetime and date

print(f&#39;anonymous string:    {Autodata.create(str)}&#39;)
print(f&#39;anonymous int:       {Autodata.create(int)}&#39;)
print(f&#39;anonymous float:     {Autodata.create(float)}&#39;)
print(f&#39;anonymous complex:   {Autodata.create(complex)}&#39;)
print(f&#39;anonymous range:     {Autodata.create(range)}&#39;)
print(f&#39;anonymous bytes:     {Autodata.create(bytes)}&#39;)
print(f&#39;anonymous bytearray: {Autodata.create(bytearray)}&#39;)
print(f&#39;anonymous datetime:  {Autodata.create(datetime)}&#39;)
print(f&#39;anonymous date:      {Autodata.create(datetime.date)}&#39;)


The code above might output the following

anonymous string:    f91954f1-96df-463f-a427-665c99213395
anonymous int:       2066712686
anonymous float:     725758222.8712853
anonymous datetime:  2017-06-19 02:40:41.000084
anonymous date:      2019-11-10 00:00:00


Creates an anonymous class


class SimpleClass:
    id &#x3D; -1
    text &#x3D; &#39;test&#39;

cls &#x3D; Autodata.create(SimpleClass)
print(f&#39;id &#x3D; {cls.id}&#39;)
print(f&#39;text &#x3D; {cls.text}&#39;)


The code above might output the following

id &#x3D; 2020177162
text &#x3D; ac54a65d-b4a3-4eda-a840-eb948ad10d5f


Create a collection of an anonymous class

class SimpleClass:
    id &#x3D; -1
    text &#x3D; &#39;test&#39;

classes &#x3D; Autodata.create_many(SimpleClass)
for cls in classes:
  print(f&#39;id &#x3D; {cls.id}&#39;)
  print(f&#39;text &#x3D; {cls.text}&#39;)
  print()


The code above might output the following

id &#x3D; 242996515
text &#x3D; 5bb60504-ccca-4104-9b7f-b978e52a6518

id &#x3D; 836984239
text &#x3D; 079df61e-a87e-4f26-8196-3f44157aabd6

id &#x3D; 570703150
text &#x3D; a3b86f08-c73a-4730-bde7-4bdff5360ef4


Creates an anonymous dataclass

from dataclasses import dataclass

@dataclass
class DataClass:
    id: int
    text: str

cls &#x3D; Autodata.create(DataClass)
print(f&#39;id &#x3D; {cls.id}&#39;)
print(f&#39;text &#x3D; {cls.text}&#39;)


The code above might output the following

id &#x3D; 314075507
text &#x3D; 4a3b3cae-f4cf-4502-a7f3-61115a1e0d2a


Creates an anonymous dataclass using fake data

@dataclass
class DataClass:
    id: int

    name: str
    address: str
    job: str

    country: str
    currency_name: str
    currency_code: str

    email: str
    safe_email: str
    company_email: str

    hostname: str
    ipv4: str
    ipv6: str

    text: str


data &#x3D; Autodata.create(DataClass, use_fake_data&#x3D;True)

print(f&#39;id:               {data.id}&#39;)
print(f&#39;name:             {data.name}&#39;)
print(f&#39;job:              {data.job}\n&#39;)
print(f&#39;address:\n{data.address}\n&#39;)

print(f&#39;country:          {data.country}&#39;)
print(f&#39;currency name:    {data.currency_name}&#39;)
print(f&#39;currency code:    {data.currency_code}\n&#39;)

print(f&#39;email:            {data.email}&#39;)
print(f&#39;safe email:       {data.safe_email}&#39;)
print(f&#39;work email:       {data.company_email}\n&#39;)

print(f&#39;hostname:         {data.hostname}&#39;)
print(f&#39;IPv4:             {data.ipv4}&#39;)
print(f&#39;IPv6:             {data.ipv6}\n&#39;)

print(f&#39;text:\n{data.text}&#39;)


The code above might output the following

id:               8952
name:             Justin Wise
job:              Chief Operating Officer

address:
65939 Hernandez Parks
Rochaport, NC 41760

country:          Equatorial Guinea
currency name:    Burmese kyat
currency code:    ERN

email:            smithjohn@example.com
safe email:       kent11@example.com
work email:       marissagreen@brown-cole.com

hostname:         db-90.hendricks-west.org
IPv4:             66.139.143.242
IPv6:             895d:82f7:7c13:e7cb:f35d:c93:aeb2:8eeb

text:
Movie author culture represent. Enjoy myself over physical green lead but home.
Share wind factor far minute produce significant. Sense might fact leader.


Create an anonymous class with nested types


class NestedClass:
    id &#x3D; -1
    text &#x3D; &#39;test&#39;
    inner &#x3D; SimpleClass()

cls &#x3D; Autodata.create(NestedClass)
print(f&#39;id &#x3D; {cls.id}&#39;)
print(f&#39;text &#x3D; {cls.text}&#39;)
print(f&#39;inner.id &#x3D; {cls.inner.id}&#39;)
print(f&#39;inner.text &#x3D; {cls.inner.text}&#39;)


The code above might output the following

id &#x3D; 1565737216
text &#x3D; e66ecd5c-c17a-4426-b755-36dfd2082672
inner.id &#x3D; 390282329
inner.text &#x3D; eef94b5c-aa95-427a-a9e6-d99e2cc1ffb2


Create a collection of an anonymous class with nested types

class NestedClass:
    id &#x3D; -1
    text &#x3D; &#39;test&#39;
    inner &#x3D; SimpleClass()

classes &#x3D; Autodata.create_many(NestedClass)
for cls in classes:
  print(f&#39;id &#x3D; {cls.id}&#39;)
  print(f&#39;text &#x3D; {cls.text}&#39;)
  print(f&#39;inner.id &#x3D; {cls.inner.id}&#39;)
  print(f&#39;inner.text &#x3D; {cls.inner.text}&#39;)
  print()


The code above might output the following

id &#x3D; 1116454042
text &#x3D; ceeecf0c-7375-4f3a-8d4b-6d7a4f2b20fd
inner.id &#x3D; 1067027444
inner.text &#x3D; 079573ce-1ef4-408d-8984-1dbc7b0d0b80

id &#x3D; 730390288
text &#x3D; ff3ca474-a69d-4ff6-95b4-fbdb1bea7cdb
inner.id &#x3D; 1632771208
inner.text &#x3D; 9423e824-dc8f-4145-ba47-7301351a91f8

id &#x3D; 187364960
text &#x3D; b31ca191-5031-43a2-870a-7bc7c99e4110
inner.id &#x3D; 1705149100
inner.text &#x3D; e703a117-ba4f-4201-a31b-10ab8e54a673


Create a Pandas DataFrame using anonymous data generated from a specified type

class DataClass:
    id &#x3D; -1
    type &#x3D; &#39;&#39; 
    value &#x3D; 0

pdf &#x3D; Autodata.create_pandas_dataframe(DataClass)
print(pdf)


The code above might output the following

          id                                  type       value
0  778090854  13537c5a-62e7-488b-836e-a4b17f2f3ae9  1049015695
1  602015506  c043ca8d-e280-466a-8bba-ec1e0539fe28  1016359353
2  387753717  986b3b1c-abf4-4bc1-95cf-0e979390e4f3   766159839


Create a Pandas DataFrame using fake data generated from a specified type

class DataClass:
    id &#x3D; -1
    type &#x3D; &#39;&#39; 
    value &#x3D; 0

pdf &#x3D; Autodata.create_pandas_dataframe(DataClass, use_fake_data&#x3D;True)
print(pdf)


The code above might output the following

  first_name    id last_name          phone_number
0   Lawrence  7670   Jimenez  001-172-307-0561x471
1      Bryan  9084    Walker         (697)893-6767
2       Paul  9824    Thomas    960.555.3577x65487*
<!--END_SECTION:feed-->

[![](https://github.com/christianhelle/christianhelle/raw/main/MVP_Badge_Horizontal_Preferred_Blue3005_RGB.jpg)](https://mvp.microsoft.com/en-us/PublicProfile/5004822)
