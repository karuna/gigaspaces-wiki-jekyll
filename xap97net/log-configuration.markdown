---
layout: post
title:  Log Configuration
categories: XAP97NET
parent: configuration.html
weight: 400
---

# This page is under construction.

# Logging and Tracing

{% toczone minLevel=2|maxLevel=2|type=flat|separator=pipe|location=top %}

GigaSpaces XAP.NET components use the tracing mechanism for logging/tracing, built-in with the .NET framework. This gives the user, control over tracing behavior using the standard .NET configuration schema. Users can:

- configure the level of events which are traced
- assign one or more trace listeners which route the events to a logging facility
- implement custom trace listeners to integrate GigaSpaces events with the application events, and more.

If the user does not specify a configuration, the default configuration is assumed.

{% refer %}GigaSpaces XAP.NET contains some of the GigaSpaces XAP components. Its logging level needs to be configured seperately -- this is described in [GigaSpaces XAP Logging]({% currentjavaurl %}/gigaspaces-logging.html){% endrefer %}.

## Basic Configuration

To configure the GigaSpaces logger, you need to add a trace source configuration named `GigaSpaces.Core` to your configuration file (`app.config`/`web.config`). Use the `switchValue` argument to set the trace level to one of the following: `Off`, `Critical`, `Error`, `Warning`, `Information`, `Verbose`.  (Naturally, each level includes all its predecessors, e.g. `Error` includes `Critical` as well). Use the `listeners` collection to add trace listeners which handle the traced events.

The following example sets the trace level to `Error`, which means that only errors and critical events are processed. It also defines a single trace listener, which writes traced messages to the Windows Event Log, under a source called `GigaSpaces.Core`.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.diagnostics>
    <sources>
      <source name="GigaSpaces.Core" switchValue="Error">
        <listeners>
          <add name="MyListener"
  type="System.Diagnostics.EventLogTraceListener"
  initializeData="GigaSpaces.Core"/>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>

{% endhighlight %}

There are several logging components split into different subjects. They should be configured in the same way, but using a different source name. The available components are:

- GigaSpaces.Core - core related loggings
- GigaSpaces.XAP.ProcessingUnit - processing unit related logins

## Default Configuration

The logger component loads the configuration during initialization. If it does not find a source element named `GigaSpaces.Core`, it loads a default configuration, which sets the trace level to `Information`, and configures an `EventLogTraceListener` with `source=GigaSpaces.Core`, (similar to the configuration shown in the basic example).

{% exclamation %} If the Windows Event Log does not contain the specified source, it is automatically created. However, you need administrator permissions to create a source in the Event Log. If you don't create this source, the default configuration is not used. We recommend that you use an administrator's profile the first time you use the product on a machine, to make sure the source is created. Subsequent runs do not require high level permissions.

## Advanced Configuration

Here are some features/scenarios which might be useful:

- You can use any of the built-in trace listeners offered by `System.Diagnostics`:
    - `ConsoleTraceListener`
    - `TextWriterTraceListener`
    - `XmlWriterTraceListener`
    - `DelimitedListTraceListener`
    - `EventLogTraceListener`

{% refer %}For more details, see: [http://msdn2.microsoft.com/en-us/library/4y5y10s7.aspx](http://msdn2.microsoft.com/en-us/library/4y5y10s7.aspx).{% endrefer %}

- You can configure a trace listener with a filter to handle specific events.

{% refer %}For more details, see: [http://msdn2.microsoft.com/en-us/library/system.diagnostics.eventtypefilter.aspx](http://msdn2.microsoft.com/en-us/library/system.diagnostics.eventtypefilter.aspx).{% endrefer %}

- You can implement a custom trace listener to handle traced events in a desired manner (e-mail, SMS, custom log, etc.). If you are planning to do this, we recommend that you examine the implementation of custom trace listeners provided in Microsoft's Logging Application Block as a reference.

{% refer %}For more details, see:

- [http://msdn2.microsoft.com/EN-US/library/aa480464.aspx](http://msdn2.microsoft.com/EN-US/library/aa480464.aspx)
- [http://msdn2.microsoft.com/en-us/library/ms228989.aspx](http://msdn2.microsoft.com/en-us/library/ms228989.aspx){% endrefer %}
{% endtoczone %}
