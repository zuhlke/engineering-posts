---
title: Effortless API-first 
subtitle: Part one of our API-first series 🛠️
slug: api-first-development
tags: api, rest-api, openapi, gradle, api-first
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690024103909/5Im8zw6FL.jpg?auto=format
domain: software-engineering-corner.hashnode.dev
ignorePost: true
hideFromHashnodeCommunity: false
publishAs: romanutti
---

The software development world is an ever-evolving landscape. Technologies and methodologies continuously evolve - each promising a more efficient and effective way to create high-quality software. 
One such approach that is gaining significant traction is *API-first* development.

In this article series we will explore the idea behind API-first, look at practicable examples and [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)'s new multiple specification files feature.

> Note: Release 7.0.1 that will fix [issue 16419](https://github.com/OpenAPITools/openapi-generator/issues/16419) is expected to be released later this year. Until then multiple specification files are only supported by the CLI version.

### What does API-first mean?

API-first design is a development paradigm that prioritizes our APIs, or application programming interfaces, and how our different software pieces communicate. In this approach, APIs are designed before any line of code is written.

[OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0) is used to outline how the API should act. It is a standard that allows us to define endpoints, operations, parameters, error messages, and other information in a way that both humans and computers understand.

A fully-fledged example of an OpenAPI specification can be found [here](https://editor.swagger.io/).

### The benefits of API-first development

API-first comes with a lot of benefits. Here are some of them:

* __Better APIs__: A good API fulfills its clients' needs, is easy to use and produces maintainable code. Thinking about the API first helps you come up with the API you want - rather than one your code accidentally generates.

* __Development teams can work in parallel__: As soon as the API specification is defined, both server and clients can start their implementation work in parallel.

* __Code-generation__: Client and server code can be generated from the API specification.

* __Integration tested__: Having the API defined first improves the chances that our client talks correctly to our server. Generating code and building your development and build processes around API-first improves the chances even more.

### The tools

Appropriate tooling is essential for API-first development. The following two will make our life easier:

#### Swagger Editor

The [Swagger Editor](https://editor.swagger.io/) is a browser-based editor where we can write OpenAPI specifications. It provides a live preview of the API documentation and allows us to generate server and client code in various languages. If you checked out the OpenAPI example before, you already have seen it in action.

#### OpenAPI Generator

A commonly used tool to generate code based on openAPI specifications is the [OpenAPI Generator](https://openapi-generator.tech/). It comes in different ways: As [CLI](https://central.sonatype.com/artifact/org.openapitools/openapi-generator-cli/7.0.0), [Maven plugin](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator-maven-plugin/README.md) or [Gradle plugin](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator-gradle-plugin).

### A first example

To give an example of how API-first can be used we will create a simple API and generate the required code.
This example uses version `7.0.1` of the [Gradle plugin](https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator-gradle-plugin).

As described earlier, we start with designing our API. In this first step, we define an endpoint that returns the details of a specific user. The specification is written in YAML and can be found in the `openapi` directory.

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
          description: User's name.
      required:
        - id
        - username
```

The specification consists of the following elements:
- **`openapi`**: Defines the version of the OpenAPI specification being used.
- **`info`**: Version, name and a brief description of the API.
- **`paths`**: Exposes all available endpoints - in our case a `GET` endpoint to retrieve user details, including supported operations, parameters and possible responses.
- **`components`**: Used to define reusable data models. Currently, we only have a `User` object.

So far, so clear.

Having our OpenAPI specification we can now generate the required code. To do so, we need to add the following plugin to our `build.gradle.kts` file:

```kotlin
plugins {
    id("org.openapi.generator") version "7.0.1"
}
```

### Generating the server

The plugin now allows us to specify further how the code should be generated. 
In this example, we want to generate a [Spring](https://spring.io/) server. Therefore, we need to add the following task to our `build.gradle.kts` file:

```kotlin
openApiGenerate {
    generatorName.set("spring")
    inputSpec.set("$rootDir/openapi/user-api.yaml")
    configFile.set("$rootDir/api-config.json")
    outputDir.set("$buildDir/generated")
}
```

Let's see what we configured: 
* `generatorName` specifies the generator to use. 
* `inputSpec` contains the OpenAPI specification that we use to generate our code.
* `configFile` allows to further customize the code generation (checkout the [docs](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator-gradle-plugin/README.adoc#configuration) to learn more about the available options).
* `outputDir` specifies the directory where the generated code should be placed.

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

And also the data transfer object (DTO) that is returned by the server was generated:

```java
@Generated
public class User {

  private Integer id;

  private String username;
  
  /**
   * Constructor with only required parameters
   */
  public User(Integer id, String username) {
    this.id = id;
    this.username = username;
  }

  public User id(Integer id) {
    this.id = id;
    return this;
  }

  /**
   * Unique identifier for the user.
   * @return id
  */
  @NotNull 
  @Schema(name = "id", description = "Unique identifier for the user.", requiredMode = Schema.RequiredMode.REQUIRED)
  @JsonProperty("id")
  public Integer getId() {
    return id;
  }

  public void setId(Integer id) {
    this.id = id;
  }

  public User username(String username) {
    this.username = username;
    return this;
  }

  /**
   * User's name.
   * @return username
  */
  @NotNull 
  @Schema(name = "username", description = "User's name.", requiredMode = Schema.RequiredMode.REQUIRED)
  @JsonProperty("username")
  public String getUsername() {
    return username;
  }

  public void setUsername(String username) {
    this.username = username;
  }
}

```

### Generating the client

Same as for the server, we can generate a client by using a suitable generator.
In our case we want to generate a [TypeScript](https://www.typescriptlang.org/) client. 
Therefore, we choose the `typescript-angular` generator in our gradle task:

```kotlin
openApiGenerate {
    generatorName.set("angular-typescript")
    inputSpec.set("$rootDir/openapi/user-api.yaml")
    configFile.set("$rootDir/api-config.json")
    outputDir.set("$buildDir/generated")
}
```

The generator will generate a service that can be used to call the API. 
As the client code in our case includes the fully-fledged implementation of the service, we can directly use it in our application.


```typescript
/**
 * NOTE: This class is auto generated by OpenAPI Generator (https://openapi-generator.tech).
 * https://openapi-generator.tech
 * Do not edit the class manually.
 */
@Injectable({
    providedIn: 'root'
})
export class UserService {

    constructor(protected httpClient: HttpClient, @Optional()@Inject(BASE_PATH) basePath: string|string[], @Optional() configuration: Configuration) {
        // ...
    }

    /**
     * Get details of a specific user by ID
     * @param userId ID of the user to retrieve
     * @param observe set whether or not to return the data Observable as the body, response or events. defaults to returning the body.
     * @param reportProgress flag to report request and response progress.
     */
    public usersUserIdGet(userId: number, observe: any = 'body', reportProgress: boolean = false, options?: {httpHeaderAccept?: 'application/json', context?: HttpContext}): Observable<any> {
        // ...
    }
}
```

And also corresponding data transfer objects were generated:

```typescript
/**
 * NOTE: This class is auto generated by OpenAPI Generator (https://openapi-generator.tech).
 * https://openapi-generator.tech
 * Do not edit the class manually.
 */
export interface User { 
    /**
     * Unique identifier for the user.
     */
    id: number;
    /**
     * User's name.
     */
    username: string;
}
```

Bonus tip on organizing code: Keep your generated code in a separate directory or module to keep it isolated from your application code. This will make it easier to manage and update your generated code as your API evolves over time.

### Conclusion

As we can see, the OpenAPI Generator is a powerful tool that can be used to generate code for both server and client side.
In the next article we will look at a more realistic example and how [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator) helps you to keep your specification files structured.