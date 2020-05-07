---
layout: post
hidden: false
title: "Angular: Thinking in pages"
series: ""
tags:
  - Angular
  - Beginner
---
* [Background](#background)
* [What is a page?](#definition)
* [Pages in Angular](#pages-in-angular)

  * [The Angular CLI approach](#angular-cli)
  * [The Ionic CLI approach](#ionic-cli)
* [Conclusion](#conclusion)

## <a name="background"></a>Background

When I started out with Angular, I had a hard time wrapping my head around building websites with `components`. I had a hard time answering the question: 

*How do I structure my components to best bring out my website structure?*

Coming from a [LAMP](https://stackoverflow.com/questions/10060285/what-is-a-lamp-stack) background, I was used to building websites around the concept of `pages`.  A homepage, a services page, a resource page, an about page, a contact page, etc. 

## <a name="definition"></a>What is a page?

We'll use excerpts from [this](https://sharepointmaven.com/sharepoint-sites-pages-web-parts/) resource on Sharepoint pages. A page is just a means to display and organize content on a given site and present it to the end user. The content itself consists of `web parts` – smaller units used to store particular content/information (i.e. documents, events, contacts). Here's a quick summary:

* **Sites**– used to **organize** various types of content (web parts)
* **Pages**– used to **display** content (web parts) on a site
* **Web Part**– used to **store** particular content/information (i.e. documents, events, contacts, etc).

## <a name="pages-in-angular"></a>Pages in Angular

Say we're creating a website front-end with Angular that has an `employees` resource, on which the following operations are permitted: `list`, `view`, `create`, `update`, and `delete`.

As with a traditional PHP/Rails/Django implementation, and also by applying our above breakdown of what a page is, we'll have the following pages:

* list employees page - content related to listing employee records
* view employee page - content related to viewing an employee record
* create employee page - content related to creating an employee record
* update employee page - content related to updating an employee record.

Where applicable, each page can further contain any of the following web parts.

* delete employee button
* view employee button
* create employee button
* update employee button
* employee form

Putting it all together, we could have the following page structure.

Now the fun part, using our above breakdown to create actual pages and components in Angular. We'll create a routable (lazy loaded) module page components out of the pages, and re-usable standard components out of the web parts.

### <a name="page-components"></a>Creating the pages

With the Angular CLI, page components are routable components. To create our above pages, we'll run the following commands:

When you create a page with the Angular CLI, 

// TODO: Build a full app with instructions at this point, step by step. Like on the chrome extension with angular blog post. Show how to create a routable module, then a page component within that module. do this for each of our above pages. Then create standalone components as per Angular 9 (no registration in modules) for each of our web parts above.

### <a name="standard-components"></a>Creating the re-usable components

sfdsfd

## <a name="conclusion"></a>Conclusion

sfsdfsdf