# AspNetCore.ApiGateway

## Pacote Asp Net Core Api Gateway

|Pacotes|Versão|Downloads|
|---------------------------|:---:|:---:|
|*AspNetCore.ApiGateway*|[![Nuget Version](https://img.shields.io/nuget/v/AspNetCore.ApiGateway)](https://www.nuget.org/packages/AspNetCore.ApiGateway)|[![Downloads count](https://img.shields.io/nuget/dt/AspNetCore.ApiGateway)](https://www.nuget.org/packages/AspNetCore.ApiGateway)|
|*AspNetCore.ApiGateway.Client*|[![Nuget Version](https://img.shields.io/nuget/v/AspNetCore.ApiGateway.Client)](https://www.nuget.org/packages/AspNetCore.ApiGateway.Client)|[![Downloads count](https://img.shields.io/nuget/dt/AspNetCore.ApiGateway.Client)](https://www.nuget.org/packages/AspNetCore.ApiGateway.Client)|
|*ts-aspnetcore-apigateway-client*|[![NPM Version](https://img.shields.io/npm/v/ts-aspnetcore-apigateway-client)](https://www.npmjs.com/package/ts-aspnetcore-apigateway-client)|[![Downloads count](https://img.shields.io/npm/dy/ts-aspnetcore-apigateway-client)](https://www.npmjs.com/package/ts-aspnetcore-apigateway-client)|

```diff
+ Este projeto foi integrado à .NET Foundation, na categoria Seed.
```

## Visão Geral

A arquitetura de microsserviços utiliza um Api Gateway, conforme mostrado abaixo.

![Arquitetura](/Docs/Architecture.png)

**O pacote:**

*	Facilita a criação de um Api Gateway!!

## Recursos

*	Swagger
*	Autenticação
*   Filtros
    *   Ação
    *   Exceção
    *   Resultado
*   Balanceamento de carga
*   Cache de resposta
*   Web sockets
*   Event sourcing
*   Agregação de requisições
*   Middleware service
*   Logging
*   Clientes disponíveis em:
    *   .NET
    *   Typescript

## Gateway como uma Facade RESTful para Microsserviços

### Sua **API Gateway** é um microsserviço que expõe endpoints como uma **facade** para os endpoints de sua API backend.

*	GET
*   HEAD
*	POST
*	PUT
*   PATCH
*	DELETE

<div>
</div>
<img src="/Docs/FacadeDesignPattern.PNG" style="width:60%;height:auto;max-width:500px;" alt="API Gateway Facade" >

## Implementação

Na solução, existem 2 **APIs backend**: **Weather API** e **Stock API**.

Por exemplo, para fazer uma chamada GET para a API backend, é necessário configurar uma API e uma Rota GET no **Api Orchestrator** do Gateway.

A aplicação cliente faria uma chamada GET para a API Gateway, que então faria uma chamada GET para a API backend usando HttpClient.

## Na sua API Backend

Se houver um endpoint GET como este:

*	**HTTP GET - /weatherforecast/forecast**

## Na sua API Gateway

Você adiciona uma rota para a chamada GET no **Api Orchestrator**.

Crie uma API backend com ApiKey chamada **weatherservice**, por exemplo.
E uma rota com RouteKey chamada **forecast**, por exemplo.

A chamada para o Gateway seria:

*	**HTTP GET - /weatherservice/forecast**

**Adicione uma referência ao pacote e...**

*	Crie uma **Orquestração de API**.

```C#
    public static class ApiOrchestration
    {
        public static void Create(IApiOrchestrator orchestrator, IApplicationBuilder app)
        {
            var serviceProvider = app.ApplicationServices;

            var weatherService = serviceProvider.GetService<IWeatherService>();

            var weatherApiClientConfig = weatherService.GetClientConfig();

            orchestrator.StartGatewayHub = true;
            orchestrator.GatewayHubUrl = "https://localhost:44360/GatewayHub";

            orchestrator.AddApi("weatherservice", "http://localhost:63969/")
                                .AddRoute("forecast", GatewayVerb.GET, new RouteInfo { Path = "weatherforecast/forecast", ResponseType = typeof(IEnumerable<WeatherForecast>) })
                                .AddRoute("forecasthead", GatewayVerb.HEAD, new RouteInfo { Path = "weatherforecast/forecast" })
                                .AddRoute("typewithparams", GatewayVerb.GET, new RouteInfo { Path = "weatherforecast/types/{index}"})
                                .AddRoute("types", GatewayVerb.GET, new RouteInfo { Path = "weatherforecast/types", ResponseType = typeof(string[]), HttpClientConfig = weatherApiClientConfig })
                                .AddRoute("type", GatewayVerb.GET, new RouteInfo { Path = "weatherforecast/types/", ResponseType = typeof(WeatherTypeResponse), HttpClientConfig = weatherApiClientConfig })
                                .AddRoute("forecast-custom", GatewayVerb.GET, weatherService.GetForecast)
                                .AddRoute("add", GatewayVerb.POST, new RouteInfo { Path = "weatherforecast/types/add", RequestType = typeof(AddWeatherTypeRequest), ResponseType = typeof(string[])})
                                .AddRoute("update", GatewayVerb.PUT, new RouteInfo { Path = "weatherforecast/types/update", RequestType = typeof(UpdateWeatherTypeRequest), ResponseType = typeof(string[]) })
                                .AddRoute("patch", GatewayVerb.PATCH, new RouteInfo { Path = "weatherforecast/forecast/patch", ResponseType = typeof(WeatherForecast) })
                                .AddRoute("remove", GatewayVerb.DELETE, new RouteInfo { Path = "weatherforecast/types/remove/", ResponseType = typeof(string[]) })
                        .AddApi("stockservice", "http://localhost:63967/")
                                .AddRoute("stocks", GatewayVerb.GET, new RouteInfo { Path = "stock", ResponseType = typeof(IEnumerable<StockQuote>) })
                                .AddRoute("stock", GatewayVerb.GET, new RouteInfo { Path = "stock/", ResponseType = typeof(StockQuote) });
        }
    }
```

*	Configurar no **Startup.cs**

```C#
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddTransient<IWeatherService, WeatherService>();
            services.AddApiGateway();
            services.AddControllers();
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new OpenApiInfo { Title = "Meu API Gateway", Version = "v1" });
            });
        }
```

Para mais detalhes sobre uso, implantação e personalização, consulte a documentação no diretório **Docs/**.

