---
layout: post
hidden: false
title: Testing File Download Links with Cypress
author: Leonel Elimpe
excerpt: Say my homepage contains the below snippet, which allows a user to download vcard files. I'd like to make sure the file download links are valid and point to existing files.
tags:
  - Cypress
last_modified_at: 2021-01-13 08:14:53
---
Say my homepage contains the below snippet, which allows a user to download vcard files. I'd like to make sure the file download links are valid and point to existing files.

```html
<a href="./assets/vcards/mtn.vcf"
   data-cy="mtn-vcard"
   download>
    MTN: 679 38 54 15
</a>
<a href="./assets/vcards/orange.vcf"
   data-cy="orange-vcard"
   download>
    Orange: 655 15 42 48
</a>
```

Here's how I would accomplish this with Cypress.

```javascript
describe("Home page test", () => {

  it("can click on and download the MTN and Orange vcards", () => {
    cy.visit("/");

    /**
     * To access your test page's document, you should use the cy.document() command to obtain a reference.
     * The document you access as a global belongs to the Cypress runner,
     * but the test page is inside an iFrame which has a different document reference.
     *
     * @see https://stackoverflow.com/a/60440805/6924437
     */
    cy.document().then((doc) => {
      // Query the mtn vcard and get it's href
      const mtnVcardUrl = doc.querySelector('[data-cy=mtn-vcard]').getAttribute('href');
      /**
       * Make an http request with the vcard url
       * @see https://docs.cypress.io/api/commands/request.html#Get-Data-URL-of-an-image
       */
      cy.request({
        url: mtnVcardUrl,
        encoding: 'base64'
      })
        .then((response) => {
          // Make sure the request is successful
          expect(response.status).to.equal(200);
        });

      const orangeVcardUrl = doc.querySelector('[data-cy=orange-vcard]').getAttribute('href');
      cy.request({
        url: orangeVcardUrl,
        encoding: 'base64'
      })
        .then((response) => {
          expect(response.status).to.equal(200);
        });
    });

  })
    
});

```

