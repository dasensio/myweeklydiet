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
First, you want to create the controller for the master data of ingredients. This data could be selected by the user to make his own meals. First, create a new table called "Ingredient" with those fields:
- Id (uniqueidentifier)
- Name (nvarchar(255))
- Unit (nvarchar(255))

> You can use SQL Management Studio for do it

Next, create a new class in **Models/Database**, called **Ingredient** with the table properties. Add the [Table("Ingredient")] attribute to the class and resolve usings.
![Azure](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/02_Create_DataModel_Structure_(2).png)

Next, you are about to create a new DataContext. This class is used by work with database, mapping a table. Create a new class in **Repositories/DataContexts**, called **IngredientContext**. Is needed that this class inherits from **DbContext** and a constructor with DbContextOptions<IngredientContext> as parameter.
  
Finally, you must to create a DbSet property for manage Ingredients data
![Azure](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/02_Create_DataModel_Structure_(1).png)
