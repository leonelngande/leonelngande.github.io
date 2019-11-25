---
layout: post
hidden: false
title: >-
  Angular: Allow Users Retry Failed Requests With A Twitter-Like Try Again
  Button
tags:
  - Angular
  - Retry Failed Requests
---
While using the Twitter web app, I noticed it displays a `Try Again` button for failed requests in different sections of the user interface. This allows the user retry each failed request without affecting the rest of the application, quite neat.

In this post, we're going to implement similar functionality in 3 steps using Angular, RxJS, [Bootstrap 4](https://getbootstrap.com/), and good old DOM events 🙂.

Feel free to use the UI library of your choice, full project source code is available on [Github](https://github.com/leonelngande/angular-retry-failed-requests-from-ui).

**1. Create a service that exposes the user's connection status** 

```typescript
import {Injectable} from '@angular/core';
import {fromEvent, merge, of} from 'rxjs';
import {mapTo} from 'rxjs/operators';

@Injectable({providedIn: 'root'})
export class ConnectionStatusService {

  /**
   * This code returning a false value means you're absolutely offline as in disconnected.
   * It returning true doesn't necessarily indicate that there's a practically usable connection.
   *
   * @see https://stackoverflow.com/a/39573363/6924437
   * @see https://justmarkup.com/articles/2016-08-18-indicating-offline/
   */
  readonly online$: Observable<boolean> = merge(
    of(navigator.onLine),
    fromEvent(window, 'online').pipe(mapTo(true)),
    fromEvent(window, 'offline').pipe(mapTo(false)),
  );

}
```

`online$` above returning a false value means you're absolutely offline as in disconnected. It returning true doesn't necessarily indicate that there's a practically usable connection. See [this](https://stackoverflow.com/a/39573363/6924437) Stackoverflow answer for a more elaborate explanation.

On a lighter note, if you've got a better name for the above service, please use the comments. Naming is hard 😅.

**2. Create a component to display the retry button and error message** 

```typescript
import {Component, EventEmitter, Input, Output} from '@angular/core';
import {ConnectionStatusService} from '../../services/connection-status.service';
import {Observable} from 'rxjs';

@Component({
  selector: 'app-try-again',
  template: `
    <div class="text-center">
      <ng-container *ngIf="(isOnline$ | async) === false">
        <span class="text-muted pb-5">
          📶🚫 <small>Offline</small>
        </span>
      </ng-container>
      <p class="text-muted">{{message}}</p>
      <button (click)="emitTryAgain()" class="btn btn-primary">
        Try Again
      </button>
    </div>

  `,
})
export class TryAgainComponent {

  /** Message to display to user in the template */
  @Input() message = 'Something went wrong.';

  /** Event emitted when the user clicks the Try Again button */
  @Output() tryAgain = new EventEmitter<boolean>();

  constructor(private connectionStatusService: ConnectionStatusService) { }

  /** Emits the tryAgain event */
  emitTryAgain() {
    this.tryAgain.emit(true);
  }

  /**
   * Getter for connection service's online$ property.
   * Primary use case is to avoid directly accessing the service in the template.
   */
  get isOnline$(): Observable<boolean> {
    return this.connectionStatusService.online$;
  }

}
```

In the template, we display a `message`, as well as a `Try Again` button which when clicked emits the `tryAgain` event. We equally check if the user is offline and display an appropriate offline icon and message. 

**3. Use the component to retry failed requests** 

For this demo, we're going to fetch Post and Todo items using the [JSONPlaceholder](https://jsonplaceholder.typicode.com/guide.html) API. We'll display them in seperate 'boxes' in the UI, adding our above retry button for each resource.

![preview of posts and todos boxes](/images/uploads/screenshot_2019-11-25-retry-failed-requests-from-ui-1-.png)

```typescript
import {Component, OnInit} from '@angular/core';
import {Observable, throwError} from 'rxjs';
import {HttpClient} from '@angular/common/http';
import {catchError, tap} from 'rxjs/operators';
import {IPost} from './models/post.interface';
import {ITodo} from './models/todo.interface';


@Component({
  selector: 'app-root',
  template: `
    <div class="container">

      <h1 class="text-center pb-5 pt-4">Retry Failed Requests From UI</h1>

      <div class="row">

        <!-- Posts start -->
        <div class="col-md-6">

          <h4 class="text-center">POSTS</h4>

          <ul class="list-group" *ngIf="posts$ | async; let posts">
            <li class="list-group-item" *ngFor="let post of posts">{{post.title}}</li>
          </ul>

          <ng-container *ngIf="fetchPostsFailed">
            <app-try-again (tryAgain)="fetchPosts()"
                           message="Failed to load Posts.">

            </app-try-again>
          </ng-container>

        </div>
        <!-- Posts end -->

        <!-- Todos start -->
        <div class="col-md-6">

          <h4 class="text-center">TODOS</h4>

          <ul class="list-group" *ngIf="todos$ | async; let todos">
            <li class="list-group-item" *ngFor="let todo of todos">{{todo.title}}</li>
          </ul>

          <ng-container *ngIf="fetchTodosFailed">
            <app-try-again (tryAgain)="fetchTodos()"
                           message="Failed to load Todos.">

            </app-try-again>
          </ng-container>

        </div>
        <!-- Todos end -->

      </div>

    </div>

  `,
})
export class AppComponent implements OnInit {

  posts$: Observable<IPost[]>;
  fetchPostsFailed: boolean;

  todos$: Observable<ITodo[]>;
  fetchTodosFailed: boolean;

  constructor(private http: HttpClient) { }

  ngOnInit() {
    this.fetchPosts();
    this.fetchTodos();
  }

  fetchPosts() {
    this.posts$ = this.http.get<IPost[]>('https://jsonplaceholder.typicode.com/posts')
      .pipe(
        // fetch is successful so set failed to false
        tap(() => this.fetchPostsFailed = false),
        // fetch has failed so set failed to true
        catchError((error) => {
          this.fetchPostsFailed = true;
          return throwError(error);
        }),
      );

  }

  fetchTodos() {
    this.todos$ = this.http.get<ITodo[]>('https://jsonplaceholder.typicode.com/todos')
      .pipe(
        // fetch is successful so set failed to false
        tap(() => this.fetchTodosFailed = false),
        // fetch has failed so set failed to true
        catchError((error) => {
          this.fetchTodosFailed = true;
          return throwError(error);
        }),
      );

  }

}
```

In the component, we keep track of failed requests with `fetchPostsFailed` and `fetchTodosFailed` for posts and todos respectively. We then use them in the template to determine if a request has failed and show the retry button.

Checkout the [demo application](https://leonelngande.github.io/angular-retry-failed-requests-from-ui/), and while on the page, disconnect from the internet, then click any of the `Reload` buttons to see the Try Again button appear.

![preview of retry buttons](/images/uploads/screenshot_2019-11-25-retry-failed-requests-from-ui-2-.png)

Also notice as soon as you reconnect to the internet, the offline icon and text are hidden.

![preview of hidden offline text and icon](/images/uploads/screenshot_2019-11-25-retry-failed-requests-from-ui-3-.png)

Clicking on the `Try Again` button re-triggers the (appropriate) http request and should it succeed, the Try Again button button is hidden.
