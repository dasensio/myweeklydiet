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
Go to Debug tab -> check "Launch browser" with empty field 

Test your API, the result must be like this:

![Swagger](https://danielasensiolabs.blob.core.windows.net/myweeklydietlab/01_Configure_Swagger.png)
