---
layout: post
title:  "API Rest coordinar consumo modelos AI"
date:   2021-03-29 00:00:00 +0200
tipue_search_active: true
excerpt_separator: <!--end_excerpt-->
tags: AI Docker Kubernetes Dev Cloud
---

Suponte que tienes una solución que involucra varios proyectos diferentes de inteligencia artificial, que has modelado de forma independiente y tienes expuestos como API Rest, pero ahora quieres coordinar desde un único punto de entrada.

Por ejemplo, suponte que quieres procesar textos y extraer por un lado de qué está hablándose y por otro el sentimiento de eso mismo. En mi caso particular, he creado dos redes neuronales independientes, que están enfocadas a realizar esta tarea de forma muy concreta y por tanto no me sirve la misma red neuronal para realizar las dos acciones.

- Tengo una red neuronal que hace muy bien NER de un párrafo
- Tengo una red que predice muy bien el sentimiento de parrafos completos que hablan específicamente con lenguaje financiero

En mi caso, ambos modelos AI los tengo ya funcionando en su respectiva API...pero ahora resulta que quiero coordinar su ejecución (dado un párrafo quiero las dos cosas). ¿Cómo lo resuelvo?

<!--end_excerpt-->

Te diré que mientras que para las API Rest que exponen mis redes neuronales he utilizado python con [FastAPI](https://fastapi.tiangolo.com/), para el caso de la coordinación he decidido utilizar ASP.NET. 

## Crear el proyecto

Vamos a utilizar VS2019 en este caso porque tiene un fantástico template para comenzar.

### Creamos solucion
![vs2019 aspnet core 5](/img/posts/api-rest-coordinacion-modelos-ai/vs2019aspnetcore5_1.png)

### Elegimos ASP.NET Core 5
![vs2019 aspnet core 5](/img/posts/api-rest-coordinacion-modelos-ai/vs2019aspnetcore5_2.png)

## Añadimos Authentication por API KEY

Dado que va a acabar desplegado en un cluster kubernetes previamente securizado interno en nuestra red privada, podemos elegir una autenticación básica basada en API_KEY para securizar minimamente el acceso a nuestra API.

Para ello, crearemos una clase dentro de "Security" a la que llamaremos "ApiKeySecurityMiddleware" y le meteremos este contenido:

```c#
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using System.Threading.Tasks;

namespace Security.Middleware
{
    public class ApiKeySecurityMiddleware
    {
        private readonly RequestDelegate _next;
        private const string APIKEYNAME = "ApiKey";
        public ApiKeySecurityMiddleware(RequestDelegate next)
        {
            _next = next;
        }
        public async Task InvokeAsync(HttpContext context)
        {
            if (!context.Request.Headers.TryGetValue(APIKEYNAME, out var extractedApiKey))
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("Api Key was not provided. (Using ApiKeySecurityMiddleware) ");
                return;
            }

            var appSettings = context.RequestServices.GetRequiredService<IConfiguration>();

            var apiKey = appSettings.GetValue<string>(APIKEYNAME);

            if (!apiKey.Equals(extractedApiKey))
            {
                context.Response.StatusCode = 401;
                await context.Response.WriteAsync("Unauthorized client. (Using ApiKeySecurityMiddleware)");
                return;
            }

            await _next(context);
        }
    }
}
```

## Activar nuestro Middleware

En el template tenemos ya lista la autenticación por API_KEY. Simplemente tenemos que activarlo, entrando en la clase `Startup.cs` y añadir la siguiente línea despues de `app.UseAuthorization()`

```c#
app.UseMiddleware<ApiKeySecurityMiddleware>();
```

![vs2019 aspnet core 5 security](/img/posts/api-rest-coordinacion-modelos-ai/vs2019aspnetcore5_3.png)

>NOTA: Evidentemente este es el de ejemplo, ahora crearemos el nuestro

## Testear con postman nuestra API

Vamos a testear que funcione correctamente, configurando nuestro request de ejemplo con [postman](https://www.postman.com/downloads/) 

Deberíamos ver el output 200 OK:

![postman](/img/posts/api-rest-coordinacion-modelos-ai/vs2019aspnetcore5_3.png)
