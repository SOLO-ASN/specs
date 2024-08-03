# Creating Swagger (Reference documentation)
AutoRest is a tool for generating HTTP client libraries from Swagger files. This document explains the conventions 
and extensions used by AutoRest in processing Swagger to produce client libraries. The Swagger specification can 
be found at [Swagger RESTful API Documentation Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md).

This page is intended to serve as reference documentation. If you have any questions about a specific part of a Swagger file or how AutoRest uses it to generate code, this is the correct place.

1. [Swagger](#Swagger)
2. [Info Object](#InfoObject)
3. [Host](#Host)
4. [Schemes](#Schemes)
5. [Consumes / Produces](#ConsumesProduces)
6. [Paths](#Paths)
7. [Path Item](#PathItem)
  1. [Operation and OperationId](#OperationOperationId)
  2. [Parameters](#Parameters)
    1. [Path Parameters](#PathParameters)
    2. [Query Parameters](#QueryParameters)
    3. [Body Parameters](#BodyParameters)
    4. [Header Parameters](#HeaderParameters)
    5. [FormData Parameters](#FormDataParameters)
  3. [Responses](#Responses)
    1. [Default Response](#DefaultResponse)
    2. [Negative Responses](#NegativeResponses)
8. [Defining Model Types](#DefiningModel)
  1. [Model Inheritance](#ModelInheritance)
  2. [Polymorphic Response Models](#PolymorphicResponse)
9. [ResourceFlattening](#ResourceFlattening)
  1. [Conditions](#Conditions)
  2. [x-ms-azure-resource](#x-ms-azure-resource)
10. [Enums with x-ms-enum](#Enum-x-ms-enum)
  1. [Structure](#enum-structure)
  2. [Understanding modelAsString](#modelAsString)
11. [Paging with x-ms-pageable](#Paging-x-ms-pageable)
  1. [Structure](#paging-structure)
  2. [Pageable Operation](#pageOperation)
  3. [Pageable Model](#pageModel)
12. [URL Encoding](#UrlEncoding)
13. [Long Running operation](#longrunning)
14. [Global parameters](#globalParam)

## Swagger<a name="Swagger"></a>
In this document, references to the 'spec' or to the 'swagger' refer to an instance of a Swagger file. AutoRest 
supports Swagger version 2.0. The version of Swagger being used must be included in the spec.

```json5
{
  "swagger": "2.0"
  ...
}
```

## Info Object<a name="InfoObject"></a>
Each spec includes an "info object."
The **title** field is used as the name of the generated client.
```json5
"info": {
  "title": "MyClientName",
}
```

```csharp
var client = new MyClientName();
```
Override the title client name by passing the `-ClientName` to AutoRest.
>autorest.exe -ClientName MyClientName

The version field specifies the service version of the API described in the spec. This is used as the default 
value for Azure clients where api-version is passed in the querystring.
```json5
"info": {
  "version": "2014-04-01-preview"
```

```
https://management.azure.com/...?api-version="2014-04-01-preview"
```
## Host<a name="Host"></a>
The host field specifies the baseURI. (everything after the protocol and before the path).
``` json
{
  "swagger": "2.0",
  "host": "management.azure.com"
}
```

## Schemes<a name="Schemes"></a>
The schemes field is an array of transfer protocols that can be used by individual operations. AutoRest supports 
*http* and *https*. The generated client uses the first scheme from the array.
```json5
{
  "swagger": "2.0",
  "schemes": [
    "https",
    "http"
  ]
  . . .
}
```

## Consumes / Produces<a name="ConsumesProduces"></a>
The *consumes* and *produces* fields declare the mime-types supported by the API. The root-level definition can 
be overridden by individual operations. AutoRest supports JSON payloads.
```json5
{
  "swagger": "2.0",
  "consumes": [
    "application/json"
  ],
  "produces": [
    "application/json"
  ]
  . . .
}
```

## Paths<a name="Paths"></a>
The paths object defines the relative paths of API endpoints (appended to the baseURI to invoke operations).
```json5
"swagger": "2.0",
"paths": {
  "/tenants": {
  ...
  },
  "/subscriptions": {
  ...
  }
}
```

```
https://management.azure.com/tenants?api-version=2014-04-01-preview
OR
https://management.azure.com/subscriptions?api-version=2014-04-01-preview
```
## Path Item<a name="PathItem"></a>
Each path item in the spec defines the operations available at that endpoint. The individual operations are 
identified by the HTTP operation (sometimes referred to as HTTP verbs). AutoRest supports: **head**, **get**, 
**put**, **post**, **delete** and **patch**.

```json5
{
  "swagger": "2.0",
  "paths": {
    "/stations": {
      "get": {
      ...
      },
      "put": {
      ...
```
### Operation and OperationId<a name="OperationOperationId"></a>
Every operation must have a unique operationId. The operationId is used to determine the generated method name.
```json5
"paths": {
  "/users": {
    "get": {
      "operationId": "ListUsers"
    }
  }
```

```csharp
var client = new MyClientName();
var listResult = client.ListUsers();
IList<User> users = listResult.Value;
```
All of the operations specified become methods on the client. The client API model can easily become a long and 
cumbersome list of methods. AutoRest allows grouping of operations by using a naming convention. Operation groups 
are identified by splitting the operationId by underscore.
```json5
"paths": {
  "/{tenantId}/users": {
    "get": {
      "operationId": "Users_List",
      ...
```
instead of
```csharp
var listResult = client.ListUsers();
```
the method becomes part of an operation group interface

```csharp
var listResult = client.Users.List();
```
The operation group interface and the interface implementation is the "core" of the operation. Other signatures of 
this API ultimately call the "core" implementation. This allows the operations to be easily mocked for testing without 
needing to mock all signature variations. Other API signatures for the same operation are generated as extension methods.
For example, consider this `GetUserById` operation on the `SampleClient`.
```json5
"/users/{userId}": {
    "get": {
      "operationId": "Users_GetById"
...
```
The generated method signature is in the `IUsers`  interface
```csharp
public interface IUsers
{
    Task<HttpOperationResponse<User>> GetByIdWithOperationResponseAsync(string userId, CancellationToken cancellationToken = default(CancellationToken));
}
```
The core signature returns a generic HttpOperationResponse which includes the HttpRequest and HttpResponse objects as 
well as the payload object. In addition to the "core" signature, there is a synchronous version and an async version 
that returns the payload object without the request and response objects.
```csharp
public static partial class UsersExtensions
{
    public static User GetById(this IUser operations, string userId) {...}

    public static async Task<User> GetByIdAsync( this IUser operations, string userId, CancellationToken cancellationToken = default(CancellationToken)) {...}
}
```

### Parameters<a name="Parameters"></a>
Each operation defines the parameters that must be supplied. An operation may not require any parameters. A parameter 
defines a name, description, schema, what request element it is `in` (AutoRest supports parameters in the **path**, **query**, 
**header**, and **body** of the request) and whether or not is it `required`.

#### Path Parameters<a name="PathParameters"></a>
The value of path parameters are replaced in the templated URL path at the position specified by the name. Parameters that 
are `in` the `path` must have a `true` value for the `required` field. In this example, the `{userId}` in this operation 
is populated with the value of userId provided in the client method.
```json5
"paths": {
  "/users/{userId}": {
    "get": {
      "operationId": "users_getById",
      "parameters": [
        {
          "name": "userId",
          "in": "path",
          "required": true,
          "type": "string",
          "description": "Id of the user."
        }
      ]
    }
  }
}
```
In the generated code, the path parameter is an argument of the method.
```csharp
string userId = "abcxyz";
var user = client.Users.GetById(userId);
```
>https://{host}/{basePath}/users/abcxyz

#### Query Parameters<a name="QueryParameters"></a>
Query parameters are appended to the request URI. They can be specified as required or optional.
```json5
"paths": {
  "/users/{userId}": {
    "get": {
      "operationId": "users_getUserById",
      "parameters": [
        {
          "name": "api-version",
          "in": "query",
          "required": true,
          "type": "string",
          "description": "Version of API to invoke."
        }
      ]
    }
  }
}
```
The user doesn't need to know where the parameter is placed. Doc comments surface the required/optional distinction.

#### Body Parameters<a name="BodyParameters"></a>
Body parameters include schema for the payload. In this example, the schema element is a `$ref` to the type details 
in the `#/definitions` section of the spec. More on the `#/definitions` later.
```json5
"paths": {
  "/subscriptions/{subscriptionId}/providers/Microsoft.Storage/checkNameAvailability": {
    "post": {
      "operationId": "StorageAccounts_CheckNameAvailability",
      "parameters": [
        {
          "name": "accountName",
          "in": "body",
          "required": true,
          "schema": {
            "$ref": "#/definitions/StorageAccountCheckNameAvailabilityParameters"
          }
        }
      ],
      ...
```

#### Header Parameters<a name="HeaderParameters"></a>
Header parameters are sent as part of the HTTP request header.
In general, reserved headers (`Content-Length`, `Content-Type`, ...) should not be documented since their values are derivable (e.g. from the request body) and not really part of the protocol specified by the OpenAPI definition.
Rather, they are part of the REST standard that the protocol is supposed to adhere to anyway.

However, there are rare scenarios for making the `Content-Type` customizable as part of a request, e.g. in case of a binary/stream request body.
The media type of binary request bodies is not reliably derivable: Maybe the service endpoint accepts PNG, JPEG or BMP images, which is expressed in OpenAPI using
``` yaml
consumes:
  - image/png
  - image/jpeg
  - image/bmp
```
and give the request body type `file`.
Now, when a request is made, the protocol has to somehow communicate to the server which of the media types the body has.
Since there is a range of possibilities (and it's certainly not a protocol's job to parse and classify binary data), we suggest adding a `Content-Type` header parameter to the operation's definition.
Unless one provides an `enum` restriction for that parameter, [AutoRest](https://github.com/Azure/autorest) will automatically make the parameter an enum with values drawn from the `consumes` declaration.
This allows for deduplication and hence prevents potential bugs.
More information on how [AutoRest](https://github.com/Azure/autorest) treats a `Content-Type` header parameter can be found [here](https://github.com/Azure/autorest/tree/master/Samples/test/stream-with-content-type).

#### FormData Parameters<a name="FormDataParameters"></a>
>Note: FormData parameters are not currently supported by AutoRest.

### Responses<a name="Responses"></a>
Each operation defines the possible responses by HTTP status code:
```json5
"/users/{userId}": {
  "get": {
    "operationId": "users_getUserById",
    "responses": {
      "200": {
        "schema": {
          "$ref": "#/definitions/user"
        }
      }
    }
  }
}
```

#### Default Response<a name="DefaultResponse"></a>
Swagger allows for specifying a `default` response. AutoRest treats the `default` response as defining an error 
response status code unless `default` is the only status code defined. The reason for imposing this convention is 
to produce higher quality API signatures. The return type of the generated API is determined by finding a common 
base type of the success responses. In practice, if the default is considered as a potential success definition, 
the common ancestor of success responses and error responses ends up being Object.

### Negative Responses<a name="NegativeResponses"></a>
You can describe all the [possible HTTP Response status codes](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) in the responses section of an operation. AutoRest generated code will deserialize the response for all the described response status codes as per their definition in the swagger. If a response status code is not defined then generated code will align to the default response behavior as described above.
- If **a schema is provided** for the negative response codes then this will have an impact on the return type of the generated method. 
  - For example: if a schema was provided for 200, and 400 was also described with a schema then,
    - the **return type** would be the Common Ancestor of both the schemas. In most cases there is nothing common between a positive and a negative response code. Hence the return type will be an `Object`. Note:This may not be very helpful to the customer
    - an exception **will NOT be thrown for 400** and the generated method will deserialize the response body as per the schema of "400".
    - any other negative response code will be treated as per the "default" response status code defined in the swagger for that operation.
- If **a schema is NOT provided** for the negative response codes then this will **NOT** have an impact on the return type of the generated method.
  - For example: if a schema was provided for 200 and 404 was described as one of the responses. However, 404 does not have a schema. In this scenario,
    - the **return type** of the generated method will be based upon the schema defined in "200".
    - an **exception will NOT be thrown** for 404 response status code
    - any other negative response code will be treated as per the "default" response status code defined in the swagger for that operation.
```json5
"/users/{userId}": {
  "get": {
    "operationId": "users_getUserById",
    "responses": {
      "200": {
       "description": "Provides User Information.",
        "schema": {
          "$ref": "#/definitions/user"
        }
      },
      "400": {
        "description": "Bad Request. ResponseBody will be deserialized as per the Error definition 
                        mentioned in schema. Exception will not be thrown.",
        "schema": {
          "$ref": "#/definitions/Error"
        }
      },
      "404": {
        "description": "Resource Not Found, ResponseBody will be null as there is no schema definition.
                        Exception will not be thrown.",
      },
      "default": {
        "description": "Default Response. It will be deserialized as per the Error definition 
                        specified in the schema. Exception will be thrown.",
        "schema": {
          "$ref": "#/definitions/Error"
        }
      }
    }
  }
}
```
### Custom Paths<a name="CustomPaths"></a>
Swagger 2.0 has a built-in limitation on paths. Only one operation can be mapped to a path and http method. There are some APIs, however, where multiple distinct operations are mapped to the same path and same http method. For example `GET /mypath/query-drive?op=file` and `GET /mypath/query-drive?op=folder` may return two different model types (stream in the first example and JSON model representing Folder in the second). Since Swagger does not treat query parameters as part of the path the above 2 operations may not co-exist in the standard "paths" element.

To overcome this limitation an "x-ms-paths" extension was introduced parallel to "paths". Urls under "x-ms-paths" are allowed to have query parameters for disambiguation, however they are removed during model parsing.

```json5
"paths":{
   "/pets": {
        "get": {
            "parameters": [
                {
                     "name": "name",
                     "required": true
                }
            ]
        }
   }
},
"x-ms-paths":{   
   "/pets?color={color}": {
        "get": {}
   },
}

```

Please note, that the use of "x-ms-paths" should be minimized to the above scenarios. Since any metadata inside the extension is not part of the default swagger specification, it will not be available to non-AutoRest tools.

## Defining Model Types<a name="DefiningModel"></a>
The request body and response definitions become simple model types in the generated code. The models include 
basic validation methods, but are generally stateless serialization definitions.

### Understanding the importance of "type" keyword while defining model types.
"Type-specific" keywords such as properties, items, minLength, etc. do not enforce a type on the schema. It works the other way around – when an instance is validated against a schema, these keywords only apply when the instance is of the corresponding type, otherwise they are ignored. Here's the relevant part of the [JSON Schema Validation](https://tools.ietf.org/html/draft-fge-json-schema-validation-00#section-4.1) spec:

> 4.1. Keywords and instance primitive types
Some validation keywords only apply to one or more primitive types. When the primitive type of the instance cannot be validated by a given keyword, validation for this keyword and instance SHOULD succeed.

For example, consider this schema:

```yaml
definitions:
  Something:
    properties:
      id:
        type: integer
    required: [id]
    minLength: 8
```
It's a valid schema, even though it combines object-specific keywords properties and required and string-specific keyword minLength. This schema means:
- If the instance is an object, it must have an integer property named id. For example, `{"id": 4}` and `{"id": -1, "foo": "bar"}` are valid, but `{}` and `{"id": "ACB123"}` are not.
- If the instance is a string, it must contain at least 8 characters. `"Hello, world!"` is valid, but `""` and `abc` are not.
- Any instances of other types are valid - `true`, `false`, `-1.234`, `[]`, `[1, 2, 3]`, `[1, "foo", true]`, etc.
(Except `null` - OpenAPI 2.0 does not have the `null` type and does not support `null` except in extension properties. OpenAPI 3.0 supports the `null` value for schemas with nullable: true.)

If there are tools that infer the `type` from other keywords (for example, handle schemas with no `type` but with `properties` as always objects), then these tools are not exactly following the OpenAPI Specification and JSON Schema.

> Bottom line: If a schema must always be an object, add `"type": "object"` explicitly. Otherwise you might get unexpected results.

**Credits** - Stack Overflow [link](https://stackoverflow.com/questions/47374980/schema-object-without-a-type-attribute-in-swagger-2-0).

### Model Inheritance<a name="ModelInheritance"></a>
Swagger schema allows for specifying that one type is `allOf` other types, meaning that the entire specification of 
the referenced schema applies is included in the new schema. By convention, if a schema has an 'allOf' that references 
only one other schema, AutoRest treats this reference as an indication that the `allOf` schema represents a base 
type being extended. In this example, the generated code would include a `Dog` model that inherits from the `Pet` model.

```json5
{
  "definitions": {
    "pet": {
      "description": "Defines the Pet model.",
      "properties": {
        "name": {
          "type": "string"
        },
      },
      "required": [
        "name"
      ]
    },
    "dog": {
      "description": "Defines the Dog model.",
      "required": [
        "breed"
      ],
      "properties": {
        "breed": {
          "type": "string"
        }
      },
      "allOf": [
        {
          "$ref": "#/definitions/pet"
        }
      ]
    }
  }
}
```

### Polymorphic Response Models<a name="PolymorphicResponse"></a>
Besides using `allOf` to define inheritance, model definitions can indicate that the payload will include a `discriminator` 
to disambiguate polymorphic payloads. The discriminator field allows the deserializer to resolve into an instance of a more 
specific type. Suppose the Dog and Cat type are `allOf` Pet and an operation will return a Dog or a Cat. If the Pet definition 
includes a `discriminator` then payloads can be Dog or Cat. A response can be defined as a Pet model and the API signature 
will indicate Pet. At runtime, the returned object is an instance of the more specific type.
```json5
{
  "definitions": {
    "Pet": {
      "type": "object",
      "discriminator": "petType",
      "properties": {
        "name": {
          "type": "string"
        },
        "petType": {
          "type": "string"
        }
      },
      "required": [
        "name",
        "petType"
      ]
    },
    "Cat": {
      "description": "A representation of a cat",
      "allOf": [
        {
          "$ref": "#/definitions/Pet"
        },
        {
          "type": "object",
          "properties": {
            "huntingSkill": {
              "type": "string",
              "description": "The measured skill for hunting",
              "default": "lazy",
              "enum": [
                "clueless",
                "lazy",
                "adventurous",
                "aggressive"
              ]
            }
          },
          "required": [
            "huntingSkill"
          ]
        }
      ]
    },
    "Dog": {
      "description": "A representation of a dog",
      "allOf": [
        {
          "$ref": "#/definitions/Pet"
        },
        {
          "type": "object",
          "properties": {
            "packSize": {
              "type": "integer",
              "format": "int32",
              "description": "the size of the pack the dog is from",
              "default": 0,
              "minimum": 0
            }
          },
          "required": [
            "packSize"
          ]
        }
      ]
    }
  }
}
```
