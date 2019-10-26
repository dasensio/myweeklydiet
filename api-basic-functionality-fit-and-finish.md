# Fit & Finish your basic functionality

Your are about to:
- Improve errors management
- Use ActionResults for return data and errors
- Use DTO for disengage your model database and your user interface.

## Improve errors management
Create a new class called **APIExceptionType** in **Exceptions** directory. This class has three properties with only get: Code (string), Message (string) and StatusCode (int) and a constructor for the three parameters (with 400 as fedault value for StatusCode). 

Next, you will create some static properties for standard errors

It seems like this:

```C#
using System;

namespace myweeklydiet.Exceptions
{
    public class APIExceptionType
    {
        public String Code { get; }
        public String Message { get; }
        public int StatusCode { get; }
        
        public APIExceptionType(String code, String message, int statusCode)
        {
            Code = code;
            Message = message;
            StatusCode = statusCode;
        }

        public static APIExceptionType Generic = new APIExceptionType("APIEX001", "Generic error", 500);
        public static APIExceptionType NotFound = new APIExceptionType("APIEX002", "Item not found", 404);
        public static APIExceptionType ValidationError = new APIExceptionType("APIEX003", "Validation error", 400);
    }
}
```

Next, create a new class in **Exceptions** directory called **APIException**. This class inherits from **Exception** and has two properties: ExtendedMessage (string) and Type (APIExceptionType). The constructor will need a exceptionType parameter and a optional extendedMessage. This is the final result:

```C#
using System;

namespace myweeklydiet.Exceptions
{
    public class APIException : Exception
    {
        public APIException(APIExceptionType type, String extendedMessage = "") : base(type.Message + ". " + extendedMessage)
        {
            Type = type;
            ExtendedMessage = extendedMessage;
        }

        public APIExceptionType Type { get; }
        public String ExtendedMessage { get; set; }
    }
}
```

Go to **InsertValidations** in the **IngredientService** and use the new APIException, resolving using issues:

```C#
private void InsertValidations(Ingredient ingredient)
{
    if (ingredient == null)
    {
        throw new APIException(APIExceptionType.ValidationError, "Ingredient is required");
    }

    if (String.IsNullOrEmpty(ingredient.Name))
    {
        throw new APIException(APIExceptionType.ValidationError, "The name of the ingredient is required");
    }

    if (String.IsNullOrEmpty(ingredient.Unit))
    {
        throw new APIException(APIExceptionType.ValidationError, "The unit of the ingredient is required");
    }
}
```

The use of APIException will improve your errors management. Your API **always** will returns the same format for errors and the status code defined by the exception type.

## Use ActionResults for return data and errors
Create a new directory called **StandardResults** in **Controller** directory, and create three new classes: ApiResult, StandardResult and ExceptionResult. All of them inherits from ActionResult.

ExceptionResults will returns the erros and the correct status code. It has the properties Code, Message, ExtendedMessage, a constructor that receive APIException, a constructor that receive Exception and a method that overrides ExecuteResult for build the response.

It seems like this:

```C#
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using myweeklydiet.Exceptions;
using Newtonsoft.Json;
using System;

namespace myweeklydiet.Controllers.StandardResults
{
    public class ExceptionResult : ActionResult
    {
        private int _statusCode;

        public ExceptionResult(APIException ex)
        {
            Code = ex.Type.Code;
            Message = ex.Type.Message;
            ExtendedMessage = ex.ExtendedMessage;
            _statusCode = ex.Type.StatusCode;
        }

        public ExceptionResult(Exception ex)
        {
            APIException apiex = new APIException(APIExceptionType.Generic);
            Code = apiex.Type.Code;
            Message = apiex.Type.Message;
            ExtendedMessage = ex.Message;
            _statusCode = apiex.Type.StatusCode;
        }

        public String Code { get; }
        public String Message { get; }
        public String ExtendedMessage { get; }

        public override void ExecuteResult(ActionContext context)
        {
            HttpResponse response = context.HttpContext.Response;
            response.StatusCode = _statusCode;
            response.ContentType = "application/json";
            response.WriteAsync(JsonConvert.SerializeObject(this));
        }
    }
}
```

StandardResult will returns a standard success result, with no other data. As ExceptionResult, this class has a Code, Message and it will override ExecuteResult. The constructor will receive a statusCode with 200 as default. This is the result:

```C#
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;
using System;

namespace myweeklydiet.Controllers.StandardResults
{
    public class StandardResult : ActionResult
    {
        private int _statusCode;
        public StandardResult(int statusCode = 200)
        {
            Code = "001";
            Message = "Success";
            _statusCode = statusCode;
        }

        public String Code { get; }
        public String Message { get; }

        public override void ExecuteResult(ActionContext context)
        {
            HttpResponse response = context.HttpContext.Response;
            response.StatusCode = _statusCode;
            response.ContentType = "application/json";
            response.WriteAsync(JsonConvert.SerializeObject(this));
        }
    }
}
```

And finally, the ApiResult will return the response data and the status code specified. It uses generic types.

```C#
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;

namespace myweeklydiet.Controllers.StandardResults
{
    public class ApiResult<T> : ActionResult where T : class
    {
        private T _result;
        private int _statusCode;

        public ApiResult(T result, int statusCode = 200)
        {
            _result = result;
            _statusCode = statusCode;
        }

        public override void ExecuteResult(ActionContext context)
        {
            HttpResponse response = context.HttpContext.Response;
            response.StatusCode = _statusCode;
            response.ContentType = "application/json";
            response.WriteAsync(JsonConvert.SerializeObject(_result));
        }
    }
}
```

Let's go to enjoy these useful classes. Go to IngredientController and change GetAll. You will add a try catch to use ExceptionResult and ApiResult:

```C#
[HttpGet]
[ProducesResponseType(typeof(ApiResult<IEnumerable<Ingredient>>), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
[Produces("application/json")]
public async Task<ActionResult<IEnumerable<Ingredient>>> GetAll()
{
    try
    {
        return new ApiResult<IEnumerable<Ingredient>>(await _service.GetAll());
    }   
    catch(Exception ex)
    {
        return new ExceptionResult(ex);
    }
}
```
By default, ApiResult returns 200 status code (OK)

Finally, make the same with Insert method:

```C#
[HttpPost]
[ProducesResponseType(typeof(Ingredient), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
[Produces("application/json")]
public async Task<ActionResult<Ingredient>> Insert(Ingredient ingredient)
{
    try
    {
        return new ApiResult<Ingredient>(await _service.Insert(ingredient), StatusCodes.Status201Created);
    }
    catch(Exception ex)
    {
        return new ExceptionResult(ex);
    }
}
```

In this case, you are returning a 201 (Created) status code with the result of the operation