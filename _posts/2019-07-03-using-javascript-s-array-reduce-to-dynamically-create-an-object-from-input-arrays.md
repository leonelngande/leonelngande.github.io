---
title: Using Javascript's Array.reduce() to Dynamically Create and Add Properties to an Object
  Arrays
hidden: false
published: false
tags:
  - Array.reduce()
  - Object Creation
  - Functional Programming
author: Leonel Elimpe
---
So I had this situation today where data to be exported had to be dynamically assigned Header Names (Excel export). Am using the Infragistics Angular [Excel Exporter](https://www.infragistics.com/products/ignite-ui-angular/angular/components/exporter_excel.html).

The data to be exported looks like this:
```typescript
const exportData = [
    	
];
```
There's equally an array with column properties that looks like this: