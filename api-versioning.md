# API Versioning

To guarantee the compatibility with the old versions of API consumers, you need to add API versioning. This means that if you evolve your API, you can create new versions for the new functionalities, but the old version still works

It is easy to do it.

Go to Startup.cs class and create a new method called ConfigureApiVersionning with IServiceCollection as parameter. For the versioning you're going to use media type for define the version on the requests, and make version 1.0 as default version

```C#
private static void ConfigureApiVersionning(IServiceCollection services)
{
  services.AddApiVersioning(o =>
  {
    o.ReportApiVersions = true;
    o.AssumeDefaultVersionWhenUnspecified = true;
    o.ApiVersionReader = new MediaTypeApiVersionReader();
    o.DefaultApiVersion = new ApiVersion(1, 0);
  });
}
```

Call this method on ConfigureServices method

```C#
public void ConfigureServices(IServiceCollection services)
{
  services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

  ConfigureDBContexts(services);
  ConfigureRepositories(services);
  ConfigureAPIServices(services);

  ConfigureSwagger(services);
  ConfigureApiVersionning(services);
}
```

API Versioning is configured, but you need to configure swagger to show correctly the methods for each version. Create a new class called **ApiVersionOperationFilter** in **Helpers** directory. In this class you will create the filter for select the correct swagger.json version in the Swagger UI

Create the Apply method for do it. This is the result of the class:

```C#
using System.Linq;
using Microsoft.AspNetCore.Mvc.Abstractions;
using Swashbuckle.AspNetCore.Swagger;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace Ooredoo_Sports_Backend.Helpers
{
    /// <summary>
    /// 
    /// </summary>
    public class ApiVersionOperationFilter : IOperationFilter
    {
        /// <summary>
        /// Apply the specified operation and context.
        /// </summary>
        /// <param name="operation">Operation.</param>
        /// <param name="context">Context.</param>
        public void Apply(Operation operation, OperationFilterContext context)
        {
            var actionApiVersionModel = context.ApiDescription.ActionDescriptor?.GetApiVersionModel();
            if (actionApiVersionModel == null)
            {
                return;
            }

            if (actionApiVersionModel.DeclaredApiVersions.Any())
            {
                operation.Produces = operation.Produces.SelectMany(p => actionApiVersionModel.DeclaredApiVersions
                    .Select(version => $"{p};v={version.ToString()}")).ToList();
            }
            else
            {
                operation.Produces = operation.Produces
                  .SelectMany(p => actionApiVersionModel.ImplementedApiVersions.OrderByDescending(v => v)
                    .Select(version => $"{p};v={version.ToString()}")).ToList();
            }
        }
    }
}
```


Go to **ConfigureSwagger** method on **Startup.cs** method and change the call of **AddSwaggerGen** to:
- Add the versions 1.0 and 1.1
- Call **DocInclusionPredicate** and **OperationFilter** to include each action in the correct version in swagger.json

It seems like this

```C#
c.SwaggerDoc("v1.1", new Info { Title = "My Weekly Diet API", Version = "v1.1" });
c.SwaggerDoc("v1.0", new Info { Title = "My Weekly Diet API", Version = "v1.0" });

c.DocInclusionPredicate((docName, apiDesc) =>
{
  var actionApiVersionModel = apiDesc.ActionDescriptor?.GetApiVersionModel();
  // would mean this action is unversioned and should be included everywhere
  if (actionApiVersionModel == null)
  {
    return true;
  }
  if (actionApiVersionModel.DeclaredApiVersions.Any())
  {
    return actionApiVersionModel.DeclaredApiVersions.Any(v => $"v{v.ToString()}" == docName);
  }
  return actionApiVersionModel.ImplementedApiVersions.Any(v => $"v{v.ToString()}" == docName);
});
c.OperationFilter<ApiVersionOperationFilter>();
```
Finally, add the Swagger endpoint for the new version 1.1 in the call to **app.UserSwaggerUI** in the method **Configure**, like this:

```C#
app.UseSwaggerUI(c =>
{
  c.SwaggerEndpoint("/swagger/v1.0/swagger.json", "MyWeeklyDiet API v1.0");
  c.SwaggerEndpoint("/swagger/v1.1/swagger.json", "MyWeeklyDiet API v1.1");
  c.RoutePrefix = string.Empty;
});
```
Swagger is ready for use API Versions!

Is the moment for define the versions of your API. Go to IngredientController and add the ApiVersion attribute for versions 1.0 and 1.1, like this:

```C#
[ApiController]
[ApiVersion("1.0")]
[ApiVersion("1.1")]
[Route("api/[controller]")]
public class IngredientController : ControllerBase
```

Create a new method called Test, and returns "Test v.1.1". Decorate this method as Get request with route "test" and with version 1.1

```C#
[HttpGet("test")]
[MapToApiVersion("1.1")]
public String Test()
{
  return "Ok v1.1";
}
```

It's time to test the API versioning! 

Execute your API and you will see the two versions
![Versioning](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Test_swagger_with_versioning_(1).png)

If you change to version 1.1, you will see the test action!!!
![Versioning](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Test_swagger_with_versioning_(2).png)

## Next steps
- [Do the challenge!](https://github.com/dasensio/myweeklydiet/blob/master/challenge.md)
