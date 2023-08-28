---
title: Effortless API-first 
subtitle: Using OpenAPI Generator's latest multiple spec-file feature ✨
slug: api-first-development
tags: api, rest-api, openapi, gradle, api-first
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690024103909/5Im8zw6FL.jpg?auto=format
domain: software-engineering-corner.hashnode.dev
ignorePost: true
hideFromHashnodeCommunity: false
publishAs: romanutti
---

The software development world is an ever-evolving landscape. Technologies and methodologies continuously evolve - each promising a more efficient and effective way to create high-quality software. One such approach that is gaining significant traction is *API-first* development.
This article describes the API-first approach and how the latest [openapi-generator](https://github.com/OpenAPITools/openapi-generator) finally supports multiple specification files.

### What does API-first mean?

API-first design is a development paradigm that prioritizes your APIs (Application Programming Interfaces) and how your different software pieces communicate. In this approach, APIs are designed before any line of code is written.

[OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0) is used to outline how the API should act. It is a standard that allows us to define endpoints, operations, parameters, error messages, and other information in a way that both humans and computers understand.

A fully-fledged example of an OpenAPI specification can be found [here](https://editor.swagger.io/).

### The benefits of API-first development

API-first comes with a lot of benefits. Here are some of them:

* __Better APIs__: A good API fulfills its clients' needs, is easy to use and produces maintainable code. Thinking about the API first helps you come up with the API you want - rather than one your code accidentally generates.

* __Development teams can work in parallel__: As soon as the API specification is defined, both (or even all, if multiple clients) can start their implementation work in parallel.

* __Code-generation__: Client and server code can be generated from the API specification.

* __Integration tested__: Having the API defined first improves the chances that your client talks correctly to your server. Generating code and building your development and build processes around API-first improves the chances even more.

### The tools

#### Swagger Editor

The [Swagger Editor](https://editor.swagger.io/) is a browser-based editor where you can write OpenAPI specifications. It provides a live preview of the API documentation and allows you to generate server and client code in various languages. If you checked out the OpenAPI example before, you already have seen it in action.

#### OpenAPI Generator

A commonly used tool to generate code based on openAPI specifications is the [openapi-generator](https://openapi-generator.tech/). It comes in different ways: As [CLI](https://central.sonatype.com/artifact/org.openapitools/openapi-generator-cli/7.0.0), [Maven plugin](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator-maven-plugin/README.md) or [Gradle plugin](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator-gradle-plugin).

### A first example

To give you an example of how API-first can be used we will create a simple API and generate the required code.
This example uses the latest version (7.0.1) of the Gradle plugin.

As described earlier, we started designing our API. In this first step, we want to define an endpoint that returns the details of a specific user. The specification is written in YAML and can be found in the `openapi` directory.

_openapi/user-api.yaml:_

```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: User API
  description: API to manage users
paths:
  /users/{userId}:
    get:
      summary: Get details of a specific user by ID
      parameters:
        - name: userId
          in: path
          description: ID of the user to retrieve
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          description: Unique identifier for the user.
        username:
          type: string
          description: User's username.
      required:
        - id
        - username
```

The specification consists of the following elements:
- **`openapi`**: Specifies the version of the OpenAPI specification being used.
- **`info`**: Version, name and a brief description of the API.
- **`/users/{userId}`**: Represents an endpoint to retrieve user details, supported operations (`get`), `parameters` and possible `responses`.
- **`components`**: Used to define reusable data models - in this case a `User` object.

So far, so clear.

Having our OpenAPI specification we can now generate the required code. To do so, we need to add the following to our `build.gradle.kts` file:

```kotlin
plugins {
    id("org.openapi.generator") version "7.0.1"
}
```

> Note: Release 7.0.1 that will fix https://github.com/OpenAPITools/openapi-generator/issues/16419 is expected to be released later this year. Until then multiple spec-files are only supported by the CLI version.

The plugin now allows you to specify further how the code should be generated. In this example, we want to generate a Spring server. Therefore, we need to add the following to our `build.gradle.kts` file:

```kotlin
openApiGenerate {
    generatorName.set("spring")
    inputSpec.set("$rootDir/openapi/user-api.yaml")
    outputDir.set("$buildDir/generated")
}
```

Let's see what we configured: 
* `generatorName` specifies the generator to use. 
* `inputSpec` contains the OpenAPI specification. 
`outputDir` specifies the directory where the generated code should be placed.

We are now ready to run the code generation. To do so, we can run the following command:

```bash
./gradlew openApiGenerate
```

This will result in an interface that can be implemented in the respective controller on sever side.

```java
/**
 * NOTE: This class is auto generated by OpenAPI Generator (https://openapi-generator.tech) (7.0.0-beta).
 * https://openapi-generator.tech
 * Do not edit the class manually.
 */
@Generated
public interface UsersApi {

    /**
     * GET /users/{userId} : Get details of a specific user by ID
     *
     * @param userId ID of the user to retrieve (required)
     * @return OK (status code 200)
     */
    @Operation(
            operationId = "usersUserIdGet",
            summary = "Get details of a specific user by ID",
            responses = {
                    @ApiResponse(responseCode = "200", description = "OK", content = {
                            @Content(mediaType = "application/json", schema = @Schema(implementation = User.class))
                    })
            }
    )
    @RequestMapping(
            method = RequestMethod.GET,
            value = "/users/{userId}",
            produces = { "application/json" }
    )
    default ResponseEntity<User> usersUserIdGet(
            @Parameter(name = "userId", description = "ID of the user to retrieve", required = true, in = ParameterIn.PATH) @PathVariable("userId") Integer userId
    ) {
        return new ResponseEntity<>(HttpStatus.NOT_IMPLEMENTED);
    }
}
```

The OpenAPI generator provides a lot of options to customize the generated code. For example, you can specify the package name, the name of the generated classes, custom type-mappings, and of course, other generators such as `javascript`, `typescript`, `kotlin` or `go`. For a complete list of options, please refer to the [documentation](https://github.com/OpenAPITools/openapi-generator)

### A more realistic example

In the real world, our APIs grow over time. Having more than one file to define all endpoints and data models can become quite messy. Therefore, we want to split our API specification into multiple files.
Thankfully, a new `inputSpecRootDirectory` option allows us to specify a directory containing multiple OpenAPI specification files. The plugin will then merge all files into one specification and generate the code based on that.

```kotlin
openApiGenerate {
    generatorName.set("spring") 
    inputSpecRootDirectory.set("$rootDir/openapi")
    outputDir.set("$buildDir/generated")
}
```

Imagine we additionally have to build an API that lists a specific order. Again, We can create an API specification file `openapi/oder-api.yaml`.

_openapi/order-api.yaml:_

```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: User API
  description: API to manage orders
paths:
  /orders/{orderId}:
    get:
      summary: Get details of a specific order by ID
      parameters:
        - name: orderId
          in: path
          description: ID of the order to retrieve
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
components:
  schemas:
    Order:
      type: object
      properties:
        id:
          type: integer
          description: Unique identifier for the order.
        comment:
          type: string
          description: Order note.
      required:
        - id
        - comment
```

If we again run the code generation, we will see that the generator created a merged file referencing both specifications.

```yaml
openapi: "3.0.0"
paths:
  /orders:
    $ref: "./order-api.yaml#/paths/~1orders"
  /users:
    $ref: "./user-api.yaml#/paths/~1users"
info:
  title: "Merged openAPI spec"
  version: "1.0.0"
```

### Extra: Reusing data models

In a scenario like this, you will most likely face the situation where you want to share a model between API specifications.
For example, let's assume we want to return the ordering user in our `/users/{orderId}` endpoint.
Thankfully OpenAPI specification allows us to reference an external schema using the `$ref` attribute (the one we saw just before).

That way, we can define a common data model in a separate file and reference it in both API specifications.

_openapi/common.yaml:_

```yaml
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          description: Unique identifier for the user.
        username:
          type: string
          description: User's username.
      required:
        - id
        - username
```

Our user API references now that common data model.

_openapi/user-api.yaml:_
```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: User API
paths:
  /users/{userId}:
    get:
      summary: Get details of a specific user by ID
      parameters:
        - name: userId
          description: ID of the user to retrieve
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: 'common.yaml#/components/schemas/User'
```

And also the order API can make use of the shared data model.

_openapi/order-api.yaml:_

```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: Order API
paths:
  /users/{orderId}:
    get:
      summary: Get details of a specific user by order ID
      parameters:
        - name: orderId
          description: ID of the order that placed the order
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: 'common.yaml#/components/schemas/User'
```

### APIs as "first-class" citizens

Using API-first as an approach to build software can benefit your organization in many ways. However, remember that more than just using and adopting the tools is required. The API-first approach requires embracing a new process - and even more crucial: Shifting from thinking in code to a code-agnostic way to design your APIs.