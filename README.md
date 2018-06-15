[New Relic](https://newrelic.com/) doesn't currently have a nuget package or a buildpack to deploy the .NET New Relic agent with a .NET Core app on Cloud Foundry. You can use the New Relic agent and configure it to work with a Cloud Foundry on the Linux stack without having to modify or install anything on the cell.
These instructions make it possible for any application developer to push a New Relic instrumented application to any Cloud Foundry environment available. The only external requirement is that the cell stack needs to have internet access to the New Relic API servers.
### Download and Install the agent into your project
1) Go to the [download](https://download.newrelic.com/dot_net_agent/latest_release) site and download the latest newrelic-netcore20-agent-VERSION.tar.gz package that corresponds to your architecture.
2) Extract the agent into a `newrelic` folder under the root of your website.
3) Add all files within the newrelic directory to your project
4) Ensure that all files within the newrelic directory have the following properties set:

 ```
 Copy to Output Directory = Always
 Build Action = Content
 ```

 Commit the newrelic folder to source control. 

### Configure the run script
Within the `newrelic` folder we need modify the `run.sh` script so that it points to the newrelic folder you created. This file is executed on app instance startup by CloudFoundry. This file sets runtime variables to enable .NET profiling and configure the New Relic agent to run from your app's newrelic folder. Finally it starts your application with the arguments you pass in.
```bash
CORECLR_NEWRELIC_HOME=${CORECLR_NEWRELIC_HOME:-"NEWRELIC_FOLDER_HERE"} CORECLR_ENABLE_PROFILING=1 CORECLR_PROFILER={36032161-FFC0-4B61-B559-F6C5D41BAE5A} CORECLR_PROFILER_PATH=$CORECLR_NEWRELIC_HOME/libNewRelicProfiler.so $@
```
>**Note (Bash Script):** The `run.sh` script uses runtime variables to execute your core application. As such this much be executed all on one line.

### Configure the NewRelic License and Application Name
There are two different ways to configure your agent for instrumentation. It is highly recommended to utilize environment variables for each application instance you wish to instrument otherwise you may instrument processes other than the desired instance.

1) Set via user defined environment variables within PCF 
```yml
NEW_RELIC_LICENSE_KEY: YOUR_LICENSE_KEY
NEW_RELIC_APP_NAME: YOUR_APP_NAME
```

2) Set `newrelic.config` inside the `newrelic` folder
```xml
  <service licenseKey="REPLACE_WITH_LICENSE_KEY" />
  <application>
    <name>REPLACE_WITH_APPLICATION_NAME</name>
  </application>
```

This sets three <a href="https://msdn.microsoft.com/en-us/library/ee471451(v=vs.100).aspx">.NET 'COR_' profiler environment variables</a> that newrelic uses to enable profiling without needing access to the registry as was required in older versions of .NET.  The final two environment variables tell New Relic where its files and configuration are, which we're dynamically pointing to in the newrelic folder under the root of our app website.
### Push to CloudFoundry
With all those files in place you can now `cf push` your application to any CloudFoundry installation. There's no need to manually install a machine wide agent. Hopefully in the future some of this manual setup will be automated and some of the configuration will also work with the [New Relic Service Broker](https://docs.pivotal.io/partners/newrelic/index.html).
