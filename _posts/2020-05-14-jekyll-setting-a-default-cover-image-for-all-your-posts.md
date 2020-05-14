---
layout: post
hidden: false
title: "Jekyll: Setting a default cover image for all your posts"
author: Leonel Elimpe
tags:
  - Jekyll
  - Blogging
---
This post assumes you're currently able to add an `image` to your post front matter, for example:

```
---
layout: post
title: Example post title
image: /assets/images/lorem-ipsum.png
tags:
  - Tag 1
  - Tag 2
author: Lorem Ipsum
---
```

A default cover image in this context refers to an image that appears above your post content, and will be added should you not include one in your post's front matter (we've included one in the above example). 

To set a default cover image for all your posts, open the config.yml file at the root of your project, and add the following entry to the `defaults` front matter:

```
# ...

# Front Matter Defaults
defaults:
  # Posts defaults
  - scope:
      path: "" # an empty string here means all files in the project
      type: posts # Apply the default values below only to posts
    values:
      image: /assets/images/some-default-image.png

# ...

```

## Background on Front Matter Defaults

[[Credits/Source](https://jekyllrb.com/docs/configuration/front-matter-defaults/)]

Using [front matter](https://jekyllrb.com/docs/front-matter/) is one way that you can specify configuration in the pages and posts for your site. Setting things like a default layout, or customizing the title, or specifying a more precise date/time for the post can all be added to your page or post front matter.

Often times, you will find that you are repeating a lot of configuration options. Setting the same layout in each file, adding the same category - or categories - to a post, etc. You can even add custom variables like author names, which might be the same for the majority of posts on your blog.

Instead of repeating this configuration each time you create a new post or page, Jekyll provides a way to set these defaults in the site configuration. To do this, you can specify site-wide `defaults` using the defaults key in the `_config.yml` file in your project’s root directory.

The `defaults` key holds an array of scope/values pairs that define what defaults should be set for a particular file path, and optionally, a file type in that path.

Read more [here](https://jekyllrb.com/docs/configuration/front-matter-defaults/).