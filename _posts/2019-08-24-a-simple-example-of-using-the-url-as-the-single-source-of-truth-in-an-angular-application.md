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

What we'll be building is a search bar component that pushes the search input to the url as a query parameter and on page reload, retrieves this value to restore the previous search state. 

Here's a link to the [live demo](https://leonelngande.github.io/material-icons-offline/) as well as a link to the [Github repository](https://github.com/leonelngande/material-icons-offline).

![Working demo](/images/uploads/search.gif)

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

Above, we're using Angular's two-way data binding to store the search input's value in `searchValue`. We equally have button to clear the search input when necessary.

 The interesting part is where we push the search value to the url in `updateSearchQueryParam` on model change. The function takes in the search value and uses the Angular Router to navigate to the current route, adding or updating the `search` query parameter with the new searchValue, and without reloading the page.

Here's a breakdown for each of the navigation extras passed to `router.navigate`:

* relativeTo: this.activatedRoute - specifies current activated route as root URI to use for relative navigation
* replaceUrl: true - navigate while replacing the current state in history
* queryParamsHandling: 'merge' - merge new with current query parameters in the router link for the next navigation
* queryParams: {search: searchValue} - sets the search query parameter to the URL

Should the user decide to reload the page or share the url, we should be able to pick up the search value and restore the previous state. That's what the following line of code in ngOnInit does:

```typescript
    this.searchValue = this.activatedRoute.snapshot.queryParams.search;
```

It picks up the value of the search query parameter from the current shapshot of the activated route and assigns it the searchValue, thereby restoring the previous search state.

This is equally useful in cases where you have another or multiple components that makes use of the search value. Within the other component(s), you simply listen for changes to the search query parameter on the activated route as below.

```typescript
export class MaterialIconsComponent implements OnInit {

  constructor(private activatedRoute: ActivatedRoute) { }

  ngOnInit() {
    // Listen for search route param changes
    this.activatedRoute.queryParams
      .pipe(
        map((queryParams: Params) => queryParams.search as string),
        distinctUntilChanged()
      )
      .subscribe((searchValue) => {
        // do something with the search value, maybe apply a filter to your list of items?
      });
  }
}
```

You can get pretty creative with this setup :-).

So that's it for this blog post, hope it's helped you in some way. Found something to improve in the blog post or have a question? Feel free to reach out to me on [Twitter](https://twitter.com/leonelngande).

Happy coding!
