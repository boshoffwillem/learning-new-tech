# ASP.NET Core

Information about ASP.NET Core.

## 1.Different services

ASP.NET can be created with different implementations.
You can have a Grpc service, an MVC service, Blazor, etc.
This section contains the information required to add to
an empty ASP.NET web service in order to get a specific
service working.

### 1.1 Empty Service

And empty ASP.NET service has to following important files.

#### Program.cs

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

namespace Templates.WebEmpty
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}
```

#### Startup.cs

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace Templates.WebEmpty
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
        }

        // This method gets called by the runtime.
        //Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapGet("/", async context =>
                {
                    await context.Response.WriteAsync("Hello World!");
                });
            });
        }
    }
}
```

#### launchSettings.json

```json
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:17072",
      "sslPort": 44342
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "Templates.WebEmpty": {
      "commandName": "Project",
      "dotnetRunMessages": "true",
      "launchBrowser": true,
      "applicationUrl": "https://localhost:5001;http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### 1.2 Grpc Serivce

In order to create a Grpc service you need to install the following
packages on the server:

- Grpc.AspNetCore

You also need to create your Protobuf files and add them to your project file.

For the client you need the following packages:

- Grpc.Net.ClientFactory
- Google.Protobuf
- Grpc.Tools

You also need to add the same Protobuf files that your server references
to the client project file.

To add a Protobuf file to your project file add the following:

```xml
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
  </ItemGroup>
```

When you're adding to the client you have to change the GrpcServices attribute
to "Client" instead of "Server".

Given the following Protobuf file:

```proto
syntax = "proto3";

option csharp_namespace = "Templates.WebGrpc";

package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
  rpc SayHelloStream (HelloRequest) returns (stream HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

You'll require the following code.
**(Remember to build the projects anytime you change the Protobuf file,
so that the code can be generated that you need to use.)**

#### Server code required

##### Properties/launchSettings.json

```json
{
  "profiles": {
    "Templates.WebGrpc": {
      "commandName": "Project",
      "dotnetRunMessages": "true",
      "launchBrowser": false,
      "applicationUrl": "http://localhost:5000;https://localhost:5001",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

Create a Services folder. Every Protobuf file is a service.

##### Services/GreeterService.cs

```csharp
namespace Templates.WebGrpc
{
    public class GreeterService : Greeter.GreeterBase
    {
        private readonly ILogger<GreeterService> _logger;

        public GreeterService(ILogger<GreeterService> logger)
        {
            _logger = logger;
        }

        public override Task<HelloReply> SayHello(HelloRequest request,
            ServerCallContext context)
        {
            return Task.FromResult(new HelloReply
            {
                Message = "Hello" + request.Name;
            });
        }

        public override async Task SayHelloStream(HelloRequest request,
            IServerStreamWriter<HelloReply> responseStream,
            ServerCallContext context)
        {
            for (int i = 0; i < 10; i++)
            {
                await responseStream.WriteAsync(new HelloReply
                {
                    Message = "Hello" + request.Name + " " + i;
                });

                await Task.Delay(TimeSpan.FromSeconds(1));
            }
        }
    }
}
```

##### Program.cs

No code necessary.

##### Startup.cs

Simply add the following endpoint to your app EndPoints:

```csharp
endpoints.MapGrpcService<GreeterService>();
```

So it will be:

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGet("/", async context =>
    {
        await context.Response.WriteAsync("Hello World!");
    });
    endpoints.MapGrpcService<GreeterService>();
});
```

Also, add the Grpc service to your service collection.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc();
}
```

#### Client code required

```csharp
using Grpc.Net.Client;

public static async Task Main()
{
    var channel = GrpcChannel.ForAddress("https://localhost:5001");
    var client = new Greeter.GreeterClient(channel);

    var response = await client.SayHelloAsync(new HelloRequest
    {
        Name = "Grpc"
    });

    Console.WriteLine("From server:" + response.Message);

    var streamResponse = client.SayHelloStream(new HelloRequest
    {
        Name = "Stream Grpc"
    });

    await foreach (var item in streamResponse.ResponseStream.ReadAllAsync())
    {
        Console.WriteLine("From server:" + item.Message);
    }
}
```

#### 1.2.1 gRPC Web

Grpc Web is a package that makes gRPC compatible with browsers and
other platforms that only support HTTP/1.1. However, when using gRPC
in this mode two features of gRPC is not supported:

- Client streaming
- Bidirectional streaming

#### 1.2.2 Stream forever

It is also possile to set up a stream bewteen the server and client
that runs forever. When the stream is no longer needed the client simply
sends a signal that raises a ```ServerCallContext.CancellationToken```.
The ```CancellationToken``` should be used on the server with async methods
so that:

- Any asynchronous work is cancelled together with the streaming call.
- The method exits quickly.

##### Server streaming method

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    while (!context.CancellationToken.IsCancellationRequested)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

##### Client streaming method

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        // ...
    }
    return new ExampleResponse();
}
```

##### Bi-directional streaming method

A bi-directional streaming method starts without the method receiving a message.
The ```requestStream``` parameter is used to read messages from the client.
The method can choose to send messages with ```responseStream.WriteAsync```.
A bi-directional streaming call is complete when the method returns:

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        await responseStream.WriteAsync(new ExampleResponse());
    }
}
```

It is possible to support more complex scenarios,
such as reading requests and sending responses simultaneously:

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    // Read requests in a background task.
    var readTask = Task.Run(async () =>
    {
        await foreach (var message in requestStream.ReadAllAsync())
        {
            // Process request.
        }
    });

    // Send responses until the client signals that it is complete.
    while (!readTask.IsCompleted)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

#### 1.2.3 gRPC Resources

- [Configuration options](https://docs.microsoft.com/en-us/aspnet/core/grpc/configuration?view=aspnetcore-5.0)
- [Logging](https://docs.microsoft.com/en-us/aspnet/core/grpc/diagnostics?view=aspnetcore-5.0)
- [Manage Protobuf references with dotnet-grpc](https://docs.microsoft.com/en-us/aspnet/core/grpc/dotnet-grpc?view=aspnetcore-5.0)
- [Protobuf types](https://docs.microsoft.com/en-us/aspnet/core/grpc/protobuf?view=aspnetcore-5.0)

### 1.3 SignalR Service

In order for you to create a SignalR Hub you need to add the following.

#### 1.3.1 Server code

##### Create a hub

You first need to create a SignalR Hub.
This is simply a class that inherits from ```Hub```.

```csharp
using Microsoft.AspNetCore.SignalR;

public class TestHub : Hub
{
    public async Task SayHello(string helloMessage)
    {
        await Clients.Others.SendAsync("SayingHello", helloMessage);
    }
}
```

The first parameter of the ```SendAsync``` method is the method
name that clients will listen for when receiving a notification from the hub.
The second parameter is the data of the notification.

##### 1.3.1.1 Startup.cs

Add the SignalR service to your services.
Add SignalR to your endpoints, specify the hub,
and specify the endpoint that the hub listens to.

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSignalR();
    }

    // This method gets called by the runtime.
    //Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapGet("/", async context =>
            {
                await context.Response.WriteAsync("Hello World!");
            });
            endpoints.MapHub<TestHub>("/hubs/test");
        });
    }
}
```

#### 1.3.2 Client code

To create a SignalR client you need to install the

- Microsoft.AspNetCore.SignalR.Client

NuGet package.

Then you create a connection. 

```csharp
connection = new HubConnectionBuilder()
    .WithUrl("http://localhost:53353/hubs/test")
    .Build();

connection.Closed += async (error) =>
{
    await Task.Delay(new Random().Next(0,5) * 1000);
    await connection.StartAsync();
};
```

You also then need to explicitely start the connection.

```csharp
await connection.StartAsync();
```

Then you simply invoke

```csharp
await connection.SendAsync("SayHello", "Hello server");
```

To call the server method.

You could also use ```InvokeAsync```.
The difference is that ```SendAsync``` will just call the server method
and return, whereas ```InvokeAsync``` will call the server method
and wait for it to finish.

You also need to create a listener for the event that the server
responds with.

```csharp
public event Action<string> HelloMessageReceived;

...

connection.On<string>("SayingHello",
    (helloMessage) => HelloMessageReceived.Invoke(helloMessage));

```
