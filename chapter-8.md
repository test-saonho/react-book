---
title: BuyButton component. Buy book logic. ReadChapter page. Checkout flow. MyBooks page. Mailchimp API. Deploy app.
seoTitle: BuyButton component. Buy book logic. ReadChapter page. Checkout flow. MyBooks page. Mailchimp API. Deploy app.
seoDescription: This book teaches you how to build a production-ready web application from scratch using React, Material-UI, Next, Express, Mongoose, MongoDB. In Chapter 8, you will write API methods, Express routes, and static methods to support 'buy book' logic. You will integrate your app with Stripe and Mailchimp and learn to deploy the app with Now.
isFree: true
---

* * *
*   BuyButton component  
*   Buy book logic  
    *   API method buyBook()
    *   Express route '/buy-book'
    *   Static method Book.buy()
    *   stripeCharge() function
    *   Purchase model
*   ReadChapter page  
    *   Excerpt and full content
    *   BuyButton
    *   Testing
*   Checkout flow  
*   MyBooks page  
    *   API method getMyBookList()
    *   Express route '/my-books'
    *   Static method Book.getPurchasedBooks()
*   Mailchimp API  
*   Deploy app  
    *   Prepare server
    *   Env keys
    *   Admin user
    *   Now
* * *

Before you start working on Chapter 8, get the `8-start` codebase. The [8-start](https://github.com/builderbook/builderbook/tree/master/book/8-start) folder is located at the root of the `book` directory inside the [builderbook repo](https://github.com/builderbook/builderbook).

*   If you haven't cloned the builderbook repo yet, clone it to your local machine with `git clone git@github.com:builderbook/builderbook.git`.
*   Inside the `8-start` folder, run `yarn` to install all packages.

These are the packages and their versions that we install specifically for Chapter 8:

*   `"react-stripe-checkout": "^2.6.3"`
*   `"sitemap": "^1.13.0"`
*   `"stripe": "^5.4.0"`
*   `"helmet": "^3.12.0"`
*   `"compression": "^1.7.2"`
*   `"babel-plugin-transform-define": "^1.3.0"`

Remember to include your `.env` at the root of your app. By the end of Chapter 8, you will add:

*   `Stripe_Test_SecretKey` and `Stripe_Live_SecretKey`,
*   `Stripe_Test_PublishableKey` and `Stripe_Live_PublishableKey`,
*   `MAILCHIMP_API_KEY`,`MAILCHIMP_REGION`, and `MAILCHIMP_PURCHASED_LIST_ID` environmental variables to your `.env` file.

* * *
  
In the previous chapter (Chapter 7), we made many UX improvements to our `ReadChapter` page. In this chapter, we will add all code related to selling a book. Here is a high-level overview of what we want to accomplish:

*   introduce a `BuyButton` component
*   define all necessary code to make the buy button functional: API method, Express route, and static methods for our Book model, Stripe API, and Purchase model
*   add a `BuyButton` component to our `ReadChapter` page and test the checkout flow
*   show books that a user bought on the `MyBooks` page (user dashboard)
*   add the email of a user who bought a book to a mailing list on Mailchimp

Like with any other new feature that needs to GET or POST data, the flow of data is usually as follows:

*   there is a _page or component_ that calls an _API method_
*   the _API method_ sends a request to a particular route, which executes a matching _Express route_
*   the function inside the matching _Express route_ calls a static method for a model, waits for data, and returns data to the client

By using this blueprint above, we've already implemented data exchange between client and server many times in this book. In Chapters 5 and 6, we added many pages/components, API methods, Express routes, and static methods related to our Admin user. In this section, we will write code related to our _Customer_ user, who buys and reads a book.

Our first step, as per the blueprint, is to create a `BuyButton` component that we eventually import and add to our `ReadChapter` page. Inside this component, we will call the `buyBook()` API method.

## BuyButton component
--------------------------------------------------

In Chapter 2, we wrote our `Header` component as a stateless functional component ([link](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#update-header-component)) and `MenuDrop` component as a regular component ([link](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#menudrop-component)).

The `BuyButton` component will have both props and state. Thus, we should write it as a regular component. Check out `components/MenuDrop.js` to remember how we write regular components. To define our new `BuyButton` component with ES6 class:

class BuyButton extends React.Component { ... }

We export a single value with [default export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#Using_the_default_export):

export default BuyButton

Since the `BuyButton` component is for our Customer user only, create a `components/customer/BuyButton.js` file.  
The carcass of the `BuyButton` component with all necessary imports is:  
`components/customer/BuyButton.js` :

    import React from 'react';
    import PropTypes from 'prop-types';
    import StripeCheckout from 'react-stripe-checkout';
    import NProgress from 'nprogress';
    
    import Button from 'material-ui/Button';
    
    import { buyBook } from '../../lib/api/customer';
    import notify from '../../lib/notifier';
    
    const styleBuyButton = {
      margin: '20px 20px 20px 0px',
      font: '14px Muli',
    };
    
    class BuyButton extends React.Component {
      // 1. propTypes and defaultProps
    
      // 2. constructor (set initial state)
    
      // 3. onToken function
    
      // 4. onLoginClicked function
    
      render() {
        // 5. define variables with props and state
    
        if (!book) {
          return null;
        }
    
        if (!user) {
          return (
            // 6. Regular button with onClick={this.onLoginClicked} event handler
          );
        }
    
        return (
          // 7. StripeCheckout button with token and stripeKey parameters
        );
      }
    }
    
    export default BuyButton;
    

We discuss all code snippets in detail below.

1.  When we wrote our `Header` component as a stateless function (`components/Header.js`), we used `Header.propTypes` to specify types of props. But when we define a regular component with ES6 class, we specify types of props with `static propTypes`. For our `BuyButton` component, we should use the latter method, since we defined this component as a child ES6 class `class BuyButton extends React.Component`. You get:
    
        static defaultProps = {
         book: null,
         user: null,
         showModal: false,
        };
        
    
    We will define and use these three props later in this section.
    
2.  Similar to how there are two ways to specify types of props, there are two popular ways to set initial state. When you don't use a prop to set an initial state object, you can simply write the following _without_ using [constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor):
    
        state = {
         open: false,
         anchorEl: undefined,
        };
        
    
    We discussed `constructor` usage cases in detail in [Chapter 2](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#menudrop-component) and [Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#readchapter-page).
    
    Open `components/MenuDrop.js` to see how this was done for our `MenuDrop` component.
    
    However, in the case of our `BuyButton` component, we want to use a `showModal` prop to set the initial state. Thus, we should use `constuctor`:
    
        constructor(props) {
         super(props);
        
         this.state = {
           showModal: !!props.showModal,
         };
        }
        
    
    Later in this chapter, we plan to pass a `showModal` prop from the `ReadChapter` page to our `BuyButton` component. `showModal: !!props.showModal` ensures that when `showModal` is true in the `ReadChapter` page, then `showModal` is true in the `BuyButton` component as well.
    
    As we discussed [in Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#page), calling `super(props)` makes props available inside constructor.
    
3.  As discussed earlier in this book ([Chapter 1, SSR](https://builderbook.org/books/builder-book/app-structure-next-js-hoc-material-ui-server-side-rendering-styles#server-side-rendering)) \- for SSR, we call an API method inside `getInitialProps()`; for CSR, we call an API method inside `componentDidMount()` or inside an event-handling function.
    
    In the `BuyButton` component, we want to call a `buyBook()` API method after our user _takes an action_ (clicks the buy button). So we defintely don't want to do SSR and execute this `buyBook()` API method on the server. To execute `buyBook()` on the client, we would _normally_ place `buyBook()` inside `componentDidMount()`. However, if placed inside `componentDidMount()`, this API method will be called _right after the component mounts_ on the client. This is not what we want - we want to call our API method _on a click event_. Thus, we place `buyBook()` inside an `onToken` function that gets executed after a user clicks the buy button.
    
    We will point `onToken` to an async anonymous function and use the `try/catch` construct along with `async/await` (as we did many times before in this book). If you want to refresh your memory, [check out Chapter 3](https://builderbook.org/books/builder-book/authentication-hoc-promise-async-await-static-method-for-user-model-google-oauth#async-await). Let's write an async anonymous function that calls the `buyBook()` API method and uses `Nprogress` and `notify()` to communicate success to our user. You should get:
    
        onToken = async (token) => {
         NProgress.start();
         const { book } = this.props;
         this.setState({ showModal: false });
        
         try {
           await buyBook({ stripeToken: token, id: book._id });
           notify('Success!');
           NProgress.done();
         } catch (err) {
           NProgress.done();
           notify(err);
         }
        };
        
    
    When a user clicks the `BuyButton` component, our code calls the `onToken` function.
    
    We pass `book` prop from the `ReadChapter` page to the `BuyButton` component. (We'll discuss this more in the next section when we add `BuyButton` to `ReadChapter`.) Thus, constant `book` is defined as `this.props.book`. Using ES6 object destructuring:
    
    const { book } = this.props
    
    After a user clicks the buy button, we want to close the modal (Stripe modal with a form for card details):
    
    this.setState({ showModal: false })
    
    Then the code calls and waits for successful execution of our API method `buyBook()` that takes two arguments:
    
    await buyBook({ stripeToken: token, id: book._id });
    
    At this point, the rest of the code should be self-explanatory.
    
4.  Why do we need an `onLoginClicked` function? If a user is logged in, then we simply call the `onToken` function and `buyBook()` API method. However, if a user is _not_ logged in, we want to redirect the user to the `/auth/google` route (Google OAuth).
    
    Similar to the `book` prop, we pass the `user` prop from our `ReadChapter` page to the `BuyButton` component:
    
    const { user } = this.props
    
    We check if a `user` object exists, and it if does not exist or is empty, we redirect to Google OAuth:
    
        if (!user) {
         window.location.href = '/auth/google';
        }
        
    
    Put these two snippets together and you define `onLoginClicked`:
    
        onLoginClicked = () => {
         const { user } = this.props;
        
         if (!user) {
           window.location.href = '/auth/google';
         }
        };
        
    
5.  Define `book` and `user` constants ([block-scoped variables](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)) as `this.props.book` and `this.props.user`, respectively. Also define the `showModal` constant as `this.state.showModal`. Using ES6 object destructuring:
    
        const { book, user } = this.props;
        const { showModal } = this.state;
        
    
6.  If a user is _not_ logged in, we simply show a button from Material-UI. This button has an `onClick` event handler that points to `{this.onLoginClicked}`:
    
        <div>
         <Button
           variant="raised"
           style={styleBuyButton}
           color="primary"
           onClick={this.onLoginClicked}
         >
           Buy for ${book.price}
         </Button>
        </div>
        
    
7.  If a user is logged in, we again show a button, but now we wrap it with `<StripeCheckout>...</StripeCheckout>` from the [react-stripe-checkout](https://github.com/azmenak/react-stripe-checkout) package.
    
    Take a look at this [example of usage](https://github.com/azmenak/react-stripe-checkout#requirements). The `StripeCheckout` component _requires_ two props: `stripeKey` and `token`. Other props are optional, and you are familiar with all of them except `desktopShowModal`. This prop controls whether the Stripe modal is open or closed. Read more about [desktopShowModal](https://github.com/azmenak/react-stripe-checkout/pull/15).
    
    Wrap the Material-UI `<Button>` with `<StripeCheckout>`:
    
        <StripeCheckout
         stripeKey={StripePublishableKey}
         token={this.onToken}
         name={book.name}
         amount={book.price * 100}
         email={user.email}
         desktopShowModal={showModal || null}
        >
         <Button variant="raised" style={styleBuyButton} color="primary">
           Buy for ${book.price}
         </Button>
        </StripeCheckout>
        
    
    As you can see from above, Stripe requires us to pass a publishable key to our component _on the client_. To do so, we need to define the global variable `StripePublishableKey` and make this variable available to our `components/customer/BuyButton.js` file. Problem is that env variables inside `.env` are _only available on server_. To make some variable available on the client, we can use babel plugin `babel-plugin-transform-define`. With this plugin, we can create file that contains universally available variables. Universally, in this context, means on both - client and server. Below, we closely follow official example from Next.js on [universal configuration](https://github.com/zeit/next.js/tree/9320d9f006164f2e997982ce76e80122049ccb3c/examples/with-universal-configuration).
    
    Create `env-config.js` at the app root. This file contains universally available environmental variables:  
    `env-config.js` :
    
        const dev = process.env.NODE_ENV !== 'production';
        
        module.exports = {
         StripePublishableKey: dev
           ? process.env.Stripe_Test_PublishableKey
           : process.env.Stripe_Live_PublishableKey,
        };
        
    
    Remember to add `process.env.Stripe_Test_PublishableKey` and `process.env.Stripe_Live_PublishableKey` to `.env` file.
    
    Modify `.babelrc` so babel uses plugin:  
    `.babelrc` :
    
        {
        "presets": ["env", "next/babel"],
        "plugins": [["transform-define", "./env-config.js"]]
        }
        
    
    Open `components/customer/BuyButton.js`, add following line as very first line of the file:  
    `/* global StripePublishableKey */`
    
    Above syntax is to prevent Eslint from showing warning about undefined global variable.
    

Plug the code from steps 1-7 into our `BuyButton` component carcass that we outlined in the beginning of this section:

    /* global StripePublishableKey */
    
    import React from 'react';
    import PropTypes from 'prop-types';
    import StripeCheckout from 'react-stripe-checkout';
    import NProgress from 'nprogress';
    
    import Button from 'material-ui/Button';
    
    import { buyBook } from '../../lib/api/customer';
    import notify from '../../lib/notifier';
    
    const styleBuyButton = {
      margin: '20px 20px 20px 0px',
      font: '14px Muli',
    };
    
    class BuyButton extends React.Component {
      static propTypes = {
        book: PropTypes.shape({
          _id: PropTypes.string.isRequired,
        }),
        user: PropTypes.shape({
          _id: PropTypes.string.isRequired,
        }),
        showModal: PropTypes.bool,
      };
    
      static defaultProps = {
        book: null,
        user: null,
        showModal: false,
      };
    
      constructor(props) {
        super(props);
    
        this.state = {
          showModal: !!props.showModal,
        };
      }
    
      onToken = async (token) => {
        NProgress.start();
        const { book } = this.props;
        this.setState({ showModal: false });
    
        try {
          await buyBook({ stripeToken: token, id: book._id });
          notify('Success!');
          NProgress.done();
        } catch (err) {
          NProgress.done();
          notify(err);
        }
      };
    
      onLoginClicked = () => {
        const { user } = this.props;
    
        if (!user) {
          window.location.href = '/auth/google';
        }
      };
    
      render() {
        const { book, user } = this.props;
        const { showModal } = this.state;
    
        if (!book) {
          return null;
        }
    
        if (!user) {
          return (
            <div>
              <Button
                variant="raised"
                style={styleBuyButton}
                color="primary"
                onClick={this.onLoginClicked}
              >
                Buy for ${book.price}
              </Button>
            </div>
          );
        }
    
        return (
          <StripeCheckout
            stripeKey={StripePublishableKey}
            token={this.onToken}
            name={book.name}
            amount={book.price * 100}
            email={user.email}
            desktopShowModal={showModal || null}
          >
            <Button variant="raised" style={styleBuyButton} color="primary">
              Buy for ${book.price}
            </Button>
          </StripeCheckout>
        );
      }
    }
    
    export default BuyButton;
    

Above, we wrote the basic version of our `BuyButton` component. As we progress in this chapter, we will improve the UX of our checkout flow and make necessary changes to this component. We will make these improvements in the section called `Checkout flow`.

We will test out our `BuyButton` component at the end of the next section, after adding it to our `ReadChapter` page.

As you've probably noticed from the code above - after a user submits card details, we call the `onToken` function that calls a `buyBook()` API method. In the next section, we discuss this API method and its corresponding Express route and static method for the Book model. We also discuss Stripe API and create a new Purchase model.

## Buy book logic
----------------------------------------

In this section, we will discuss the `buyBook()` API method, Express route `/buy-book`, and static method `buy()` for our Book model. As you learned in Chapters 5 and 6:

*   a page or component calls an API method
*   the API method sends a request to an Express route
*   that Express route calls and waits for a static method
*   the static method fetches or updates data using Mongoose methods (aka Mongoose Queries)

#### API method buyBook()

Let's decide which type of method to use for a request sent by our `buyBook` API method. Since we send `book._id` (as `id`) and `token` (as `stripeToken`) to the _server_, the method should be POST. Because the method is POST, you don't need to specify it explicitly, as POST is a default method (check out `lib/api/sendRequest.js`, which we wrote [in Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#api-methods)).

Next, let's decide the route to use for the API endpoint; for example, let it be `/buy-book`.

If you don't remember how we wrote API methods, check out `lib/api/admin.js` and [Chapter 5, Section Internal APIs](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#internal-apis). For example, to display a list of books on our Admin dashboard:  
`lib/api/admin.js` :

    import sendRequest from './sendRequest';
    
    const BASE_PATH = '/api/v1/admin';
    
    export const getBookList = () =>
      sendRequest(`${BASE_PATH}/books`, {
        method: 'GET',
      });
    

Create a `lib/api/customer.js` file with the `buyBook` API method:  
`lib/api/customer.js` :

    import sendRequest from './sendRequest';
    
    const BASE_PATH = '/api/v1/customer';
    
    export const buyBook = ({ id, stripeToken }) =>
      sendRequest(`${BASE_PATH}/buy-book`, {
        body: JSON.stringify({ id, stripeToken }),
      });
    

Notice that we pass stringified `id` and `stripeToken` values to the request body. We will get these values from the request body when we write our Express route.

#### Express route '/buy-book'

First things first, we need to make sure that Customer-related API endpoints are indeed defined on the server. Open your `server/api/index.js` file and find the line `server.use('/api/v1/customer', customerApi);`:  
`server/api/index.js` :

    import publicApi from './public';
    import customerApi from './customer';
    import adminApi from './admin';
    
    export default function api(server) {
      server.use('/api/v1/admin', adminApi);
      server.use('/api/v1/public', publicApi);
      server.use('/api/v1/customer', customerApi);
    }
    

To remember how you wrote these Express routes, open up `server/api/admin.js`. Here is one Express route that calls and waits for the static method `Book.list()`:  
`server/api/admin.js` :

    import express from 'express';
    
    import Book from '../models/Book';
    import { getContent, getRepos } from '../github';
    import User from '../models/User';
    import logger from '../logs';
    
    const router = express.Router();
    
    router.use((req, res, next) => {
      if (!req.user || !req.user.isAdmin) {
        res.status(401).json({ error: 'Unauthorized' });
        return;
      }
    
      next();
    });
    
    router.get('/books', async (req, res) => {
      try {
        const books = await Book.list();
        res.json(books);
      } catch (err) {
        res.json({ error: err.message || err.toString() });
      }
    });
    
    export default router;
    

To write an Express route for our `buyBook` API method, keep in mind a few differences:

*   inside the Express middleware that checks for user permissions, we don't need to check for `!req.user.isAdmin`
*   the route is `/buy-book`
*   the method is `POST`; thus, use `router.post()` instead of `router.get()`
*   call `Book.buy()` instead of `Book.list()`
*   pass three parameters to the `buy()` static method: book id as `id`, Stripe token as `stripeToken`, and `user`
    
    We get the first two parameters from the request body `req.body` using ES6 object destructuring:
    
    const { id, stripeToken } = req.body;
    
    The third parameter is from `req.user` that we discussed in detail [in Chapter 3](https://builderbook.org/books/builder-book/authentication-hoc-promise-async-await-static-method-for-user-model-google-oauth#google-oauth-auth-function).
    

Go to your `server/api/customer.js` file.  
Take into account these specifics above and put together the Express middleware and Express route:  
`server/api/customer.js` :

    import express from 'express';
    
    import Book from '../models/Book';
    import logger from '../logs';
    
    const router = express.Router();
    
    router.use((req, res, next) => {
      if (!req.user) {
        res.status(401).json({ error: 'Unauthorized' });
        return;
      }
    
      next();
    });
    
    router.post('/buy-book', async (req, res) => {
      const { id, stripeToken } = req.body;
    
      try {
        await Book.buy({ id, stripeToken, user: req.user });
        res.json({ done: 1 });
      } catch (err) {
        logger.error(err);
        res.json({ error: err.message || err.toString() });
      }
    });
    
    export default router;
    

In a typical Express route with the `GET` method, we would send a response that contains some data from our database (for example, a `book` or `chapter` object). However, in a typical Express route with the `POST` method, we can simply respond with the object `done: 1` or `save: 1` or `success: 1`.

#### Static method Book.buy()

With the API method and Express route out of way, let's discuss the static method `buy()` in our Book model. This static method will do multiple things:

1.  use Mongoose's `findById()` to find a book
2.  check if a user already bought a book by searching (and waiting) for a `Purchase` document with `userId` and `bookId`
3.  if a user did not buy a book, call and wait for a `stripeCharge()` method defined at `server/stripe.js` (discussed later in this section)
4.  send an email to a user (same way we sent a welcome email in [Chapter 4](https://builderbook.org/books/builder-book/testing-with-jest-debugging-with-winston-transactional-emails-in-app-notifications#transactional-emails-with-aws-ses))
5.  finally, create a new `Purchase` document with all necessary parameters

High-level structure of our `buy` static method that takes the three arguments discussed earlier:  
`server/models/Book.js` :

    static async buy({ id, user, stripeToken }) {
      if (!user) {
        throw new Error('User required');
      }
    
      // 1. find book by id
    
      if (!book) {
        throw new Error('Book not found');
      }
    
      // 2. check if user bought book already
    
      // 3. call stripeCharge() method
    
      // 4. send transactional email confirming purchase
    
      // 5. create new Purchase document
    }
    

The only two snippets of code that we did not mention are those that throw errors in case `user` and `book` objects do not exist.

Let's discuss each step in detail:

1.  Open `server/models/Book.js` and find the static method `syncContent()` that finds a book by id using the Mongoose method `findById()`:
    
    const book = await this.findById(id, 'userId githubRepo githubLastCommitSha');
    
    In our case, instead of returning `userId githubRepo githubLastCommitSha` parameters, we want to return `name slug price`. Thus, the code for step 1 is simply:
    
    const book = await this.findById(id, 'name slug price');
    
2.  At this point in the code, we have both `user` and `book` objects. Here we use `user._id` from `user` and `id` from `book` to check if a `Purchase` document exists on our database. If a user bought a book already, we create a `Purchase` document in the `Purchase` collection of our database. It's important to check if this `Purchase` document exists, so a user does not buy a book twice. This is an unlikely edge case, but let's take care of it.
    
    Define the boolean parameter `isPurchased`, which is true if the count of found documents is greater than zero:
    
    const isPurchased = (await Purchase.find({ userId: user._id, bookId: id }).count()) > 0;
    
    If `isPurchased` is true, then we throw an error. Our code for step 2 is:
    
        const isPurchased = (await Purchase.find({ userId: user._id, bookId: id }).count()) > 0;
        if (isPurchased) {
         throw new Error('Already bought this book');
        }
        
    
3.  In this step, we pass three parameters to the `stripeCharge()` method. Parameters: `book.price` from `book` as `amount`, `stripeToken.id` from `stripeToken` (we passed this to the server from client via our `buyBook` API method) as `token`, `user.email` from `user` as `buyerEmail` (we define this method in the next subsection). Code for step 3:
    
        const chargeObj = await stripeCharge({
         amount: book.price * 100,
         token: stripeToken.id,
         buyerEmail: user.email,
        });
        
    
    For bookkeeping, we will save `chargeObj` as a parameter in the `Purchase` document in step 5.
    
4.  Remember how we send the transactional welcome email to new users - open `server/models/User.js` and find the static method `signInOrSignUp()` inside it:
    
        const template = await getEmailTemplate('welcome', {
         userName: displayName,
        });
        
        try {
         await sendEmail({
           from: `Kelly from Builder Book <${process.env.EMAIL_SUPPORT_FROM_ADDRESS}>`,
           to: [email],
           subject: template.subject,
           body: template.message,
         });
        } catch (err) {
         logger.error('Email sending error:', err);
        }
        
    
    The code for sending an email after a user bought a book will be very similar. Here are small differences to keep in mind:
    
    *   First, in addition to passing the `userName` parameter to `getEmailTemplate()`, we will pass `bookTitle` and `bookUrl`.
    *   Second, the template name is `purchase` instead of `welcome` (we don't want to send the welcome email again).
    *   Third, since we are not inside the User model but Book model, the value of `to` inside the `sendEmail()` method is `[user.email]` instead of `[email]`.
    
    With these change in mind, you will get the following code for step 4:
    
        const template = await getEmailTemplate('purchase', {
         userName: user.displayName,
         bookTitle: book.name,
         bookUrl: `${ROOT_URL}/books/${book.slug}/introduction`,
        });
        
        try {
         await sendEmail({
           from: `Kelly from builderbook.org <${process.env.EMAIL_SUPPORT_FROM_ADDRESS}>`,
           to: [user.email],
           subject: template.subject,
           body: template.message,
         });
        } catch (error) {
         logger.error('Email sending error:', error);
        }
        
    
    Since we used `ROOT_URL`, we have to define it within a scope of `server/models/Book.js`. Add following line of code right after section with imports:  
    ``const ROOT_URL = process.env.ROOT_URL || `http://localhost:${process.env.PORT || 8000}`;``
    
    We don't have an EmailTemplate with the name `purchase`. Open `server/models/EmailTemplate.js` and add a `purchase` template right after the `welcome` template:
    
        {
         name: 'purchase',
         subject: 'You purchased book at builderbook.org',
         message: `{{userName}},
           <p>
             Thank you for purchasing our book! You will get confirmation email from Stripe shortly.
           </p>
           <p>
             Start reading your book: <a href="{{bookUrl}}" target="_blank">{{bookTitle}}</a>
           </p>
           <p>
             If you have any questions while reading the book, 
             please fill out an issue on 
             <a href="https://github.com/builderbook/builderbook/issues" target="blank">Github</a>.
           </p>
        
           Kelly & Timur, Team Builder Book
         `,
        },
        
    
    Done. The next time you start your app, a new EmailTemplate document with `"name": "purchase"` will be created on MongoDB.
    
5.  This one is straightforward. To create a new `Purchase` document, we use the Mongoose method `create()` on our Purchase model. Besides adding a `createdAt` timestamp, we pass the following parameters to the document: `user._id`, `book._id`, `book.price`, and `chargeObject` as `userId`, `bookId`, `amount`, and `stripeCharge`, respectively:
    
        return Purchase.create({
         userId: user._id,
         bookId: book._id,
         amount: book.price * 100,
         stripeCharge: chargeObj,
         createdAt: new Date(),
        });
        
    

Plug in the code from steps 1 to 5 into our high-level structure of the `buy` static method. Add this method within `BookClass` of our Book model at `server/models/Book.js`:

    static async buy({ id, user, stripeToken }) {
      if (!user) {
        throw new Error('User required');
      }
    
      const book = await this.findById(id, 'name slug price');
      if (!book) {
        throw new Error('Book not found');
      }
    
      const isPurchased = (await Purchase.find({ userId: user._id, bookId: id }).count()) > 0;
      if (isPurchased) {
        throw new Error('Already bought this book');
      }
    
      const chargeObj = await stripeCharge({
        amount: book.price * 100,
        token: stripeToken.id,
        buyerEmail: user.email,
      });
    
      const template = await getEmailTemplate('purchase', {
        userName: user.displayName,
        bookTitle: book.name,
        bookUrl: `${ROOT_URL}/books/${book.slug}/introduction`,
      });
    
      try {
        await sendEmail({
          from: `Kelly from builderbook.org <${process.env.EMAIL_SUPPORT_FROM_ADDRESS}>`,
          to: [user.email],
          subject: template.subject,
          body: template.message,
        });
      } catch (error) {
        logger.error('Email sending error:', error);
      }
    
      return Purchase.create({
        userId: user._id,
        bookId: book._id,
        amount: book.price * 100,
        stripeCharge: chargeObj,
        createdAt: new Date(),
      });
    }
    

This is pretty much the final version of our `buy()` static method. We will make only one upgrade - later in this chapter, we will call the `subscribe()` method to add a user's email to a Mailchimp list.

Remember to add all missing imports to `server/models/Book.js`:

    import Purchase from './Purchase';
    import { stripeCharge } from '../stripe';
    import getEmailTemplate from './EmailTemplate';
    import sendEmail from '../aws';
    

In the next two subsections, we will discuss `stripeCharge()` and Purchase model. We imported and used them inside `Book.buy()`, but we have not defined them.

#### stripeCharge() method

To create a charge with Stripe, we use the [stripe](https://github.com/stripe/stripe-node) package and follow this [official example](https://github.com/stripe/stripe-node#usage):

    var stripe = require('stripe')('sk_test_...');
    
    var customer = await stripe.customers.create(
      { email: 'customer@example.com' }
    );
    

There is no strong reason for us to create `customer` as shown in the official example, since buying a digital book is a one-time transaction. Instead of creating `customer`, we want to create `charge`, so the code will be `stripe.charges.create()` instead of `stripe.customers.create`.

Let's define `API_KEY` for development and production environments:

    const dev = process.env.NODE_ENV !== 'production';
    const API_KEY = dev ? process.env.Stripe_Test_SecretKey : process.env.Stripe_Live_SecretKey;
    

Next, define the constant `client` as well as `stripe` with the passed `API_KEY`:

const client = stripe(API_KEY);

We will specify only five parameters out of two dozen possible parameters for `stripeCharge()` (to see all parameters, check [Charge, Stripe API](https://stripe.com/docs/api/node#charges)). In addition to the three parameters that we passed to `stripeCharge()` inside our `Book.buy()` static method, we will add two more:

    currency: 'usd',
    description: 'Payment for the book at builderbook.org',
    

For charge creation, you get:

    return client.charges.create({
      amount,
      source: token,
      receipt_email: buyerEmail,
      currency: 'usd',
      description: 'Payment for the book at builderbook.org',
    });
    

Create a `server/stripe.js` file. Add definitions of `API_KEY`, `client` and above snippet that creates charge:  
`server/stripe.js`:

    import stripe from 'stripe';
    
    export function stripeCharge({
      amount, token, buyerEmail,
    }) {
      const dev = process.env.NODE_ENV !== 'production';
      const API_KEY = dev ? process.env.Stripe_Test_SecretKey : process.env.Stripe_Live_SecretKey;
      const client = stripe(API_KEY);
    
      return client.charges.create({
        amount,
        source: token,
        receipt_email: buyerEmail,
        currency: 'usd',
        description: 'Payment for the book at builderbook.org',
      });
    }
    

Remember to get `Stripe_Test_SecretKey` and `Stripe_Live_SecretKey` values from your Stripe account and add them to the `.env` file.

You will need to add a total of 4 Stripe keys to `.env` file, 2 publishable and 2 secret. After you create Stripe account, find keys at [https://dashboard.stripe.com/account/apikeys](https://dashboard.stripe.com/account/apikeys).

#### [_link_](#purchase-model) Purchase model

Our Purchase model is straighforward to write. There are no static methods to define. We want to ensure the uniqueness of our index with:

mongoSchema.index({ bookId: 1, userId: 1 }, { unique: true });

We discussed MongoDB indices in detail in [Chapter 5, section Chapter model](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#index-in-mongodb).

Check out User, Book, Chapter, and EmailTemplate models to remember how to create a model and specify Schema:

const Purchase = mongoose.model('Purchase', mongoSchema);

We create a Purchase document inside the `Book.buy()` static method with:

    return Purchase.create({
      userId: user._id,
      bookId: book._id,
      amount: book.price * 100,
      stripeCharge: chargeObj,
      createdAt: new Date(),
    });
    

Based on the information above, define Schema:

    const mongoSchema = new Schema({
      userId: {
        type: Schema.Types.ObjectId,
        required: true,
      },
      bookId: {
        type: Schema.Types.ObjectId,
        required: true,
      },
      amount: {
        type: Number,
        required: true,
      },
      stripeCharge: {
        id: String,
        amount: Number,
        created: Number,
        livemode: Boolean,
        paid: Boolean,
        status: String,
      },
      createdAt: {
        type: Date,
        required: true,
      },
    });
    

Create a `server/models/Purchase.js` file and add the following code:  
`server/models/Purchase.js` :

    import mongoose, { Schema } from 'mongoose';
    
    const mongoSchema = new Schema({
      userId: {
        type: Schema.Types.ObjectId,
        required: true,
      },
      bookId: {
        type: Schema.Types.ObjectId,
        required: true,
      },
      amount: {
        type: Number,
        required: true,
      },
      stripeCharge: {
        id: String,
        amount: Number,
        created: Number,
        livemode: Boolean,
        paid: Boolean,
        status: String,
      },
      createdAt: {
        type: Date,
        required: true,
      },
    });
    
    mongoSchema.index({ bookId: 1, userId: 1 }, { unique: true });
    
    const Purchase = mongoose.model('Purchase', mongoSchema);
    
    export default Purchase;
    

In the next section `ReadChapter page`, we will add our `BuyButton` component to the `ReadChapter` page and make a few UX improvements to our checkout flow.

## ReadChapter page
--------------------------------------------

In this section, we will make necessary changes to our `ReadChapter` page so that a logged-out user can see _preview_ content and then buy the _full_ content of a book. Two main goals for this section are:

*   conditionally show `chapter.htmlExcerpt` (preview content) and `chapter.htmlContent` (full content)
*   add the `BuyButton` component and pass props to it

#### Excerpt and full content

In this subsection, we modify the `ReadChapter` page so it shows either `chapter.htmlExcerpt` or `chapter.htmlContent` depending on whether a user bought a book or not.

Open `pages/public/read-chapter.js` and find instances of `htmlContent`. [In Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#readchapter-page), we simply defined `htmlContent` as:

    let htmlContent = '';
    if (chapter) {
      htmlContent = chapter.htmlContent;
    }
    

With this definition, the content you write inside a `.md` file of a Github repo will be displayed on the webpage, unpaywalled. This is fine for writing a free book or documentation. However, we would like our app to support both free and paid content. Thus, we introduce the `isFree` parameter for chapters.

In Chapter 6, you forked our [demo-book repo](https://github.com/builderbook/demo-book). Navigate to the `chapter-1.md` file and click the Edit icon. Here is a [link](https://github.com/builderbook/demo-book/edit/master/chapter-1.md) to the original file, and here is snapshot of the page:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36941706-6871066a-1f16-11e8-87e8-d4171f0243e2.png)

If you add `isFree: true` to the metadata section of the `chapter-1.md` file, then the Chapter document created from this file will get a `"isFree": true` parameter after an Admin user clicks the `Sync with Github` button. We introduced the code responsible for saving parameters from metadata to a Chapter document [in Chapter 6](https://builderbook.org/books/builder-book/github-integration-admin-dashboard-testing-admin-ux-and-github-integration#synccontent-for-chapter-model).

To make sure that this code works, navigate to the `introduction.md` file of your forked `demo-book` repository and click the Edit icon. You will see `isFree: true` inside the metadata section:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36941749-a2733490-1f17-11e8-9b1c-7d523a31e6d8.png)

Now go to your database dashboard at mLab, go to the chapters collection, and find the chapter document created from the `introduction.md` file. You will indeed see the proper `isFree` parameter:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36941770-109e69d0-1f18-11e8-8779-0781ac06110b.png)

Alright, now we have a useful boolean parameter, `chapter.isFree`, that we can use to control whether we show `chapter.htmlExcerpt` or `chapter.htmlContent`. When `chapter.isFree` is true, the page should display `chapter.htmlContent`. When false, `chapter.htmlExcerpt`.

The next step is to introduce another boolean parameter: `chapter.isPurchased`. This will allow users who purchased a book to see its content. When the `chapter.isPurchased` parameter is true, the page should display `chapter.htmlContent`, even when `chapter.isFree` is false. To set `isPurchased` on the `chapter` object, we have to modify the `getBySlug()` static method inside our Chapter model.

Open `server/models/Chapter.js` and find the `getBySlug()` static method:

    static async getBySlug({ bookSlug, chapterSlug }) {
      const book = await Book.getBySlug({ slug: bookSlug });
      if (!book) {
        throw new Error('Book not found');
      }
    
      const chapter = await this.findOne({ bookId: book._id, slug: chapterSlug });
    
      if (!chapter) {
        throw new Error('Chapter not found');
      }
    
      const chapterObj = chapter.toObject();
      chapterObj.book = book;
    
      return chapterObj;
    }
    

[In Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#express-route), when we wrote the `getBySlug` static method, we passed `userId` and `isAdmin` parameters to this method from corresponding Express route. But we haven't used them yet (though we mentioned that we would in Chapter 8).

Add these two parameters in the `getBySlug()` function like this:

    static async getBySlug({
      bookSlug, chapterSlug, userId, isAdmin,
    })
    

Now, let's modify the `getBySlug` static method by adding code that checks whether the chapter `isPurchased`. We'll add the code _before_ we return chapterObj with `return chapterObj`. Follow the example from our `Book.buy()` static method located at `server/models/Book.js`. There, we defined `isPurchased` like this:

const isPurchased = (await Purchase.find({ userId: user._id, bookId: id }).count()) > 0;

Here, we will make some slight changes. We will check:

*   that the purchase document exists (`Purchase.findOne({ userId, bookId: book._id })`)
*   whether a user is Admin (`isAdmin`),
*   whether the chapter is free (`chapter.isFree`)

To check for all three, you may get something like this:

    if (userId) {
      const purchase = await Purchase.findOne({ userId, bookId: book._id });
    
      chapterObj.isPurchased = !!purchase || isAdmin;
    }
    
    const isFreeOrPurchased = chapter.isFree || chapterObj.isPurchased;
    
    if (!isFreeOrPurchased) {
      delete chapterObj.htmlContent;
    }
    

First note - `chapterObj.isPurchased` is true if a purchase exists or if a user is Admin.

Second note - we _remove_ the full content of `htmlContent` from the `chapterObj` object when `chapterObj.isFreeOrPurchased` is false. If a chapter is not free and not purchased, then `chapterObj` will _not contain_ `htmlContent`. This is a very important step, since our app renders `ReadChapter` page initially _on the server_. Writing conditional logic that hides full content on the client is _not_ sufficient that's why we added above server-side logic.

Add the above code snippet to the `getBySlug` method in our Chapter model. Paste the snippet above this line of code:

return chapterObj;

Remember to import the Purchase model to `server/models/Chapter.js` with:

import Purchase from './Purchase';

Great, we have _both_ boolean parameters we wanted: `chapter.isFree` and `chapter.isPurchased`!

Writing a conditional construct inside the `ReadChapter` page will be easy. Open `pages/public/read-chapter.js` and find two places where we defined `htmlContent`. First, inside `constructor` :

    let htmlContent = '';
    if (chapter) {
      htmlContent = chapter.htmlContent;
    }
    

Second, inside `componentWillReceiveProps()`:

    const { htmlContent } = chapter;
    

Replace _both_ of these definitions of `htmlContent` with:

    let htmlContent = '';
    if (chapter && (chapter.isPurchased || chapter.isFree)) {
      htmlContent = chapter.htmlContent;
    } else {
      htmlContent = chapter.htmlExcerpt;
    }
    

The condition `(chapter.isPurchased || chapter.isFree)` ensures that we show the full content of `chapter.htmlContent` _only_ in the following cases:

*   user bought the book
*   user is an Admin
*   Admin user specifies `isFree: true` in the metadata of a `.md` file

#### BuyButton

Now that we've set up conditions for showing free or paid content to particular users, let's add a buy button to appear on paid books.

Go to the `ReadChapter` page. Open the `pages/public/read-chapter.js` file and import the `BuyButton` component:

import BuyButton from '../../components/customer/BuyButton';

In the same file, find the `renderMainContent()` function. Inside this function, find the element:

    <div
      // eslint-disable-next-line react/no-danger
      dangerouslySetInnerHTML={{ __html: htmlContent }}
    />
    

Under this element, which is `htmlContent`, add the buy button (`BuyButton` component):

    {!chapter.isPurchased && !chapter.isFree ? (
      <BuyButton user={user} book={chapter.book} />
    ) : null}
    

This conditional construct ensures that a page does not display the buy button on a chapter when `chapter.isPurchased` or `chapter.isFree` is true.

We pass `user`, `book`, and `showModal` props to our `BuyButton` component. We have already defined the props `user` and `book` inside the `ReadChapter` page component. Go ahead and find them:

*   we defined a `user` constant inside the `render()` method as `const { user } = this.props;`, since we pass `user` prop to the `Header` component
*   we defined a `book` constant inside the `renderSidebar()` function as `const { book } = chapter;`, we use `book` for `book.slug`

So we know how to define `user` and `book` inside the local scope of the `renderMainContent()` function:

    renderMainContent() {
      const { user } = this.props;
    
      const {
        chapter, htmlContent, showTOC, isMobile,
      } = this.state;
    
      const { book } = chapter;
    
      let padding = '20px 20%';
      if (!isMobile && showTOC) {
        padding = '20px 10%';
      } else if (isMobile) {
        padding = '0px 10px';
      }
    
      return (
        <div style={{ padding }} id="chapter-content">
          <h2 style={{ fontWeight: '400', lineHeight: '1.5em' }}>
            {chapter.order > 1 ? `Chapter ${chapter.order - 1}: ` : null}
            {chapter.title}
          </h2>
          <div
            // eslint-disable-next-line react/no-danger
            dangerouslySetInnerHTML={{ __html: htmlContent }}
          />
          {!chapter.isPurchased && !chapter.isFree ? (
            <BuyButton user={user} book={book} />
          ) : null}
        </div>
      );
    }
    

Time to test.

#### Testing

We should make UX improvements to our checkout flow, but before we do, let's test the code we wrote so far in this chapter.

Start your app with `yarn dev`, log in with a _non-Admin_ user, and navigate to

[http://localhost:8000/books/demo-book/example](http://localhost:8000/books/demo-book/example)

As expected, you will see `chapter.htmlExcerpt` and the `BuyButton` component!  
![Builder Book](https://user-images.githubusercontent.com/10218864/36943901-685bd918-1f46-11e8-8649-d85886211d92.png)

Log out, and log in with an `Admin` user (if you have only one Google account, set `"isAdmin": true` on your user document at MongoDB).

Navigate to `http://localhost:8000/books/demo-book/example`.  
As expected, the page displays the full content of `chapter.htmlContent`, and the `BuyButton` component has disappeared:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36943925-f3a2ad3a-1f46-11e8-8ce8-60b48eb0f88a.png)

If you try to test the checkout flow, it will fail with the error `Not found`. The `Book.buy()` method fails because the EmailTemplate document with the name `purchase` can't be found in our database.

To make our app create this missing EmailTemplate document, you need to go to mLab and _delete_ the `emailtemplates` collection. Then stop your app and start it again with `yarn dev`. Check the newly created `emailtemplates` collection - it will contain two documents with the names `welcome` and `purchase`. Now we are ready for testing the checkout flow.

Notice that at this point your database has no `purchases` collection.

Log out and log in with a _non-Admin_ user again (or remove `"isAdmin": true` from your user document at MongoDB).  
Navigate to `http://localhost:8000/books/demo-book/example`.  
Click the `Buy book` button and you will see the Stripe modal:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36943969-8aadc534-1f47-11e8-9ecd-0a0723678fc8.png)

In test mode, Stripe suggests using the card number 4242 4242 4242 4242 with any CVC (why not 4242) and any date in the future.  
Click the `Pay $49` button on the Stripe modal, and you will see a `Success!` in-app message. Then _after reloading_ the page, you will see the full content:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36944140-edd07500-1f4a-11e8-982e-dc6558804d55.png)

In your database, the app created a new collection called `purchases`. The very first Purchase document looks like:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36944152-2906aa18-1f4b-11e8-9736-34ffc7dec561.png)

Success, the checkout flow indeed works.

Reloading a page manually to see the full content is not a good UX. In the next section, we will reload the page automatically and make a few more UX improvements.

## Checkout UX
----------------------------------

In the previous section, we implemented a checkout flow and showed that it works as expected. However, a few experiences inside our checkout flow are not smooth. For example:

1.  After you buy a book, you have to _manually_ refresh the page in order to see the full content.
2.  If you are logged out while viewing the `ReadChapter` page and then log in, you will be redirected to the `MyBooks` page. It would be better to redirect _back_ to the `ReadChapter` page.
3.  If you are logged out and click the `Buy book` button (on the `ReadChapter` page), you will be redirected to the `MyBooks` page. It would be better to redirect _back_ to the `ReadChapter` page you were reading when you clicked the `Buy book` button.
4.  If you are logged out and click the `Buy book` button, you have to sign in and then click the `Buy book` button _again_. It would better to see the Stripe modal right after logging in, so you only need to click the `Buy Book` button _once_.

Let's discuss how we will solve each of the UX problems above.

1.  You may think of at least two ways to solve this problem. Ultimately, the goal is to re-run the `getInitialProps()` method inside the `ReadChapter` page (`pages/public/read-chapter.js`). Re-rendering a component with `setState()` will _not_ trigger `getInitialProps()` to execute again. We need to re-run the `getInitialProps()` method to add `htmlContent` parameter to the `chapterObj` so full content can be displayed on the page. One complication is that, to re-run `getInitialProps()`, we need to pass `headers.cookie` to `getInitialProps()` again. We have to save `headers.cookie` in props.
    
    Here is the reason why the `getInitialProps()` method requires `cookie` as an argument. As discussed earlier in [Chapter 2, section Session](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#session), the server needs a cookie to identify a session. Then the server uses session to get a user id, and finally the server uses user id to get a user from `req.user`. In our `Book.buy()` static method (`server/models/Book.js`), we use `user` and `user.isAdmin` to see if a user bought a book or if a user is an Admin.
    
    A simpler solution is to trigger a window reload after the `buyBook()` API method has executed. Open `components/customer/BuyButton.js` and find this snippet:
    
        try {
         await buyBook({ stripeToken: token, id: book._id });
         notify('Success!');
         NProgress.done();
        } catch (err) {
         NProgress.done();
         notify(err);
        }
        
    
    As you know from [Chapter 3, section Async/await](https://builderbook.org/books/builder-book/authentication-hoc-promise-async-await-static-method-for-user-model-google-oauth#async-await), the code pauses at the line with `await buyBook()`. After this line, add a new line:  
    `window.location.reload(true);`
    
    This method [location.reload(true)](https://developer.mozilla.org/en-US/docs/Web/API/Location/reload) reloads a page from the server.
    
2.  Imagine you bought a book but you logged out of the app. Say you go back to Chapter 2 of the book. Let's modify the code so that when you click the `Log in` link (in the `Header`) and log in with Google, you'll be _automatically_ redirected _back_ to Chapter 2 _instead_ of the `MyBooks` page.
    
    Take a look at the code responsible for redirecting a user after Google login at `server/google.js`. Find this anonymous arrow function:
    
        (req, res) => {
         if (req.user && req.user.isAdmin) {
           res.redirect('/admin');
         } else {
           res.redirect('/my-books');
         }
        },
        
    
    We need to modify the above conditional logic. We still want to redirect an Admin user to `/admin`. But a Customer user should be redirected to the `ReadChapter` page \_if\_ the user clicked the `Log in` link or `Buy book` button _while_ on the `ReadChapter` page.
    
    We suggest creating a new parameter, `redirectUrl`, that contains the URL of the `ReadChapter` page.
    
    We can _append_ this `redirectUrl` value to the `Login` page URL (`/login`) and Google OAuth URL (`/auth/google`). Then, after a user logs in, we can _read_ `redirectUrl` from the appended URL and redirect a user _back_ to the `ReadChapter` page. For example, for the URL:
    
    https://builderbook.org/books/builder-book/table-of-contents-highlight-for-section-hide-header-mobile-browser
    
    The `redirectUrl` would be:
    
    %2Fbooks%2Fbuilder-book%2Ftable-of-contents-highlight-for-section-hide-header-mobile-browser
    
    URL encoded `%2F` stands for slash `/`.
    
    First, let's work on the redirect when a user clicks the `Log in` link inside the `Header` component. Open `server/google.js` and find the `/auth/google` Express route:
    
        server.get(
         '/auth/google',
         passport.authenticate('google', {
           scope: ['profile', 'email'],
           prompt: 'select_account',
         }),
        );
        
    
    We want to re-write this route in order to save a `redirectUrl` string value to the `req.session` object _before_ calling `passport.authenticate('google')`. We want both `req.query` and `req.query.redirectUrl` to exist and make sure that the latter starts with `/`. We do so with the JavaScript method [startsWith('/')]((https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/startsWith).) that outputs true or false:
    
        if (req.query && req.query.redirectUrl && req.query.redirectUrl.startsWith('/')) {
         req.session.finalUrl = req.query.redirectUrl;
        } else {
         req.session.finalUrl = null;
        }
        
    
    If all three conditions are true, we _save_ `redirectUrl` to our session object as `req.session.finalUrl`.  
    The updated Express route will look like:
    
        server.get('/auth/google', (req, res, redirectUrl) => {
         if (req.query && req.query.redirectUrl && req.query.redirectUrl.startsWith('/')) {
           req.session.finalUrl = req.query.redirectUrl;
         } else {
           req.session.finalUrl = null;
         }
        
         passport.authenticate('google', {
           scope: ['profile', 'email'],
           prompt: 'select_account',
         })(req, res, redirectUrl);
        });
        
    
    At this point, we saved a `redirectUrl` value to `req.session`. The next step is to use it for an actual redirect. In the same file, find this function inside the `/oauth2callback` Express route:
    
        (req, res) => {
         if (req.user && req.user.isAdmin) {
           res.redirect('/admin');
         } else {
           res.redirect('/my-books');
         }
        },
        
    
    _Before_ the `else` condition, we should add an `else if` condition to redirect a user to `req.session.finalUrl` with `res.redirect(req.session.finalUrl)`:
    
        else if (req.session.finalUrl) {
         res.redirect(req.session.finalUrl);
        }
        
    
    Updated function:
    
        (req, res) => {
         if (req.user && req.user.isAdmin) {
           res.redirect('/admin');
         } else if (req.session.finalUrl) {
           res.redirect(req.session.finalUrl);
         } else {
           res.redirect('/my-books');
         }
        },
        
    
    This conditional logic redirects an Admin user to `/admin` and redirects a Customer user to `req.session.redirectUrl` if `req.query.redirectUrl` exists. However, based on the last statement, our redirect feature does not work yet. It will only work when `req.query` (i.e. the part of the URL after `?redirectUrl=`) has a value (see how we defined `req.session.finalUrl` earlier). Our next step is to engineer `req.query.redirectUrl`.
    
    Let's keep in mind what we want to achieve. We want a user who is on the `ReadChapter` page and who clicks the `Log in` link in the `Header` component to be redirected back to the same `ReadChapter` page after logging in. That means that:
    
    *   The URL of the `Login` page _must_ contain the `?redirectUrl=` query with a value.
    *   Once the URL of the `Login` page has this query, the app will be able to read the query value and pass it to `/auth/google?redirectUrl=`.
    *   After that, the updated Express route `server.get('/auth/google)` will take care of the rest.
    
    Start your app and make sure you are logged out. Go to Chapter 1 (titled "Example") of the [demo-book](https://github.com/builderbook/demo-book) repo you forked earlier. The URL of this page is:
    
    http://localhost:8000/books/demo-book/example
    
    We want to make sure that when you click `Log in` in the `Header`, the URL of the Login page is:
    
    http://localhost:8000/public/login?redirectUrl=%2Fbooks%2Fdemo-book%2Fexample
    
    Instead of:
    
    http://localhost:8000/public/login
    
    The easiest way to achieve this is to pass the `redirectUrl` prop from the `ReadChapter` page to the `Header` component. Let's make neccesary changes to `ReadChapter` and `Header`.
    
    *   In `pages/public/read-chapter.js`, make the following 3 changes:
        
        1.  To `static PropTypes`, add a new `url` prop:
            
                url: PropTypes.shape({
                asPath: PropTypes.string.isRequired,
                }).isRequired,
                
            
        2.  Inside the main `render()` method, define a `url` constant as `this.props.url`:  
            `const { user, url } = this.props;`
        3.  Pass a `url.asPath` value to the `redirectUrl` prop on the `Header` component like this:
            
            Header user={user} hideHeader={hideHeader} redirectUrl={url.asPath}
            
            Read about `url` and `asPath` properties in the Next.js [docs for routing](https://github.com/zeit/next.js/#routing).  
            In our case, `url.asPath` is `/books/demo-book/example`.  
            If in doubt, print this value to the browser console (`Developer tools > Console`) by adding `console.log(url.asPath);` after `const { user, url } = this.props;` inside `render()` of the `pages/public/read-chapter.js` file.
            
    *   In `components/Header.js`, make the following 3 changes:
        
        1.  Add a `redirectUrl` prop to the `Header` function:
            
            function Header({ user, hideHeader, redirectUrl })
            
        2.  Update the `href` value in the `<Link>` element for `Log in`. It becomes:
            
                <Link
                prefetch
                href={{ pathname: '/public/login', asPath: '/login', query: { redirectUrl } }}>
                <a style={{ margin: '0px 20px 0px auto' }}>Log in</a>
                </Link>
                
            
            We specified that query is a value after `?redirectUrl=` with this [property shorthand](https://eslint.org/docs/rules/object-shorthand): `query: { redirectUrl }`.
            
        3.  Add the `redirectUrl` prop to `propTypes` and `defaultProps`. Add `redirectUrl: PropTypes.string` and `redirectUrl: ''` to `propTypes` and `defaultProps` blocks respectively.
    
    We are ready to test.  
    Start your app. While logged out, go to `http://localhost:8000/books/demo-book/example`.  
    Click the `Log in` link in the `Header`. You will be redirected to the `Login page` that has URL:
    
    http://localhost:8000/public/login?redirectUrl=%2Fbooks%2Fdemo-book%2Fexample
    
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36952085-f5476954-1fbf-11e8-88e6-0835ef698596.png)  
    It works!
    
    However, if you click the red `Log in with Google` button and log in as a Customer (non-Admin) user, you _won't_ be redirected back to Chapter 1. That's because we did not pass the `redirectUrl` query to the `/auth/google` URL yet.
    
    Open `pages/public/login.js` and make these 4 changes:
    
    1.  Rewrite `Login` as a function instead of a constant
    2.  Add the `url` prop to `Login.propTypes`
    3.  Define `redirectUrl` as:
        
        const redirectUrl = (url.query && url.query.redirectUrl) || '';
        
    4.  Pass `redirectUrl` to the `href` attribute of the `<Button>` element with:
        
        /auth/google?redirectUrl=${redirectUrl}
        
    
    After all changes, the updated `pages/public/login.js` file is:  
    `pages/public/login.js` :
    
        import Head from 'next/head';
        import PropTypes from 'prop-types';
        import Button from 'material-ui/Button';
        
        import withAuth from '../../lib/withAuth';
        import withLayout from '../../lib/withLayout';
        import { styleLoginButton } from '../../components/SharedStyles';
        
        function Login({ url }) {
         const redirectUrl = (url.query && url.query.redirectUrl) || '';
        
         return (
           <div style={{ textAlign: 'center', margin: '0 20px' }}>
             <Head>
               <title>Log in to Builder Book</title>
               <meta name="description" content="Login page for builderbook.org" />
             </Head>
             <br />
             <p style={{ margin: '45px auto', fontSize: '44px', fontWeight: '400' }}>Log in</p>
             <p>You’ll be logged in for 14 days unless you log out manually.</p>
             <br />
             <Button
               variant="raised"
               style={styleLoginButton}
               href={`/auth/google?redirectUrl=${redirectUrl}`}
             >
               <img src="https://storage.googleapis.com/nice-future-2156/G.svg" alt="Log in with Google" />
                   Log in with Google
             </Button>
           </div>
         );
        }
        
        Login.propTypes = {
         url: PropTypes.shape({
           query: PropTypes.shape({
             redirectUrl: PropTypes.string,
           }),
         }).isRequired,
        };
        
        export default withAuth(withLayout(Login), { logoutRequired: true });
        
    
    Now we can test this entire new feature.  
    Start your app. While logged out, go to:
    
    http://localhost:8000/books/demo-book/example
    
    Click \`Log in\` in the \`Header\`. You will be redirected to the \`Login page\` with this URL:
    
    http://localhost:8000/public/login?redirectUrl=%2Fbooks%2Fdemo-book%2Fexample
    
    Hover your cursor over the \`Log in with Google\` button, and you will see the following URL in the bottom-left corner of your browser window:
    
    http://localhost:8000/auth/google?redirectUrl=/books/demo-book/example
    
    Log in as a Customer (non-Admin) user, and you will be redirected back to:
    
    http://localhost:8000/books/demo-book/example
    
    Hooray!
    
3.  The next step is to work on a slightly different redirect - taking a user back to the `ReadChapter` page when a logged-out user clicks the `Buy book` button. We've actually written most of the code for this feature. We will rely on the code we added to `server/google.js`. That means that we need to add a `redirectUrl` query to the URL when a logged-out user clicks the `Buy book` button on the `ReadChapter` page.
    
    Do you remember which function handles a click on `Buy book` from a logged-out user? It's the `onLoginClicked` function from the `BuyButton` component (`components/customer/BuyButton.js`):
    
        onLoginClicked = () => {
         const { user } = this.props;
        
         if (!user) {
           window.location.href = '/auth/google';
         }
        };
        
    
    We need to make sure that `/auth/google` gets the `redirectUrl` query. In this local scope, define `redirectUrl` as [location.pathname](https://www.w3schools.com/jsref/prop_loc_pathname.asp).
    
    `location.pathname` gives the same value as Next.js's `url.asPath` (see above). It's the part of the URL without the domain root:
    
    const redirectUrl = window.location.pathname;
    
    Append the value of `redirectUrl` to `/auth/google?redirectUrl=`. Set the page's URL to this appended URL by using [location.href](https://www.w3schools.com/jsref/prop_loc_href.asp):
    
        const redirectUrl = window.location.pathname;
        window.location.href = `/auth/google?redirectUrl=${redirectUrl}`;
        
    
    Updated `onLoginClicked` function:
    
        onLoginClicked = () => {
         const { user } = this.props;
        
         if (!user) {
           const redirectUrl = window.location.pathname;
           window.location.href = `/auth/google?redirectUrl=${redirectUrl}`;
         }
        };
        
    
    Time to test it out.
    
    Make sure you are logged out and navigate to: `http://localhost:8000/books/demo-book/example`  
    Click on the blue `Buy book` button.  
    You will see the Google OAuth prompt. After logging in (as a Customer), you will be automatically redirected back to:
    
    http://localhost:8000/books/demo-book/example
    
    Good job.
    
    The only part that still sucks, UX-wise, is that the Stripe modal did not appear automatically. You clicked the `Buy book` button because you intend to buy a book. But our app requires you to click `Buy book` one more time. That's annoying.
    
4.  To show the Stripe modal to a user who clicked on the `Buy book` button while logged out, we need to somehow change the value of the `showModal` prop inside the `BuyButton` component. We need to detect that a logged-user clicked the `Buy book` button on the `ReadChapter` page. Then we need to pass a `showModal` prop with the value `true` from the `ReadChapter` page to the `BuyButton` component.  
    Earlier in this chapter, we set the initial state of the `Buy book` component with:
    
        this.state = {
         showModal: !!props.showModal,
        };
        
    
    This code ensures that `showModal` from the `ReadChapter` page has the same value as `showModal` in the scope of the `BuyButton` component.
    
    The question is _when_ should the `ReadChapter` page pass this `showModal` prop to the `BuyButton` component?
    
    An easy solution is to modify our `onLoginClicked` function so that `/auth/google?redirectUrl=${redirectUrl}` is _further_ appended with a unique query, say `?buy=1`. Inside our `onLoginClicked` function, append `?buy=1` to `/auth/google?redirectUrl=${redirectUrl}` so that it becomes:
    
    /auth/google?redirectUrl=${redirectUrl}?buy=1
    
    Next, make changes to the `ReadChapter` page component so it passes `showModal` to the `BuyButton` component:
    
    1.  Update the `<BuyButton>` component by passing this extra prop:
        
        BuyButton user={user} book={book} showModal={showStripeModal}
        
        Inside `renderMainContent`, define `showStripeModal` as `this.props.showStripeModal` with ES6 destructuring:
        
        const { user, showStripeModal } = this.props;
        
    2.  Add `showStripeModal` to `static PropTypes`:  
        `showStripeModal: PropTypes.bool.isRequired`
    3.  Finally, write code that makes `showStripeModal` detect the `buy=1` query in the URL:
        
        const showStripeModal = req ? !!req.query.buy : window.location.search.includes('buy=1');
        
        This code above says that, when `req` exists, `showStripeModal` is true when the query `?buy=` exists (`req.query.buy` exists thus `!!req.query.buy` is true). When `req` does not exist, then `showStripeModal` is true when `window.location.search.includes('buy=1')` exists. [location.search](https://www.w3schools.com/jsref/prop_loc_search.asp) returns the query part of the URL - in our case, everything after the first `?` symbol. For example, for Chapter 1 of our demo book, `window.location.search`:
        
        ?redirectUrl=/books/demo-book/example?buy=1
        
        The JavaScript method [includes('buy=1')](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/includes) checks if `buy=1` is included in `window.location.search`.
        
        We defined `showStripeModal` with conditional logic because sometimes the page is server-side rendered (`req.query.buy` is available on the server) and sometimes client-side rendered (`window.location.search` is available on client).
        
        Add the above definition of `showStripeModal` inside the `getInitialProps()` method like this:
        
            static async getInitialProps({ req, query }) {
            const { bookSlug, chapterSlug } = query;
            
            const headers = {};
            if (req && req.headers && req.headers.cookie) {
              headers.cookie = req.headers.cookie;
            }
            
            const chapter = await getChapterDetail({ bookSlug, chapterSlug }, { headers });
            
            const showStripeModal = req ? req.query.buy : window.location.search.includes('buy=1');
            
            return { chapter, showStripeModal };
            }
            
        
    
    Our code is ready for testing.
    
    While logged out, navigate to: `http://localhost:8000/books/demo-book/example`  
    Click on the blue `Buy book` button. Log in as a Customer (non-Admin) user.  
    You will be automatically redirected back to Chapter 1:  
    `http://localhost:8000/books/demo-book/example?buy=1`  
    This time you will the Stripe modal and there is no need for you to click `Buy book` again:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36957272-1c36e9e0-1fe8-11e8-8b61-5a95f63bd037.png)
    

We are done with UX improvements to our checkout flow. In the next section, we will work on the `MyBooks` page. Our main goal is to display a hyperlinked title for the book purchased by a Customer user.

## MyBooks page
------------------------------------

The `MyBooks` page is a Customer page, and it contains a list of books purchased by a user. Think of it as the Customer's dashboard.

Before we start writing code for any new page, the first step is to decide how to render that page - server-side or client-side.

For example, the `Admin` page (`pages/admin/index.js`) is client-side rendered, and the `getBookList()` API method is called inside the `componentDidMount()` lifecycle hook. If you load the `/admin` page, you will notice that the page loads _without_ data (without the list of books). Data appears with a small but noticable delay. If Admin UX is our priority, we would display some kind of loading UI while the Admin user waits for data to appear. For example, we could've displayed Nprogress, a spinner, or simply text `loading...`.

In comparison, the `ReadChapter` page is server-side rendered (on initial load), and the `getChapterDetail()` API method is called inside the `getInitialProps()` method instead of `componentDidMount()`. The UX advantage of a server-side rendered page is that it does not require loading UI. The page loads _with_ data, never without data.

Admin UX is not our first priority - Customer UX is. Thus, we will implement the `MyBook` page so that it is server-side rendered on initial load. We will place the API method `getMyBookList()` inside the `getInitialProps()` method.

You created multiple page at this point. The most recent server-side rendered page was the `ReadChapter` page. Take a look at the `getInitialProps()` method at `pages/public/read-chapter.js`:

    static async getInitialProps({ req, query }) {
      const { bookSlug, chapterSlug } = query;
    
      const headers = {};
      if (req && req.headers && req.headers.cookie) {
        headers.cookie = req.headers.cookie;
      }
    
      const chapter = await getChapterDetail({ bookSlug, chapterSlug }, { headers });
      const showStripeModal = req ? !!req.query.buy : window.location.search.includes('buy=1');
    
      return { chapter, showStripeModal };
    }
    

One similarity between the `MyBooks` and `ReadChapter` pages is that we will pass a `cookie` as `headers.cookie` to the server. If we don't do this, then `req.user` won't be available on the server. (You remember from Chapter 3 that cookie has encoded session id, session has user id, and with this user id, the server can find a user and create `req.user` that is used in many Express routes to check user permissions or to update user parameters.)

The difference between these pages is that `ReadChapter` shows `chapter.htmlExcerpt` to a logged-out user, but `MyBooks` should not be available at all to a logged-out user. Good UX would be to redirect the logged-out user, who stumbled upon the `/my-books` route, to the `Login` page:

    if (req && !req.user) {
      res.redirect('/login');
      return { purchasedBooks: [] };
    }
    

If a user is logged in, then we define and return an array of purchased books `purchasedBooks` like this:

    const { purchasedBooks } = await getMyBookList({ headers });
    return { purchasedBooks };
    

Summarize the above description of the `getInitialProps()` method:

    static async getInitialProps({ req, res }) {
      if (req && !req.user) {
        res.redirect('/login');
        return { purchasedBooks: [] };
      }
    
      const headers = {};
      if (req && req.headers && req.headers.cookie) {
        headers.cookie = req.headers.cookie;
      }
    
      const { purchasedBooks } = await getMyBookList({ headers });
      return { purchasedBooks };
    }
    

Remember how you rendered a list of sections using the `map()` JavaScript method. Open `pages/public/read-chapter.js` and find the `renderSections()` function:

    renderSections() {
      const { sections } = this.state.chapter;
      const { activeSection } = this.state;
    
      if (!sections || !sections.length === 0) {
        return null;
      }
    
      return (
        <ul>
          {sections.map(s => (
            <li key={s.escapedText} style={{ paddingTop: '10px' }}>
              <a
                style={{
                  color: activeSection && activeSection.hash === s.escapedText ? '#1565C0' : '#222',
                }}
                href={`#${s.escapedText}`}
                onClick={this.closeTocWhenMobile}
              >
                {s.text}
              </a>
            </li>
          ))}
        </ul>
      );
    }
    

Let's use this example above for `purchasedBooks`:

    {purchasedBooks && purchasedBooks.length > 0 ? (
      <div>
        <h3>Your books</h3>
        <ul>
          {purchasedBooks.map(book => (
            <li key={book._id}>
              <Link
                as={`/books/${book.slug}/introduction`}
                href={`/public/read-chapter?bookSlug=${book.slug}&chapterSlug=introduction`}
              >
                <a>{book.name}</a>
              </Link>
            </li>
          ))}
        </ul>
      </div>
    ) : (
      <div>
        <h3>Your books</h3>
        <p>You have not purchased any book.</p>
      </div>
    )}
    

That's it, the page is pretty much done. Go to your `pages/customer/my-books` file and put together the `getInitialProps()` and `render()` methods. Inside `render()`, remember to define `purchasedBooks` with:

const { purchasedBooks } = this.props;

Add missing imports, propTypes, defaultProps, and `<Head>` with `<title>`. Wrap the page component with `withLayout` and `withAuth` HOCs. Export with `export default withAuth(withLayout(MyBooks));`:  
`pages/customer/my-books.js` :

    import React from 'react';
    import PropTypes from 'prop-types';
    import Link from 'next/link';
    import Head from 'next/head';
    
    import { getMyBookList } from '../../lib/api/customer';
    import withLayout from '../../lib/withLayout';
    import withAuth from '../../lib/withAuth';
    
    
    class MyBooks extends React.Component {
      static propTypes = {
        purchasedBooks: PropTypes.arrayOf(PropTypes.shape({
          name: PropTypes.string.isRequired,
        })),
      };
      static defaultProps = {
        purchasedBooks: [],
      };
    
      static async getInitialProps({ req, res }) {
        if (req && !req.user) {
          res.redirect('/login');
          return { purchasedBooks: [] };
        }
    
        const headers = {};
        if (req && req.headers && req.headers.cookie) {
          headers.cookie = req.headers.cookie;
        }
    
        const { purchasedBooks } = await getMyBookList({ headers });
        return { purchasedBooks };
      }
    
      render() {
        const { purchasedBooks } = this.props;
    
        return (
          <div>
            <Head>
              <title>My Books</title>
            </Head>
            <div style={{ padding: '10px 45px' }}>
              {purchasedBooks && purchasedBooks.length > 0 ? (
                <div>
                  <h3>Your books</h3>
                  <ul>
                    {purchasedBooks.map(book => (
                      <li key={book._id}>
                        <Link
                          as={`/books/${book.slug}/introduction`}
                          href={`/public/read-chapter?bookSlug=${book.slug}&chapterSlug=introduction`}
                        >
                          <a>{book.name}</a>
                        </Link>
                      </li>
                    ))}
                  </ul>
                </div>
              ) : (
                <div>
                  <h3>Your books</h3>
                  <p>You have not purchased any book.</p>
                </div>
              )}
            </div>
          </div>
        );
      }
    }
    
    export default withAuth(withLayout(MyBooks));
    

This page won't get any data yet, since we did not define the `getMyBookList` API method and did not write an Express route nor static method for `getMyBookList` in our Book model.

#### API method getMyBookList

Since we pass `headers` to `getMyBookList`, we should take a look at the `getChapterDetails` API method that accepts `headers` as well. Open `lib/api/public.js`:

    export const getChapterDetail = ({ bookSlug, chapterSlug }, options = {}) =>
      sendRequest(
        `${BASE_PATH}/get-chapter-detail?bookSlug=${bookSlug}&chapterSlug=${chapterSlug}`,
        Object.assign(
          {
            method: 'GET',
          },
          options,
        ),
      );
    

Re-write the above API method. Keep in mind that we pass `headers` as `options.headers`, and the non-base part of API endpoint is `/my-books`:  
`lib/api/customer.js` :

    export const getMyBookList = (options = {}) =>
      sendRequest(
        `${BASE_PATH}/my-books`,
        Object.assign(
          {
            method: 'GET',
          },
          options,
        ),
      );
    

#### Express route '/my-books'

Our next step is to write an Express route with the method `GET` and route `/my-books`. At this point, you know that a function inside `/my-books` Express route should call and wait for static method and then return `purchasedBooks`. Let's make the static method `Book.getPurchasedBooks()` return `purchasedBooks`:

const { purchasedBooks } = await Book.getPurchasedBooks({ purchasedBookIds });

The above static method requires ids of books that are purchased by a user. Where do we get these ids? Since we did pass a cookie to the server, then `req.user` exists. We can look into `req.user.purchaseBookIds` for ids of books purchased by a user. This means that we need to modify our `Book.buy()` static method so that the user document of a user who buys a book gets `purchaseBookIds` array as a parameter.

Open `server/models/User.js` and make 2 small changes to the file:

*   add `purchasedBookIds: [String]` to mongoSchema,
*   add `'purchasedBookIds'` to `static publicFields()`

Open `server/models/Book.js`. Add a line of code that _finds and updates_ a user who bought a book. Use the Mongoose method `findByIdAndUpdate()`:

User.findByIdAndUpdate(user.id, { $addToSet: { purchasedBookIds: book.id } }).exec();

Add the above line of code right before the line with `const template = await getEmailTemplate()` inside the static method `buy()`.  
Remember to import `User` to `server/models/Book.js`. Eslint should tell you if an import is missing.

Back to the Express route.

Let's define `purchasedBooksIds` with ES6 object destructuring:

const { purchasedBookIds = \[\] } = req.user;

We are ready to put our Express route together. As always, we suggest using `try/catch` with `async/await`:  
`server/api/customer.js` :

    router.get('/my-books', async (req, res) => {
      try {
        const { purchasedBookIds = [] } = req.user;
    
        const { purchasedBooks } = await Book.getPurchasedBooks({ purchasedBookIds });
    
        res.json({ purchasedBooks });
      } catch (err) {
        res.json({ error: err.message || err.toString() });
      }
    });
    

The only missing part in our client-server data exchange is the static method `getPurchasedBooks()`.

#### Static method Book.getPurchasedBooks()

In the previous section, we wrote code that saves a purchased book id to a user document.  
Start your app. Make sure you use a Customer user who did not buy `demo-book`. If necessary, go to your purchases collection in mLab and delete the Purchase document for your Customer user and `demo-book`. Now buy a book and go to your users collection in mLab. Open the user document and you will see a new array:

    "purchasedBookIds": [
        "5a90d1c1b0466e2eee5f8508"
    ]
    

The static method `getPurchasedBooks()` finds all books in the books collection using the `purchasedBookIds` array, and the method returns `purchasedBooks` to the Express route `/my-books`. We will use the `find()` and `sort()` Mongoose methods that you know well by this point:  
`server/models/Book.js` :

    static async getPurchasedBooks({ purchasedBookIds }) {
      const purchasedBooks = await this.find({ _id: { $in: purchasedBookIds } }).sort({
        createdAt: -1,
      });
      return { purchasedBooks };
    }
    

You may not be familiar with the `{ $in: [array] }` operator. [$in](https://docs.mongodb.com/manual/reference/operator/query/in/#op._S_in) allows us to find all documents with a `field` value matching one of the values inside an array:

{ field: { $in: \[, , ...  \] } }

Start your app, log in, and navigate to `http://localhost:8000/my-books`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/37013260-d727d2f8-20ad-11e8-800b-80dd8a0c4d26.png)

Ta-da, you now see a list of purchased books.

In the next section, we discuss how to add user emails to a Mailchimp list. In the section after that, you will learn how to prepare your app for production and deploy it.

## Mailchimp API
--------------------------------------

In this section, our goal is to modify the static method `Book.buy()`, which we wrote earlier in this chapter, so that the email of a user who purchases a book gets added to a Mailchimp list.

After publishing and selling a book, you may occasionally need to send announcements to everyone who bought the book. For example, you may add a new chapter or make a major update to the book. We'll set up an API so that you can email people who bought a book by using a Mailchimp list.

Open `server/models/Book.js` and find the static method `buy()`. Find this line of code that creates a new Purchase document:

return Purchase.create()

Before this line, add a `try/catch` construct that calls and waits for the `subscribe()` method (which we will define at `server/mailchimp.js`):

    try {
      await subscribe({ email: user.email });
    } catch (error) {
      logger.error('Mailchimp error:', error);
    }
    

The `subscribe()` method will take a user's email and add it to a Mailchimp list. The list name is not the actual name of the list on your Mailchimp dashboard but the name of the variable that points to a unique `List ID`. More on this below.

Inside the `subscribe()` method, our goal is to send a POST request from our server to a Mailchimp server.

In this book, you wrote code related to third party APIs for Google OAuth, Github, AWS SES,and Stripe. If you remember, for Google OAuth, AWS SES, Stripe - we _did not_ write any code related to an actual _request_ (request from our server to a third party server). We used packages to send requests (usually, GET and/or POST) from our server to third party platform services. We wrote request-related code only in one instance - building Github integration.

Go ahead and open `server/github.js` and find the code for request:

    request.post(
      {
        url: TOKEN_URI,
        headers: { Accept: 'application/json' },
        form: {
          client_id: CLIENT_ID,
          code,
          client_secret: API_KEY,
        },
      },
      async (err, response, body) => {
        // code that manages error
        // code that manages success
      },
    );
    

This request is sent via POST (`request.post`) to Github's server when an Admin user authorizes our app to access data on his/her Github account. You see that we pass `CLIENT_ID` and `API_KEY` of our app to Github's server with `form`. For Mailchimp, we will pass the API key with [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) (`headers.Authorization`). Syntax for Authorization header: `Authorization: <type> <credentials>`

In our case:

Authorization: Basic apikey:API_KEY

The `API_KEY` must be base64 encoded. Recall how we did base64 decoding for `chapter.data.content` in [Chapter 6](https://builderbook.org/books/builder-book/github-integration-admin-dashboard-testing-admin-ux-and-github-integration#synccontent-for-book-model).

After encoding:

Authorization: \`Basic ${Buffer.from(\`apikey:${API_KEY}`).toString('base64')}`

Accept header in a Mailchimp request is the same as in a Github request. Follow the above example of a request to Github to put together a request to Mailchimp:

    request.post(
      {
        uri: `${ROOT_URI}${path}`,
        headers: {
          Accept: 'application/json',
          Authorization: `Basic ${Buffer.from(`apikey:${API_KEY}`).toString('base64')}`,
        },
        json: true,
        body: data,
      },
      (err, response, body) => {
        if (err) {
          reject(err);
        } else {
          resolve(body);
        }
      },
    );
    

We used the variables `ROOT_URI` and `API_KEY`.

`ROOT_URI` is the Mailchimp API endpoint to which our app sends a request. In general, it is:

https://usX.api.mailchimp.com/3.0/lists/{list_id}/members

Read more about the [API to add members to a list](http://developer.mailchimp.com/documentation/mailchimp/reference/lists/members/).

Region `usX` is a subdomain. Follow these steps to find the subdomain for an API endpoint:

*   sign up or log in to Mailchimp
*   go to `Account > Extras > API keys > Your API keys`
*   your API key will look like `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-us17`

That means the region is `us17` and your app will send requests to the Mailchimp subdomain:

https://us17.api.mailchimp.com/3.0/lists/{list_id}/members

Let's add the `LIST_IDS` variable to request uri:

https://us17.api.mailchimp.com/3.0/lists/${LIST_IDS}/members/

We can add an actual value, but let's add the variable in case we decide to save emails to a different list at some point. We define `LIST_IDS` as:

    const LIST_IDS =  process.env.MAILCHIMP_PURCHASED_LIST_ID;
    

And we define `API_KEY` as:

const API\_KEY = process.env.MAILCHIMP\_API_KEY;

This is a good place to add `MAILCHIMP_PURCHASED_LIST_ID` and `MAILCHIMP_API_KEY` to your `.env` file. We discussed how to find the `API_KEY` above, it looks like: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx-us17`.

To find `List ID`, follow these steps:

*   on your Mailchimp dashboard, go to `Lists > click the list name > Settings > List name and defaults`
*   find the section `List ID`
*   get the `xxxxxxxxxx` value from this section

Now we are ready to put together the `subscribe()` method that sends POST request to Mailchimp server. Follow these steps:

*   Create `server/mailchimp.js` file.
*   Import `request`
*   Add `require('dotenv').config();` since access env variables with `process.env`.
*   Save `parameter` email to `data.email_address`. Set `data.status` to `subscribed` (we pass `data` to request's `body`).

You will get:  
`server/mailchimp.js` :

    import request from 'request';
    
    require('dotenv').config();
    
    export async function subscribe({ email }) {
      const data = {
        email_address: email,
        status: 'subscribed',
      };
    
      const LIST_IDS = process.env.MAILCHIMP_PURCHASED_LIST_ID;
    
      const API_KEY = process.env.MAILCHIMP_API_KEY;
    
      await new Promise((resolve, reject) => {
        request.post(
          {
            uri: `https://us17.api.mailchimp.com/3.0/lists/${LIST_IDS}/members/`,
            headers: {
              Accept: 'application/json',
              Authorization: `Basic ${Buffer.from(`apikey:${API_KEY}`).toString('base64')}`,
            },
            json: true,
            body: data,
          },
          (err, response, body) => {
            if (err) {
              reject(err);
            } else {
              resolve(body);
            }
          },
        );
      });
    }
    

Done!

In above construct we used `async/await` as well as `new Promise()`. You may remember from [Chapter 3, section Async/await](https://builderbook.org/books/builder-book/authentication-hoc-promise-async-await-static-method-for-user-model-google-oauth#async-await) that function after `await` must return a Promise.

Time to test. Make sure that the actual values of `MAILCHIMP_PURCHASED_LIST_ID` and `MAILCHIMP_API_KEY` are in your `.env` file.

To test the `subscribe()` method, we need to run the static method `buy()` from our Book model. This means you have to buy a book. When testing, make sure you login with a Customer user who _has not bought a book_. In case your Customer user _bought a book_, here's how to un-buy a book:

*   on your database, delete the user's corresponding Purchase document from the purchases collection
*   find the user document for your User (users collection) and delete the parameter `purchasedBookIds`.

Go to your Mailchimp dashboard and access `Lists > click on list name`. Notice that the list is empty:  
![Builder Book](https://user-images.githubusercontent.com/10218864/37052749-08b1c424-212f-11e8-89db-e7338cbc6582.png)

Start your app and go to `http://localhost:8000/books/demo-book/example`. Click the `Buy book` button and go through the checkout.

After successfully purchasing the book, refresh the page on Mailchimp. You will see a new subscriber:  
![Builder Book](https://user-images.githubusercontent.com/10218864/37053390-def0baee-2130-11e8-88cd-959f39519cf8.png)

Nice! Now you have a communication method with people who bought your book. Use it wisely, never spam.

For deeper dive into Mailchimp integration, check our [our tutorial](https://medium.freecodecamp.org/how-to-integrate-mailchimp-in-a-javascript-web-app-2a889fb43f6f) at freeCodeCamp. In this tutorial, we did extensive testing of Mailchimp integration with [Postman](https://www.getpostman.com) and [browser console](https://developers.google.com/web/tools/chrome-devtools/console).

In the next and final section of our book, we will prepare our app for production and deploy it.

## Deploy app
--------------------------------

So far we've been running our app locally at `http://localhost:8000`. However to deploy our app, we need to:

*   set the root URL to `https://builderbook.org` instead of `http://localhost:8000`
*   set `NODE_ENV` to production
*   when `NODE_ENV` is in production, tell our app to use production (i.e. live) API keys instead of development (i.e. test) keys.

Once we prepare our app for production, we will deploy it with [Now](https://zeit.co/about) and optimize the app for search engines.

#### NODE\_ENV and ROOT\_URL

Throughout our app, we defined `dev` with this line of code:

const dev = process.env.NODE_ENV !== 'production';

Most recently, we wrote this line at `env-config.js`. Find this line of code in `server/app.js`, `server/stripe.js`, and `server/github.js` as well.

This code says that `dev` is true when `NODE_ENV` is not `production`. Open `server/app.js` and find this snippet:

    const dev = process.env.NODE_ENV !== 'production';
    const MONGO_URL = process.env.MONGO_URL_TEST;
    
    mongoose.connect(MONGO_URL);
    
    const port = process.env.PORT || 8000;
    const ROOT_URL = process.env.ROOT_URL || `http://localhost:${port}`;
    

Add the following three `console.log()` statements right after the code snippet above:

    console.log(process.env.NODE_ENV);
    console.log(dev);
    console.log(ROOT_URL);
    

Start your app with `yarn dev` and pay attention to the terminal output:

    undefined
    true
    http://localhost:8000
    

So `dev` is true because `process.env.NODE_ENV` is undefined (we did not set it!), and `ROOT_URL` is `http://localhost:8000`.

Our goal is to set `process.env.NODE_ENV` and once it is set, use it to specify a _production-specific_ `ROOT_URL` and other environmental variables, such as API keys.

Open `package.json` and find the `scripts` block.  
Prepend `NODE_ENV=production` to the `dev` command, so it becomes:

"dev": "NODE_ENV=production nodemon server/app.js --watch server --exec babel-node",

Start your app with `yarn dev` and now the terminal prints:

    production
    false
    http://localhost:8000
    > Could not find a valid build in the '.next' directory! Try building your app with 'next build' before starting the server.
    [nodemon] app crashed - waiting for file changes before starting...
    

Alright, not bad! You successfully set the environment to `production`.

Next.js tells us that we need to build our app with `NODE_ENV=production` before we run it. In the `scripts` of `package.json`, modify the `build` command like this:

"build": "NODE_ENV=production next build",

Run `yarn build`. When complete, start your app with `yarn dev`. Now the app runs locally but with `NODE_ENV=production`. You'll notice that the `ROOT_URL` is still `http://localhost:8000`. Let's change that by writing a conditional construct. Replace this line inside `server/app.js`:

const ROOT\_URL = process.env.ROOT\_URL || \`http://localhost:${port}\`;

with:

const ROOT_URL = dev ? \`http://localhost:${port}\` : 'https://builderbook.org';

Now run `yarn build` and `yarn dev`. The terminal outputs:

    production
    false
    https://builderbook.org
    

Try logging in - you'll see that it will fail. This makes sense because we _did not_ add routes from `server/google.js`, `https://builderbook.org/auth/google`, and `https://builderbook.org/oauth2callback`, to our Google OAuth app on Google Cloud Platform. We only added `http://localhost:8000/auth/google` and `http://localhost:8000/oauth2callback`.

We added Express routes `/auth/google` and `/oauth2callback` to our server with:

auth({ server, ROOT_URL });

Thus we took care of `ROOT_URL` for Google OAuth.

However, the `sendRequest()` function inside `lib/api/sendRequest.js` also uses `ROOT_URL`, and all API methods in our app use the `sendRequest()` function to send a request (`GET` or `POST`) from client to server. Open `lib/api/sendRequest.js` and find this snippet:

    const response = await fetch(
      `${ROOT_URL}${path}`,
      Object.assign({ method: 'POST', credentials: 'include' }, options, { headers }),
    );
    

We suggest, for the sake of reusability, creating a `getRootUrl()` function that contains conditional logic and outputs the proper `ROOT_URL` depending on `NODE_ENV`. Create a new file `lib/api/getRootUrl.js` with the following content:

    export default function getRootURL() {
      const port = process.env.PORT || 8000;
      const dev = process.env.NODE_ENV !== 'production';
      const ROOT_URL = dev ? `http://localhost:${port}` : 'https://builderbook.org';
    
      return ROOT_URL;
    }
    

To use `getRootUrl` in `lib/api/sendRequest.js`, follow these steps:

*   import `getRootUrl` with
    
    import getRootUrl from './getRootUrl';
    
*   update the snippet that contains `ROOT_URL` like this:
    
        const response = await fetch(
        `${getRootUrl()L}${path}`,
        Object.assign({ method: 'POST', credentials: 'include' }, options, { headers }),
        );
        
    
*   remove unnecessary code:
    
        const port = process.env.PORT || 8000;
        const ROOT_URL = process.env.ROOT_URL || `http://localhost:${port}`;
        
    

Go ahead and use `getRootUrl` inside `server/app.js` as well:

*   import `getRootUrl` with
    
    import getRootUrl from '../lib/api/getRootUrl';
    
*   update the snippet that contains `ROOT_URL` by replacing:
    
    const ROOT_URL = dev ? \`http://localhost:${port}\` : 'https://builderbook.org';
    
    with:  
    `const ROOT_URL = getRootUrl();`
    
*   keep the following line of code, since `server.listen()` uses `port`:  
    `const port = process.env.PORT || 8000;`
    

The third and final location where we will use `getRootUrl` is `server/models/Book.js`:

*   import `getRootUrl` with
    
    import getRootUrl from '../lib/api/getRootUrl';
    
*   replace this line:
    
    const ROOT\_URL = process.env.ROOT\_URL || \`http://localhost:${process.env.PORT || 8000}\`;
    
    with:
    
    const ROOT_URL = getRootUrl();
    

Start your app with `yarn dev` and look at the terminal:

    production
    false
    https://builderbook.org
    

This output proves that `getRootUrl()` successfully set the proper value for `ROOT_URL`.

We should make sure that our app uses live API keys instead of test ones. Let's test it out for our Github keys. Open `server/github.js`. Find the `const API_KEY` line of code and add `console.log()` right after it:

    const API_KEY = dev ? process.env.Github_Test_SecretKey : process.env.Github_Live_SecretKey;
    console.log(API_KEY);
    

Important note - the Github OAuth app _does not_ support multiple domains. Therefore, you should create a _second_ Github OAuth app and set `https://builderbook.org` and `https://builderbook.org/auth/github/callback` for the domain and callback URL. Reminder - in your first Github OAuth app, you set `http://localhost:8000` and `http://localhost:8000/auth/github/callback`.

Paste `process.env.Github_Live_SecretKey` and `process.env.Github_Live_ClientID` to your `.env` file.  
Start your app with `yarn dev` and you will see that the terminal printed the proper value for `API_KEY`, which is the value you specified for `process.env.Github_Live_SecretKey` inside `.env`.

Before we deploy our app, we need to modify the `start` command. We stopped using the `yarn next` command to start our app since [Chapter 2](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#express-server), where we introduced Express server. When we deploy our app on `Now`, `Now` will use the `start` command to start our app. Thus, we should update it to start our custom Express/Next server. Update it like this:

"start": "babel-node server/app.js"

For development, remember to remove `NODE_ENV=production` from the `dev` command that we used to run our app locally. The final `scripts` section inside `package.json` should be:

    "scripts": {
      "build": "NODE_ENV=production next build",
      "start": "babel-node server/app.js",
      "dev": "nodemon server/app.js --watch server --exec babel-node",
      "lint": "eslint components pages lib server",
      "test": "jest --coverage"
    },
    

After you are done with testing, remove all the `console.log()` statements from `server/app.js` and `server/github.js`.

#### Security

Besides setting `NODE_ENV` and `ROOT_URL`, we should prepare our server for production from a security point of view. We encourage you to read about the best practices for Express in the [official Express docs](https://expressjs.com/en/advanced/best-practice-security.html).

In this subsection, we will discuss the following security settings:

*   helmet package
*   trust proxy
*   cookie.secure

*   The [Helmet](https://helmetjs.github.io/) dependency is a collection of 12 Express middleware functions that set headers to prevent some standard attacks. Read more about each of the 12 middleware functions in the [Helmet docs](https://helmetjs.github.io/docs/). You can use all functions or only one. By default, helmet mounts 7 out 12 middleware functions. One of them is [helmet.hidePoweredBy()](https://helmetjs.github.io/docs/hide-powered-by) that simply hides the `X-Powered-By` header, making it a bit harder to guess which technology you use.
    
    Start your app, go to the `MyBooks` page, and look at the `Response Headers`. To do so, open `Developer tools > Network > click on my-book.js > select Headers`. You will clearly see that our app uses Express since `X-Powered-By:Express`:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37220585-ec0c0b96-237b-11e8-8c82-f5fba8cfdd1b.png)
    
    Your app should have the `helmet` package if you ran `yarn` at the start of this chapter. To use `helmet`, add the following line of code right after `const server = express();`:
    
    server.use(helmet())
    
    Check the `Response Headers` of `my-books.js`, and this time you won't see `X-powered-By`:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37220802-8f7dbbd0-237c-11e8-8592-95255dc7cd02.png)
    

*   Most hosting platforms scale an app by adding [load balancers](https://www.nginx.com/resources/glossary/load-balancing) that distribute requests from the client among multiple servers. A load balancer is a [proxy server](https://en.wikipedia.org/wiki/Proxy_server#Web_proxy_servers), an intermediary server that sits between client and server. To pass information from client to server, we need to make the server trust the proxy server. We only need to do so in our production environment:
    
        if (!dev) {
          server.set('trust proxy', 1);
        }
        
    
    The snippet of code above ensures that our server trusts its immediate proxy. Once set, the client can pass the following information to the server via a proxy server:
    
    *   `req.hostname` inside the [X-Forwarded-Host](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host) header
    *   `req.protocol` inside the [X-Forwarded-Proto](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto) header
    *   `req.ip` inside the [X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For) header
    
    If we don't set `server.set('trust proxy', 1);`, then our server will see the IP address of the proxy server as the IP address of the _client_.
    

*   In this step, we set a production-only [cookie.secure](https://github.com/expressjs/session#cookiesecure) setting for session:
    
        if (!dev) {
          sess.cookie.secure = true;
        }
        
    
    Our app will set `cookie` only when the client accesses the app via HTTPS protocol. Our app won't set `cookie` if the client used HTTP.
    
    Combine this `cookie` setting with the `trust proxy` setting, and you get:
    
        if (!dev) {
          server.set('trust proxy', 1);
          sess.cookie.secure = true;
        }
        
    

Add the above code snippet right before the `server.use(session(sess));` line of code in your `server/app.js` file.

#### SEO

This step is optional, but if you want better SEO, you can provide extra information to search engine bots that index your website. There are two ways to provide information to indexing bots:

*   `sitemap.xml` (placed at `/sitemap.xml`),
*   `robots.txt` (placed at `/robots.txt`).

Inside `sitemap.xml`, you specify which routes to index (`url`), how often to index them (`changefreq`), and how important each one is to index (`priority`).

Inside `robots.txt`, you can create disallow rules, which specify routes that indexing bots should not crawl.

Let's discuss the content of each file:

*   To generate content for `sitemap.xml`, we use a third party package [sitemap](https://github.com/ekalinin/sitemap.js) that you installed at the beginning of this chapter. We will follow the [official example for an Express server](https://github.com/ekalinin/sitemap.js#example-of-using-sitemapjs-with-express):
    
        var sm = require('sitemap');
        
        var sitemap = sm.createSitemap ({
          hostname: 'http://example.com',
          cacheTime: 600000,        // 600 sec - cache purge period
          urls: [
            { url: '/page-1/',  changefreq: 'daily', priority: 0.3 },
            { url: '/page-2/',  changefreq: 'monthly',  priority: 0.7 },
            { url: '/page-3/'},    // changefreq: 'weekly',  priority: 0.5
            { url: '/page-4/',   img: "http://urlTest.com" }
          ]
        });
        
        app.get('/sitemap.xml', function(req, res) {
          sitemap.toXML( function (err, xml) {
              if (err) {
                return res.status(500).end();
              }
              res.header('Content-Type', 'application/xml');
              res.send( xml );
          });
        });
        
    
    In our web app, the most important pages to crawl are chapters that are located at the `/books/builder-book/${chapter.slug}` route for our book with the slug `builder-book`. Take a look at the address bar of this page to see an example of that route. In the above example, objects inside the `urls` are manually added. However, we want to add the `urls` to our sitemap automatically from our database. Let's use the Chapter model and Mongoose method/query `find()` to find all chapters in our database:
    
        Chapter.find({}, 'slug').then((chapters) => {
          chapters.forEach((chapter) => {
            sitemap.add({
              url: `/books/builder-book/${chapter.slug}`,
              changefreq: 'daily',
              priority: 1,
            });
          });
        });
        
    
    Important note - if you have multiple books in your database, you should modify this code. For example, instead of fetching _all_ chapter documents from the database, find chapters by `bookId` for each book and set the proper book slug in the route for the `url` parameter.
    
*   Create a `static/robots.txt` file with the following content:
    
        User-agent: *
        Allow: /books/builder-book/
        Disallow: /admin
        
    
    These rules tell all indexing bots (`User-agent: *`) to crawl all routes under `/books/builder-book/` but disallow crawling under the `/admin` route.
    
    In Express, we use the [res.sendFile()](http://expressjs.com/en/api.html#res.sendFile) method to serve files from a particular path:
    
    res.sendFile(path.join(__dirname, '../static', 'robots.txt'));
    
    By now, you know how to write an Express route with the `GET` method:
    
        server.get('/robots.txt', (req, res) => {
          res.sendFile(path.join(__dirname, '../static', 'robots.txt'));
        });
        
    

We are done with the `sitemap.xml` and `robots.txt` files. The only remaining step is to add routes to our Express server. Let's combine the Express routes under one function `setup()`, export this function, and import it to `server/app.js`.

Put together all the code discussed above. Keep in mind the following:

*   in the sitemap example above, replace all instances of `app` to `server` (since we defined our Express server as `server` inside `server/app.js`)
*   `hostname` is `https://builderbook.org`
*   instead of writing `urls` manually, we do it with `Chapter.find()`
*   remember to add the `/robots.txt` Express route
*   remember to import `sitemap` and `path` (`path` is a Node module; it is not listed in the `package.json` file)

You will get:

    import sm from 'sitemap';
    import path from 'path';
    
    import Chapter from './models/Chapter';
    
    const sitemap = sm.createSitemap({
      hostname: 'https://builderbook.org',
      cacheTime: 600000, // 600 sec - cache purge period
    });
    
    export default function setup({ server }) {
      Chapter.find({}, 'slug').then((chapters) => {
        chapters.forEach((chapter) => {
          sitemap.add({
            url: `/books/builder-book/${chapter.slug}`,
            changefreq: 'daily',
            priority: 1,
          });
        });
      });
    
      server.get('/sitemap.xml', (req, res) => {
        sitemap.toXML((err, xml) => {
          if (err) {
            res.status(500).end();
            return;
          }
    
          res.header('Content-Type', 'application/xml');
          res.send(xml);
        });
      });
    
      server.get('/robots.txt', (req, res) => {
        res.sendFile(path.join(__dirname, '../static', 'robots.txt'));
      });
    }
    

Put this code into a new `server/sitemapAndRobots.js` file.

Update `server/app.js` in the following ways:

*   import `sitemapAndRobots`:
    
    import sitemapAndRobots from './sitemapAndRobots';
    
*   under the line with `routesWithSlug({ server, app });`, add this new line:
    
    sitemapAndRobots({ server });
    

Start your app with `yarn dev`. Check out the `/sitemap.xml` and `/robots.txt` routes! For example, in our case, the `/sitemap.xml` route shows the `sitemap.xml` file:  
![Builder Book](https://user-images.githubusercontent.com/10218864/37223674-1f5d04a0-2386-11e8-9286-915bf0030854.png)

You can make sure that Google bots find `sitemap.xml` and `robots.txt` at [Search Console](https://www.google.com/webmasters/tools/home?hl=en).

#### Now

At this point, we've prepared our app for production and are ready to deploy. You will learn how to deploy your app with [Now by Zeit](https://zeit.co/now). `Now` is a command-line interface tool that allows you to deploy a JavaScript app in under a minute and with one command, `now`. This tool will definitely make you more productive. It's built by `Zeit` on top of AWS and GCP infrastructure.

In this subsection, we will discuss some features of `Now`. For a list of all features, check out the [official documentation](https://zeit.co/docs).

We will discuss the following topics:

1.  installing `Now`
2.  first deployment
3.  frozen state and scale
4.  logs, source
5.  custom domain
6.  useful commands

1.  To install `Now`, run:
    
    npm install -g now
    
    To see all available commands (this confirms that you installed \`Now\`):
    
    now help
    
    Attempt running:
    
    now
    
    This will produce an error and ask you to sign up for Zeit and click a verification link inside an email from Zeit.
    
    After you click the verification link, you will be able to deploy and use the rest of the `Now` commands.
    
2.  Create a `now.json` file at the root of your app folder. File content:
    
        {
         "env": {
             "NODE_ENV": "production"
         },
        
         "dotenv": true
        }
        
    
    The first parameter sets `NODE_ENV`, and the second parameter tells `Now` that our app uses the `dotenv` dependency and that our environmental variables are in a `.env` file.
    
    In your terminal, navigate to the root directory of your app.  
    Deploy your app with this command:
    
    now
    
    `Now` will:
    
    *   upload code
    *   install dependencies from `yarn.lock`
    *   run `npm build`
    *   run `npm start`
    
    The terminal prints out all events:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37228004-fb690c8e-2393-11e8-9b2e-f9046ca8c7c4.png)
    
    Access your deployment at the URL provided by `Now`. In our case, it's:  
    
    \> Ready! [https://8-end-lpbrrsahsc.now.sh](https://8-end-lpbrrsahsc.now.sh)
    
      
    On our browser:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37228050-2c52b6ce-2394-11e8-94ba-98b00b2445ec.png)
    
    This URL is publicly accessible but has a randomly-generated and hard-to-guess part, `lpbrrsahsc`.
    
    The name of deployment is the same as a name inside `package.json` file.
    
    Important note - `Now` will upload all files from your app root directory. Tell `Now` to ignore any file or folder by adding its name to a `.npmignore` file. More on this in [Now's docs](https://zeit.co/docs/features/now-cli#selecting-files-and-directories-to-be-uploaded).
    
3.  Your newly created deployment will go into a `frozen` state if it does not get any requests. Run:
    
    now scale ls
    
    Output:
    
        url                                           cur      min      max     auto      age
        8-end-lpbrrsahsc.now.sh                         1        0        1        ✔       9m
        
    
    Scale number indicates number of live instances of deployment. The value at `min` equals 0, this means that the deployment will go into a `frozen` state. To start `frozen` deployment, you have to visit it and wait for 10-20 seconds.
    
    To keep deployment alive, even when your app receives no traffic, you have to scale it to 1 or more instances:
    
    now scale https://8-end-lpbrrsahsc.now.sh 1
    
    If you want to deploy to 2 or more instances _automatically_ when your app gets more traffic, run:
    
    now scale https://8-end-lpbrrsahsc.now.sh 1 2 auto
    
    Check the [scaling docs](https://zeit.co/docs/getting-started/scaling) for more features.
    
4.  Go to `https://zeit.co/dashboard`. You will see something like:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37228712-601c2a9c-2396-11e8-98af-660a0563c96e.png)
    
    Click the hyperlinked deployment URL, and you will be redirected to a deployment overview page:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37228783-a4cdd190-2396-11e8-9036-92c468a26112.png)
    
    Click on `Logs` and `Source`:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37228843-d928eb8c-2396-11e8-8967-dceb8d8b75bb.png)
    
    ![Builder Book](https://user-images.githubusercontent.com/10218864/37228852-e21412da-2396-11e8-87da-a97387f061e3.png)
    
    Both the production logs and deployment's actual code are useful for debugging.
    
5.  The URL of our deployment is not memorable. It simply allows you to deploy an app and test it out before mapping it to an actual, memorable domain.  
    One way to create a more memorable URL is:
    
    now ln 8-end-lpbrrsahsc.now.sh builderbook.now.sh
    
    Now we can access our deployment at `https://builderbook.now.sh`.
    
    Another (more common) way is to point your deployment to a custom domain. Run this (remember to use _your_ actual domain):
    
    now ln 8-end-lpbrrsahsc.now.sh test.builderbook.org
    
    Now we can access our deployment at `https://test.builderbook.org`.
    
    If the `now ln` command produces an error, then you need to either add `CNAME` to your domain records or add a domain to [zeit.world](https://zeit.co/world). Here is a [link](https://zeit.co/docs/getting-started/assign-a-domain-name) with instructions for either case.
    
6.  You do not pay for deployments that are in a `frozen` state. The cool part about the `now ln` command is that a newly aliased deployment will _assume_ the scale settings of a previous deployment that was aliased to the same custom domain. At the same time, the old deployment will become `frozen`. For example, if the scale setting for our domain `test.builderbook.org` is `1 2 auto` and then we alias a new deployment to this domain, the new deployment will get `1 2 auto` settings automatically. Our old deployment, which got un-aliased, will get `0 1 auto` settings and will go into a `frozen` state.
    
    As a consequence, you will accumulate many `frozen` deployments. To reduce clutter, you can remove them with:
    
    now rm 8-end-lpbrrsahsc.now.sh
    
    If you don't like the `Are you sure?` step, run:
    
    now rm 8-end-lpbrrsahsc.now.sh -y
    
    If you like to remove _all_ deployments with a particular name that _are not_ aliased, use:
    
    now rm 8-end --safe -y
    
    The above command will remove all deployments that have the name `8-end` accept deployment that is aliased.
    

This was the last subsection of the last section in the last chapter of this book.  
If you got this far - wow!

We hope that you learned a lot and that you glued many concepts together while building this web application.

By now, you should answer this question from the Introduction chapter with a [resounding Yes](https://youtu.be/fcbj8BBsWSA?t=66) :

Have you ever built a production-ready web application from scratch by yourself?

If you have any questions or feedback, feel free to create an issue at:  
[https://github.com/builderbook/builderbook/issues](https://github.com/builderbook/builderbook/issues)

To give a review of our book, please fill out this form:  
[https://goo.gl/forms/JdevtnCWsLwZTAio2](https://goo.gl/forms/JdevtnCWsLwZTAio2)

* * *

At the end of Chapter 8, your codebase should look like the codebase in `8-end`. The [8-end](https://github.com/builderbook/builderbook/tree/master/book/8-end) folder is located at the root of the `book` directory inside the [builderbook repo](https://github.com/builderbook/builderbook).

Compare your codebase and make edits if needed.

* * *
