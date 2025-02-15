# NLog Loki Target

![CI](https://github.com/corentinaltepe/nlog.loki/workflows/CI/badge.svg?branch=master)
[![NuGet](https://img.shields.io/nuget/v/CorentinAltepe.NLog.Loki)](https://www.nuget.org/packages/CorentinAltepe.NLog.Loki)
[![codecov](https://codecov.io/gh/corentinaltepe/nlog.loki/branch/master/graph/badge.svg?token=84N5XB4J09)](https://codecov.io/gh/corentinaltepe/nlog.loki)

This is an [NLog](https://nlog-project.org/) target that sends messages to [Loki](https://grafana.com/oss/loki/) using Loki's HTTP Push API.

> Loki is a horizontally-scalable, highly-available, multi-tenant log aggregation system inspired by Prometheus. It is designed to be very cost effective and easy to operate.

## Installation

The NLog.Loki NuGet package can be found [here](https://www.nuget.org/packages/CorentinAltepe.NLog.Loki). You can install it via one of the following commands below:

NuGet command:

    Install-Package CorentinAltepe.NLog.Loki

.NET Core CLI command:

    dotnet add package CorentinAltepe.NLog.Loki

## Usage

Under .NET Core, [remember to register](https://github.com/nlog/nlog/wiki/Register-your-custom-component) `NLog.Loki` as an extension assembly with NLog. You can now add a Loki target [to your configuration file](https://github.com/nlog/nlog/wiki/Tutorial#Configure-NLog-Targets-for-output):

```xml
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  
  <extensions>
    <add assembly="NLog.Loki" />
  </extensions>

  <!-- Loki target is async, so there is no need to wrap it in an async target wrapper. -->
  <targets>
    <target 
      name="loki" 
      xsi:type="loki"
      batchSize="200"
      taskDelayMilliseconds="500"
      queueLimit="10000"
      endpoint="http://localhost:3100"
      username="myusername"
      password="secret"
      layout="${level}|${message}${onexception:|${exception:format=type,message,method:maxInnerExceptionLevel=5:innerFormat=shortType,message,method}}|source=${logger}">
      <!-- Loki requires at least one label associated with the log stream. 
      Make sure you specify at least one label here. -->
      <label name="level" layout="${level:lowercase=true}" />
      <label name="server" layout="${hostname:lowercase=true}" />
    </target>
  </targets>

  <rules>
    <logger name="*" minlevel="Info" writeTo="loki" />
  </rules>

</nlog>
```

The `@endpoint` attribute is a [Layout](https://github.com/NLog/NLog/wiki/Layouts) that must ultimately resolve to a fully-qualified absolute URL of the Loki Server when running in a [Single Proccess Mode](https://grafana.com/docs/loki/latest/overview/#modes-of-operation) or of the Loki Distributor when running in [Microservices Mode](https://grafana.com/docs/loki/latest/overview/#distributor). When an invalid URI is encountered, all log messages are silently discarded.

`username` and `password` are optional fields, used for basic authentication with Loki.

`label` elements can be used to enrich messages with additional [labels](https://grafana.com/docs/loki/latest/design-documents/labels/). `label/@layout` support usual NLog layout renderers.
