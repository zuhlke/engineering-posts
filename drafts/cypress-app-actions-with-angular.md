---
title: Cypress app actions with Angular
domain: campzulu.hashnode.dev
tags: web-development, frontend-development, angular, testing, cypress
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/_zKxPsGOGKg/upload/v1666550567525/TYz2JBy-V.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp
publishAs: culas
ignorePost: true
---

It was no surprise when Angular announced last August [on their official blog](https://blog.angular.io/the-state-of-end-to-end-testing-with-angular-d175f751cb9c) that they've decided to deprecate Protractor.
They've been public about evaluating the future of Protractor.
Version 12 of Angular already added support for other e2e testing frameworks.
As a result, we migrated the existing Protractor tests at one of our customers to Cypress.
We used this as a chance to improve the tests themselves.
One way to do this was using Cypress app actions.

# The Problem

Due to the nature of the application, many tests required multiple steps to reach the desired start conditions.
The existing tests would go through the same parts of the UI again and again.
That was slow and often the source of flakiness on the CI.

We first tried using `cy.request()` directly to set up and manipulate test data.
Sadly, for a couple of reasons, this turned into more effort than we were willing to put in.
Especially since our Angular app already knew how to handle all communication with the API.

# App Actions

In a [blog post from 2019](https://www.cypress.io/blog/2019/01/03/stop-using-page-objects-and-start-using-app-actions/#application-actions), Cypress engineer Gleb Bahmutov explained the concept of "application actions".
Cypress ambassador Filip Hric also went into [the difference between app actions and page objects](https://applitools.com/blog/page-objects-app-actions-cypress/).¨
They leverage the fact that Cypress runs inside the browser and therefore has access to the application.
That allows us to skip interacting with sections of the UI that aren't part of the test and directly manipulate the state.
The command `window` gives us access to the window object.

```ts
cy.window()
    .then(({app}) => app.sayTheLineBart());
```

# Angular

First, we needed to make the desired service accessible on the window object.
We decided against overusing app actions for the time being.
The interface, respectively dependency, between Cypress and our Angular app should be kept small.
As we had a `BackendService` that centrally handled all requests to our API, it was an obvious choice.
In the constructor, we verify whether the app is running inside Cypress and expose the service.

`src/app/.../backend.service.ts`:
```typescript
export class BackendService {
    constructor() {
        if (window.hasOwnProperty('Cypress')) {
            window.app4cy = window.app4cy ?? {};
            window.app4cy.BackendService = this;
        }
    }

    public get<T>(url: string): Promise<T> {
        // ...
    }

    // ...
}
```

# Abstraction through Commands

<!--
    - add Command and Utils function
    - reasoning
-->

The action should be easy to use in Cypress, so we created a custom command: `backendRequest`.
This way, we keep the dependency on the app in one central place and abstract the app action.

`cypress/support/commands.ts`:
```typescript
Cypress.Command.add('backendRequest',
    {prevSubject: false},
    <M extends keyof BackendService>(method: M, ...args: Parameters<BackendService[M]>) =>
    cy.log(`Use BackendService for a ${method.toUpperCase()} request to '${args[0]}'`)
        .window({log: false})
        .its('app4cy.BackendService', {log: false})
        // @ts-ignore (TS compiler sadly struggles with the relationship between the types of "method" and "args")
        .then(async (service: BackendService) => service[method](..args));
);
```

That also allowed us to specify the types of parameters and return values.

`cypress/support/commands.ts`:
```typescript
declare global {
    namespace Cypress {
        interface Chainable<Subject> {
            backendRequest<M extends keyof BackendService>(
                method: M,
                ...args: Parameters<BackendService[M]>
            ): Chainable<ReturnType<BackendService[M]>>;
        }
    }
}
```

We then went even one step further.
Repeated actions, such as creating a specific item, were put into a utility function.
One could argue this was unnecessary.

`cypress/support/utils.ts`:
```typescript
export function createNewItem(itemName: string): void {
    const urlPath = 'items';
    cy.backendRequest('post', urlPath, itemName);
}
```

# Conclusion

Once we set this app action up, the affected tests greatly improved stability, readability and speed.
Using app actions halved the duration of some tests.
By choosing to expose only one central service, we kept the impact on the Angular app itself minimal.
In another app, it might make sense to use more specified services or components to manipulate the state.
Our experiences with this setup are very positive.
I would recommend looking into making use of the potential of Cypress if you have similar problems.
The setup is straightforward.

