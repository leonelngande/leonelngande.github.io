---
layout: post
hidden: false
title: Adding Custom Content To Akveo's @nebular/theme 4.5.0 Sidebar Component
image: 
  path: /images/uploads/akveo-nebular.jpg
  thumbnail: /images/uploads/akveo-nebular.jpg
  caption: "Akveo Nebular 4.0"
tags:
  - Angular
  - Nebular UI Kit
  - nb-sidebar
author: Leonel Elimpe
---
So I've been working with Akveo's [Nebular UI Kit](https://akveo.github.io/nebular/) for a while now and it's been a real issue adding custom content to `nb-sidebar`.

Today, I had a look at the component's source code on [Github](https://github.com/akveo/nebular/tree/master/src/framework/theme/components/sidebar) and found out you can basically project anything into it, specifically into `nb-sidebar > div.main-container > div.scrollable` .

```typescript
@Component({
  selector: 'nb-sidebar',
  styleUrls: ['./sidebar.component.scss'],
  template: `
    <div class="main-container"
         [class.main-container-fixed]="containerFixedValue">
      <ng-content select="nb-sidebar-header"></ng-content>
      <div class="scrollable" (click)="onClick($event)">
        <ng-content></ng-content>
      </div>
      <ng-content select="nb-sidebar-footer"></ng-content>
    </div>
  `,
})
```

Full source code [here](https://github.com/akveo/nebular/blob/master/src/framework/theme/components/sidebar/sidebar.component.ts).

Notice the sidebar component projects `nb-sidebar-header` and `nb-sidebar-footer` into dedicated containers, and all other content into `div.scrollable`.

What this means is any custom content you put into `<nb-sidebar>...</nb-sidebar>` will be projected into `div.scrollable`. For example:

```html
<nb-layout windowMode>

  <nb-layout-header fixed>
    <xc-header></xc-header>
  </nb-layout-header>


  <nb-sidebar class="menu-sidebar" tag="menu-sidebar" responsive>

    <ng-container *ngIf="menu$ | async; let menu">
      <nb-menu [items]="menu"></nb-menu>
    </ng-container>

    <!--
      -- Found out I can just project any content into nb-sidebar template's .scrollable div.
      -- So decided to add the footer here and style it well so it displays properly.
      -->
    <xc-footer></xc-footer>


  </nb-sidebar>


  <nb-layout-column>
    <router-outlet></router-outlet>
  </nb-layout-column>


</nb-layout>
```

Notice my `xc-footer` in the above sample. I was trying to add a footer to the sidebar but didn't want a "sticky footer" which is what nb-sidebar-footer offers. So I placed it below nb-menu and added css to push it well below `nb-menu`.

```css
nb-menu {
  /**
    * Assuming here the footer (xc-footer) is directly below nb-menu
    * Push the footer a good distance below the menu.
    */
  padding-bottom: 3.5rem;
}
```

So basically you can add any content to the sidebar and if need be, properly style it 🙂.
