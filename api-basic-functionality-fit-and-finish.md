# Fit & Finish your basic functionality

Your are about to:
- Improve errors management
- Use ActionResults for return data
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
        public APIException(APIExceptionType type, String extendedMessage = "")
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
