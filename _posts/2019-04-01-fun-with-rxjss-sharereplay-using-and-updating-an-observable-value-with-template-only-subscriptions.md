---
layout: post
hidden: false
title: >-
  Fun with RXJS's shareReplay, using and updating an observable value with
  template only subscriptions
tags: []
---
```typescript
<igx-expansion-panel>
  <igx-expansion-panel-header>
    <igx-expansion-panel-title>BUCKETS</igx-expansion-panel-title>
  </igx-expansion-panel-header>
  <igx-expansion-panel-body>
    <div>{{bucketOptions$ | async | json}}</div>
    <app-multi-select-box-card [optionsList]="bucketOptions$ | async"
                               (optionSelectionChanged)="updateOptionSelection(bucketOptions$, $event)">

    </app-multi-select-box-card>
  </igx-expansion-panel-body>
</igx-expansion-panel>
```



```typescript
bucketOptions$: Observable<Array<IServerDropdownOption>>;

ngOnInit() {
  this.bucketOptions$ = this.contactsService.bucketList().pipe(shareReplay());
}

updateOptionSelection(optionsList$: Observable<Array<IServerDropdownOption>>, $event: IOptionMultiSelectBox) {
  optionsList$ = optionsList$.pipe(
    map(options => {
      return options.map(option => {
        if (option.value === $event.value) {
          option.selected = $event.selected;
        }
        return option;
      });
    }),
  );
}
```
