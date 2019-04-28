---
layout: post
hidden: false
title: Format and Display a Javascript Date Object as Date and Time
tags: []
---
```typescript
cellRenderer: ( data ) => {
    return data.value ? (new Date(data.value)).toLocaleDateString() + ' ' + (new Date(data.value)).toLocaleTimeString() : '';
}
```
