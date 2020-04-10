# ICOS

Internal Communications Orchestration Services

# Installation

### Requirements
A Kubernetes Cluster

{{TODO need to create the contracts first}}

# Description of what ICOS does

ICOS is a program that takes your monolithic code and recompiles it or reweaves it into separate microservices that can then be deployed onto a kubernetes cluster. It does this by using interfaces as a natural separation between logic and simple dependency injection to bind the code by creating a proxy implementation of the interfaces; an internal communication protocol that is generated to form a custom binary protocol can then be introduced.

You simply write a program in C# with different interfaces for every microservice, you mark them with a couple of attributes and ICOS will automatically rebuild them into separate executables and microservices, all neatly dockerized up with a kubernetes Spec file to deploy them .

# How To/ FAQ

### The Template Engine is Escaping all my HTML
Instead of ``{{foo}}`` do ``{{{foo}}}`` thats 3 Brackets and it will work

### How to use Cookies
To add your cookie to the ``HttpResponce.Cookies`` to create your Cookie. Thereafter your Cookie will be present in ``HttpRequest.Cookies``.

### How to add ``Access-Control-Allow-Origin`` header

### P2P
Note: P2P is currently considered to be in beta and is not to be used for mission critical systems.

Simply use ``Action<>`` or ``Func<>`` and call it to create p2p communication for more details, look at the following example

### TCP and UDP

To use a Custom TCP or UDP port, create an Http Microservice and add the CustomPort Attribute:

```csharp
    [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
    [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
    [IcosCfg(Cfg.Protocol, Protocol.Http)]
    [IcosCfg(Cfg.Domain, "foo.example.com")]
    [IcosCfg(Cfg.CustomPort, 25565)]
    public interface Foo
    {
            [IcosHttpPath("/")]
            HttpResponse Index(HttpRequest req);
    }

```

This will still create a web page that is accessible, but when you implement the interface with the custom port, simply open a Tcp or UDP server in the constructor and it will have the open ports.

NOTE: You will have to add proper configs to your Kubernetes Cluster and Load Balancers to enable your custom port to be ingressed.

### How Do I Use SSL
Follow this tutorial:
https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes
Then Add this to your ``_res`` file:
``Ingress.yaml``
```yaml
# NAME={{NAME}} NAMESPACE={{NAMESPACE}} DOMAIN={{DOMAIN}} PATH={{PATH}} SERVICEPORT={{SERVICEPORT}}
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: {{NAME}}
  namespace: {{NAMESPACE}}
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.org/proxy-connect-timeout: "30s"
    nginx.org/proxy-read-timeout: "20s"
    nginx.org/client-max-body-size: "4m"
spec:
  tls:
    - hosts:
      - {{DOMAIN}}
      secretName: {{NAME}}-tls
  rules:
  - host: {{DOMAIN}}
    http:
      paths:
      - path: {{PATH}}
        backend:
          serviceName: {{NAME}}
          servicePort: {{SERVICEPORT}}

```

# Microservices
ICOS has two primary types of microservices and three methods of inter microservice communication. The first kind is the APi, the second kind is the Internal microservice. The three methods of communication are http, ICP and P2P .

Http microservices have two methods of operation. The first method is a rest API and the second method is a render based engine with templating. Http microservices are exposed to the internet using Ingress,  whereas ICP or internal microservices are never exposed to the internet and are designed for internal use only.

 Use HTTP microservices to create web pages and API's. Use internal microservices to create workers buffers and queues or any internal services that need completion to facilitate modular scaling. 

# Structure
A recommended structure for your project is to create a few folders in your project, one for implementation and one for the services and finally for models. This is not mandatory but it helps to maintain a properly structured and easy to follow and understanding way of structuring things so that groups of programmers can work together without the confusion that inevitably results from poorly defined standards. 

# ICP Service
The ICP service is the simplest form of microservice to create. Create an interface with all the methods you want that service to do, then mark it with a couple of attributes. Generally these attributes do not change so you can just copy and paste them. It is recommended that you place these interfaces in the services folder .

```csharp
    [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
    [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
    [IcosCfg(Cfg.Protocol, Protocol.Icp)]
    public interface ITimeProvider
    {
        string GetTime();
    }

```

Next you will merely implement the interface again. It is recommended to place the implementation in the implementation folder 

```csharp
    public class TimeProvider : ITimeProvider
    {
        public string GetTime()
        {
            return DateTime.Now.ToString();
        }
    }

```

Using the iTimeProvider interface in another microservice will be discussed in the dependency injection section of this documentation 

# HTTP Service

HTTP microservices are a bit more complicated.  There are two methods of exposing an http interface. The first is to simply use an API, the second method is to use the template engine. Let's take a look at both .

## API
First let's take a look at an example of a simple API that gives you the time of day 
```csharp
    [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
    [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
    [IcosCfg(Cfg.Protocol, Protocol.Http)]
    [IcosCfg(Cfg.Domain, "foo.example.com")]
    [IcosCfg(Cfg.DomainPath, "/")]
    public interface IGateway
    {
            [IcosHttpPath("/")]
            HttpResponse Index(HttpRequest req);
    }
```

Again it is recommended to place this in the services folder. The two attributes that you want to change or the last to the ``domain`` and ``domain path``. Every method in the API way of doing things must return an HTTP response and accept an HTTP request as the first argument. The attribute defining the path is also mandatory. We will discuss this attribute in a later section in this documentation entitled attributes .

Now let's look at the implementation of the Gateway :
```csharp
public class Gateway : IGateway
{
        private ITimeProvider _provider;

        public HttpResponse Index(HttpRequest req)
        {
            return new HttpResponse(_provider.GetTime());
        }
}
```
Let's take a look at some of the constructors for the HTTP response .

Returns the string as a webpage with the default content type of ``text/html``
```csharp
public HttpResponse(string res)
```
Will return String as Response body with the provided mimeType
```csharp
public HttpResponse(string res, string mimeType)
```
The Object will be serialized using ``Newtonsoft.Json``, with the default mimeType of ``application/json``.
```csharp
public HttpResponse(object json)
```
Will Return the provided byte[] as the body of the Response.
```csharp
public HttpResponse(byte[] res)
```
Will Return the provided byte[] as the body of the Response, with the provided mimeType
```csharp
public HttpResponse(byte[] res, string mimeType)
```

## Template Engine

Using the template engine is easy. First: create an ``index.html`` file in the ``www`` Folder. You should find this folder in your ``_res`` folder. Next create an interface for the file and place it in services :

```html
<html
<head>
    <title>Test</title>
</head>
<body>

<h1>{{Username}}</h1>
<h2>{{Email}}</h2>
</body>
</html>
```
And now for the ``IUserView``
```csharp
    [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
    [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
    [IcosCfg(Cfg.Protocol, Protocol.Http)]
    [IcosCfg(Cfg.Domain, "foo.example.com")]
    [IcosCfg(Cfg.ViewEngine, ViewEngine.Static)]
    {
        [IcosHttpPath("/")]
        User Index(HttpRequest req);
    }

```
Note that the  index has the same Name as the file index. Multiple files can be served by creating more methods in the same interface. You can have as many interfaces as you like to separate them into different microservices .
Now create a class called ``User`` in the ``Model`` folder:

```csharp
public class User
{
        public User(string username, string email)
        {
            Username = username;
            Email = email;
        }

        public string Username { get; set; }
        public string Email { get; set; }
}
```
Now we can implement  the user view  as follows :

```csharp
public class UserView : IUserView
{
        public User Index(HttpRequest req)
        {
            return new User("myvar", "foo@gmail.com");
        }
}
```
Note that we are returning the user object and that the properties of the object are used in the templating engine to replace the moustache handlebars.

## Static Host
Any content example used earlier:
```csharp
public class Gateway : IGateway
{
        private ITimeProvider _provider;

        public HttpResponse Index(HttpRequest req)
        {
            return new HttpResponse(_provider.GetTime());
        }
}

```
 We can see that the implementation uses the time provider microservice to get the time. In this example, by creating a private field for the micro service, the dependency injection system will automatically assign the proxy interface that will use a custom TCP binary protocol to communicate with the actual implementation of the interface, running in another container .

# CFG File 
The config file must be placed in the ``_res`` file. Simply look at the example below with the comments describing what every setting does .
```yaml
kubernetes-namespace: test # the kubernetes namespace, NOTE: the ns will not be auto created you must create it yourself

docker-registry: foo/ # the docker registry you plan on using 

content: www #the static and template content directory

Domain-overides: #domain overrides look at Attributes section on docs for more examples
    api_sub_domain: stage
```

# Attributes
## IcosHttpPath
When creating an Http Microservice, every method must have an ``IcosHttpPath`` attribute. To determine the path it will serve, Paths may also include Arguments for eg:
```csharp
    [IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
    [IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
    [IcosCfg(Cfg.Protocol, Protocol.Http)]
    [IcosCfg(Cfg.Domain, "api.example.com")]
    [IcosCfg(Cfg.DomainPath, "/")]
    public interface IQueueApi
    {
            [IcosHttpPath("/")]
            HttpResponse Index(HttpRequest req);
        
            [IcosHttpPath("/v1/q/spot/{guid}")]
        HttpResponse GetSpotInQueue(HttpRequest req);
    }
```
The Method GetSpotInQueue has the Argument ``guid`` it can be used as follows:
```chsarp
 public HttpResponse GetSpotInQueue(HttpRequest req)
        {
           return new HttpResponse(req.Argument["guid"]);
        }

```
If you open the webpage in your web browser: ``http://api.example.com/v1/q/spot/some_test_id/``
The page will respond with ``some_test_id``

## IcosCfg

All Services must be Stateless in Type, there are future plans for stateful services.
```charp
[IcosCfg(Cfg.ServiceType, ServiceType.Stateless)]
```
The only Load Balance Strategy available is Round Robin, other Strategies are planned.
```charp
[IcosCfg(Cfg.LoadBalanceStrategy, LoadBalanceStrategy.RoundRobin)]
```
The Protocol can be one of two options. The first is ``Protocol.Http`` to create a web api or page, or ``Protocol.Icp`` to mark a service for Internal use only
```charp
[IcosCfg(Cfg.Protocol, Protocol.Http)]
```
The domain this service should bind to its ingress.
```charp
[IcosCfg(Cfg.Domain, "exmaple.com")]
```
The DomainPath provided in the kubernetes ingress.
```charp
[IcosCfg(Cfg.DomainPath, "/")]
```
Custom ports will be added to the Service and Ingress Setup, you will be responsible to use the port yourself.
```charp
[IcosCfg(Cfg.CustomPort, 9090)]
```
Used to enable the Static View Engine. NOTE: View engines may only be used on HTTP services
```charp
[IcosCfg(Cfg.ViewEngine, ViewEngine.Static)]
```


# Kubespec Overrides (Optional)
If you want to change the kubernetes specs, to add resource constraints or use a cert issue with ssl, you can easily create a file in the ``_res`` folder with the following names and base content, Note you must be sure to add the proper reg-cred for your private docker registries.

Deployment.yaml
```yaml
#  REGISTRY={{REGISTRY}} NAME={{NAME}} NAMESPACE={{NAMESPACE}} PORT={{PORT}} PORTS={{PORTS}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{NAME}}
  namespace: {{NAMESPACE}}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{NAME}}
  template:
    metadata:
      labels:
        app: {{NAME}}
    spec:
      imagePullSecrets:
      - name: regcred
      containers:
      - image: {{REGISTRY}}{{NAMESPACE}}{{NAME}}
        imagePullPolicy: Always
        name: {{NAME}}
        ports:
        - containerPort: {{PORT}}
{{PORTS}}    
#        livenessProbe:
#          exec:
#            command:
#            - /bin/bash
#            - -c
#            - cat /tmp/healthy; rm -rf /tmp/healthy
#          initialDelaySeconds: 1
#         periodSeconds: 15
#        readinessProbe:
#          exec:
#            command:
#            - /bin/bash
#            - -c
#            - cat /tmp/healthy; rm -rf /tmp/healthy
#          initialDelaySeconds: 1
#          periodSeconds: 5
#        resources:
#          limits:
#            cpu: 0.1
#            memory: 128Mi
#          requests:
#            cpu: 0.1
#            memory: 128Mi
#---
#apiVersion: autoscaling/v2beta2
#kind: HorizontalPodAutoscaler
#metadata:
#  name: {{NAME}}
#  namespace: {{NAMESPACE}}
#spec:
#  scaleTargetRef:
#    apiVersion: apps/v1
#    kind: Deployment
#    name: {{NAME}}
#  minReplicas: 1
#  maxReplicas: 10
#  metrics:
#  - type: Resource
#    resource:
#      name: cpu
#      target:
#        type: Utilization
#        averageUtilization: 50

```

Ingress.yaml
```yaml
# NAME={{NAME}} NAMESPACE={{NAMESPACE}} DOMAIN={{DOMAIN}} PATH={{PATH}} SERVICEPORT={{SERVICEPORT}}
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: {{NAME}}
  namespace: {{NAMESPACE}}
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.org/proxy-connect-timeout: "30s"
    nginx.org/proxy-read-timeout: "20s"
    nginx.org/client-max-body-size: "4m"
spec:
  rules:
  - host: {{DOMAIN}}
    http:
      paths:
      - path: {{PATH}}
        backend:
          serviceName: {{NAME}}
          servicePort: {{SERVICEPORT}}

```
Service.yaml
```yaml
# NAME={{NAME}} NAMESPACE={{NAMESPACE}} DEPLOYMENTPORT={{DEPLOYMENTPORT}} SERVICEPORT={{SERVICEPORT}}
apiVersion: v1
kind: Service
metadata:
  name: {{NAME}}
  namespace: {{NAMESPACE}}
spec:
  ports:
  - name: icos-port
    port: {{SERVICEPORT}}
    targetPort: {{DEPLOYMENTPORT}}
    protocol: TCP
{{PORTS}}
  selector:
    app: {{NAME}}

```
