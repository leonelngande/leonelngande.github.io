---
layout: post
hidden: false
title: Specify a default value for Netlify CMS string widget
author: Leonel Elimpe
tags:
  - Netlify CMS
  - Blogging
  - Jekyll
---
To specify a default value for Netlify CMS's `string` widget, add the `default` property to it's definition and assign it a value, as follows:

```yaml
# ...

collections:
  - name: "post"
    # ..

    fields:
      - {label: "Author", name: "author", widget: "string", default: "Leonel Elimpe"}
      # ..
```

As seen above in my posts collection, I've added an `author` field and given it a `default` value.

Have a question you'd like cleared up? Please use the comments, or DM me on [Twitter](https://twitter.com/leonelngande).
