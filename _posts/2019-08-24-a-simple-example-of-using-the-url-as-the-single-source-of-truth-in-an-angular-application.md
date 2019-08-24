---
layout: post
hidden: false
title: >-
  A Simple Example of Using the Url as the Single Source of Truth in an Angular
  Application
tags:
  - Angular
  - State Management
---
Lately i've been trying to apply the concept of using the url as the single source of truth in Angular applications I work on. Today I wrote some functionality demonstrates this quite well.

What we'll be building is a search bar component that pushes the search input to the url as a query parameter and on page reload, retrieves this value to restore the previous state. 

Here's a link to the [live demo](https://leonelngande.github.io/material-icons-offline/) as well as a link to the [Github repository](https://github.com/leonelngande/material-icons-offline).

// Add working demo gif here

Let's get started with the search bar component.

```typescript
import {Component, OnInit} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-search-bar',
  templateUrl: './search-bar.component.html',
  styleUrls: ['./search-bar.component.scss']
})
export class SearchBarComponent implements OnInit {

  searchValue: string;

  constructor(
    private router: Router,
    private activatedRoute: ActivatedRoute,
  ) { }

  ngOnInit() {
    this.searchValue = this.activatedRoute.snapshot.queryParams.search;
  }

  clearSearch() {
    this.searchValue = '';
    this.updateSearchQueryParam(this.searchValue);
  }

  updateSearchQueryParam(searchValue: string) {
    this.router.navigate(
      [],
      {
        relativeTo: this.activatedRoute,
        queryParams: {search: searchValue},
        replaceUrl: true,
        queryParamsHandling: 'merge'
      });
  }
}
```

```html
<form>
  <mat-form-field appearance="outline">

    <input type="search"
           placeholder="Search"
           matInput
           name="searchValue"
           [(ngModel)]="searchValue"
           (ngModelChange)="updateSearchQueryParam(searchValue)">

    <button mat-button *ngIf="searchValue" matSuffix mat-icon-button aria-label="Clear" (click)="clearSearch()">
      <mat-icon>close</mat-icon>
    </button>

  </mat-form-field>

</form>
```

Above, we're using Angular's two-way data binding to store the user's search value in `searchValue`. The interesting part is where on model change, we push the search value to the url in `updateSearchQueryParam`.

The function takes in the search value, and then uses the Angular Router to navigate to the current route with new query params, which will not reload the page, but will update query params.
