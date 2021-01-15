---
layout: post
hidden: false
title: Using Cypress to Test Input Fields in a Page Prerendered with Scully
author: Leonel Elimpe
tags:
  - Cypress
  - Scully
last_modified_at: 2021-01-15 03:05:53
---
When testing form inputs in pages prerendered with Scully, Cypress seems to detect the input field twice, once when the prerendered HTML is loaded, and a second time when Angular finishes loading and re-renders the page.

For example, say I have a page that is used to verify phone numbers. The page's URL can contain query parameters `code` and `phone`, that should be read and appropriate form input fields auto filled. The form looks something like this (simplified):

```html
<form>
    <input
           formControlName="code"
           type="text"
           matInput
           name="code"
           data-cy="code">
  
    <input formControlName="phone"
           type="tel"
           name="phone"
           data-cy="phone">
</form>
```

Below is how I would write Cypress tests for the described functionality.

```javascript
describe("Phone verification page test", () => {
  it("can bind query parameters to appropriate form input fields", () => {
    const code = 'SOME-CODE';
    const phone = '6xxxxxxxx';
    cy.visit(`/phone-verification?code=${code}&phone=${phone}`);

    cy.get('[data-cy=code]')
        .should('have.value', code);
      
    cy.get('[data-cy=phone]')
        .should('have.value', phone);
  });

});
```

If the page is not prerendered, the tests pass with flying colors. 

After the page is prerendered however, these tests fail due to Cypress finding more than one input element for each field, as mentioned at the beginning.

I don't fully understand why this happens, if you do, please let me know by reaching out or leaving a comment so I update the post.

To get the tests passing, I had to update them as below.

```javascript
describe("Verification page test", () => {
  it("can bind query parameters to appropriate form input fields", () => {
    const code = 'SOME-CODE';
    const phone = '6xxxxxxxx';
    cy.visit(`/phone-verification?code=${code}&phone=${phone}`);

    cy.get('[data-cy=code]')
        .last() // 1️⃣
        .should('have.value', code);
      
    cy.get('[data-cy=phone]')
        .last() // 2️⃣
        .should('have.value', phone);
  });

});
```

Notice I've added `.last()` to the `cy.get()` query so it returns the latest input field, the one created after Angular loaded and re-rendered the page.