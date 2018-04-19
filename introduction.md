---
title: Introduction
seoTitle: Builder Book
seoDescription: Learn web development by building a modern web application from scratch with React, Material-UI, Next, Express, Mongoose, MongoDB. Perfect for developers who know the basics of HTML, CSS, JavaScript, and React and are ready to build a complete web app.
isFree: true
---

Introduction
------------

*   Table of Contents  
*   Why this book?  
*   Who is this book for?  
*   What will you learn?  
*   Project structure  
*   Screenshots  
    *   Customer pages
    *   Admin pages
*   Authors  

* * *

[_link_](#table-of-contents) Table of Contents
----------------------------------------------

In this book, you'll build a full-stack JavaScript web application from scratch, using ES6 syntax. The final app can be used to publish and sell books. We use it ourselves for this book!

Below is a list of all chapters and the main topics covered. Click any title to see a free preview of that chapter. You can also access the Table of Contents by clicking the menu icon at the top left.

Book Chapters:

1.  [App structure. Next.js. HOC. Material-UI. Server-side rendering. Styles.](https://builderbook.org/books/builder-book/app-structure-next-js-hoc-material-ui-server-side-rendering-styles)
2.  [Server. Database. Session. Header and MenuDrop components.](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components)
3.  [Authentication HOC. Promise. Async/await. Static method for User model. Google OAuth.](https://builderbook.org/books/builder-book/authentication-hoc-promise-async-await-static-method-for-user-model-google-oauth)
4.  [Testing with Jest. Debugging with Winston. Transactional emails. In-app notifications.](https://builderbook.org/books/builder-book/testing-with-jest-debugging-with-winston-transactional-emails-in-app-notifications)
5.  [Book and Chapter models. Internal API. Render chapter.](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter)
6.  [Github integration. Admin dashboard. Testing Admin UX and Github integration.](https://builderbook.org/books/builder-book/github-integration-admin-dashboard-testing-admin-ux-and-github-integration)
7.  [Table of Contents. Highlight for section. Hide Header. Mobile browser.](https://builderbook.org/books/builder-book/table-of-contents-highlight-for-section-hide-header-mobile-browser)
8.  [BuyButton component. Buy book logic. ReadChapter page. Checkout flow. MyBooks page. Mailchimp API. Deploy app.](https://builderbook.org/books/builder-book/buybutton-component-buy-book-logic-readchapter-page-checkout-flow-mybooks-page-mailchimp-api-deploy-app)

* * *

[_link_](#why-this-book-) Why this book?
----------------------------------------

_"What I cannot create, I do not understand." Richard Feynman_

Now is the best time to learn how to code - the number of learning resources has never been so high. You can choose from numerous reference guides, video courses, blog posts, books, and online boards (e.g. Stack Overflow).

We personally used (and still use) all of these resources while learning web development. An abundance of learning resources is a good thing, but it does not guarantee that you will become a good engineer. An abundance of resources and limited time force you to choose resources carefully. In other words, learning _how to learn_ and _what to learn_ has never been as important as it is today.

If you just started learning web development, then reference guides, video courses, and tutorials will help you get familiar with basic principles and syntax. Great resources will show you not only _how_ to do something but also explain _why_.

Once you are familiar with principles and some syntax, the next step is to build your own web application. At this point, you've learned how to think logically like a developer, so you'll enjoy learning backend engineering as much as frontend. You will still use reference guides and other resources when you forget something or need to learn a new concept. However, despite the abundance of resources to learn the basics, the number of resources that teach you how to build a modern, large-scale, production-ready web application _from scratch_ is relatively low.

We don't argue that everyone who learns coding should start with building a large project. But we do strongly suggest that developers - who have learned the basics and want to take the next step in advancing their web development skills - build a complete application from scratch. Since knowledge is organized as a tree-like structure, building one large web application will force you to glue all the disconnected pieces of knowledge you've learned into one, tree-like, systematic structure.

Say you've read about HTML, CSS, and JavaScript. You've followed tutorials, solved exercises, perhaps even cloned some large apps from Github and edited the code to learn how it works. You've learned a lot! However, at this point, the knowledge that you've acquired is fragmented. You've learned pieces of information here and there, but you haven't seen all that information used in one application. This fragmented knowledge is easy to forget and hard to apply. Building a large web application from scratch will help you see all of your knoweldge put together. Thus, our book.

This book walks you through building a modern web application from scratch. We wrote it because we wish we had a similar resource when we were learning how to build web apps. You will start from 0 lines of code in Chapter 1 and end up with over 12,000 lines of code by Chapter 8. In the end, you'll have your own production-ready web app, which you can add to your portfolio or even use to start a business.

You may have learned about server-side rendering, session, cookie, API endpoint, internal API, GET/POST methods, Promise, async/await, model, schema, index, routing, express, http, request/response, server/client, third-party APIs, lifecycle hooks, OAuth, and many other concepts. But have you ever put them all together into one working application?

To rephrase one popular meme

Do you even lift, bro?

would be

Have you ever built a production-ready web application from scratch by yourself?

[_link_](#who-is-this-book-for-) Who is this book for?
------------------------------------------------------

This book assumes basic knowledge of HTML, CSS, React, and JavaScript. However, we've aimed to explain every line of code in our book. In cases where we do not describe a feature of JavaScript, React, or frameworks and packages that we use, we provide links for you to learn about them.

We provide a free preview for every chapter to help you decide whether this book is for you. You can see the following for free in each chapter:

*   Detailed table of contents with sections and subsections
*   Complete codebases for the chapter's start and end
*   List of packages discussed in the chapter
*   A preview of the chapter's beginning content

To see a detailed list of all the technologies that you will learn, check out [package.json](https://github.com/builderbook/builderbook/blob/master/package.json).

This book will be a great resource if you have a basic level of experience with React and JavaScript. Perhaps you worked on a large project with someone but never built one by yourself. Perhaps you wrote an asynchronous callback using `Promise.then` but you prefer `async/await`. If so, this book will help you organize many unrelated concepts that you've learned from reference guides and tutorials into a single system.

[_link_](#what-will-you-learn-) What will you learn?
----------------------------------------------------

In this book, you will build your own production-ready web application (see the project structure below). Throughout the book, you will:

*   get familar with React, Next.js, and Material-UI
*   create an Express server and Session
*   connect your app to MongoDB with the help of Mongoose
*   add Google OAuth 2.0 for user authentication
*   integrate your app with third-party applications: Github, Stripe, AWS SES, and Mailchimp
*   create User, Book, Chapter, EmailTemplate, and Purchase models
*   write dozens of static methods for these models, as well as for Express routes and API methods
*   write dozens of pages, components, and more

You could spend weeks searching these topics on Google. Our book puts everything about building a web app into one place.

[_link_](#project-structure) Project structure
----------------------------------------------

Take a look at the [final code](https://github.com/builderbook/builderbook) and the [codebase for each chapter](https://github.com/builderbook/builderbook/tree/master/book).

In total, you will write more than 12,000 lines code over 8 chapters. Your final web app will have the following structure:

    .
    ├── components                  # React components
    │   ├── admin                   # Components used on Admin pages
    │   │   ├── EditBook.js         # Edit title, price, and repo of book
    │   ├── customer                # Components used on Customer pages
    │   │   ├── BuyButton.js        # Buy book
    │   ├── Header.js               # Header component
    │   ├── MenuDrop.js             # Dropdown menu
    │   ├── Notifier.js             # In-app notifications for app's users
    │   ├── SharedStyles.js         # List of _reusable_ styles
    ├── lib                         # Code available on both client and server
    │   ├── api                     # Client-side API methods
    │   │   ├── admin.js            # Admin user methods
    │   │   ├── customer.js         # Customer user methods
    │   │   ├── getRootURL.js       # Returns ROOT_URL
    │   │   ├── public.js           # Public user methods
    │   │   ├── sendRequest.js      # Reusable code for all GET and POST requests
    │   ├── context.js              # Context for Material-UI integration
    │   ├── notifier.js             # Contains notify() function that loads Notifier component
    │   ├── withAuth.js             # HOC that passes user to pages and more
    │   ├── withLayout.js           # HOC for SSR with Material-UI and more
    ├── pages                       # Pages
    │   ├── admin                   # Admin pages
    │   │   ├── add-book.js         # Page to add a new book
    │   │   ├── book-detail.js      # Page to view book details and sync content with Github
    │   │   ├── edit-book.js        # Page to update title, price, and repo of book
    │   │   ├── index.js            # Main Admin page that has all books and more
    │   ├── customer                # Customer pages
    │   │   ├── my-books.js         # Customer's dashboard
    │   ├── public                  # Public pages (accessible to logged out users)
    │   │   ├── login.js            # Login page
    │   │   ├── read-chapter.js     # Page with chapter's content
    │   ├── _document.js            # Allows to customize pages (feature of Next.js)
    │   ├── index.js                # Homepage
    ├── server                      # Server code
    │   ├── api                     # Express routes, route-level middleware
    │   │   ├── admin.js            # Admin routes
    │   │   ├── customer.js         # Customer routes
    │   │   ├── index.js            # Mounts all Express routes on server
    │   │   ├── public.js           # Public routes
    │   ├── models                  # Mongoose models
    │   │   ├── Book.js             # Book model
    │   │   ├── Chapter.js          # Chapter model
    │   │   ├── EmailTemplate.js    # Email Template model
    │   │   ├── Purchase.js         # Purchase model
    │   │   ├── User.js             # User model
    │   ├── utils                   # Server-side utilities
    │   │   ├──slugify.js           # Generates slug for any Model
    │   ├── app.js                  # Custom Express/Next server
    │   ├── aws.js                  # AWS SES API
    │   ├── github.js               # Github API
    │   ├── google.js               # Google OAuth API
    │   ├── logs.js                 # Logger
    │   ├── mailchimp.js            # MailChimp API
    │   ├── routesWithSlug.js       # Express routes that contain slug
    │   ├── sitemapAndRobots.js     # Express routes for sitemap.xml and robots.txt
    │   ├── stripe.js               # Stripe API
    ├── static                      # Static resources
    │   ├── robots.txt              # Rules for search engine bots
    ├── test/server/utils           # Tests
    │   ├── slugify.test.js         # Unit test for generateSlug() function
    ├── .babelrc                    # Config for Babel
    ├── .eslintrc.js                # Config for Eslint
    ├── .gitignore                  # List of ignored files and directories
    ├── env-config.js               # Make Stripe's public keys available on client
    ├── now.json                    # Settings for now from Zeit
    ├── package.json                # List of packages and scripts
    ├── yarn.lock                   # Exact versions of packages. Generated by yarn.
    

[_link_](#screenshots) Screenshots
----------------------------------

In this book, you will build your own [Builder Book app](https://github.com/builderbook/builderbook). This website itself is a Builder Book app! You are welcome to add the final app you build in this book to your portfolio or resume. You can even use it to start a business.

The main use cases for this app, besides learning, are:

*   to write and host free documentation
*   to write, host, and sell books

The app has two primary users, `Customer` and `Admin`:

*   the Customer user reads chapters (for free or buys a book)
*   the Admin user writes content and hosts content (free or paid) on his/her website

The app uses both server-side and client-side rendering. For initial load, pages are rendered by the server; for subsequent loads, pages are rendered on the client.

Below are screenshots of the main pages you'll build for your app.

#### [_link_](#customer-pages) Customer pages

*   Chapter excerpts: this page shows some free, sample content to visitors and registered users who have not bought a book yet.

![Builder Book](https://user-images.githubusercontent.com/26158226/38517453-e84a7566-3bee-11e8-82cd-14b4dfbe6a78.png)

*   Purchased book content: after logging in (with Google) and purchasing the book (via Stripe), the Customer can see the full content of the chapters.

![Builder Book](https://user-images.githubusercontent.com/26158226/38518394-9ee97306-3bf1-11e8-8df2-8c05fb75249a.png)

#### [_link_](#admin-pages) Admin pages

*   Add/edit book: to create a book, the Admin chooses a price, writes a title, and selects a Github repo that contains the book content.

![Builder Book](https://user-images.githubusercontent.com/26158226/38517449-e5faaa38-3bee-11e8-9c02-740096dc860e.png)

*   Book details: to update the content of a book, the Admin syncs with content hosted on a Github repo.

![Builder Book](https://user-images.githubusercontent.com/26158226/38517450-e7005bd0-3bee-11e8-9916-81f32d3d1827.png)

[_link_](#authors) Authors
--------------------------

Together, we've built [Builder Book](https://github.com/builderbook/builderbook) and [Harbor](https://github.com/builderbook/harbor) web apps.  
Stay tuned for our next open-source app, [Async](https://github.com/async-labs/saas-boilerplate).

*   [Kelly Burke](https://github.com/klyburke)
*   [Timur Zhiyentayev](https://github.com/tima101)
