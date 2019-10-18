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
- Models: Classes for store data
- Helpers: Useful classes
- Exceptions: Exceptions classes
