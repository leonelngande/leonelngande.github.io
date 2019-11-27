---
layout: post
hidden: false
title: Why you should use trackBy with Angular's *ngFor loop
tags:
  - Angular
  - '*ngFor'
  - trackBy
author: Leonel Elimpe
---
> trackBy is a function which will return a unique identifier for each item in the array provided to *ngFor. 
>
> Normally when the array changes, Angular re-renders the whole DOM tree. But if you use trackBy, Angular will know which element has changed and will only make DOM changes for that particular element.

I am currently working on Xamcademy (an examination questions revision platform), and it supports commenting on questions and reaction to comments by users.

`Logged in users can comment on questions they open.`

`Logged in users can react to comments of other users on the question.`

Here's how data currently flows through the components:

`QuestionOverviewComponent > ResponseListComponent > ResponseComponent > ReactionSelectComponent`

I took the approach of fetching all necessary data in the parent component (QuestionOverviewComponent) and then passing down the relevant pieces to the children as needed.

The ReactionSelectComponent emits an onSelect event, which the response and response list components pick up so it bubble up to the question overview component.

```typescript
import {Component, EventEmitter, Input, OnInit, Output, ViewChild} from '@angular/core';
import {IReactionType} from '../../@core/models/reaction.model';
import {NbPopoverDirective, NbPosition} from '@nebular/theme';

@Component({
  selector: 'xc-reaction-select',
  templateUrl: './reaction-select.component.html',
  styleUrls: ['./reaction-select.component.scss'],
})
export class ReactionSelectComponent implements OnInit {

  @Input() reactions: IReactionType[];
  position = NbPosition.BOTTOM;

  @Output() onSelect: EventEmitter<IReactionType> = new EventEmitter<IReactionType>();

  @ViewChild(NbPopoverDirective) popover: NbPopoverDirective;

  constructor() { }

  ngOnInit() {
  }

  selected(type: IReactionType) {
    this.onSelect.emit(type);
    this.popover.hide();
  }

}
```

_reaction-select.component.ts_

__

```typescript
import {Component, EventEmitter, Input, Output} from '@angular/core';
import {IComment} from '../../../@core/models/comment';
import {IReaction, IReactionEvent, IReactionType, ReactionType} from '../../../@core/models/reaction.model';
import {NbAccessChecker} from '@nebular/security';

@Component({
  selector: 'xc-response',
  templateUrl: './response.component.html',
  styleUrls: ['./response.component.scss'],
})
export class ResponseComponent {

  @Input() response: IComment;
  reactionTypes = ReactionType.all();

  @Output() onReaction: EventEmitter<IReactionEvent<IComment>> = new EventEmitter<IReactionEvent<IComment>>();

  constructor(public accessChecker: NbAccessChecker) { }

  react(reactionType: IReactionType) {
    this.onReaction.emit({
      reactionType,
      reactable: this.response,
    });
  }

  trackByFn(index, reaction) {
    return reaction.id; // unique id corresponding to the item
  }

  getCount(reaction: IReaction, reactions: IReaction[]): number {
    return reactions && reactions.filter(single => single.type === reaction.type).length;
  }
}
```

_response.component.ts_

__

```typescript
import {Component, EventEmitter, Input, Output} from '@angular/core';
import {IComment} from '../../../@core/models/comment';
import {IReactionEvent} from '../../../@core/models/reaction.model';

@Component({
  selector: 'xc-response-list',
  templateUrl: './response-list.component.html',
  styleUrls: ['./response-list.component.scss'],
})
export class ResponseListComponent {

  @Input() responses: IComment[] = [];

  @Output() onReaction: EventEmitter<IReactionEvent<IComment>> = new EventEmitter<IReactionEvent<IComment>>();

  constructor() { }

  react($event: IReactionEvent<IComment>) {
    this.onReaction.emit($event);
  }

  trackByFn(index, response) {
    return response.id; // unique id corresponding to the item
  }
}
```

_response-list.component.ts_

__

Therefore, on every comment or reaction event, the data is posted and the comments list in QuestionOverviewComponent re-fetched from the api.

The problem is this creates a situation where the comments in view dissappear for a moment and then reappear as Angular re-renders the comments list component. This is also quite ineficient.

```html
<xc-response *ngFor="let response of responses"
             (onReaction)="react($event)"
             [response]="response">
             
</xc-response>
```

_response-list.component.html without trackBy_ 



This is where `trackBy` comes in to save the day. 

```html
<xc-response *ngFor="let response of responses; trackBy: trackByFn"
             (onReaction)="react($event)"
             [response]="response">
             
</xc-response>
```

_response-list.component.html with trackBy_ 



trackBy is a function which will return a unique identifier for each item in the array provided to *ngFor. 

Normally when the array changes, Angular re-renders the whole DOM tree. But if you use trackBy, Angular will know which element has changed and will only make DOM changes for that particular element.

Now whenever the comments list is re-fetched, only the affected comment's view is updated as Angular re-renders.

So, always make sure to provide a trackBy function in you *ngFor loops, especially when trying to optimize performance.

Happy coding!
