---
layout: post
hidden: false
title: Straightforward Guide to Adding Bootstrap 4 Styles in an Angular Project
tags: []
---
FYI, this is a to the point guide on adding and Bootstrap 4 styles (scss) to an Angular project.

1. Add Bootstrap 4. This is done with `npm install bootstrap --save` 
2. In your styles.scss (usually src/styles.scss though you may have a custom structure), import [bootstrap-reboot](https://getbootstrap.com/docs/4.3/content/reboot/), [bootstrap-grid](https://getbootstrap.com/docs/4.3/layout/grid/), [tables](https://getbootstrap.com/docs/4.3/content/tables/), and [utilities](https://getbootstrap.com/docs/4.3/layout/utilities-for-layout/) as below:

```scss
@import '~bootstrap/scss/bootstrap-reboot';
@import '~bootstrap/scss/bootstrap-grid';
@import '~bootstrap/scss/tables';
@import '~bootstrap/scss/utilities';
```

Note that you can leave out any of these in case you're using bootstrap to complement functionality missing in another css framework you're using, say Angular Material for instance.

After doing the above, you're good to go with Bootstrap's rich list of utility, layout, and grid classes in your project. Feel free to import any other styles you require, you can find all under `'~bootstrap/scss/'`  in the node_modules folder.

Visit [this github repository](https://github.com/leonelngande/bootstrap4-styles-angular) for a sample project.



Happy coding!
