run_code

# ICOS

Welcome to the ICOS testbed, using this you can try ICOS with out any installation or delay. First what is ICOS, Internal Comunication Orcistration Services is an automated microservice platfrom that takes monolithic applications and reweaves it into sperated microservices that are automaticly docerized and deployed to kubernetes. We will be creating a website that gives you the date and time, using two microservices althow using multiple micro-services are overkill the point is to provide an example.

The first microserivec will be called ``ITimeApi`` the goal of this microservice is to handel incomming http requests, then to call the second microservice called ``ITimeProvider`` and ask it for a string containg the date and time. 

# Services
1. Create a new file called ``ITimeApi`` in the folder with this content:
    ```csharp

        [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
        [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
        [IcosCfg(Cfg.Protocol, Protocol.Http)]
        [IcosCfg(Cfg.Domain, "{{slug}}.hatchery.cloud")]
        [IcosCfg(Cfg.DomainPath, "/")]


        public interface ITimeApi
        {
            [IcosHttpPath("/")]
            HttpResponse Index(HttpRequest req);
        }
    
    ```
2. Create a new file called ``ITimeProvider`` in the folder with this content:
    ```csharp

        [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
        [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
        [IcosCfg(Cfg.Protocol, Protocol.Icp)]
        public interface ITimeProvider
        {
            string GetTime();
        }
    
    ```
# Implementation

1. Create a new folder called ``Implementation``
2. Create a file called ``TimeApi.cs`` in the folder with the following content:
    ```csharp
        public class TimeApi : ITimeApi
        {
            
            private ITimeProvider _provider;
            
            public HttpResponse Index(HttpRequest req)
            {
                return new HttpResponse("Time: " + _provider.GetTime());
            }
        }
    
    ```
3. Create a file called ``TimeProvider.cs`` in the folder with this content:
    ```csharp
            public class TimeProvider : ITimeProvider
            {
                public string GetTime()
                {
                    return DateTime.Now.ToString();
                }
            }
        
    ```
    
run_code

Lets take a look at the console output down bellow to understand what icos is doing:

First the Server Does some Setup.
```
Connection To Server Open
starting
Login to docker registry
```
Next The nuget packages are restored, and icos does its thing
```
Restoring Nuget Packages

[LOG] Starting
[LOG] Working Directory: /tmp/4ada6570ea824dcca859bb950e2f6751/TestBedProject
[LOG] Running dotnet publish
[DEBUG] [ReweaverEngine::LoadAssembly(String)]Loading Assembly: TestBedProject.dll
[DEBUG] [AssemblyManager::LoadAssembly(String)]Found Service: itimeapi
[DEBUG] [AssemblyManager::LoadAssembly(String)]Found Service: itimeprovider
[LOG] Reweaving
[DEBUG] [ReweaverEngine::Build()][itimeapi] Loaded Template Host Binary: Icos.Host.Http.dll
[DEBUG] [ReweaverEngine::Build()][itimeapi] Running Builder: StaticHostBuilder
[DEBUG] [ReweaverEngine::Build()][itimeapi] Running Builder: ProxyInterfaceBuilder
[DEBUG] [ReweaverEngine::Build()][itimeapi] Running Builder: InterfaceHostBuilder
[DEBUG] [ReweaverEngine::Build()][itimeapi] Running Builder: LambdaBuilder
[DEBUG] [ReweaverEngine::Build()][itimeapi] Running Builder: K8SSpecBuilder
[DEBUG] [ReweaverEngine::Build()][itimeapi] Running Builder: BuildScriptBuilder
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Loaded Template Host Binary: Icos.Host.Icp.dll
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Running Builder: StaticHostBuilder
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Running Builder: ProxyInterfaceBuilder
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Running Builder: InterfaceHostBuilder
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Running Builder: LambdaBuilder
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Running Builder: K8SSpecBuilder
[DEBUG] [ReweaverEngine::Build()][itimeprovider] Running Builder: BuildScriptBuilder
[LOG] Injecting Dependencies
[LOG] Writing Binaries
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Newtonsoft.Json.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Mono.Cecil.Pdb.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Icos.Core.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: YamlDotNet.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: TestBedProject.pdb
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Mono.Cecil.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Icos.Reweaver.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Mono.Cecil.Rocks.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Mono.Cecil.Mdb.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: Handlebars.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeapi] Found Dependence: CommandLine.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Newtonsoft.Json.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Mono.Cecil.Pdb.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Icos.Core.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: YamlDotNet.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: TestBedProject.pdb
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Mono.Cecil.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Icos.Reweaver.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Mono.Cecil.Rocks.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Mono.Cecil.Mdb.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: Handlebars.dll
[DEBUG] [ReweaverEngine::Write(String)][itimeprovider] Found Dependence: CommandLine.dll
[LOG] Done
```

Now that ICOS has recompiled and reweaved the monolitchic program into sperated docker containers with custom binary protocals to facilitate Internal Remote Procedure calls.

```
[LOG] Pushing Docker Images
Sending build context to Docker daemon 1.781MB

Step 1/5 : FROM mcr.microsoft.com/dotnet/core/sdk:3.1
---> d9d656b4ceb2
Step 2/5 : WORKDIR /app
---> Using cache
---> 355a5a7be62d
Step 3/5 : COPY . .
---> 8383a8d1791a
Step 4/5 : EXPOSE 8080
---> Running in b5d90410e142
Removing intermediate container b5d90410e142
---> aaef19c01342
Step 5/5 : ENTRYPOINT ["dotnet", "/app/entrypoint.dll"]
---> Running in 7c2b5cb12bb0
Removing intermediate container 7c2b5cb12bb0
---> 551a80e81e3c
Successfully built 551a80e81e3c
Successfully tagged registry.myvar.cloud/aa6e48a59f4e6495c9eb250e6ef42b37aitimeapi:latest
The push refers to repository [registry.myvar.cloud/aa6e48a59f4e6495c9eb250e6ef42b37aitimeapi]
[...]
caa9bb437898: Pushed
latest: digest: sha256:7c707edda18ddd112560097e2aedcf5815765514e0f2aed93f7eb37fba91ad06 size: 2217
Sending build context to Docker daemon 1.775MB

Step 1/5 : FROM mcr.microsoft.com/dotnet/core/sdk:3.1
---> d9d656b4ceb2
Step 2/5 : WORKDIR /app
---> Using cache
---> 355a5a7be62d
Step 3/5 : COPY . .
---> bff9eebd9942
Step 4/5 : EXPOSE 8081
---> Running in c41aaefe133f
Removing intermediate container c41aaefe133f
---> cf7e41129959
Step 5/5 : ENTRYPOINT ["dotnet", "/app/entrypoint.dll"]
---> Running in 4aa42b21535a
Removing intermediate container 4aa42b21535a
---> 41b6dca7c0ae
Successfully built 41b6dca7c0ae
Successfully tagged registry.myvar.cloud/aa6e48a59f4e6495c9eb250e6ef42b37aitimeprovider:latest
The push refers to repository [registry.myvar.cloud/aa6e48a59f4e6495c9eb250e6ef42b37aitimeprovider]
[...]
50378a43a5f7: Pushed
latest: digest: sha256:db42300460c6b471e2eb7dda9cdd5fa02a89f76812b6a9539d7e490bc5d4b1be size: 2217
[LOG] Build completed in 00:00:42.3400877
namespace/aa6e48a59f4e6495c9eb250e6ef42b37a created
Deleting Old Instance if it exists
[...]
```

Now that the containers have been uploaded to the registry, the autogenerate kubernetes spec file is applied

First a Deployment is Created for the Time Api
```
deployment.apps/itimeapi created
```
next a service is created for the Time Api
```
service/itimeapi created
```
next icos creates an ingress method for the Time Api service
```
ingress.networking.k8s.io/itimeapi created
```
next the deployment for the Time Provider Microservice is created
```
deployment.apps/itimeprovider created
```
finally the service is created for the TimeProvider Microservice, TimeProvider does not need an ingress because it's only for internal usage
```
service/itimeprovider created
```

