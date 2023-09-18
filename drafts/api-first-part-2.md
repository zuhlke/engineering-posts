---
title: Multi-File OpenAPI Specifications
subtitle: Part two of our API-first series ✨
tags: api, rest, openapi, gradle, api-first, rest-api, apis
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1693750140935/5qa-wBJUA.jpg?auto=format
domain: software-engineering-corner.hashnode.dev
ignorePost: true
hideFromHashnodeCommunity: false
publishAs: romanutti
---

The API-first approach prioritizes APIs over code and puts them at the beginning of the software development process.
Our previous article looked at the idea behind API-first development and some first examples.
This article will focus on how [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)'s new multiple specification files feature lets us organize our APIs.

> Note: Release 7.0.1 that will fix [issue 16419](https://github.com/OpenAPITools/openapi-generator/issues/16419) and allows multiple specification files is expected to be released later this month 🔥 Until then, the feature is only supported by the CLI version.

### Multiple specifications

In the real world, our APIs grow over time. Having more than one file to define all endpoints and data models can become quite messy. Therefore, we want to split our API specification into multiple files.
Thankfully, the new `inputSpecRootDirectory` option allows us to specify a directory containing multiple OpenAPI specification files. The plugin will then merge all files into one specification and generate the code based on that.

```kotlin
openApiGenerate {
    generatorName.set("spring") 
    inputSpecRootDirectory.set("$rootDir/openapi")
    configFile.set("$rootDir/api-config.json")
    outputDir.set("$buildDir/generated")
}
```

In the last article, we created a simple API specification file `user-api.yaml` for a user management API.
Imagine we additionally have to build an API that lists a specific order. Again, we can create an API specification file `order-api.yaml`.

_openapi/order-api.yaml:_
```yaml
openapi: 3.0.0
info:
  version: 1.0.0
  title: Order API
  description: API to manage orders
paths:
  /orders/{orderId}:
    get:
      summary: Get details of a specific order by ID
      tags:
        - orders
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

If we again run the code generation (`./gradlew openApiGenerate`), we will see that the generator created a merged file referencing both specifications.

_openapi/api.yaml:_
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
The OpenAPI specification allows us to reference external contents using the `$ref` attribute.
In our example `./order-api.yaml#/~1order` means we 
1. take the `order-api.yaml` file, 
2. read the contents of the `paths/orders` node in that file (`~1` escapes `/`, see [JSON Pointer](https://swagger.io/docs/specification/using-ref/) rules) and 
3. substitute the `$ref` with that contents.

### Don't repeat yourself

In a scenario like this, we will likely face a situation where we want to share a model between API specifications.
Let's assume we want to return the user in
* our user API to get the user details via `/users/{userId}` and in
* order API to the user that placed an order via `/users/{orderId}`.

Again, `$ref` is our friend: We can define a common data model in a separate file and reference it in both API specifications.

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
          description: User's name.
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
      tags:
        - users
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

And the order API can use the shared data model, too.

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
      tags:
        - orders
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

Using API-first to build software can benefit your organization in many ways. 
In this article series, we looked at what API-first means and how OpenAPI Generator can help you to adopt this approach.
However, remember that more than just using and adopting the tools is required. The API-first approach requires embracing a new process - and even more crucial: Shifting from thinking in code to a code-agnostic way to design your APIs.