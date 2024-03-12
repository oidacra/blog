---
title: "Implementing selectSignal in Component-Store with Angular's New Template Syntax"
seoTitle: "SelectSignal: Component-Store, Angular Template"
datePublished: Sat Jan 06 2024 03:14:32 GMT+0000 (Coordinated Universal Time)
cuid: clr1hrin9000009lbdlin7n4i
slug: implementing-selectsignal-in-component-store-with-angulars-new-template-syntax
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/HpMihL323k0/upload/b8147ea0052780f9640579b348bca705.jpeg
tags: angular, ngrx, component-store

---

In this article, we'll explore more about `component-store` with Angular. Instead of using the usual `selectors`, we'll use `selectSignal` from `component-store`. At the same time, we'll update our template to include the advanced template syntax introduced in Angular 17.

## Why?

We'll use `selectSignal` instead of `select()` because it helps show view-related changes in the store and includes the new Angular template syntax. Some advantages we get are:

* Eliminate the use of the `async` pipe
    
* Simplify the template with `@if` to manage the loading state
    
* Use `@for` and `@empty` to display the options
    
* Avoid overusing `ng-container` and `ng-template` to handle the `async` pipe and display different templates
    

## What we'll change

### First, the store,

```typescript
// series.store.ts
// from normal selectors
  readonly series$ = this.select((state) => state.series);
  readonly isLoading$ = this.select((state) => state.state === 'loading');

// to use selectSignal
  readonly seriesSignal: Signal<Serie[]> = this.selectSignal(
    (state) => state.series
  );
  readonly isLoadingSignal: Signal<boolean> = this.selectSignal(
    (state) => state.state === 'loading'
  );
```

By using `selectSignal`, it retrieves a Signal from the store not an observable.

### Second, the component

```typescript
// series.component.ts
// viewModel from the store removed
readonly vm$: Observable<ViewModelComponent> = this.store.vm$;

// converted to selectSignal()
readonly vm: Signal<ViewModelComponent> = this.store.vm

// viewModel replaced
```

We now have the same View Model, but it's a Signal instead.

### Third, the view,

```xml
<!--series.components.html -->
  <h1>Series</h1>
  @if (vm().isLoading){
    <p>Loading...</p>
  }
  @else{
    <ul>
        @for (item of vm().series; track item.id;) {
            <li>{{item.name}}</li>
        } @empty {
            <li>No series found</li>
        }
    </ul>
   }
```

We used the signal to show the information, and also, made use of the new Angular Template Syntax.

## Conclusion

The new features in Angular 16 (Signals) and Angular 17 (New template syntax) make displaying information in Angular much simpler. These changes create a new way of handling things compared to before.

Right now, I think using observables with RxJs for handling data and requests is the best approach. For things related to the view, now I prefer using Signals.

You can see all the changes I made in the code by checking out this [PR](https://github.com/oidacra/series-post/pull/2/files). It lets you compare what we changed.

Do you have any experiences with ComponentStore to share? Or any questions about what we discussed? I'd love to hear your thoughts and talk about them in the comments below.