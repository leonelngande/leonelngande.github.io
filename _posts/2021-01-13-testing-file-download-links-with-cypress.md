---
layout: post
hidden: false
title: Testing File Download Links with Cypress
author: Leonel Elimpe
excerpt: Say my homepage contains the below snippet, which allows a user to
  download vcard files. I'd like to make sure the file download links are valid
  and point to existing files.
tags:
  - Cypress
last_modified_at: 2021-01-13 08:14:53
---
Say my homepage contains the below snippet, which allows a user to download vcard files. I'd like to make sure the file download links are valid and point to existing files.

```html
<a href="./assets/vcards/mtn.vcf"
   data-cy="mtn-vcard"
   download>
    Download MTN Vcard
</a>
<a href="./assets/vcards/orange.vcf"
   data-cy="orange-vcard"
   download>
    Download Orange Vcard
</a>
```

Here's how I would accomplish this with Cypress.

```javascript
describe("Home page test", () => {

  it("can click on and download the MTN and Orange vcards", () => {
    cy.visit("/");

    // 0️⃣
    cy.document().then((doc) => {
      // 1️⃣
      const mtnVcardUrl = doc.querySelector('[data-cy=mtn-vcard]').getAttribute('href');
      // 2️⃣
      cy.request({
        url: mtnVcardUrl,
        encoding: 'base64'
      })
        .then((response) => {
          // 3️⃣
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

- 0️⃣: To access your test page's document, you should use the `cy.document()` command to obtain a reference. The document you access as a global belongs to the Cypress runner, but the test page is inside an iFrame which has a different document reference. Visit this [Stackoverflow answer](https://stackoverflow.com/a/60440805/6924437) to learn more.
- 1️⃣: Query the MTN vcard and get it's `href`.
- 2️⃣: Make an HTTP request with the vcard URL. For more on making http requests, visit the [Cypress documentation](https://docs.cypress.io/api/commands/request.html#Get-Data-URL-of-an-image).
- 3️⃣: Make sure the request is successful. You can further read and verify the file extension from the base64 encoded file string in the response body.