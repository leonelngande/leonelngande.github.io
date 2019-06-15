---
title: 'Fix: Angular Route Resolver Receiving Data But Still Preventing Navigation
  To Route'
hidden: false
image:
  path: "/images/angular-banner-large.jpg"
  thumbnail: ''
  caption: Angular logo banner image
categories: []
tags:
- Router Resolver
- Angular
last_modified_at: 

---
Ever had this issue where for some reason you can't navigate to a route even though everything seems ok, and the route's data resolvers are all executing?

Well, for the past two days this is just what I've been going through. At first I thought it was due to changes I'd made to my application (actually it's part of the reason), but today I finally found the real culprit.

> The Angular router waits for an observable to complete in order for the navigation to complete.

There! [Here's the GitHub issue](https://github.com/angular/angular/issues/10556#issuecomment-240284735) that made my headache go away.

**Background:**

I had this data resolver for the current logged in user:

     import { Injectable } from '@angular/core';
    import {Resolve} from '@angular/router';
    import {catchError} from 'rxjs/operators';
    import {EMPTY, Observable} from 'rxjs';
    import {IUser} from '../models/user';
    import {AuthService} from '../auth/auth.service';
    
    @Injectable()
    export class LoggedInUserResolver implements Resolve<IUser> {
    
      constructor(private authService: AuthService) { }
    
      resolve(): Observable<IUser> | Observable<never> {
        return this.authService.getCurrentUser().pipe(
          catchError(err => {
            return EMPTY;
          }),
        );
      }
    }
    

As above, it simply called the getCurrentUser() method in the AuthService, which returned an Observable of type IUser retrieved from the jwt token in local-storage.

    ﻿import { Injectable } from '@angular/core';
    import {filter, map, mergeMap} from 'rxjs/operators';
    import {Observable} from 'rxjs';
    import {NbAuthService} from '@nebular/auth';
    import {IUser} from '../models/user';
    import {UserService} from './users.service';
    import {ApiService} from './api.service';
    
    @Injectable({providedIn: 'root'})
    export class AuthService {
    
      constructor(public api: ApiService,
                  public nbAuth: NbAuthService) { }
    
      getCurrentUser(): Observable<IUser> {
    
        return this.nbAuth.getToken().pipe(
          filter(res => res && res.getPayload()),
          map(res => res.getPayload().user),
          filter(user => user && user.id),
          map(user => UserService.adapt(user)),
        );
      }
    }

FYI, I am using Akveo's [Nebular UI Kit](https://akveo.github.io/nebular/) and [ngx-admin](https://github.com/akveo/ngx-admin).

This worked out find until I modified the api to no longer return user details as part of the jwt token after login. Now, only the token is returned and the user details fetched at /auth/user, which caused me to modify the AuthService as follows:

    import {Injectable, OnDestroy} from '@angular/core';
    import {filter, map, mergeMap, switchMap, takeWhile} from 'rxjs/operators';
    import {BehaviorSubject, Observable} from 'rxjs';
    import {NbAuthJWTToken, NbAuthService} from '@nebular/auth';
    import {IUser} from '../models/user';
    import {UserService} from './users.service';
    import {ApiService} from './api.service';
    
    @Injectable({providedIn: 'root'})
    export class AuthService implements OnDestroy {
    
      private readonly _user = new BehaviorSubject<IUser>(null);
      // Expose the observable$ part of the _user subject (read only stream)
      readonly user$ = this._user.asObservable();
    
      alive = true;
    
      constructor(
        public api: ApiService,
        public nbAuth: NbAuthService,
      ) {
        // listen for token change and update logged in user accordingly
        this.nbAuth.onTokenChange()
          .pipe(
            takeWhile(_ => this.alive),
            filter((token: NbAuthJWTToken) => token && token.isValid()),
            switchMap(res => {
              return this.updateUser();
            }),
          )
          .subscribe();
      }
    
      ngOnDestroy(): void {
        this.alive = false;
      }
    
      getCurrentUser(): Observable<IUser> {
        return this.user$;
      }
    
      updateUser(): Observable<IUser> {
        return this.api.get('/auth/user').pipe(
          map(userObject => {
            const user = userObject ? UserService.adapt(userObject) : null;
            // update _user behaviour subject
            this._user.next(user);
            return user;
          }),
        );
      }
    }

With this modification, the logged in user is now exposed as an observable of the readonly __user_ BehaviourSubject, which is updated by fetching the user from the api every time a token change event (after login, after first app load) is fired from Nebular's NbAuthService.

Equally, the getCurrentUser() method is modified to simply return the new user$ observable of the AuthService.

This is where my routing issue began (hadn't realized). Now instead of getCurrentUser() returning an Observable that completes after being called, it is returning an Observable derived from a [BehaviourSubject](https://www.learnrxjs.io/subjects/behaviorsubject.html) which _almost never completes_ (in this case, it'll be destroyed when this.alive === false).

Going back to the LoggedInUserResolver, the use of the BehaviourSubject observable (authService.user$) means any route definition that previously used this resolver will stall and the route is never navigated to.

Here's a sample usage of the resolver in a route definition:

    const routes: Routes = [
      {
        path: ':section_question_id',
        component: SectionQuestionOverviewComponent,
        resolve: {
          section_question: ExamSectionQuestionResolver,
          loggedInUser: LoggedInUserResolver,
        },
      },
    ];

As a quick workaround, I used the rxjs [take](https://www.learnrxjs.io/operators/filtering/take.html) operator to force the user$ observable to complete, which is also suggested in the GitHub [issue](https://github.com/angular/angular/issues/10556#issuecomment-240284735) mentioned at the beginning. From the thread, the issue has not yet been handled at the framework level.

The LoggedInUserResolver now looks like this:

    import {Injectable} from '@angular/core';
    import {Resolve} from '@angular/router';
    import {catchError, take} from 'rxjs/operators';
    import {EMPTY, Observable} from 'rxjs';
    import {IUser} from '../models/user';
    import {AuthService} from '../services/auth.service';
    
    @Injectable({providedIn: 'root'})
    export class LoggedInUserResolver implements Resolve<IUser> {
    
      constructor(private authService: AuthService) { }
    
      resolve(): Observable<IUser> | Observable<never> {
        return this.authService.user$
          .pipe(
            /**
             * Angular router waits for an observable to complete in order for the navigation to complete.
             * @see https://github.com/angular/angular/issues/10556#issuecomment-240284735
             */
            take(1),
            catchError(err => {
              console.error('Error fetching logged in user ', err);
              return EMPTY;
            }),
          );
      }
    }

Well, that's it for this post, feel free to point out any issues, or ask any question in the comments section below.

Happy coding!