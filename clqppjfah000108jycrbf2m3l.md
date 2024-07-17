---
title: "A Practical Guide to NgRx ComponentStore: Managing Local/Component States"
seoTitle: "NgRx ComponentStore: Efficient Local State"
seoDescription: "Master NgRx ComponentStore with this practical guide: learn basics, best practices, real-world examples, and enhance app consistency and performance"
datePublished: Thu Dec 28 2023 21:18:57 GMT+0000 (Coordinated Universal Time)
cuid: clqppjfah000108jycrbf2m3l
slug: a-practical-guide-to-ngrx-componentstore-managing-localcomponent-states
canonical: https://medium.com/@oidacra/a-practical-guide-to-ngrx-componentstore-managing-local-component-states-ceee619cba2c
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/NIJuEQw0RKg/upload/cf5028f82c3e2853d70f6b9df8836e0e.jpeg
tags: angular, rxjs, ngrx, component-store

---

In this series about `ComponentStore`, I'll share my experiences using it for about a year. We'll start with the basics of what it is and how to use it, eventually diving deeper into patterns and best practices.

The official definition of `ComponentStore` from [NgRx's official site](https://ngrx.io/guide/component-store):

> ComponentStore is a stand-alone library that helps to manage local/component state. It's an alternative to reactive push-based "Service with a Subject" approach.

To put it in simpler terms: `ComponentStore` is a dedicated library designed to manage local/component states. It grants us a consistent methodology to initialize, read, update, and tackle side effects within our store.

I will create a real example in [GitHub](https://github.com/oidacra/series-post) with all the code running, we will create an easy app for Browse TV Series, using [https://www.tvmaze.com](https://www.tvmaze.com) \[[Api Doc](https://www.tvmaze.com/api)\].

## What gives to us ComponentStore

The core idea behind a store is to serve as a single source of truth—a centralized location where data is always accurate and up-to-date. This ensures consistency throughout our application.

With this perspective, `ComponentStore` offers methods to **initialize** the store (which can be done lazily), **update** it, replace its contents(**patch**), and manage side **effects**."

To summarize, `ComponentStore` allow us to use these methods:

* `setState()`
    
* `patchState()`
    
* `select()`
    
* `updater()`
    
* `effect()`
    

Additionally, it has a few [lifecycle hooks](https://ngrx.io/guide/component-store/lifecycle) to do tasks after the ComponentStore is instantiated.

All right, let's get down to business. [**This is the repo if you want to follow all**.](https://github.com/oidacra/series-post)

### Initialization of the store

* We can do this through the constructor of the store (my favorite way),
    

```typescript
// series.store.ts
export interface SeriesState  {
  series: Serie[];
  selectedId: number;
}

@Injectable()
export class SeriesStore extends ComponentStore<SeriesState> {
    constructor() {
        super({series: []}); // <--- Initialization
    }
}
```

* Or using `setState` (the lazy way from the component)
    

```typescript
// series.component.ts

  ngOnInit() {
    this.store.setState({ series: [] }); // <--- Initialization
  }
}
```

### Read from the store

To obtain or read a value we use the `select` method to create a `selector` to use in our components:

```typescript
// series.store.ts

export interface SeriesState  {
  series: Serie[]; // <- the selector get this slice
  selectedId: number;
}

@Injectable()
export class SeriesStore extends ComponentStore<SeriesState> {
  // we are reading an array of series
  readonly series$ = this.select((state) => state.series);

  constructor() {
    super(initialState);
  }
}
```

It can be combined with other selectors or observables (really useful to create view model)

```typescript
// series.store.ts
// code removed ...

// observable created with the array of series    
readonly series$ = this.select((state) => state.series);
// observable created with the selectedId    
readonly selectedSerieId$ = this.select((state) => state.selectedId);

// new selector with a combination of observables
readonly selectedSerie$ = this.select(
  this.series$,
  this.selectedSerieId$,
  (series, id) => series.find((serie) => serie.id === id)
);

// code removed ...
```

Starting with Angular 16, we can use `signal`, and `component-store` allows us to convert from observable to signals using `selectSignal` as selector. To explore how to use it, take a look at the [branch](https://github.com/oidacra/series-post/tree/use-signal-with-component-store) `use-signal-with-component-store`. We will employ this approach in upcoming posts.

### Update the store

We need a way to update the store, `ComponentStore` gives us 3 ways to do that, using `setState`, `patchState` or creating an `updater` (one of my favorite ways):

* Using `setState()`, this method is for completely replacing the state or applying a function that operates on the previous state and returns a new state.
    

```typescript
this.setState(state => ({ ...state, data: newData, loading: false }));
```

* `patchState()`, partial update of the store, it's useful when you only need to change one or a few properties of the state without having to touch the rest of the state.
    

```typescript
this.patchState({ state: 'loading' });
```

* `update()`, provides a way to update the store by sending an input value to update the store.
    

```typescript
// definition of the updater
readonly selectedSerieId = this.updater((state, selectedId: number) => {
    return { ...state, selectedId };
  });

// use of the updater in the component
selectedMovie(selectedId: number) {
  this.store.selectedSerieId(selectedId); // <- the updater act as a method with one parameter
}
```

### Effects

Effects in `ComponentStore` are tools that separate actions with side effects (such as HTTP calls) from components, allowing components to focus only on displaying data and triggering updates.

```typescript
// Updater: add a serie to the store
  readonly addSeries = this.updater((state, series: Serie[]) => {
    return { ...state, series, state: 'loaded' };
  });

// Effect
// Gets a string asynchronously and uses the updater to 
// add the result to the store.
readonly getAllSeries = this.effect<void>((trigger$) =>
    trigger$.pipe(
      // Define the state of the component as loading
      tap(() => this.patchState({ state: 'loading' })),
      switchMap(() =>
        this.seriesService.getSeries().pipe(
          tapResponse({
            // When the request is successful, update the store
            next: (movies) => this.addSeries(movies),
            error: (e: HttpErrorResponse) => {
              // When the request fails, update the store with the error state
              this.patchState({ state: 'error' });
              this.handleError(e);
            },
          })
        )
      )
    )
  );
```

Effects take either 0 or 1 parameter. This parameter can be an item or an object with the structure required for making the request.

An example with a parameter:

```typescript
 readonly getSerieDetail = this.effect((serieId$: Observable<string>) => {
        return serieId$.pipe(
            // Choose the proper choice of the flattening operator.
            switchMap((id) => this.seriesService.fetchSerie(id).pipe(
                // code removed
            )),
        );
    });
```

## Using the store in our component

Our component,

```typescript
@Component({
  selector: 'app-series',
  standalone: true,
  imports: [AsyncPipe, NgForOf, NgIf],
  templateUrl: './series.component.html',
  styleUrls: ['./series.component.scss'],
  providers: [SeriesStore, SeriesService],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class SeriesComponent implements OnInit {
  private readonly store = inject(SeriesStore);
  readonly vm$: Observable<ViewModelComponent> = this.store.vm$; // Our ViewModel exposed to the template

  ngOnInit(): void {
    this.store.getAllSeries();
  }
}
```

and our template:

```xml
<ng-container *ngIf="vm$ | async as vm">
  <h1>Series</h1>

  <!--Show this while loading:-->
  <ng-container *ngIf="vm.isLoading; else contentTpl">
    <p>Loading...</p>
  </ng-container>

  <!--Template when the data is loaded:-->
  <ng-template #contentTpl>
    <ul *ngIf="vm.series.length; else noItemsTpl">
      <li *ngFor="let serie of vm.series">
        {{ serie.name }}
      </li>
    </ul>

    <!-- If dont have any series, show this:-->
    <ng-template #noItemsTpl>
      <p>No series found</p>
    </ng-template>
  </ng-template>
</ng-container>
```

## Recommendations

In my experience, I consistently use a View Model to display the final information that will be rendered in the template. Nearly every 'if' statement in the template can be converted into a selector and exposed in the View Model.

The Store typically contains all the raw data I utilize, component state, and any unprocessed data. All of this data, whether it involves calculations, filtering, or selection, is transformed into a selector or a combination of selectors to be exposed in the View Model.

## Conclusion

The NgRx ComponentStore is a useful tool to handle local/component states in an app. It can start, update, and manage side effects in the store, making state management consistent and efficient. Methods like `setState()`, `patchState()`, `select()`, `updater()`, and `effect()` give flexible and complete control of the store. The ComponentStore's connection with lifecycle hooks makes it even more versatile. By using these features well, developers can keep data consistent in their apps, leading to better performance and user experience.

Do you have any experiences with ComponentStore to share? Or any questions about what we discussed? I'd love to hear your thoughts and talk about them in the comments below.