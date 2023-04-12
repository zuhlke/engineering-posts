---
title: RxJS Memory Leaks in Angular
domain: software-engineering-corner.hashnode.dev
tags: web-development, rxjs, form, bugs-and-errors
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/xqFCy9AbHP4/upload/a6072d56d7c313baa3d0eafda3ebbd24.jpeg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp
publishAs: tispBe
hideFromHashnodeCommunity: false
ignorePost: true
---

Memory leaks do not sound like something you want in your Angular application. Yet, nearly all applications using [RxJS Observables](https://rxjs.dev/guide/observable) face the challenge of mitigating this threat. You might already wonder, what these memory leaks are and how they can affect your application. And what is more, how can you prevent them?

This article describes RxJS Observable memory leaks and presents various techniques and patterns to tackle them in Angular applications.

### Memory Leaks in a Nutshell

In Angular web applications, memory leaks are caused by a mismanagement of [RxJs Observables](https://rxjs.dev/guide/observable). Thereby, allocated heap memory space is not freed up by the application when a component with an Observable is destroyed. This memory leak will result in performance issues, i.e. the page gets slower and slower. Higher load times negatively impact the user experience and increase the bounce rate [[1]](https://www.nngroup.com/articles/response-times-3-important-limits/). If a page load goes from 1s to 5s, then the bounce rate increases by 90% [[2]](https://www.thinkwithgoogle.com/consumer-insights/consumer-trends/mobile-page-speed-new-industry-benchmarks/). A page refresh will clear the heap memory again, and "reset" the memory leak. So, the longer an Angular component lives on your page, the higher the chance of experiencing a memory leak.

Luckily, we can easily prevent those memory leaks. But first, let us go into more detail about the origins of memory leaks.

### RxJs Observables

Let us start at the beginning - what are [RxJs Observables](https://rxjs.dev/guide/observable)? The _Reactive Extensions for Javascript_ (RxJs), is a library for reactive programming concerned with asynchronous data streams and the propagation of data changes. Thereby, you can consider a data stream as a _collection_ of data arriving and potentially changing over time, e.g. when fetching data from backend services using HTTP.

To that end, RxJS provides one core type - the _Observable_ - representing a collection of future values and events. Observables are lazy push-based systems, similar to JavaScript [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global%5FObjects/Promise). The consumer of the data is unaware of when it will receive the data. It is the data producer that determines when to "push" data to its consumers. Compared to Promises, however, an Observable can produce multiple values over time, i.e. a stream of data. This data stream can return zero to potentially infinite values from its invocation onward, synchronously or asynchronously.

![RxJs Observables](https://cdn.hashnode.com/res/hashnode/image/upload/v1681311720510/6W-qbqxuM.png?auto=compress align="center")

Observables are lazy computations, so, unless you _subscribe_ to them, no data will be transmitted. In Angular applications, we use Observables mostly to fetch data from backend services using HTTP, for routing purposes, and to respond to user events e.g. in forms.

**Please note**: There are Observables that auto-complete, i.e. don't need a specific unsubscribe. This is the case for Angular HttpClient or Angular Router Observables [[3]](https://lukaonik.medium.com/do-we-need-to-unsubscribe-http-client-in-angular-86d781522b99). However, we still recommend applying the same mechanisms to prevent memory leaks as with other Observables, ensuring consistency among the code.

### Subscriptions

Subscriptions are used to consume data provided by Observables. Subscribers - or rather data consumers - thereby, _subscribe_ to the Observables, which creates the subscription. Subscriptions are disposable resources that live in the heap memory. By _unsubscribing_ from an Observable, the resource is destroyed and the allocated heap memory is freed up again. Unsubscribing will also cancel the Observable execution.

### **Memory Leaks - A Detailed Example**

**A Memory Leak in the Wild - Example Setup**  
Let us look at a practical example. You are requested to implement an autocomplete search feature in your web application. The requirement states that after 1s of no new user input, a search in the backend should be triggered. The returned search results should be displayed below the search input.

![live search input](https://cdn.hashnode.com/res/hashnode/image/upload/v1681311768972/_YpoEOqLH.png?auto=compress align="center")

We use a simple [FormControl](https://angular.io/api/forms/FormControl) for the search input field and subscribe to the search term changes when the component is created, i.e. in the `ngOnInit` lifecycle hook. Our `SearchComponent` looks as follows:

```typescript
@Component({
  selector: "search-input",
  templateUrl: "./search.component.html",
  styleUrls: ["./search.component.scss"],
})
export class SearchComponent implements OnInit {
  searchCtrl: FormControl = new FormControl();

  ngOnInit() {
    this.searchCtrl.valueChanges.pipe(debounceTime(1000)).subscribe((searchTerm: string) => {
      this.searchService.updateSearchResults(searchTerm);
    });
  }
}
```

```xml
<mat-form-field appearance="fill">
  <mat-label>Search</mat-label>
  <input matInput [formControl]="searchCtrl" (focus)="onSearchFieldFocus()" />
</mat-form-field>
```

The `valueChanges` returns an Observable of type _any_. It emits an event when the value of the search control changes, so, with every new user keystroke. We use the [RxJs operator](https://rxjs.dev/api/operators/debounceTime) `debounceTime` to only emit a value when there is no user input for at least 1s. In our callback, we simply call the backend service to update the search results.

**Identifying the Memory Leak**  
Now, when we subscribe to the search input changes, a subscription is created in the heap memory. Currently, we do not unsubscribe from the `valueChanges` Observable.  
As soon as the `SearchComponent` is destroyed, the subscription is still active and not cleaned up by the garbage collector. This becomes a problem when the `SearchComponent` is created and destroyed frequently, e.g. when the search input is only displayed after clicking a search button before. In that case, more and more heap space is occupied by those obsolete `valueChanges` subscriptions. The browser will become slower and slower and the search feature unusable. We have a memory leak.

A page refresh would clear the heap memory and reset the problem state. Making it difficult to identify issues like that during development with frequent re-compilations and page refreshes.

Luckily there are various techniques to deal with memory leaks. In the following, we present the most popular ones. Additionally, we outline the extension of the linting rules to prevent memory leaks before they occur.

## Fixing Memory Leaks

There are various techniques to fix memory leaks. We abstain from presenting third-party packages that deal with this issue as oftentimes this is not an option in corporate projects.

Here are the most common ones.

1. **ngOnDestroy**
2. **Scalable ngOnDestroy**
3. **Mixin**

**1\. ngOnDestroy**  
A straightforward approach is to simply unsubscribe from all Observables within a component before it gets destroyed. It is recommended to do this in the `ngOnDestroy` lifecycle hook. To that end, we assign the subscription to a variable `searchTerm$` and unsubscribe from this subscription when the component gets destroyed.

```typescript
@Component({
  selector: 'app-search',
  templateUrl: './search.component.html',
  styleUrls: ['./search.component.scss'],
})
export class SearchComponent implements OnInit, OnDestroy {
  searchCtrl: FormControl = new FormControl();
  searchTerm$: Subscription;

  ngOnInit(): void {
    this.searchTerm$ = this.searchCtrl.valueChanges
      .pipe(debounceTime(1000))
      .subscribe((searchTerm: string) => {
        ...
      });
  }

  ngOnDestroy(): void {
    this.searchTerm$.unsubscribe();
  }
```

This works well for smaller components with only a few subscriptions. However, it does not scale well. Imagine an advanced search form with more than twenty different filters and inputs that you listen to using subscriptions. The `SearchComponent` would be overflowing with class-scoped subscription variables and unsubscriptions in the `ngOnDestroy`.

**2\. Scalable ngOnDestroy**  
We can easily deal with the scalability issue of the first approach by using the [RxJs `takeUntil` operator](https://www.google.com/search?client=safari&rls=en&q=rxjs+takeuntil&ie=UTF-8&oe=UTF-8). This operator allows us to automatically unsubscribe from the Observable at a given trigger. The trigger has to be an Observable itself.  
In our case, the trigger is the destruction of the component, i.e. we want to keep the subscription active _until_ the component is destroyed. The `ngOnDestroy` does not expose an Observable we could use in the `takeUntil` operator, thus, we have to create our own:

```typescript
@Component({
  selector: 'app-search',
  templateUrl: './search.component.html',
  styleUrls: ['./search.component.scss'],
})
export class SearchComponent implements OnInit, OnDestroy {
  searchCtrl: FormControl = new FormControl();
  private readonly destroy$ = new Subject<void>();

  ngOnInit(): void {
    this.searchCtrl.valueChanges
      .pipe(
        debounceTime(1000),
        takeUntil(this.destroy$)
      )
      .subscribe((searchTerm: string) => {
        ...
      });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
```

We create a new `destroy$` [Subject](https://rxjs.dev/guide/subject) acting as our trigger Observable. In all our subscriptions, we simply add the `takeUntil(this.destroy$)` operator. Lastly, we execute our trigger in the `ngOnDestroy` hook. All subscriptions are stopped and destroyed.

Note that `takeUntil` has to be the last operator in your pipe, otherwise, you risk a [takeUntil leak](https://ncjamieson.com/avoiding-takeuntil-leaks/). You can easily enforce this using the `no-unsafe-takeuntil` [eslint rule](https://github.com/cartant/eslint-plugin-rxjs/blob/main/docs/rules/no-unsafe-takeuntil.md).

**3\. Fancy a Mixin?**  
This approach targets code duplication in the `ngOnDestroy` hook. The two calls to `this.destroy$` will be similar for every class using observables. Thus, the idea is to extract and centralise this part of the `ngOnDestroy` hook.

To that end, we introduce a new global function - `WithDestroy`. This function extends a base class _T_ and adds the common `ngOnDestroy` hook with our two calls to the `destroy$` Subject.

```typescript
export function WithDestroy<T extends Constructor<{}>>(Base: T = class {} as never) {
  return class extends Base implements OnDestroy {
    public destroy$ = new Subject<void>();

    public ngOnDestroy(): void {
      this.destroy$.next();
      this.destroy$.complete();
    }
  };
}
```

Additionally, we need a new type `Constructor` that we add in a separate file.

```typescript
export type Constructor<T> = new (...args: any[]) => T;
```

This might look quite unusual, but it makes use of standard Typescript constructs. Our class `SearchComponent` can now be facilitated to the following:

```typescript
@Component({
  selector: 'app-search',
  templateUrl: './search.component.html',
  styleUrls: ['./search.component.scss'],
})
export class SearchComponent extends WithDestroy implements OnInit, OnDestroy {
  searchCtrl: FormControl = new FormControl();

  constructor() {
    super();
  }

  ngOnInit(): void {
    this.searchCtrl.valueChanges
      .pipe(
        debounceTime(1000),
        takeUntil(this.destroy$)
      )
      .subscribe((searchTerm: string) => {
        ...
      });
  }
```

We can completely remove the `ngOnDestroy` hook and `destroy$` Subject in our component. The only thing we have to do is extend the class with the newly added `WithDestroy` and add a constructor. Much cleaner than before, isn't it?

## Linting to Prevent Memory Leaks

To prevent memory leaks in our project and ensure that all subscriptions are terminated, we introduce new linting rules.

We recommend adding the following eslint plugins and rules:

- [eslint-plugin-rxjs](https://github.com/cartant/eslint-plugin-rxjs)
  - `no-unsafe-takeuntil` \- disallows operators after the `takeUntil` operator
- [rxjs-tslint-rules](https://github.com/cartant/rxjs-tslint-rules)
  - `rxjs-prefer-angular-takeuntil` - enforces the `takeUntil` operator when calling subscribe

## **Conclusion**

We showed how to identify memory leaks and how to fix them. Unfortunately, there is no integrated solution from RxJs to automatically unsubscribe Observables with component destruction. Let's hope that this will change at some point in the future.

[1] [Nielsen Norman Group, Response Times: The 3 Important Limits, State April 2023](https://www.nngroup.com/articles/response-times-3-important-limits/)  
[2] [Google Consumer Insights, State April 2023](https://www.thinkwithgoogle.com/consumer-insights/consumer-trends/mobile-page-speed-new-industry-benchmarks/)  
[3] [Luka Onikadze, Do we need to unsubscribe HTTP client in Angular?, State April 2023](https://lukaonik.medium.com/do-we-need-to-unsubscribe-http-client-in-angular-86d781522b99)
