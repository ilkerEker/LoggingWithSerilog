# LoggingWithSerilog
Applied from https://codingsonata.com/logging-with-serilog-in-asp-net-core-web-api/
https://github.com/aram87/LoggingWithSerilog


Serilog Sinks
Serilog has the concept of sinks as plugins, comes bundled with a numerous predefined sinks that you can configure and directly use to start writing logs.

Such sinks range from simple ones like Console or File to more advanced sinks like ElasticSearch or sinks that that can directly connect and dump the logs to databases such as SQL Server, MongoDB, or to cloud logging services like Azure Analytics, Amazon CloudWatch or cloud databases such as Azure Cosmos DB or Aws Dynamo DB, and even you can find sinks that would dump the logs into the event log of windows and so many other options.

In this tutorial, we have learned how to connect the ASP.NET Core Web API with Serilog, you can experiement with different types of provided Serilog sinks.

Check the following link to view the full list of provided Serilog sinks.

Custom Serilog Sink
In some cases, you might need to write your own sink, Serilog enables you to achieve this by implementing the ILogEventSink in your own class and writing the needed custom logging code in the Emit method

Let’s create a Custom Sink, add a folder with name Sinks, and create a new class with name CustomSink

using Serilog.Core;
using Serilog.Events;

namespace LoggingWithSerilog.Sinks
{
    public class CustomSink : ILogEventSink
    {
        public void Emit(LogEvent logEvent)
        {
            var result = logEvent.RenderMessage();

            Console.ForegroundColor = logEvent.Level switch
            {
                LogEventLevel.Debug => ConsoleColor.Green,
                LogEventLevel.Information => ConsoleColor.Blue,
                LogEventLevel.Error => ConsoleColor.Red,
                LogEventLevel.Warning => ConsoleColor.Yellow,
                _ => ConsoleColor.White,
            };
            Console.WriteLine($"{logEvent.Timestamp} - {logEvent.Level}: {result}");
        }
    }
}

using Serilog.Core;
using Serilog.Events;
 
namespace LoggingWithSerilog.Sinks
{
    public class CustomSink : ILogEventSink
    {
        public void Emit(LogEvent logEvent)
        {
            var result = logEvent.RenderMessage();
 
            Console.ForegroundColor = logEvent.Level switch
            {
                LogEventLevel.Debug => ConsoleColor.Green,
                LogEventLevel.Information => ConsoleColor.Blue,
                LogEventLevel.Error => ConsoleColor.Red,
                LogEventLevel.Warning => ConsoleColor.Yellow,
                _ => ConsoleColor.White,
            };
            Console.WriteLine($"{logEvent.Timestamp} - {logEvent.Level}: {result}");
        }
    }
}
In the above example, we are changing the console text color according to the level of the log.

To be able to configure our application to use the new Custom Sink, we must define an extension method for the new Custom Sink

using Serilog;
using Serilog.Configuration;

namespace LoggingWithSerilog.Sinks
{
    public static class CustomSinkExtensions
    {
        public static LoggerConfiguration CustomSink(
                  this LoggerSinkConfiguration loggerConfiguration)
        {
            return loggerConfiguration.Sink(new CustomSink());
        }
    }
}

using Serilog;
using Serilog.Configuration;
 
namespace LoggingWithSerilog.Sinks
{
    public static class CustomSinkExtensions
    {
        public static LoggerConfiguration CustomSink(
                  this LoggerSinkConfiguration loggerConfiguration)
        {
            return loggerConfiguration.Sink(new CustomSink());
        }
    }
}
Now, the last part is to glue to components together in the Program.cs, by introducing the new Custom Sink within the Serilog logging configuration

var logger = new LoggerConfiguration()
        .ReadFrom.Configuration(builder.Configuration)
        .WriteTo.CustomSink()
        .Enrich.FromLogContext()
        .CreateLogger();

var logger = new LoggerConfiguration()
        .ReadFrom.Configuration(builder.Configuration)
        .WriteTo.CustomSink()
        .Enrich.FromLogContext()
        .CreateLogger();
As you can see above, we are using 2 sinks, one from configuration (File Sink) , and the other is using the Custom Sink defined using the WriteTo Custom Sink Extension method.
