# Web API

## Infrastructure
You need some Azure infrastructure for develop and deploy this API:
- AppService for host the API
- SQL Server to store the data

Check the [Azure infrastructure guide](https://github.com/dasensio/myweeklydiet/blob/master/azure-infrastructure-guide.md) for setup

## Start to develop
> [Doc reference](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-2.2&tabs=visual-studio)

Open Visual Studio and create a new API project

Select ASP.NET Core Web Application and press **Next**
![VisualStudio](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Create_new_webapi_project_(1).png)

Type the name of the project and press **Create**
![VisualStudio](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Create_new_webapi_project_(2).png)

Select ASP.NET Core 2.2 version and API tpye. Then press **Create**
![VisualStudio](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Create_new_webapi_project_(3).png)

Let's see the created solution
![VisualStudio](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Create_new_webapi_project_(4).png)

The basic architecture of a Web API can be like this:
- Controllers: Endpoints for the API
- Services: Business layer
- Repositories: Database operations
- Repositories/DataContexts: Classes for DataContexts
- Models: Classes for store data
- Models/Databases: Clases for store data from database
- Helpers: Useful classes
- Exceptions: Exceptions classes

Let's go to create this structure

Next, you need some nuget packages for work with swagger, authorization and more:
- Microsoft.AspNetCore.Mvc.Versioning (3.1.2): For API Versioning
- Swashbuckle.AspNetCore (4.0.1): For Swagger
- System.IdentityModel.Tokens.Jwt (5.5.0): For authorization
- System.Configuration.ConfigurationManager (4.6.0): For read appsettings.json

## Prepare the application settings
You need to store in appsettings.json the ConnectionString of your database. Get this connection string from azure SQL configuration, and create a new key in appsettings.json, like this

```json
"ConnectionStrings": {
    "DefaultConnection": "Server=tcp:{yourServer},1433;Initial Catalog=myweeklydiet;Persist Security Info=False;User ID={yourUser};Password={yourPassword};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
  }
```

## Creating the first functionality
### Create the table
First, you want to create the controller for the master data of ingredients. This data could be selected by the user to make his own meals. First, create a new table called "Ingredient" with those fields:
- Id (uniqueidentifier)
- Name (nvarchar(255))
- Unit (nvarchar(255))

> You can use SQL Management Studio for do it

### Create the model
Next, create a new class in **Models/Database**, called **Ingredient** with the table properties. Add the [Table("Ingredient")] attribute to the class and resolve usings.
```C#
using System;
using System.ComponentModel.DataAnnotations.Schema;

namespace myweeklydiet.Models
{
    [Table("Ingredient")]
    public class Ingredient
    {
        public Guid? Id { get; set; }
        public String Name { get; set; }
        public String Unit { get; set; }
    }
}
```

### Create the data context
Next, you are about to create a new DataContext. This class is used by work with database, mapping a table. Create a new class in **Repositories/DataContexts**, called **IngredientContext**. Is needed that this class inherits from **DbContext** and a constructor with DbContextOptions<IngredientContext> as parameter.
  
Finally, you must to create a DbSet property for manage Ingredients data
```C#
using Microsoft.EntityFrameworkCore;
using myweeklydiet.Models;

namespace myweeklydiet.Repositories.DataContexts
{
    public class IngredientContext : DbContext
    {
        /// <summary>
        /// 
        /// </summary>
        /// <param name="options"></param>
        public IngredientContext(DbContextOptions<IngredientContext> options) : base(options)
        {
        }

        /// <summary>
        /// 
        /// </summary>
        public DbSet<Ingredient> Ingredients { get; set; }
    }
}
```

### Add the new DbContext to the application context
You need to add de new DataContext to the services collection in startup class. Open Startup.cs and create a new method called ConfigureDbContexts, with IServiceCollection as input parameter:

```C#
private void ConfigureDBContexts(IServiceCollection services)
{
    services.AddDbContext<IngredientContext>(options => options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
}
```

Next, modify the ConfigureServices Method and call the new method
```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    ConfigureDBContexts(services);
}
```
Now, the DbContext is ready for use in your repository

### Create the repository
You need a repository for manage the database operations. First, you are going to create a interface with the definition of the operations. Create a new interface in **Repositories/Interfaces** directory, called **IIngredientRepository**, with some operations: Insert, Update, Delete, Get and GetAll.

```C#
using myweeklydiet.Models;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace myweeklydiet.Repositories.Interfaces
{
    public interface IIngredientRepository
    {
        Task<Ingredient> Insert(Ingredient ingredient);
        Task<Ingredient> Update(Ingredient ingredient);
        Task Delete(Guid id);
        Task<Ingredient> Get(Guid id);
        Task<IEnumerable<Ingredient>> GetAll();
    }
}
```

Next, you are about to create the repository with the operations. You are focusing in the methods Insert and GetAll, not implementing the rest for now. Create a new class in **Repositories** directory called **IngredientRepository**, implementing IIngredientRepository.

You need to create a constructor with the IngredientContext for work with the database. 

The GetAll method will return context.Ingredients and the Insert method will set the Id and insert the new object. The class seems like this:

```C#
using Microsoft.EntityFrameworkCore;
using myweeklydiet.Models;
using myweeklydiet.Repositories.DataContexts;
using myweeklydiet.Repositories.Interfaces;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace myweeklydiet.Repositories
{
    public class IngredientRepository : IIngredientRepository
    {
        private IngredientContext _context;

        public IngredientRepository(IngredientContext context)
        {
            _context = context;
        }

        public async Task<IEnumerable<Ingredient>> GetAll()
        {
            return await _context.Ingredients.ToListAsync();
        }

        public async Task<Ingredient> Insert(Ingredient ingredient)
        {
            ingredient.Id = Guid.NewGuid();
            await _context.AddAsync(ingredient);
            await _context.SaveChangesAsync();

            return ingredient;
        }

        public async Task Delete(Guid id)
        {
            throw new NotImplementedException();
        }

        public async Task<Ingredient> Get(Guid id)
        {
            throw new NotImplementedException();
        }

        public async Task<Ingredient> Update(Ingredient ingredient)
        {
            throw new NotImplementedException();
        }
    }
}

```

Finally, you are about to add the repository to the app context for use it in the future. Go to Startup.cs and create a new method called ConfigureRepositories with IServiceCollection as input param. It seems like this:

```C#
private void ConfigureRepositories(IServiceCollection services)
{
    services.AddScoped<IIngredientRepository, IngredientRepository>();
}
```

And then, add the new method to ConfigureServices method, like this:

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    ConfigureDBContexts(services);
    ConfigureRepositories(services);
}
```

### Create service
Next step is to create the service. As the repository, you need an interface and a class that implements it. First, create a new interface in **Services/Interfaces** called **IIngredientService** with the methods Insert, Update, Delete, Get and GetAll.

```C#
using myweeklydiet.Models;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace myweeklydiet.Services.Interfaces
{
    public interface IIngredientService
    {
        Task<Ingredient> Insert(Ingredient ingredient);
        Task<Ingredient> Update(Ingredient ingredient);
        Task Delete(Guid id);
        Task<Ingredient> Get(Guid id);
        Task<IEnumerable<Ingredient>> GetAll();
    }
}
```

Then, you need to create the class **IngredientService** inside **Services** for implement **IIngredientService**. You're focused in implement GetAll and Insert methods. Also, you need to create a constructor with the **IIngredientRepository** for work with **IngredientRepository** and a method to make the insert validations.

The method GetAll will return the result of repository.GetAll. The method Insert will make validations, and will return the result of repository.Insert method.

```C#
using myweeklydiet.Models;
using myweeklydiet.Repositories.Interfaces;
using myweeklydiet.Services.Interfaces;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace myweeklydiet.Services
{
    public class IngredientService : IIngredientService
    {
        private IIngredientRepository _repository;

        private void InsertValidations(Ingredient ingredient)
        {
            if (ingredient == null)
            {
                throw new Exception("Ingredient is required");
            }

            if (String.IsNullOrEmpty(ingredient.Name))
            {
                throw new Exception("The name of the ingredient is required");
            }

            if (String.IsNullOrEmpty(ingredient.Unit))
            {
                throw new Exception("The unit of the ingredient is required");
            }
        }

        public IngredientService(IIngredientRepository repository)
        {
            _repository = repository;
        }
        public async Task Delete(Guid id)
        {
            throw new NotImplementedException();
        }

        public async Task<Ingredient> Get(Guid id)
        {
            throw new NotImplementedException();
        }

        public async Task<IEnumerable<Ingredient>> GetAll()
        {
            return await _repository.GetAll();
        }

        public async Task<Ingredient> Insert(Ingredient ingredient)
        {
            InsertValidations(ingredient);

            return await _repository.Insert(ingredient);
        }

        public async Task<Ingredient> Update(Ingredient ingredient)
        {
            throw new NotImplementedException();
        }
    }
}
```

Finally, as the repository, you need to add the service to the app context for use it in the future. Go to Startup.cs and create a new method called **ConfigureAPIServices** with IServiceCollection as input param. It seems like this:

```C#
private void ConfigureAPIServices(IServiceCollection services)
{
    services.AddScoped<IIngredientService, IngredientService>();
}
```

And call the method inside the ConfigureServices method:

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    ConfigureDBContexts(services);
    ConfigureRepositories(services);
    ConfigureAPIServices(services);
}
```

### And finally... create the controller
The controller will recieve Http requests and will call the services. This is the place where you will define your API Restful endpoints. Let's go to create a new class called **IngredientController** inside **Controllers** directory, inheriting from **ControllerBase**, with a constructor with a **IIngredientService** parameter and two methods: GetAll and Insert

This seems like this:

```C#
using Microsoft.AspNetCore.Mvc;
using myweeklydiet.Models;
using myweeklydiet.Services.Interfaces;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace myweeklydiet.Controllers
{
    public class IngredientController : ControllerBase
    {
        private IIngredientService _service;

        public IngredientController(IIngredientService service)
        {
            _service = service;
        }

        public async Task<IEnumerable<Ingredient>> GetAll()
        {
            return await _service.GetAll();
        }

        public async Task<Ingredient> Insert(Ingredient ingredient)
        {
            return await _service.Insert(ingredient);
        }
    }
}
```

BUT...

**It isn't enough**

A API controller need some decoration with attributes that defines [the behavior of the methods of our API Restful](https://github.com/dasensio/myweeklydiet/blob/master/api-restful-behavior.md).

The most important attributes are:
- **[ApiController]**: Class attribute. Defines that the class is an API Controller
- **[Route("api/[controller]")]**: Class attribute. Defines the route of the controller. [controller] will be **Ingredient** in this case, part of the name of the **Ingredient**Controller.
- **[HttpGet]**, **[HttpPost]**, **[HttpPut]**, **[HttpDelete]**: Method attribute. Verbs of the request. You can change the default route if you need. Example:
```C#
[HttpGet("test")]
```
Indicates that the url for the request will be http://yourdomain.com/api/ingredient/test
- **[ProducesResponseType]**: Method attribute that indicates the type of the response. You need one for each response. For example: 
```C#
[ProducesResponseType(typeof(Ingredient), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
```
Indicates that the method can return 201, 400 or 500 responses.
- **[Produces]**: Method attribute that indicates the Content-Type of the response. For Example: 
```C#
[Produces("application/json")]
```
Indicates that the method returns a json result

The final result of your controller is like this:

```C#
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using myweeklydiet.Models;
using myweeklydiet.Services.Interfaces;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace myweeklydiet.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class IngredientController : ControllerBase
    {
        private IIngredientService _service;

        public IngredientController(IIngredientService service)
        {
            _service = service;
        }

        [HttpGet]
        [ProducesResponseType(typeof(Ingredient), StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status500InternalServerError)]
        [Produces("application/json")]
        public async Task<IEnumerable<Ingredient>> GetAll()
        {
            return await _service.GetAll();
        }

        [HttpPost]
        [ProducesResponseType(typeof(Ingredient), StatusCodes.Status201Created)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status500InternalServerError)]
        [Produces("application/json")]
        public async Task<Ingredient> Insert(Ingredient ingredient)
        {
            return await _service.Insert(ingredient);
        }
    }
}
```

This is the moment for test your API!

## Next Steps
1. API Versioning (Comming soon)
2. Document your API using Swagger (Comming soon)
