---
layout: post
hidden: false
title: Two Ways to get a Form Input's Value in Cypress
author: Leonel Elimpe
tags:
  - Cypress
last_modified_at: 2021-01-15 05:25:31
---
Given the below form, how do we get any individual input's value inside a Cypress test?

```html
<form>
    <input type="text"
           name="code"
           data-cy="code">
                                
    <input type="tel"
           name="phone"
           data-cy="phone">
</form>
```

<br>

## Using Cypress's API methods

If you need to hold a reference to the value, you can query it using one of the methods below. 

```javascript
cy.get('[data-cy=code]').should(($input) => {
  const value = $input.val();
    
  console.log(value); // do something with the value
    
})
```

```javascript
cy.get('[data-cy=code]').invoke('val').should((value) => {

    console.log(value);

})
```

<br>

## Querying the DOM

You can also retrieve an input's value by querying the DOM directly.

```javascript
cy.document().then((doc) => {
    const value = doc.querySelector('[data-cy=code]').value;
    
    console.log(value);
});
```

Note that to access the test page's [document](https://developer.mozilla.org/en-US/docs/Web/API/Document), we have to use the `cy.document()` command to obtain a reference. The `document` you access as a global belongs to the Cypress runner, but the test page is inside an iFrame which has a different document reference.

<br>

### Further reading

* [How do I get an input’s value? - Cypress Documentation](https://docs.cypress.io/faq/questions/using-cypress-faq.html#How-do-I-get-an-input%E2%80%99s-value)
* [document.querySelectorAll doesn't work in cypress - Stackoverflow](https://stackoverflow.com/a/60440805/6924437)