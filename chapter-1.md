---
title: App structure. Next.js. HOC. Material-UI. Server-side rendering. Styles.
seoTitle: Set up your web app with Next.js and Material-UI
seoDescription: This book teaches you how to build a production-ready web application from scratch using React, Material-UI, Next, Express, Mongoose, MongoDB. Chapter 1 shows you how to structure your app, write pages and components, and integrate Next.js and Material-UI.
isFree: true
---

* * *
*   Setup  
    *   Node and Yarn
    *   package.json
*   Code editor and lint  
    *   VS editor
    *   Eslint
    *   Prettier
*   App structure  
    *   Next.js
    *   Document
    *   Project structure
*   Index page  
*   Header component  
*   withLayout HOC  
*   Material-UI integration  
    *   Add styles on server
    *   Remove server-side styles
*   Server-side rendering  
* * *

Before you start working on Chapter 1, get the `1-start` codebase. The [1-start](https://github.com/builderbook/builderbook/tree/master/book/1-start) folder is located at the root of the `book` directory inside the [builderbook repo](https://github.com/builderbook/builderbook).

*   If you haven't cloned the builderbook repo yet, clone it to your local machine with `git clone git@github.com:builderbook/builderbook.git`.
*   Inside the `1-start` folder, run `yarn` to install all packages.

These are the packages and their versions that we install specifically for Chapter 1:

*   `"material-ui": "^1.0.0-beta.41"`
*   `"next": "^5.1.0"`
*   `"prop-types": "^15.6.0"`
*   `"react": "^16.2.0"`
*   `"react-dom": "^16.2.0"`
*   `"react-jss": "^8.2.1"`
*   `"babel-preset-env": "^1.6.1"`

* * *

You've read about the motivation for writing this book and building a web application. Motivation aside, this book will teach you how to build a modern-stack, production-ready web application _from scratch_. Together, we will go from 0 to over 12,000 lines of code in 8 chapters of this book.

In this very first chapter, we have multiple goals:

*   set up our coding environment
*   create our app structure
*   get familiar with Next.js (Next)
*   create our first page and component
*   create a higher-order component
*   integrate Next with Material-UI
*   learn about server-side rendering
*   add global and shared styles

## Setup
----------------------

We work on Ubuntu 16.04 LTS. we skipped the 17.04 release and decided to wait for stable 18.04. Thus, we provide installation instructions specific to a Linux-based OS (for example, Ubuntu and MacOS). We use the [Visual Studio](https://code.visualstudio.com) editor (VS editor), which we find easier to use and automate than any other popular editor. The web application that we build in this book will allow you, via integration with Github, to use VS editor for writing documentation and books.

The core technologies of the Builder Book app are React, Material-UI, Next, Express, Mongoose, and MongoDB. By the end of this chapter, we will create a static web app (no server and no database yet). To do so, we will install and integrate the first three technologies - React, Material-UI, and Next.

#### Node and Yarn

We are building a [Node](https://nodejs.org) app, and many of the tools that we use in this app also require Node.

We suggest using Node with the help of [nvm](https://github.com/creationix/nvm) (Node Version Manager). On Linux, press Ctrl+Alt+T to open your terminal (alternatively, use the search bar to search for terminal).

*   Run the command below to install nvm:  
    `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash`
    
*   Check the nvm version to confirm successful installation:  
    `nvm --version`
    
*   Trigger nvm:  
    `. ~/.nvm/nvm.sh`
    
*   Install Node 8.9.3:  
    `nvm install 8.9.3`
    
*   Make it default:  
    `nvm alias default 8.9.3`
    
*   Check Node version to confirm successful installation:  
    `node -v`
    

Node version should be 8.9.3.

Once Node is installed, we can install Yarn, a manager for third-party packages (also called dependencies or modules). Whenever we need to use code developed by other developers - we add the package name and version to a `package.json` file and run `yarn` in the app's directory. More on `package.json` in the next section.

*   Install [Yarn](https://yarnpkg.com/en/docs/install#linux-tab) by running the following two commands in your terminal:
    
        curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
        echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
        
    
*   Check Yarn version to confirm successful installation:  
    `yarn -v`
    

In this book, the Yarn version is `1.5.1`.

#### package.json

`package.json` is a required file at the root of any Node application. This file contains the app's metadata: app name, app version, scripts, and dependenices (described by name and version), among other properties. Read about [working with a package.json](https://docs.npmjs.com/getting-started/using-a-package.json) file to learn more about metadata. Some parameters are optional (`keywords`, `license`), but some are required (`name`, `version`).

Open the `1-start` folder located at `builderbook/book/`. Take a look at our `package.json` file:  
`package.json` :

    {
      "name": "1-start",
      "version": "0.0.1",
      "license": "MIT",
      "scripts": {
        "build": "next build",
        "start": "next start",
        "dev": "next"
      },
      "dependencies": {
        "material-ui": "^1.0.0-beta.41",
        "next": "^5.1.0",
        "prop-types": "^15.6.0",
        "react": "^16.2.0",
        "react-dom": "^16.2.0",
        "react-jss": "^8.2.1"
      },
      "devDependencies": {
        "babel-preset-env": "^1.6.1"
      }
    }
    

You can see required metadata like `name` and `version` (version format is `major.minor.patch`).

The section with `scripts` contains shortcuts for commands. At the end of this book, the `scripts` section will contain the following command:

"dev": "nodemon server/app.js --watch server --exec babel-node"

This shortcut allows us to run our app locally by typing `yarn start` in the terminal _instead_ of:

yarn nodemon server/app.js --watch server --exec babel-node`

Inside the `scripts` commands, you may choose to pass environmental variables. You would set it up by adding `NODE_ENV=production` to one of the commands in the `scripts` section. Let's consider an example. At the end of this book, we will start our app _locally_ using `yarn dev`, which has the following shortcut in the `scripts` section:

"dev": "nodemon server/app.js --watch server --exec babel-node"

We have _no environmental variables_ inside this command. Thus, `NODE_ENV` and `ROOT_URL` _default_ to `development` and `http://localhost:8000`, respectively.

To start our app on a _remote_ production server, we use `yarn start` with this shortcut:

"start": "NODE\_ENV=production ROOT\_URL=https://builderbook.org babel-node server/app.js"

In this case, the values of `NODE_ENV` and `ROOT_URL` are explicitly set, so they do not assume their default values. `NODE_ENV` and `ROOT_URL` equal `production` and `https://builderbook.org`, respectively.

In Chapter 2, we'll introduce the [dotenv](https://www.npmjs.com/package/dotenv) package, which will manage most of our environmental variables (over a dozen in total) in our app.

The next section in our `package.json` file is `dependencies`. This section contains a list of third-party packages that we need in production _and_ development environments. To install all packages from `package.json`, simply run `yarn` in your terminal while inside the app's directory.

To check if Yarn successfully installed the packages, look for a newly generated `node_modules` folder and `yarn.lock` lockfile at the app's root directory. The former folder contains the code of third-party packages, and the latter file contains the exact versions of packages and their dependencies.

In the `scripts` section, you can specify a combination of commands or run the scripts directly from your files. From our `package.json` file, if you run `yarn dev` in your terminal, you will first run `yarn next`.

`devDependencies` are dependencies that our app uses in development but _not_ in production. Typically, developers use packages in `devDependencies` to run tests, compile code, or lint code locally.

If you ran the `yarn` command inside the `1-start` folder, then you successfully installed all packages we need for Chapter 1. We will discuss each installed package later in this chapter.

## Code editor and lint
----------------------------------------------------

In this section, we will set up preferences for Visual Studio Code editor (VS editor) and discuss/install two extensions that help us _format_ code: Eslint and Prettier.

#### VS editor

We prefer the [Visual Studio](https://code.visualstudio.com) code editor, because it easily integrates with Github and Eslint (which formats code), and it comes with Terminal (this allows you to stay within the editor while running commands). Here is a typical view of the editor with terminal. A list of staged changes is one click away, and you can see that we have 2 staged changes:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36002112-cf3d1eb0-0cdd-11e8-8ca6-3e483fc659f8.png)

In addition, VS editor has many optional settings that may increase your productivity. You can access User Settings by going to `File`>`Preferences`>`Settings`>`User Settings`. Here is a list of our User Settings:

    {
      "window.zoomLevel": -1,
      "files.autoSave": "afterDelay",
      "git.enableSmartCommit": true,
      "editor.formatOnSave": true,
      "eslint.autoFixOnSave": true,
      "prettier.eslintIntegration": true,
    }
    

And here is a snapshot of our User Settings:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36043261-97adb33c-0d83-11e8-887c-01bc611790a2.png)

*   `"window.zoomLevel": -1` sets the zoom level to 80% - our personal preference, since we like to scroll less, especially when writing a tutorial or book.
*   `"files.autoSave": "afterDelay"` saves files automatically for you, so you don't need to save your files manually.
*   `"git.enableSmartCommit": true` stages changes automatically, so you don't need to stage your changes manually.
*   `"editor.formatOnSave": true` detects formatters, such as Eslint, and formats your code on each save event. To format code on a save event, you have to _manually_ save the file.

Below, we will discuss the remaining 2 settings that are related to Eslint and Prettier code formatters.

To format code, we use two extensions in Visual Studio: _ESLint_ by [Dirk Baeumer](https://github.com/dbaeumer) and _Prettier-JavaScript formatter_ by [Esben Petersen](https://github.com/esbenp):  
![Builder Book](https://user-images.githubusercontent.com/10218864/36002213-3473c176-0cde-11e8-83e3-021eb3709668.png)

#### Eslint

[Eslint](https://eslint.org) lints code, i.e. it checks code for potential formatting problems and fixes them. After you install the Eslint extension on VS editor, you will see new settings. Read the [full list of settings here](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint). One setting that we want to set to true is:

"eslint.autoFixOnSave": true

This setting makes Eslint apply fixes to your code when you save a file manually. Before we jump in and test Eslint out, we need to create a configuration file and install missing dependencies.

Eslint requires a `.eslintrc.js` file that contains a list of formatting rules. Formatting rules can specify the type of quotes and commas to use, the maximum length for a line of code, and more. Make sure to place the `.eslintrc.js` file at the app's root directory:  
`.eslintrc.js` :

    module.exports = {
      parser: 'babel-eslint',
      extends: 'airbnb',
      env: {
        browser: true,
        jest: true,
      },
      plugins: ['react', 'jsx-a11y', 'import'],
      rules: {
        'max-len': ['error', 100],
        'no-underscore-dangle': ['error', { allow: ['_id'] }],
        'prefer-destructuring': [
          'error',
          {
            VariableDeclarator: {
              array: false,
              object: true,
            },
            AssignmentExpression: {
              array: true,
              object: false,
            },
          },
          {
            enforceForRenamedProperties: false,
          },
        ],
        'import/prefer-default-export': 'off',
        'jsx-a11y/anchor-is-valid': 'off',
        'react/react-in-jsx-scope': 'off',
        'react/jsx-filename-extension': [
          'error',
          {
            extensions: ['.js'],
          },
        ],
      },
    };
    

We included the `.eslintrc.js` file with all necessary rules in our [1-end](https://github.com/builderbook/builderbook/tree/master/book/1-end) folder.

To make Eslint work properly, we need to install the missing packages that Eslint relies on.  
Add the following dependencies to your `package.json` file and run `yarn`:

    "devDependencies": {
        "babel-eslint": "^8.2.1",
        "babel-preset-env": "^1.6.1",
        "eslint": "^4.15.0",
        "eslint-config-airbnb": "^16.1.0",
        "eslint-plugin-import": "^2.8.0",
        "eslint-plugin-jsx-a11y": "^6.0.3",
        "eslint-plugin-react": "^7.5.1"
      }
    }
    

A list of all Eslint rules is in the [official docs](https://eslint.org/docs/rules). Check out, for example, the [max-len](https://eslint.org/docs/rules/max-len) rule.

Here's an example of code with an Eslint error for `max-len`:

var foo = { "bar": "This is a bar.", "baz": { "qux": "This is a qux" }, "difficult": "to read" };

And here's the same code without the `max-len` error:

    var foo = {
      "bar": "This is a bar.",
      "baz": { "qux": "This is a qux" },
      "easier": "to read"
    };
    

You may have noticed that we installed the [`eslint-config-airbnb`](https://www.npmjs.com/package/eslint-config-airbnb) package and [extended](https://eslint.org/docs/user-guide/configuring#extending-configuration-files) Eslint with `extends: 'airbnb'`. In addition to rules that we specified in `.eslintrc.js`, we use rules from the `eslint-config-airbnb` package.

Now that you've properly installed and configured Eslint, let's test it.

Create a `pages/index.js` file inside your `1-start` folder, then paste the following code:  
`pages/index.js` :

    [1, 2, 3].map(function (x) {
      const y = x + 1;
      return x * y;
    });
    

Eslint will highlight this code, and when you _hover over the highlighted code_, you will see a description of the error(s):

    [eslint] Unexpected unnamed function. (func-names)
    [eslint] Unexpected function expression. (prefer-arrow-callback)
    

This type of Eslint error is a result of [Airbnb's format rule for arrow functions](https://github.com/airbnb/javascript#arrow-functions). To fix this error, simply save the file manually with `Ctrl+S`, and the code will be edited into:  
`pages/index.js` :

    [1, 2, 3].map((x) => {
      const y = x + 1;
      return x * y;
    });
    

As we mentioned above, Eslint is able to fix some problems automatically when you save the file manually and havef `"eslint.autoFixOnSave": true` in your User Settings on VS editor.

You can also find Eslint problems in all files of your application. To do so, add one new script shortcut to the `scripts` section of your `package.json` file:  
`"lint": "eslint components pages lib server"`

The `scripts` section becomes:

    "scripts": {
        "build": "next build",
        "start": "next start",
        "dev": "next",
        "lint": "eslint components pages lib server"
    }
    

Go back to `pages/index.js` and re-add the code that contains Eslint errors:  
`pages/index.js` :

    [1, 2, 3].map(function (x) {
      const y = x + 1;
      return x * y;
    });
    

Run `yarn lint` inside your `1-start` folder. Eslint will find errors and a warning in `pages/index.js`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36347051-4a395432-1402-11e8-81f7-130a2ca90e6b.png)

Cool part - you can fix some of these code style errors automatically by running `yarn lint --fix`.  
Run `yarn lint --fix` and check out the `pages/index.js` file. The code will be automatically fixed by Eslint.

In case you want to prevent Eslint from editing or noting errors on part of your code, you can specify lines of code or files that Eslint should not run on.

To disable Eslint on a line of code, add the following code to that line:

// eslint-disable-line

To disable Eslint inside a file, add the following code to the top of that file:

/\* eslint-disable */

#### Prettier

[Prettier](https://prettier.io/docs/en/index.html) is also a code formatter. We use the [Prettier extension for VS editor](https://github.com/prettier/prettier-vscode) to complement Eslint.

Let's do a practical example to understand why we need Prettier. Make sure that the Prettier extension is _uninstalled_ and Eslint extension is _installed_.

Open your `pages/index.js` file and paste the following code:

foo(reallyLongArg(), reallyReallyLongArg(), omgSoManyParameters(), IShouldRefactorThis(), isThereSeriouslyAnotherOne());

Hover over the first highlight by Eslint:

\[eslint\] Line 17 exceeds the maximum line length of 100. (max-len)

Manually save file the with `Ctrl+S` \- the code does not get fixed with Eslint. Unlike the arrow function example from above, in this case, Eslint is not able to fix the error. To correct a `max-len` error, we use Prettier.

Install the Prettier extension. Tell Prettier to first run `prettier` and then run `eslint --fix` by specifying the Prettier setting:

"prettier.eslintIntegration": true

Go back to `pages/index.js` and manually save the file with `Ctrl+S`. Unlike before, the code will be automatically formatted:

    foo(
      reallyLongArg(),
      reallyReallyLongArg(),
      omgSoManyParameters(),
      IShouldRefactorThis(),
      isThereSeriouslyAnotherOne(),
    );
    

Now you're set for writing cleaner, formatted code. In the next section, we will discuss how to structure our app and get familiar with the Next.js framework.

## App structure
--------------------------------------

In this section we will discuss how to organize our app. We'll discuss where to put:

*   client-only code
*   code that is available on both client and server
*   server-only code

#### Next.js

[Next.js](https://github.com/zeit/next.js) provides our app with simple routing, error handling (404 and 500 HTTP errors), server-side rendering on initial load, and hot code reloading. In addition, Next.js uses [webpack](https://webpack.js.org) to automatically bundle modules (for example, pages and components) without any manual configuration by developers. Basically, if you like to start quickly and don't want to build these basic features of your web app from scratch, Next.js is a good choice.

We'll be discussing Next.js as we go through this book. To learn the basics, check out this [tutorial](https://learnnextjs.com/basics/getting-started) created by [Arunoda Susiripala](https://arunoda.me) from [Zeit](https://zeit.co).

We will discuss server-side rendering in more detail in the last three sections of this chapter, especially in the section titled `Server-side rendering`.

If you ran `yarn` inside the `1-start` folder, then Next.js is already installed. To check it - run `yarn build` in your terminal, and the `.next` folder will be generated at the root. Alternatively, run `yarn dev`, and the app should start at `http://localhost:3000`. You won't see any page on your browser, since we haven't created a page yet.

Next.js specifies rules on how to structure your app. You store code for pages in a `/pages` folder, and the name of each page file becomes that page's route.

You store components and static files (such as images) in `/components` and `/static` folders, respectively. Important note - when we prepare our app for production, we strongly recommend moving files from `/static` to some content delivery network (CDN). We store static resources at Google Cloud.

    ├── components                  # React components
    ├── lib                         # Code available on both client and server
    ├── pages                       # Pages
    ├── server                      # Server code
    ├── static                      # Static resources
    ├── package.json                # List of packages and scripts
    

We place code that should be available on both client and server (for example, higher-order components and API methods) in the `/lib` folder. Code in this folder should be available on the server for server-side rendering of pages (more on this below), and also on the client for client-side rendering.

We will put all server-only code, such as Express server, Express routes, and third-party APIs, in the `/server` folder.

Next.js lets you configure webpack by creating a `next.config.js` file at the root. You can also customize error pages 404 and 500 by creating a custom page inside `pages/_error.js`.

We won't customize webpack or error pages, but we will customize `<Document>` by creating `pages/_document.js`.

#### Document

In Next.js, you don't need to specify `<head>`, `<html>`, and `<body>` elements on every page. These elements are included automatically, but you can [customize](https://github.com/zeit/next.js#custom-document) them by extending the `Document` class inside a `pages/_document.js` file.

Create your `pages/_document.js` file:  
`pages/_document.js` :

    import Document, { Head, Main, NextScript } from 'next/document';
    
    export default class MyDocument extends Document {
      render() {
        return (
          <html lang="en">
            <Head />
            <body>
              <Main />
              <NextScript />
            </body>
          </html>
        )
      }
    }
    

Our web app, like any other, should have the following three elements inside its `<Header>`:

*   useful metadata for browsers
*   static resources (favicon, font, Material icons, minified css files for Nprogress bar and code highlighting)
*   global styles

In addition:

*   though not inside `<Header>`, we should add some styles for the `<body>` element as well

Translate English into HTML and update the code snippet above. You get this carcass for our page:  
`pages/_document.js` :

    import Document, { Head, Main, NextScript } from 'next/document';
    
    class MyDocument extends Document {
      render() {
        return (
          <html lang="en">
            <Head>
    
              // 1. metadata
    
              // 2. static resources (from CDN)
    
              // 3. global styles
    
            </Head>
            <body
              style={{ /* styles for body */ }}
            >
              <Main />
              <NextScript />
            </body>
          </html>
        )
      }
    }
    
    export default MyDocument;
    

Let's fill it out.

1.  Add the following four [meta tags](https://www.w3schools.com/tags/tag_meta.asp), though all of them are optional:
    
        <meta charSet="utf-8" /> 
        // tells browser that content is UTF-8 encoded
        
        <meta name="viewport" content="width=device-width, initial-scale=1.0" /> 
        // sets page width to screen width, sets initial zoom
        
        <meta name="google" content="notranslate" /> 
        // tell google to not show translate modals
        
        <meta name="theme-color" content="#1976D2" /> 
        // specifies color of browser on mobile device
        
    
2.  Add these four resources from CDN (we use Google Cloud, but you can upload your resources to any other CDN). We suggest adding favicon image, font, styles for Nprogress (introduced in Chapter 3), and styles for highlighting markdown (introduced in Chapter 5):
    
        <link
         rel="shortcut icon"
         href="https://storage.googleapis.com/builderbook/favicon32.png"
        />
        <link
         rel="stylesheet"
         href="https://fonts.googleapis.com/css?family=Muli:300,400:latin"
        />
        <link
         rel="stylesheet"
         href="https://fonts.googleapis.com/icon?family=Material+Icons"
        />
        <link
         rel="stylesheet"
         href="https://storage.googleapis.com/builderbook/nprogress.min.css"
        />
        <link rel="stylesheet" href="https://storage.googleapis.com/builderbook/vs.min.css" />
        
    
3.  Add global styles for `<a>`, `<blockquote>`, `<pre>`, and `<code>`. Our book content will contain the latter three elements. In Chapter 5, you will learn about the `marked` package and how it converts `markdown` into `HTML`. Styles:
    
        <style>
         {`
           a, a:focus {
             font-weight: 400;
             color: #1565C0;
             text-decoration: none;
             outline: none
           }
           a:hover, button:hover {
             opacity: 0.75;
             cursor: pointer
           }
           blockquote {
             padding: 0 1em;
             color: #555;
             border-left: 0.25em solid #dfe2e5;
           }
           pre {
             display:block;
             overflow-x:auto;
             padding:0.5em;
             background:#FFF;
             color: #000;
             border: 1px solid #ddd;
           }
           code {
             font-size: 14px;
             background: #FFF;
             padding: 3px 5px;
           }
         `}
        </style>
        
    
4.  Finally, add a `<body>` element with these styles:
    
        <body
         style={{
           font: '16px Muli',
           color: '#222',
           margin: '0px auto',
           fontWeight: '300',
           lineHeight: '1.5em',
           backgroundColor: '#F7F9FC',
         }}
        >
         // some code
        </body>
        
    

Paste the code snippets from steps 1 to 4 into the page carcass, and you get:  
`pages/_document.js` :

    import Document, { Head, Main, NextScript } from 'next/document';
    
    class MyDocument extends Document {
      render() {
        return (
          <html lang="en">
            <Head>
              <meta charSet="utf-8" />
              <meta name="viewport" content="width=device-width, initial-scale=1.0" />
              <meta name="google" content="notranslate" />
              <meta name="theme-color" content="#1976D2" />
    
              <link
                rel="shortcut icon"
                href="https://storage.googleapis.com/builderbook/favicon32.png"
              />
              <link
                rel="stylesheet"
                href="https://fonts.googleapis.com/css?family=Muli:300,400:latin"
              />
              <link
                rel="stylesheet"
                href="https://fonts.googleapis.com/icon?family=Material+Icons"
              />
              <link
                rel="stylesheet"
                href="https://storage.googleapis.com/builderbook/nprogress.min.css"
              />
              <link rel="stylesheet" href="https://storage.googleapis.com/builderbook/vs.min.css" />
    
              <style>
                {`
                  a, a:focus {
                    font-weight: 400;
                    color: #1565C0;
                    text-decoration: none;
                    outline: none
                  }
                  a:hover, button:hover {
                    opacity: 0.75;
                    cursor: pointer
                  }
                  blockquote {
                    padding: 0 1em;
                    color: #555;
                    border-left: 0.25em solid #dfe2e5;
                  }
                  pre {
                    display: block;
                    overflow-x: auto;
                    padding: 0.5em;
                    background: #FFF;
                    border: 1px solid #ddd;
                  }
                  code {
                    font-size: 14px;
                    background: #FFF;
                    padding: 3px 5px;
                  }
                `}
              </style>
            </Head>
            <body
              style={{
                font: '16px Muli',
                color: '#222',
                margin: '0px auto',
                fontWeight: '300',
                lineHeight: '1.5em',
                backgroundColor: '#F7F9FC',
              }}
            >
              <Main />
              <NextScript />
            </body>
          </html>
        );
      }
    }
    
    export default MyDocument;
    

#### Babel

[Babel](https://babeljs.io) is a JavaScript compiler. It will take the Javascript code that we wrote and convert it into JavaScript that browsers understand. To configure Babel, we need to tell Babel which presets to use.

Babel preset is a set of plugins that support particular language features. For example, preset `react` supports `JSX` features, while preset `es2015` supports `ES2015/ES6` features. You specify presets in a `.babelrc` file at your app's root.

We use [preset env](https://github.com/babel/babel/tree/master/packages/babel-preset-env) (listed in the `devDependencies` section inside `package.json`) to _automatically_ determine the Babel plugin that we need.

Create a `.babelrc` file and add `env` to it:  
`.babelrc` :

    {
      "presets": ["env"]
    }
    

[According to Next.js docs](https://github.com/zeit/next.js#customizing-babel-config) \- if we use `.babelrc`, then the default behaviour of Next.js will be overwritten, and we must specify an additional preset `next/babel` that is used by Next.js. Thus, the final version of our `.babelrc` file:  
`.babelrc` :

    {
      "presets": ["env", "next/babel"]
    }
    

Besides the `.babelrc` and `.eslintrc.js` files, we provided you with a `.gitignore` file that contains a list of sensitive files that we strongly recommend _not_ to store on a remote Github repository. For example, `.gitignore` has the `.env` file that contains sensitive API keys and secrets (introduced in Chapter 2).

In the next section, we will write our first page and component. After that, we will start our app with `yarn dev` and check out `<head>`, `<html>`, and `<body>` of our page using `Developer tools > Elements` on Chrome.

## Index page
--------------------------------

Time to create our very first page - the `Index` page. In ES6, `const Index = React.createClass({})` becomes `class Index extends React.Component {}`. Thus, you get:  
`pages/index.js` :

    import React from 'react';
    
    class Index extends React.Component {
      render() {
        return <div>some content</div>;
      }
    };
    

We will use the above syntax for components with `state`. _However_, since our first version of the `Index` component _has no_:

*   `state`,
*   [refs](https://reactjs.org/docs/refs-and-the-dom.html),
*   lifecycle methods (such as `componentDidMount` or any other).

Thus we may choose to write component as stateless functional component. Example of a stateless functional component, `Pure`:

    const Pure = () => {
      return <div>some content</div>
    };
    
    export default Pure;
    

In the case of our `Index` component, we get:  
`pages/index.js` :

    import Head from 'next/head';
    
    const Index = () => (
      <div style={{ padding: '10px 45px' }}>
        <Head>
          <title>Index page</title>
          <meta name="description" content="This is description of Index page" />
        </Head>
        <p>Content on Index page</p>
      </div>
    );
    
    export default Index;
    

Check out [this blog](https://hackernoon.com/react-stateless-functional-components-nine-wins-you-might-have-overlooked-997b0d933dbc) post about the pros and cons of using stateless functional component.

We imported `Head` from `next/head` and specified a title and description for proper indexing by search engines (good for SEO). Title and description will be added to the `<head>` element of our `Index` page. We also added some padding to the page by using the inline (local) style `style={{ padding: '10px 45px' }}`.

Time to test. Start your app with `yarn dev`, go to `http://localhost:3000`. You will see a page with `Content on Index page`. On Chrome, go to `Developer tools > Elements`. Find and click on the `<head>` element:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36058930-b507ef7a-0de1-11e8-9c6c-184f3bd321cd.png)

Good job if you see the proper `<title>` and `<meta name="description">` from `pages/index.js`. You can also see the link tags and styles, as well as other meta tags, that we added earlier to `pages/_document.js`. Now you know how customize `<Document>` and create a page in a React/Next.js app!

Rename `pages/index.js` to `pages/about.js`. Start your app with `yarn dev` and go to `http://localhost:3000`. You see a simple 404 error page provided by Next.js:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36058956-8015dff6-0de2-11e8-9f71-7881897d8065.png)

To see our page, you have to navigate to the `/about` route (`http://localhost:3000/about`), since the file name is the page route in Next.js.

Remember to rename the file back to `pages/index.js`.

## Header component
--------------------------------------------

You may have noticed that our `Index` page has no Header. Let's create a `Header` component as a stateless functional component in the same way we wrote the `Index` component:  
`components/Header.js` :

    const Header = () => {
      return <div>some content</div>
    };
    
    export default Header;
    

We use Next.js's `<Link>` element for routing - [read more about it](https://github.com/zeit/next.js#with-link). It's possible to prefetch a route with `<Link prefetch>`. The [prefetch feature](https://github.com/zeit/next.js#prefetching-pages) is production-only. If you specify a `prefetch` option, then your app will download the static code of a page with the route specified in the `href` parameter of `<Link>`.

If your page has only static code and no data to wait for, then loading the prefetched page will feel almost instant. When a page needs to wait for data, there will be a delay; however, the page will still load faster than without using `prefetch`.

We will use the `prefetch` option to almost instantly load our `Login` page (introduced in Chapter 3 and tested, after deployment, in Chapter 8). The `Login` page is a static page that has no data to wait for. In Chapter 3, we will test how prefetch works. For now, we will only add `prefetch` to `<Link>`:  
`components/Header.js` :

    import Link from 'next/link';
    
    const Header = () => (
      <div>
        <Link prefetch href="/login">
          <a style={{ margin: '0px 20px 0px auto' }}>Log in</a>
        </Link>
      </div>
    );
    
    export default Header;
    

Import this `Header` component into the `Index` page:

    import Head from 'next/head';
    import Header from '../components/Header';
    
    const Index = () => (
      <div style={{ padding: '10px 45px' }}>
        <Head>
          <title>Index page</title>
          <meta name="description" content="This is description of Index page" />
        </Head>
        <Header />
        <p>Content on Index page</p>
      </div>
    );
    
    export default Index;
    

Start your app with `yarn dev` and go to `http://localhost:3000`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36064797-89e2f240-0e45-11e8-84bf-526a8ae3302d.png)

That's a starting point but not acceptable. We should make some UI improvements.

Eventually, in our `Header` component, we want to place multiple items in separate columns. In the Material-UI library (installed with `yarn` at the start of this chapter), the [`Grid`](https://material-ui-next.com/layout/grid) element creates a column grid, and the [`Toolbar`](https://material-ui-next.com/demos/app-bar) element adds a bar with action items. Click the hyperlinks to learn more about their properties. We use them as follows:  
`components/Header.js` :

    import Link from 'next/link';
    
    import Toolbar from 'material-ui/Toolbar';
    import Grid from 'material-ui/Grid';
    
    import { styleToolbar } from './SharedStyles';
    
    const Header = () => (
      <div>
        <Toolbar style={styleToolbar}>
          <Grid container direction="row" justify="space-around" align="center">
            <Grid item xs={12} style={{ textAlign: 'right' }}>
              <Link prefetch href="/login">
                <a style={{ margin: '0px 20px 0px auto' }}>Log in</a>
              </Link>
            </Grid>
          </Grid>
        </Toolbar>
      </div>
    );
    
    export default Header;
    

Earlier in this chapter, we added some _global_ styles to our `pages/_document.js`. We call these styles global because our app applies them to _all_ elements. However, in some situations, you want to use a style only in a few places. We can call these styles _shared_ to emphasize that they can be exported/imported and shared among a few components and pages.

We place _shared_ styles in `components/SharedStyles.js`. Create this file with the following content:

    const styleToolbar = {
      background: '#FFF',
      height: '64px',
      paddingRight: '20px'
    }
    
    module.exports = {
      styleToolbar
    }
    

As you can see, we imported `styleToolbar` to our `Header` component and used this style on the `<Toolbar>` element.

In [`1-end/components/SharedStyles.js`](https://github.com/builderbook/builderbook/blob/master/book/1-end/components/SharedStyles.js), we provide a few more styles that we will use in this book.

Start your app with `yarn dev` and go to `http://localhost:3000`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36064939-fc9dfa94-0e47-11e8-9307-c20d31c34e74.png)

Looking better. We will add one more improvement in the next section.

## withLayout HOC
----------------------------------------

In the previous section, we imported and added the `Header` component to our `Index` page. Our app will have 5+ pages, so importing and adding the `Header` component to _each_ page will be time-consuming. It will be much faster to _wrap_ our pages with a [higher-order component](https://reactjs.org/docs/higher-order-components.html) that contains the `Header` component. In addition to the `Header` component, this higher-order component (HOC) can pass some props to our pages. In Chapter 2, we will use a HOC to pass a `user` prop to our `Header` component.

In React, by definition, a HOC (or HOC function) is a function that takes a component (`BaseComponent`) and returns a new component (`App`):

const App = higherOrderComponent(BaseComponent)

In our case, we will _wrap_ our `Index` component with our `withLayout` HOC during export:

export default withLayout(Index)

Before we start using the `withLayout` HOC for wrapping pages, we need to construct it. Based on the HOC definition above, our `withLayout` function should take `BaseComponent` as an argument and return a new component `App`. You _already_ know from the `Index page` section that, in ES6, we define an `App` component with `class App extends React.Component {}`:

    class App extends React.Component {
      render() {
        return (
          <div>
            <CssBaseline />
            <BaseComponent {...this.props} />
          </div>
        );
      }
    }
    

We'll wrap this `App` component with our `withLayout()` function that takes `BaseComponent` as an argument and returns a new `App` component:

    function withLayout(BaseComponent) {
      class App extends React.Component {
        render() {
          return (
            <div>
              <CssBaseline />
              <BaseComponent {...this.props} />
            </div>
          );
        }
      }
    
      return App;
    }
    

Create a `lib/withLayout.js` file. Add missing imports and the code block above to the file:  
`lib/withLayout.js` :

    import React from 'react';
    import CssBaseline from 'material-ui/CssBaseline';
    
    function withLayout(BaseComponent) {
      class App extends React.Component {
        render() {
          return (
            <div>
              <CssBaseline />
              <BaseComponent {...this.props} />
            </div>
          );
        }
      }
    
      return App;
    }
    
    export default withLayout;
    

You may have noticed that we added `<CssBaseline />` from Material-UI. This element adds some basic styles to our `withLayout` HOC. For example, `<CssBaseline />` adds this style:

    html: {
          WebkitFontSmoothing: 'antialiased',
          MozOsxFontSmoothing: 'grayscale',
        },
    

Fonts look a bit better with this style. You can see exactly [what adding CssBaseline does](https://github.com/mui-org/material-ui/blob/v1-beta/src/CssBaseline/CssBaseline.js).

At this point, we made a basic version of our `withLayout` HOC. Now it's time to test.  
Import and add the `Header` component to our `withLayout` component:  
`lib/withLayout.js` :

    import React from 'react';
    import CssBaseline from 'material-ui/CssBaseline';
    import Header from '../components/Header';
    
    function withLayout(BaseComponent) {
      class App extends React.Component {
        render() {
          return (
            <div>
              <CssBaseline />
              <Header {...this.props} />
              <BaseComponent {...this.props} />
            </div>
          );
        }
      }
    
      return App;
    }
    
    export default withLayout;
    

At this point, Eslint will suggest you to re-write the component as a stateless function. Since our `withLayout` HOC will get more complicated, you can ignore this suggestion by adding `/* eslint-disable */` to the top of `lib/withLayout.js`.

Add our `withLayout` HOC and remove the `Header` component on our `Index` page. Remember to include all imports:  
`pages/index.js` :

    import Head from 'next/head';
    import withLayout from '../lib/withLayout';
    
    const Index = () => (
      <div style={{ padding: '10px 45px' }}>
        <Head>
          <title>Index page</title>
          <meta name="description" content="This is description of Index page" />
        </Head>
        <p>Content on Index page</p>
      </div>
    );
    
    export default withLayout(Index);
    

This is a good place to test. Start your app with `yarn dev` and go to `http://localhost:3000`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36066890-8940c390-0e67-11e8-97a7-ff31a91a8e3d.png)

Good job if you can see the `Header` component.

Open `lib/withLayout.js`, remove `<Header {...this.props} />` and `import Header from '../components/Header';`. Save the file, and Next.js will reload the app. Go to `http://localhost:3000`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36066924-fdd57f02-0e67-11e8-9323-5b59f18dc7f4.png)

As we expect, no `Header` component this time. Remember to add the two lines back to your `withLayout.js` file.

We should make one more improvement to our `withLayout` HOC. In Chapter 2, we will pass a `user` prop to the `Index` page using `Index.getInitialProps()`. The method `getInitialProps()` is a Next.js method that acts on a page component and populates the page props with data. We discuss `getInitialProps()` in more detail in Chapters 2 and 3.

Here we can write a a block of code that checks whether `BaseComponent` (for example the `Index` component) has initial data, and if it does, then pass that data to props of the `App` component. If `BaseComponent` has no initial data, then we pass an empty object:

    App.getInitialProps = (ctx) => {
      if (BaseComponent.getInitialProps) {
        return BaseComponent.getInitialProps(ctx);
      }
    
      return {};
    };
    

Why is this logic useful in our `withLayout` HOC? Consider this situation - in Chapter 2, we will pass a `user` prop to our `Index` page with `Index.getInitialProps()`. The code above will pass the `user` prop to `App`, and as a result, the `Header` component will get the `user` prop as well because of `<Header {...this.props} />`. Then, by using a `user` object, we are able to show a user his/her avatar and name on the `Header` component. This is a typical UX for dashboards.

Add the code block above to the current version of our `withLayout` HOC, and you get:  
`lib/withLayout.js` :

    import React from 'react';
    import CssBaseline from 'material-ui/CssBaseline';
    import Header from '../components/Header';
    
    function withLayout(BaseComponent) {
      class App extends React.Component {
        render() {
          return (
            <div>
              <CssBaseline />
              <Header {...this.props} />
              <BaseComponent {...this.props} />
            </div>
          );
        }
      }
    
      App.getInitialProps = (ctx) => {
        if (BaseComponent.getInitialProps) {
          return BaseComponent.getInitialProps(ctx);
        }
    
        return {};
      };
    
      return App;
    }
    
    export default withLayout;
    

## Material-UI integration
----------------------------------------------------------

[Material Design](https://material.io/guidelines) is a design framework for web and mobile apps. It was developed and released by Google under an Apache-2 license. Since then, developers have created React-specific libraries for material design. We use the [Material-UI](https://github.com/mui-org/material-ui) library.

To test out if we properly integrated Material-UI with Next.js, let's add `<Button>` from Material-UI to our `Index` page:  
`pages/index.js` :

    import Head from 'next/head';
    import Button from 'material-ui/Button';
    import withLayout from '../lib/withLayout';
    
    const Index = () => (
      <div style={{ padding: '10px 45px' }}>
        <Head>
          <title>Index page</title>
          <meta name="description" content="This is description of Index page" />
        </Head>
        <p>Content on Index page</p>
        <Button variant="raised">
          MUI button
        </Button>
      </div>
    );
    
    export default withLayout(Index);
    

Start your app with `yarn dev` and go to `http://localhost:3000`. Look at the button while refreshing the tab a few times. You will see a flash of style - at first, the button looks like a simple HTML button:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36067926-818a8792-0e7d-11e8-9b26-18cf3711de26.png)

After 500 ms or so, the button gets styling from Material-UI and looks like:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36067911-547db864-0e7d-11e8-96b7-8fa45876981c.png)

You see a flash of style because the server renders the page with HTML only, _without styles_. The server sends this HTML-rendered page to the client (browser), where you see a simple HTML button. A few moments later, the client injects styles to HTML, and you see the button styled with Material-UI.

To better understand what we mean by `the client injects styles to HTML`, go to `http://localhost:3000`. Open `Developer tools > Elements`. Click on the `<head>` element and delete the following two `<style>` tags:

*   `<style type="text/css" data-jss="" data-meta="MuiButtonBase"></style>`
*   `<style type="text/css" data-jss="" data-meta="MuiButton"></style>`

Here is a screenshot to help you find these tags:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36069169-7f721c1a-0e99-11e8-8583-aabdf9dd59f4.png)

After you delete these `<style>` tags, you will see a simple HTML button with no style, i.e. the client has no style to inject to the HTML:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36069201-d304ef88-0e99-11e8-9385-be9bf034d9cc.png)

Next.js renders pages on the server (for initial load); however, server-side rendering does not come out-of-the-box in Material-UI. We need to write code to integrate Material-UI with Next.js. It's a big task, but luckily, Material-UI contributors maintain a [good example of Next.js/Material-UI integration](https://github.com/mui-org/material-ui/blob/v1-beta/examples/nextjs). We encourage you to check out this official example, since we will follow it closely.

From our testing, we understand what happens between server and client to cause the flash of style. We see that the client injects styles to HTML, but our app _does not_ add styles to HTML _on the server_.

We want to set up our app to add styles on the server. But after we successfully inject styles to our HTML on the server (server-side styles), we'll need to _remove_ the server-side styles once the client injects styles (client-side styles). If we don't remove server-side styles, then the same HTML elements will have both server-side and client-side styles, and this may lead to errors. Remember that with Next.js, only the initial load is server-rendered. Subsequent renders happen on the client, so there's no reason to keep server-side styles after injecting client-side styles.

To be able to pass styles to a page on the server, we need to create a `pageContext` object that contains `theme`, `sheetManager`, `sheetRegistry`, and `generateClassName`. We'll discuss the purpose of these 4 parameters in the next section. To inject styles on the server, we will pass `pageContext` to our `MyDocument` component (`pages/_document.js`) before rendering.

Let's summarize what we need to do:

1.  Create page context
2.  Inject styles on the server
3.  Remove server-side styles to avoid side effects

#### Create page context

We will create `pageContext` by following the [official Next.js/Material-UI example](https://github.com/mui-org/material-ui/blob/v1-beta/examples/nextjs) closely.

`pageContext` defined as:

    function createPageContext() {
      return {
        theme,
        sheetsManager: new Map(),
        sheetsRegistry: new SheetsRegistry(),
        generateClassName: createGenerateClassName(),
      };
    }
    

To inject styles on the server, we need to stringify our CSS styles. To stringify CSS styles, we use [SheetsRegistry](http://cssinjs.org/js-api/?v=v9.8.0#style-sheets-registry) from the [react-jss](https://github.com/cssinjs/react-jss) package. We access `sheetRegistry` inside `pageContext` and apply the `toString()` method:

pageContext.sheetsRegistry.toString()

1.  The [theme](https://github.com/cssinjs/react-jss#theming) object allows us to theme our `withLayout` HOC (at `lib/withLayout.js`) with simply:
    
        <MuiThemeProvider
         theme={this.pageContext.theme}
        >
         <BaseComponent {...this.props} />
        </MuiThemeProvider>
        
    
    For example, we can add `primary` and `secondary` colors to our theme:
    
        const theme = createMuiTheme({
         palette: {
           primary: { main: blue[700] },
           secondary: { main: grey[700] },
         },
        });
        
    
    Once specified, we can use these colors to style any element, such as a button:  
    `<Button color="primary" variant="raised">MUI button</Button>`
    
2.  [sheetsManager](http://cssinjs.org/js-api?v=v9.8.0#style-sheets-manager) counts how many elements use the same [Style Sheet](http://cssinjs.org/js-api?v=v9.8.0#create-style-sheet) and automatically attaches/detaches the Style Sheet to/from elements. We use `sheetsManager` to inject styles to our `withLayout` HOC:
    
        <MuiThemeProvider
         theme={this.pageContext.theme}
         sheetsManager={this.pageContext.sheetsManager}
        >
         <BaseComponent {...this.props} />
        </MuiThemeProvider>
        
    
3.  [sheetsRegistry](http://cssinjs.org/js-api?v=v9.8.0#style-sheets-registry) gets all CSS styles as a string with:
    
    pageContext.sheetsRegistry.toString()
    
    We need to add a string of styles to the `<style>` tag using [innerHTML](https://www.w3schools.com/jsref/prop_html_innerhtml.asp) (DOM method that sets the HTML content of an element). React uses [dangerouslySetInnerHTML](https://reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml) instead of `innerHTML` to set HTML content. Thus, we will pass a string of styles on the server with:
    
        <style
         id="jss-server-side"
         dangerouslySetInnerHTML={{
           __html: pageContext.sheetsRegistry.toString(),
         }}
        />
        
    
    We use `dangerouslySetInnerHTML` for styles, since JSX does not support HTML inside the `<style>` tag.
    
    Here is an [example](https://github.com/cssinjs/react-jss#server-side-rendering) of passing Style Sheets to `<App />` with `<JssProvider>`:
    
        <JssProvider registry={sheets}>
           <App />
        </JssProvider>
        
    
    In our case:
    
        <JssProvider
         registry={pageContext.sheetsRegistry}
        >
         <Component pageContext={pageContext} {...props} />
        </JssProvider>
        
    
4.  [generateClassName](http://cssinjs.org/js-api?v=v9.8.0#generate-your-own-class-names) generates unique class names for HTML elements. For example, a rendered button may get the unique class `class="jss111 jss96 jss101 jss104"`. Here is an [example](https://github.com/cssinjs/react-jss#multi-tree-setup) of passing `generateClassName` to `<App />` with `<JssProvider>`:
    
        <JssProvider generateClassName={generateClassName}>
         <App />
        </JssProvider>
        
    
    In our case:
    
        <JssProvider
         registry={pageContext.sheetsRegistry}
         generateClassName={pageContext.generateClassName}
        >
         <Component pageContext={pageContext} {...props} />
        </JssProvider>
        
    

Let's put together a `lib/context.js` file based on the [official example](https://github.com/mui-org/material-ui/blob/v1-beta/examples/nextjs/src/getPageContext.js):  
`lib/context.js` :

    import { SheetsRegistry } from 'react-jss';
    import { createMuiTheme, createGenerateClassName } from 'material-ui/styles';
    import blue from 'material-ui/colors/blue';
    import grey from 'material-ui/colors/grey';
    
    const theme = createMuiTheme({
      palette: {
        primary: { main: blue[700] },
        secondary: { main: grey[700] },
      },
    });
    
    function createPageContext() {
      return {
        theme,
        sheetsManager: new Map(),
        sheetsRegistry: new SheetsRegistry(),
        generateClassName: createGenerateClassName(),
      };
    }
    
    export default function getContext() {
      if (!process.browser) {
        return createPageContext();
      }
    
      if (!global.INIT_MATERIAL_UI) {
        global.INIT_MATERIAL_UI = createPageContext();
      }
    
      return global.INIT_MATERIAL_UI;
    }
    

The following code snippet calls the function `createPageContext()`, or creates new page context for every _server-side_ request (`!process.browser`):

    if (!process.browser) {
      return createPageContext();
    }
    

The following code snippet points `global.INIT_MATERIAL_UI` to `createPageContext()` and returns (i.e. makes available on the client) `global.INIT_MATERIAL_UI`:

    if (!global.INIT_MATERIAL_UI) {
      global.INIT_MATERIAL_UI = createPageContext();
    }
    
    return global.INIT_MATERIAL_UI;
    

When we are done integrating Material-UI with Next.js, we will check the value of `global.INIT_MATERIAL_UI` on the client (browser).

#### Inject styles on server

In this subsection, our goal is to render a page inside our custom document (`pages/_document.js`) with the `renderPage()` function as shown in the [Next.js example for custom Document](https://github.com/zeit/next.js/blob/canary/examples/with-styled-components/pages/_document.js#L5):

    static getInitialProps ({ renderPage }) {
      const sheet = new ServerStyleSheet()
      const page = renderPage(App => props => sheet.collectStyles(<App {...props} />))
      const styleTags = sheet.getStyleElement()
    
      return { ...page, styleTags }
    }
    

However, we need to adapt the example to our situation:

1.  In our situation, we want to import context:
    
    import getContext from '../lib/context';
    
    And point `pageContext` to it:
    
    const pageContext = getContext();
    
2.  We also want to pass the `pageContext` prop and other props to `<Component>`:
    
3.  And pass `sheetsRegistry` and `generateClassName` to `<JssProvider>` as discussed in the previous subsection. Wrap `<Component>` in `<JssProvider>`:
    
        <JssProvider
         registry={pageContext.sheetsRegistry}
         generateClassName={pageContext.generateClassName}
        >
         <Component pageContext={pageContext} {...props} />
        </JssProvider>
        
    
4.  Finally, in addition to the page, we want to return `pageContext` and `styles`:
    
        return {
         ...page,
         pageContext,
         styles: (
           <style
             id="jss-server-side"
             // eslint-disable-next-line
             dangerouslySetInnerHTML={{
               __html: pageContext.sheetsRegistry.toString(),
             }}
           />
         ),
        };
        
    

Let's make a short detour to understand how the spread operator works.

You may have noticed `...` or [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax). The spread operator creates a new object with parameters that are cloned from the original object. To better understand it, paste the following code into your browser console. Go to Chrome, open a tab, and access `Developer tools > Console`:

    var foo = { a:1, b:2, c:3 }
    
    console.log({ ...foo });
    console.log(Object.assign({}, foo));
    

Run the code by pressing `Enter`.  
![Builder Book](https://user-images.githubusercontent.com/10218864/36225224-d742e8ec-117e-11e8-8c0a-76f8e87db5db.png)

The line `console.log({ ...foo });` outputs a new object with parameters cloned from the `foo` object:

{a: 1, b: 2, c: 3}

Since the outputs of both `console.log()` statements are the same, you can see that the statement `{ ...foo }` is equivalent to `Object.assign({}, foo)`. [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) with an empty first parameter also creates a new object with cloned parameters from the original object.

In this book, we strive for shorter and more readable syntax. Thus, we pick the spread operator (`...`) over `Object.assign()`.

Back to our situation. Instead of cloning the parameters of a `foo` object, we clone parameters of our `page` object and add two more parameters - `pageContext` and `styles` \- to our newly created object:

    {
      ...page,
      pageContext,
      styles
    }
    

The code snippet above is equivalent to:

    var foo = { a:1, b:2, c:3 }
    console.log({ ...foo, d:4, e:5 });
    

Run this code snippet in your browser console:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36225880-fe0a8316-1180-11e8-883e-309f7faf7807.png)

As expected, the output is:

{a: 1, b: 2, c: 3, d: 4, e: 5}

Let's go back to editing `pages/_document.js`. Modify the [official example](https://github.com/zeit/next.js/blob/canary/examples/with-styled-components/pages/_document.js#L5) using our discussion from steps 1-4 above, and you get:

    MyDocument.getInitialProps = ({ renderPage }) => {
      const pageContext = getContext();
    
      const page = renderPage(Component => props => (
        <JssProvider
          registry={pageContext.sheetsRegistry}
          generateClassName={pageContext.generateClassName}
        >
          <Component pageContext={pageContext} {...props} />
        </JssProvider>
      ));
    
      return {
        ...page,
        pageContext,
        styles: (
          <style
            id="jss-server-side"
            // eslint-disable-next-line
            dangerouslySetInnerHTML={{
              __html: pageContext.sheetsRegistry.toString(),
            }}
          />
        ),
      };
    };
    

The final, updated `MyDocument` component:  
`pages/_document.js` :

    import React from 'react';
    import Document, { Head, Main, NextScript } from 'next/document';
    import JssProvider from 'react-jss/lib/JssProvider';
    import getContext from '../lib/context';
    
    class MyDocument extends Document {
      render() {
        return (
          <html lang="en">
            <Head>
              <meta charSet="utf-8" />
              <meta name="viewport" content="width=device-width, initial-scale=1.0" />
              <meta name="google" content="notranslate" />
              <meta name="theme-color" content="#1976D2" />
    
              <link
                rel="shortcut icon"
                href="https://storage.googleapis.com/builderbook/favicon32.png"
              />
              <link
                rel="stylesheet"
                href="https://fonts.googleapis.com/css?family=Muli:300,400:latin"
              />
              <link
                rel="stylesheet"
                href="https://storage.googleapis.com/builderbook/nprogress.min.css"
              />
              <link rel="stylesheet" href="https://storage.googleapis.com/builderbook/vs.min.css" />
    
              <style>
                {`
                  a, a:focus {
                    font-weight: 400;
                    color: #1565C0;
                    text-decoration: none;
                    outline: none
                  }
                  a:hover, button:hover {
                    opacity: 0.75;
                    cursor: pointer
                  }
                  blockquote {
                    padding: 0 1em;
                    color: #555;
                    border-left: 0.25em solid #dfe2e5;
                  }
                  pre {
                    display:block;
                    overflow-x:auto;
                    padding:0.5em;
                    background:#FFF;
                    color: #000;
                    border: 1px solid #ddd;
                  }
                  code {
                    font-size: 14px;
                    background: #FFF;
                    padding: 3px 5px;
                  }
                `}
              </style>
            </Head>
            <body
              style={{
                font: '16px Muli',
                color: '#222',
                margin: '0px auto',
                fontWeight: '300',
                lineHeight: '1.5em',
                backgroundColor: '#F7F9FC',
              }}
            >
              <Main />
              <NextScript />
            </body>
          </html>
        );
      }
    }
    
    MyDocument.getInitialProps = ({ renderPage }) => {
      const pageContext = getContext();
    
      const page = renderPage(Component => props => (
        <JssProvider
          registry={pageContext.sheetsRegistry}
          generateClassName={pageContext.generateClassName}
        >
          <Component pageContext={pageContext} {...props} />
        </JssProvider>
      ));
    
      return {
        ...page,
        pageContext,
        styles: (
          <style
            id="jss-server-side"
            // eslint-disable-next-line
            dangerouslySetInnerHTML={{
              __html: pageContext.sheetsRegistry.toString(),
            }}
          />
        ),
      };
    };
    
    export default MyDocument;
    

#### Remove server-side styles

We are almost done. Here, we modify our `withLayout` HOC to remove server-side injected styles after this component is mounted on the browser. We'll closely follow this [official example](https://github.com/mui-org/material-ui/blob/v1-beta/examples/nextjs/src/withRoot.js).

Our tasks for modifying `withLayout` are:

1.  before the component renders ([componentWillMount()](https://reactjs.org/docs/react-component.html#unsafe_componentwillmount)), define `pageContext` by pointing it to an existing context (`this.props.pageContext`) or new context (`getContext()`):
    
        componentWillMount() {
         this.pageContext = this.props.pageContext || getContext();
        }
        
    
    However, `componentWillMount` is in the process of depreciation and React won't support it in versions 17+. Thus let's set component's `pageContext` with React's `constructor(props)` like this:
    
        constructor(props, context) {
         super(props, context);
         this.pageContext = this.props.pageContext || getContext();
        }
        
    
2.  after the component renders ([componentDidMount()](https://reactjs.org/docs/react-component.html#componentdidmount)), _remove server-side styles_ by selecting an element with the id `id="jss-server-side"` with the [document.querySelector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) method:
    
        componentDidMount() {
         const jssStyles = document.querySelector('#jss-server-side');
         if (jssStyles && jssStyles.parentNode) {
           jssStyles.parentNode.removeChild(jssStyles)
         }
        }
        
    
3.  pass `theme` and `sheetsManager` to `<MuiThemeProvider>`, then wrap `<MuiThemeProvider>` around `<BaseComponent {...this.props} />`:
    
        <MuiThemeProvider
         theme={this.pageContext.theme}
         sheetsManager={this.pageContext.sheetsManager}
        >
         <CssBaseline />
         <div>
           <Header {...this.props} />
           <BaseComponent {...this.props} />
         </div>
        </MuiThemeProvider>
        
    

Add missing imports and the code snippets from above:  
`lib/withLayout.js` :

    import React from 'react';
    import PropTypes from 'prop-types';
    import { MuiThemeProvider } from 'material-ui/styles';
    import CssBaseline from 'material-ui/CssBaseline';
    
    import getContext from '../lib/context';
    import Header from '../components/Header';
    
    function withLayout(BaseComponent) {
      class App extends React.Component {
        constructor(props, context) {
          super(props, context);
          this.pageContext = this.props.pageContext || getContext();
        }
    
        componentDidMount() {
          const jssStyles = document.querySelector('#jss-server-side');
          if (jssStyles && jssStyles.parentNode) {
            jssStyles.parentNode.removeChild(jssStyles);
          }
        }
    
        render() {
          return (
            <MuiThemeProvider
              theme={this.pageContext.theme}
              sheetsManager={this.pageContext.sheetsManager}
            >
              <CssBaseline />
              <div>
                <Header {...this.props} />
                <BaseComponent {...this.props} />
              </div>
            </MuiThemeProvider>
          );
        }
      }
    
      App.propTypes = {
        pageContext: PropTypes.object, // eslint-disable-line
      };
    
      App.defaultProps = {
        pageContext: null,
      };
    
      App.getInitialProps = (ctx) => {
        if (BaseComponent.getInitialProps) {
          return BaseComponent.getInitialProps(ctx);
        }
    
        return {};
      };
    
      return App;
    }
    
    export default withLayout;
    

I've used `propTypes` and `defaultProps` on the `App` component. This usage is _optional_ but recommended. We discuss `propTypes` and `defaultProps` in more detail in [Chapter 2](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#index-page). If you choose to use them, remember to import:

import PropTypes from 'prop-types';

Start your app with `yarn dev` and navigate to `http://localhost:3000`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36134596-9e98a878-103a-11e8-832a-da70dc828b7d.png)

The button on the `Index` page, which originally had a flash of style, now loads _without_ any flash. We successfully integrated Material-UI with Next.js!

We should run two experiments to confirm our conclusion.

1.  On Chrome, go to `Developer tools > Elements` and click `Ctrl+F` to search for `jss-server-side`. You won't be able to find any element.
    
    Open `lib/withLayout.js` and comment out the following line:
    
    // jssStyles.parentNode.removeChild(jssStyles);
    
    Save the file, reload your tab, and go back to `Developer tools > Elements`. Click `Ctrl+F` to search for `jss-server-side`. Now, you _will see_ `<style id="jss-server-side">`, which contains server-side styles:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36134544-46c2af04-103a-11e8-8ee2-7aca6db5f04e.png)
    
    Remember to uncomment this line.
    
2.  Go to `lib/context.js` and add `console.log(global.INIT_MATERIAL_UI);` in this location:
    
        if (!global.INIT_MATERIAL_UI) {
         global.INIT_MATERIAL_UI = createPageContext();
         console.log(global.INIT_MATERIAL_UI);
        }
        
        console.log(global.INIT_MATERIAL_UI);
        
        return global.INIT_MATERIAL_UI;
        
    
    Save the file, reload your tab, and go to `Developer tools > Console`. You will see a context object that consists of `theme`, `sheetsManager`, `sheetsRegistry`, and `generateClassName`:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36134671-1d0f9798-103b-11e8-9510-27a5487460e0.png)
    
    If you comment out this line:
    
    // global.INIT\_MATERIAL\_UI = createPageContext();
    
    or this line:
    
    // return global.INIT\_MATERIAL\_UI;
    
    Then you'll get an error:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36134728-9c9488ac-103b-11e8-8eef-3b7568813340.png)
    
    _As expected_, since context is not created (first line) or not made available on the client (second line) - the error says that `theme` (`this.pageContext.theme`) is undefined.
    

Good job! You successfully finished Next.js/Material-UI integration.

## Server-side rendering
------------------------------------------------------

Let's look at a default behaviour of Next.js: server-side rendering (SSR) on _initial load_.

Unlike client-side rendering (CSR), SSR has a few advantages. Two specific advantages are SEO (search engine optimization) and UX (user experience):

*   server-rendered JavaScript is properly indexed by search engine crawling bots
    
*   server-side rendered pages have no loading delay, i.e. a user sees no empty page with a loading spinner - pages appear on the browser fully rendered
    

One more _potential_ advantage of SSR is faster loading speed for initial load. Compare server-side rendering with client-side:

*   SSR (server-side rendering). For initial page load with SSR, there is only _one_ over-network round trip between client (browser) and server. It takes the server time to render the page with data; however, the client gets both HTML and data for the requested page in _one round trip_.
    
*   CSR (client-side rendering). For initial page load with CSR, there are _two_ over-network round trips between client and server. After the first round trip, the client gets a page's HTML. After the second round trip, the client gets that page's data. Typically, after the second round trip, the client gets HTML for all other app pages. Thus, subsequent page loads (after initial load) are faster, since they do not require HTML but smaller JSON data.
    

Doing SSR on initial load and CSR on subsequent loads will be best for user experience. That's exactly what Next.js does: SSR on initial load and CSR for subsequent loads.

Here is an [informative blog post](https://medium.com/@adamzerner/client-side-rendering-vs-server-side-rendering-a32d2cf3bfcc) that compares SSR with more traditional CSR.

The best way to see that Next.js renders on the server for initial load is to disable SSR. Let's do an experiment and disable server-side rendering using [dynamic import](https://github.com/zeit/next.js/tree/5e7f990039b2e513fd09bc8078e17207a348a1ee#3-with-no-ssr).

Start your app with `yarn dev` and navigate to `http://localhost:3000`.

*   With SSR.
    
    1.  Refresh the browser tab a few times.  
        Notice that the `Header` component and `Index` page show up on the browser at the same time. There is _no empty page_ with a loading element.
        
    2.  Right click anywhere on the page and select `View page source`.  
        Find the `Log in` link (inside the `Header` component).  
        Notice that the `<a style="margin:0px 20px 0px auto" href="/login">Log in</a>` element is wrapped in the server-rendered `<div id="__next">...</div>` element.
        
*   Without SSR. Let's see what happens if we disable server-side rendering for the `Header` component.
    
    Open `lib/withLayout.js` and comment out this line:
    
    // import Header from '../components/Header';
    
    Then, import `dynamic`:
    
    import dynamic from 'next/dynamic';
    
    Finally, define `Header` as:
    
    const Header = dynamic(import('../components/Header'), { ssr: false });
    
    Save the file and reload your tab.
    
    1.  Refresh your browser tab. Notice how the `Header` component now arrives _after_ the `Index` page, with a slight but noticable delay. For a split second, you see that the `Index` page arrives empty (without the `Header` component). And while the page is empty, it shows a loading element `<p>loading...</p>`. After the `Index` page loads, `<p>loading...</p>` gets replaced by the `Header` component.
        
    2.  Right click on the page and select `View page source`. Find the server-rendered `<div id="__next">...</div>` element. Inside it, you won't find `<a style="margin:0px 20px 0px auto" href="/login">Log in</a>`. But you will find a `<p>loading...</p>` element instead of the entire `Header` component. The element `<p>loading...</p>` is server-rendered, and the `Header` component replaces the `<p>loading...</p>` element on the client.
        

Besides loading UX and SEO, SSR page may load faster than CSR page when network is slow, since in case of CSR there is one extra trip over network. We did a detailed comparison of SSR and CSR in our [tutorial at Hackernoon](https://hackernoon.com/server-side-vs-client-side-rendering-in-react-apps-443efd6f2e87). We compared performance metrics in addition to loading UX. Here is link to sample app that is used to compare SSR to CSR pages: [https://ssr-csr.builderbook.org](https://ssr-csr.builderbook.org).

In our app, we will use both SSR and CSR:

*   if we want SSR on initial load, we will make sure that the page gets data with the `getInitialProps()` method. This method executes on the server for _initial page load_ and then on the client if a user gets to the page with `<Link href="route">` or `Router.push('route')`.
    
*   if we want CSR only, we will call an API method inside the `componentDidMount()` lifecycle hook (see Chapter 5 and 6 for detailed examples) when data loads passively, without any extra action from user. If we want to call an API method _after a user takes an action_, such as clicking a button (see [Chapter 8, BuyButton](https://builderbook.org/books/builder-book/buybutton-component-buy-book-logic-mailchimp-api-readchapter-page-mybooks-page-deploy-app#buybutton-component)), we would place the API method inside an event-handling function (for example, `onButtonClick()`) _instead_ of `componentDidMount()`.
    

In this chapter, you learned integrated Next.js with Material-UI, and learned about server-side rendering. At this point, you have a basic template for static websites. In the next chapter (Chapter 2), we will create an Express server, connect our app to a database, and learn about `session` and `cookie`.

* * *

At the end of Chapter 1, your codebase should look like the codebase in `1-end`. The [1-end](https://github.com/builderbook/builderbook/tree/master/book/1-end) folder is located at the root of the `book` directory inside the [builderbook repo](https://github.com/builderbook/builderbook).

Compare your codebase and make edits if needed.

Enjoying the book so far? Please share a quick [review](https://goo.gl/forms/JdevtnCWsLwZTAio2). You can update your review at any time.

* * *
