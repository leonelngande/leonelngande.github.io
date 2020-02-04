---
layout: post
hidden: false
title: 'How To Pass Data To An Ionic 4 Modal, and Return Data From The Modal'
tags: []
---
Say you're working on an Ionic app and would like to collect some data from the user using a modal, as well as pass initial data into the modal, how do you do that? Let's go ahead and take a quick look.

PS, our case study is a form which has a `market` field.

```typescript
export interface Market {
    id: string;
    imageUrl: string;
    title: string;
}
```

We'll leverage a modal for collecting the user's location through a Google Map in the modal component, passing in an initial location (userLocation).

1. The form component

2. The modal component

3. Passing initial data and presenting the modal

4. Closing the modal and returning the user selected location
