# Configure Swagger for your API

You know as swagger is an excelent tool for document and test your API, and it is easy to use.

Go to Startup.cs and create a method called ConfigureSwagger with a IServiceCollection parameter. Inside this method you are about to add Swagger generator using the XML comments and generating a swagger.json in a v1.0 directory.

```C#
private void ConfigureSwagger(IServiceCollection services)
{
  services.AddSwaggerGen(c =>
  {
    c.SwaggerDoc("v1.0", new Info { Title = "My Weekly Diet API", Version = "v1.0" });

    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    c.IncludeXmlComments(xmlFile);
  });
}
```

Next, call ConfifureSwagger on your ConfigureServices method

```C#
public void ConfigureServices(IServiceCollection services)
{
  services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

  ConfigureDBContexts(services);
  ConfigureRepositories(services);
  ConfigureAPIServices(services);

  ConfigureSwagger(services);
}
```

And finally, call the methods UseSwagger and UseSwaggerUI methods inside Configure method:

```C#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
  if (env.IsDevelopment())
  {
    app.UseDeveloperExceptionPage();
  }
  else
  {
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
  }

  app.UseHttpsRedirection();
  app.UseMvc();

  app.UseSwagger();
  app.UseSwaggerUI(c =>
  {
    c.SwaggerEndpoint("/swagger/v1.0/swagger.json", "MyWeeklyDiet API");
    c.RoutePrefix = string.Empty;
  });
}
```

You have to configure your project to generate XML with documentation. Push right button of the mouse in the project and then, click on Properties.

Go to Build tab -> Output and check XML documentantion file
![Swagger](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Configure_Swagger_(2).png)

Go to Debug tab -> check "Launch browser" with empty field 
![Swagger](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Configure_Swagger_(3).png)

Test your API, the result must be like this:
![Swagger](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Configure_Swagger.png)

Go to your IngredientController and create comments for the methods, like this.

```C#
/// <summary>
/// Returns all the ingredients
/// </summary>
/// <returns></returns>
[HttpGet]
[ProducesResponseType(typeof(ApiResult<IEnumerable<IngredientDTO>>), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
[Produces("application/json")]
public async Task<ActionResult<IEnumerable<IngredientDTO>>> GetAll()
{
  try
  {
    return new ApiResult<IEnumerable<IngredientDTO>>(await _service.GetAll());
  }
  catch(APIException ex)
  {
    return new ExceptionResult(ex);
  }
  catch(Exception ex)
  {
    return new ExceptionResult(ex);
  }
}

/// <summary>
/// Creates a new ingredient
/// </summary>
/// <param name="ingredient">The ingredient to create</param>
/// <returns></returns>
[HttpPost]
[ProducesResponseType(typeof(IngredientDTO), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
[Produces("application/json")]
public async Task<ActionResult<IngredientDTO>> Insert(IngredientDTO ingredient)
{
  try
  {
    return new ApiResult<IngredientDTO>(await _service.Insert(ingredient), StatusCodes.Status201Created);
  }
  catch (APIException ex)
  {
    return new ExceptionResult(ex);
  }
  catch (Exception ex)
  {
    return new ExceptionResult(ex);
  }
}
```
right-click on the file myweeklydiet.xml and click on **Properties**. Change the property **Copy to Output Directory** to **Copy always** in order to include this file on deployments.
![Swagger](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Configure_Swagger_(4).png)


Debug your API and... 
**Amazing! you can explore and test your API using swagger!**

## Next steps
- [API Versioning](https://github.com/dasensio/myweeklydiet/blob/master/api-versioning.md)
