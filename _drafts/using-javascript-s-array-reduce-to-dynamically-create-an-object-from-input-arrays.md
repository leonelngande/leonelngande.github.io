---
title: Using Javascript's Array.reduce() to Dynamically Create an Object from Input
  Arrays
hidden: false
image:
  path: ''
  thumbnail: ''
  caption: ''
categories: []
tags: []
last_modified_at: 

---
So I had this situation today where data to be exported had to be dynamically assigned Header Names (Excel export). Am using the Infragistics Angular [Excel Exporter](https://www.infragistics.com/products/ignite-ui-angular/angular/components/exporter_excel.html).

The data to be exported looks like this:

    const exportData = [
    	
    ];

There's equally an array with column properties that looks like this: