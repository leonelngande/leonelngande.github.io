---
layout: post
hidden: false
title: "Netlify: Redirect Users by Country or Language"
author: Leonel Elimpe
tags:
  - Netlify
date: 2020-06-23 10:53:19
last_modified_at: 2020-06-23 10:53:20
---
Today at [Switchn](https://switchn.net) we had the need to redirect users by country and browser language on our Netlify hosted site.  That is, send those visiting our site from English speaking countries to `/en/` and those from French speaking countries to `/fr/`. Also if the user's browser sends an `accept-language` header in the request (e.g. `accept-language: en-US,en;q=0.9`), we'd like to take that into consideration too.

I had the thought, isn't it possible the awesome engineers at Netlify have thought of this use case for their customers? Well, surprise!! They have.

* [Redirect by country or language - Netlify docs](https://docs.netlify.com/routing/redirects/redirect-options/#redirect-by-country-or-language)

<br>

## Netlify Support links

* [Do language based redirects take into account browser’s language?](https://community.netlify.com/t/do-language-based-redirects-take-into-account-browsers-language/2577)
* [Redirect by browser language](https://community.netlify.com/t/redirect-by-browser-language/4849)

<br>

Here's an example _redirects file configuration:

```
# Redirect API requests to your API server
/api/* https://api.your-domain-name.com/:splat

# Redirect users with English language (browser) preference to the English site
/* /en/index.html 200  Language=en

# Redirect users with french language (browser) preference to the french site
/* /fr/index.html 302 Language=fr

# If browser-specific language preference is not available,
# redirect by default to the English site
/* /en/index.html 200
```

Please note I've not tested the above configuration with a live site, I might later add a section with the configuration we end up using.