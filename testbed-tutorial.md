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
