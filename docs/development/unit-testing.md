---
id: unit-testing
title: Unit Testing
sidebar_label: Unit Testing
sidebar_position: 1
---

GraphQL ASP.NET has more than `3500 unit tests and 91% code coverage`. Much of this is powered by a test component designed to quickly build a configurable, fully mocked server instance to perform a query. It may be helpful to download the code and extend it for harnessing your own controllers.

The `TestServerBuilder<TSchema>` can be found in the `graphql-aspnet-testframework` project of the primary repo and is dependent on `Moq`. As its part of the core library solution you'll want to remove the project reference to `graphql-aspnet` project and instead add a reference to the nuget package.

This document explains how to perform some common test functions for your own controller methods.

## Create a Test Server

1. Create a new instance of the `TestServerBuilder`. The builder takes in a set of flags to perform some auto configurations for common scenarios such as exposing exceptions or altering the casing of graph type names.
2. Configure your test scenario

    - Use `.User` to add any permissions to the mocked user account
    - Use `.Authorization` to add any security policy definitions if you wish to test security
    - Use `.AddGraphQL()` to mimic the functionality of schema configuration used when your application starts.
    - The `TestServerBuilder` implements `IServiceCollection`, add any additional mocked services as needed to ensure your controllers are wired up correctly by the runtime.

3. Build the server instance using `.Build()`

```csharp title="Configuring a Test Server Instance"
[Test]
public async Task MyController_InvocationTest()
{
    var builder = new TestServerBuilder();
    builder.AddGraphQL(o => {
        o.AddController<MyController>();
    });

    var server = builder.Build();
    //...
}

```

## Execute a Query

1. Mock the query execution context (the object that the runtime acts on) using `.CreateQueryContextBuilder()`
2. Configure the text, variables etc. on the builder.
3. Build the context and submit it for processing:
    - Use `server.ExecuteQuery()` to process the context. `context.Result` will be filled with the final `IQueryExecutionResult` which can be inspected for resultant data fields and error messages.
    - Use `server.RenderResult()` to generate the json string a client would recieve if they performed the query.


```csharp title="Executing a Test Query"
[Test]
public async Task MyController_InvocationTest()
{
    // ...
    var server = builder.Build();
    var contextBuilder = server.CreateQueryContextBuilder();
    contextBuilder.AddQueryText("query { controller { actionMethod { property1 } } }");

    var context = contextBuilder.Build();
    var result = await server.RenderResult(context);

    /* result contains the string for:
    {
        "data" : {
            "controller": {
                "actionMethod" : {
                    "property1" : "value1"
                }
            }
        }
    }
    */
}

```
