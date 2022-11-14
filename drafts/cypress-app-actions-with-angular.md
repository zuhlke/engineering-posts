---
title: Cypress app actions with Angular
domain: campzulu.hashnode.dev
tags: web-development, frontend-development, angular, testing, cypress
cover: https://cdn.hashnode.com/res/hashnode/image/unsplash/_zKxPsGOGKg/upload/v1666550567525/TYz2JBy-V.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp
publishAs: culas
ignorePost: true
---

It was no surprise when Angular announced last August [on their official blog](https://blog.angular.io/the-state-of-end-to-end-testing-with-angular-d175f751cb9c) that they've decided to deprecate Protractor.
They've been public about evaluating the future of Protractor and version 12 of Angular already added support for other e2e testing frameworks.
At one of our customers, we therefore migrated the existing Protractor tests to Cypress.
We used this as a chance to improve the tests themselves.
One way to do this were Cypress app actions.

# The Problem

<!--
    - repeating actions going through ui 
    - slow, brittle, many requests
    - due to structure of application and general testing strategy
    - different ports/apps, CORS handling
-->

# App Actions

<!--
    - link to Cypress best practices about app actions
    - explanation of purpose and functionality
-->

# Angular

<!--
    - how to setup in Angular
    - different ways
    - "backendService" example
-->

`src/app/.../backend.service.ts`:
```typescript
@Injectable()
export class BackendService {
    constructor() {
        if (window.hasOwnProperty('Cypress')) {
            window.app4cy = window.app4cy ?? {};
            window.app4cy.BackendService = this;
        }
    }

    public get<T>(url: string): Observable<T> {
        return this.http.get<T>(url);
    }

    // ...
}
```

# Abstraction through Commands

<!--
    - add Command and Utils function
    - reasoning
-->

`cypress/support/commands.ts`:
```typescript
Cypress.Command.add('backendRequest',
    {prevSubject: false},
    <M extends keyof BackendService>(method: M, ...args: Parameters<BackendService[M]>) =>
    cy.log(`Use BackendService for a ${method.toUpperCase()} request to '${args[0]}'`)
        .window({log: false})
        .its('app4cy.BackendService', {log: false})
        // @ts-ignore (TS compiler sadly struggles with the relationship between "method" and "args")
        .then(async (service: BackendService) => firstValueFrom(service[method](..args)))
);
```

`cypress/support/commands.ts`:
```typescript
declare global {
    namespace Cypress {
        /**
         * Make a request to API via the Angular app's `BackendService`.
         * ...
         */
        interface Chainable<Subject> {
            backendRequest<M extends keyof BackendService>(
                method: M,
                ...args: Parameters<BackendService[M]>
            ): Chainable<ReturnType<BackendService[M]>>;
        }
    }
}
```

`cypress/support/utils.ts`:
```typescript
export function createNewItem(itemName: string): void {
    const urlPath = 'items';
    cy.backendRequest('post', urlPath, itemName);
}
```

# Conclusion