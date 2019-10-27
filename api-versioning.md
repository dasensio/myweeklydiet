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


Go to **ConfigureSwagger** method on **Startup.cs** method and change the call of **AddSwaggerGen** to call **DocInclusionPredicate** and **OperationFilter** to include each action in the correct version in swagger.json:

```C#
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
