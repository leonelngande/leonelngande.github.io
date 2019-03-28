---
layout: post
hidden: false
title: Fetching the Current Route Fragment in Angular 7
tags:
  - Angular
  - Fragment Navigation
---
I've spent a few hours trying to get fragment navigation working properly for a use case on Xamcademy (https://xamcademy.com).

I want the browser (Angular) to automatically scroll to a section in the catalog when it's fragment link is clicked under the Catalog sidebar menu.

From my understanding so far, the current implementation of Angular's fragment navigation is still limited and works as expected only with static data.

https://github.com/angular/angular/issues/24547

https://www.bennadel.com/blog/3534-restoring-and-resetting-the-scroll-position-using-the-navigationstart-event-in-angular-7-0-4.htm

So I settled on fetching the fragment from an injected ActivatedRoute and manually scrolling to the location with a helper function.

Here's my courses list ts file.

```typescript
import {Component, OnDestroy, OnInit} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {IExam} from '../../../@core/models/exam';
import {takeWhile} from 'rxjs/operators';

@Component({
  selector: 'xc-courses-list',
  templateUrl: './courses-list.component.html',
  styleUrls: ['./courses-list.component.scss'],
})
export class CoursesListComponent implements OnInit, OnDestroy {

  exams: Array<IExam>;
  alive = true;

  constructor(private route: ActivatedRoute) { }

  ngOnInit() {
    this.exams = this.route.snapshot.data.exams;

    this.route.fragment.pipe(
      takeWhile(_ => this.alive),
    ).subscribe((fragment: string) => {
      this.scrollToAnchor(fragment);
    });
  }

  /**
   * Scroll to anchor
   *
   * @param {string} location Element id
   * @param {string} wait     Wait time in milliseconds
   */
  public scrollToAnchor(location: string, wait = 0): void {
    const element = document.querySelector('#' + location);
    if (element) {
      setTimeout(() => {
        element.scrollIntoView({behavior: 'smooth', block: 'start', inline: 'nearest'});
      }, wait);
    }
  }

  ngOnDestroy(): void {
    this.alive = false;
  }
}
```

And the course list html file in which an id is set for each course category (exam) which can be scrolled to.

```typescript
<div class="container">
  <div class="text-center">
    <h3>All Courses/Subjects</h3>
    <p>This is the full catalog of our courses and subjects.</p>
  </div>
</div>

<div class="row" *ngIf="exams; else noExams">
  <div class="col-md-9">
    <div class="row">
      <div class="col-md-12" *ngFor="let exam of exams">

        <nb-card [id]="'exam' + exam.id">
          <nb-card-header>
            <!--<nb-badge text="{{exam.courses.length}}" status="success" position="top right"></nb-badge>-->
            {{ exam.name }}
          </nb-card-header>
          <nb-list>
            <nb-list-item *ngFor="let course of exam.courses">

              <a routerLink="/courses/{{course.id}}" class="xc-list-link">
                <nb-user [name]="course.name"
                         size="large"
                         [title]="course.exam_sessions.length ? course.exam_sessions.length + ' sessions' : 'Sessions unavailable'">

                </nb-user>
              </a>
              
            </nb-list-item>
          </nb-list>
        </nb-card>

      </div>
    </div>
  </div>
  <div class="col-md-3" #stickyMenu [class.sticky]="sticky">
    <div class="row">
      <div class="col-md-12">
        <nb-card class="exams-sidebar">
          <nb-list>
            <nb-list-item *ngFor="let exam of exams">

              <a [href]="'/courses/list#exam' + exam.id" class="xc-list-link">
                <nb-user [name]="exam.name"
                         size="large"
                         [title]="exam.courses?.length ? exam.courses.length + ' courses' : 'Courses unavailable'">

                </nb-user>
              </a>
              
            </nb-list-item>
          </nb-list>
        </nb-card>
      </div>
    </div>
  </div>
</div>

<ng-template #noExams>
  <div class="row">
    <div class="col-md-6">
      <nb-card>
        <nb-card-body>
          Sorry, no course is currently available. Please keep visiting to be updated on course availability.
        </nb-card-body>
      </nb-card>
    </div>
  </div>
</ng-template>
```

So we listen to changes in the route fragment, find it's id on the page, and if it exists, we use the utility function to scroll to the id.

Here's a live implementation <https://www.xamcademy.com/courses/list#exam2>

If you found a bug or have any question, please do mention in the comments section.

Happy coding!
