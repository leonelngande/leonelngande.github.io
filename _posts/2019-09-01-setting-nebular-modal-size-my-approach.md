---
layout: post
hidden: false
title: 'Setting Nebular Modal Size: My Approach'
tags:
  - Angular
  - Nebular
  - NbDialogService
---
I've had this longstanding issue with setting the size of modals created with [NbDialogService](https://akveo.github.io/nebular/docs/components/dialog/overview#nbdialogservice).

The examples in the Nebular docs use a card for the modal structure, which makes sense, with this setup however the modal has the tendency to not adjust well to dynamic content, using up the full viewport sometimes.

To fix this issue, I've had a look at the [bootstrap modal](https://getbootstrap.com/docs/4.0/components/modal/) (which functions exactly as i'd like my modal to), the [large one](https://getbootstrap.com/docs/4.0/components/modal/#large-modal) to be precise.

![Large Bootstrap modal](/images/uploads/bootstrap-large-modal-width.png)

Notice the max-width of the modal - `max-width: 800px`. I decided to use this value to set the max-width of my modal.

Next up was limiting the modal's height with the max-height property. I decided to use the value of NbComponentSize `large`, which currently is `44.25rem` (from inspecting the value).

```typescript
export declare type NbComponentSize = 'tiny' | 'small' | 'medium' | 'large' | 'giant';
```

This effectively gave me a modal component that properly adjust it's width and height to the content passed to it. Here's the full component code:

```typescript
import {Component, OnInit, TemplateRef, ViewChild} from '@angular/core';
import {NbDialogService} from '@nebular/theme';


@Component({
  selector: 'xc-modal-view',
  template: `
    <ng-template #modalTemplate let-data let-ref="dialogRef">
      <nb-card>
        <nb-card-header>
          <span>Title</span>
          <button class="close" aria-label="Close" (click)="ref.close()">
            <nb-icon icon="close"></nb-icon>
          </button>
        </nb-card-header>
        <nb-card-body>
          <router-outlet></router-outlet>
        </nb-card-body>
      </nb-card>
    </ng-template>
  `,
  styles: [`
    nb-card {
      /**
       * This is the max-width value of the Bootstrap giant modal
       * Using it here ensures the modal will properly adjust it's width to the content
       */
      max-width: 800px;
      /**
       * This is the height value of NbComponentSize 'giant'
       * By setting max-height of the modal card to this value, we ensure the modal will properly adjust it's height
       * to the content
       */
      max-height: 44.25rem;
    }
  `],
})
export class ModalViewComponent implements OnInit {

  @ViewChild('modalTemplate', {static: true}) modalTemplate: TemplateRef<any>;

  constructor(
    private dialogService: NbDialogService,
  ) { }

  ngOnInit() {
    this.dialogService.open(this.modalTemplate, { context: {} });
  }

}
```

I use the modal as a routable modal, reason for the router-outlet in the view. For more on this you can have a look [here](https://www.bennadel.com/blog/3620-most-of-your-modal-windows-should-be-directly-accessible-by-route-in-angular-7-2-15.htm).

Have a question or found an error in the post? Feel free to reach out on [Twitter](https://twitter.com/leonelngande), or just use the comments section.

Happy coding!
