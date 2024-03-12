---
title: "How to Convert ChartModule to Spectator for Easy Testing"
seoTitle: "ChartModule to Spectator: Easier Testing"
seoDescription: "Optimize ChartModule conversion to Spectator for easy Jest testing; overcome PrimeNG graphics module issues by mocking browser APIs or ChartModule"
datePublished: Fri Oct 06 2023 21:25:47 GMT+0000 (Coordinated Universal Time)
cuid: clnf48idd000109k03ghc8v0a
slug: how-to-convert-chartmodule-to-spectator-for-easy-testing
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/cckf4TsHAuw/upload/70334d51f968ddde5e37436686038dda.jpeg
tags: angular, jest, primeng, spectator

---

Recently, we migrated some libraries to Jest and encountered issues with the tests we had created for a component that used the PrimeNg ChartModule (which, is a wrapper for ChartJS).

At first glance, we identified that the issue stemmed from Jest's lack of support for `Canvas`, `MutationObserver`, and other browser APIs necessary to run the tests (Karma uses a real browser to run the test, but Jest does not).

Finally, we realized that we had two options: either mock the browser APIs or mock the `ChartModule`. Since we were using Spectator for testing, we managed to do this easily (after much searching, test, and error).

Firstly, we created a mock to replace what we needed. In this case, since PrimeNG still uses modules, we created a component and a test module specifically for this purpose.

```typescript
// Component containing chart.js in PrimeNG's ChartModule
@Component({
    selector: 'p-chart',
    template: 'PrimeNg Chart - mocked component'
})
class MockUIChart {
    // Here we add all the inputs that we are using
    @Input()
    data: unknown;
    @Input()
    options: unknown;
    @Input()
    plugins: unknown;
}
// Module that imports and exports the component
@NgModule({
    declarations: [MockUIChart],
    exports: [MockUIChart]
})
export class MockChartModule {}
```

In our test, we do the following:

```typescript
describe('ComponentStandalone', () => {
    let spectator: Spectator<ComponentStandalone>;

    const createComponent = createComponentFactory({
        component: ComponentStandalone,
        overrideComponents: [
            [
                ComponentStandalone,
                {
                    remove: { imports: [ChartModule] },
                    add: { imports: [MockChartModule] }
                }
            ]
        ]
    });
});
```

With this, we prevent the `p-chart` component from attempting to instantiate ChartJS in the [`ngAfterViewInit`](https://github.com/primefaces/primeng/blob/master/src/app/components/chart/chart.ts), which was the root cause of all the Jest warnings.

Do you have any experience with `Jest`, PrimeNG, or testing tools that you'd like to share? Leave your comments below!