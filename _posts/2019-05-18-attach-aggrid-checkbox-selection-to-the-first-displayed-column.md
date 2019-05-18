---
layout: post
hidden: false
title: Attach agGrid Checkbox Selection to the First Displayed Column
tags:
  - Angular
  - agGrid
  - agGrid Checkbox Selection
---
From the agGrid [documentation](https://www.ag-grid.com/javascript-grid-selection/#header-checkbox-selection), 

`colDef.checkboxSelection can also be a function that returns true/false - use this if you want only checkboxes on some rows but not others. gridOptions.checkboxSelection can also be specified as a function - use this if you want, for example, the first column to have checkbox selection regardless of which column it is (you would do this by looping the columns using the column API, and check if the first column is the current one (in checkboxSelection).` 

The described looping and setting of checkboxSelection can be done as follows:

```typescript
const columnDefs = [
    {
        headerName: 'My Header',
        field: 'myHeader',
        width: 100,
        checkboxSelection: function(params) {
            const displayedColumns = params.columnApi.getAllDisplayedColumns();
            return displayedColumns[0] === params.column;
        },
    },  
];
```

This is based on the headerCheckboxSelection function solution provided in the [agGrid documentation](https://www.ag-grid.com/javascript-grid-selection/#header-checkbox-selection).
