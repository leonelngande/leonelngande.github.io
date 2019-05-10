---
layout: post
hidden: false
title: Angular 2+ PDF Generation with jsPDF - Sample Project
tags:
  - Angular
  - jsPDF
  - PDF Generation
---
I may flesh this out with a detailed post later, but if you have any questions or encounter any issues feel free to reach out to me.

Here's the [sample project on GitHub](https://github.com/leonelngande/javascript-pdf-generation-with-angular), it demonstrates generating a PDF document in Angular with [jsPDF](https://github.com/MrRio/jsPDF), with features like a header, a footer, header and footer formatting, tables, page numbers, font size, and margins.

You should however note that generating PDF documents with Javascript won't really get you much, you're better off using a backend technology. This is suitable for quickly printing stuff like invoices, event tickets, reports, certificates,etc.

When you click the Generate button, a copy of the generated pdf will be downloaded automatically and it'll also be displayed in a dynamic iframe on the [demo page](https://leonelngande.github.io/javascript-pdf-generation-with-angular/). The demo pulls a couple paragraphs from the api https://jsonplaceholder.typicode.com/comments to flesh out the document.



Hope this helps you out somehow, happy coding!
