---
layout: post
hidden: false
title: 'Setting Nebular Modal Size: My Approach'
tags: []
---
I've had this longstanding issue with setting the size of modals created with [NbDialogService](https://akveo.github.io/nebular/docs/components/dialog/overview#nbdialogservice).

The examples in the Nebular docs use a card for the modal structure, which makes sense, with this setup however the modal has the tendency to not adjust well to dynamic content, using up the full viewport sometimes.

To fix this issue, I've had a look at the [bootstrap modal](https://getbootstrap.com/docs/4.0/components/modal/) (which functions exactly as i'd like my modal to), the [large one](https://getbootstrap.com/docs/4.0/components/modal/#large-modal) to be precise.
