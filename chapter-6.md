---
title: Github integration. Admin dashboard. Testing Admin UX and Github integration.
seoTitle: Integrate your web app with Github
seoDescription: This book teaches you how to build a production-ready web application from scratch using React, Material-UI, Next, Express, Mongoose, MongoDB. Chapter 6 shows you how to integrate your app with Github, create Express routes, and write client-side API methods.
isFree: true
---

* * *
*   Github integration  
    *   Set up server
    *   syncContent() for Book model
    *   syncContent() for Chapter model
*   Markdown to HTML  
*   Admin dashboard  
    *   Express routes
    *   API methods
    *   Admin pages and components
*   Update Header component  
*   Testing  
    *   Connecting Github
    *   Adding new book
    *   Editing existing book
    *   Syncing content
* * *

Before you start working on Chapter 6, get the `6-start` codebase. The [6-start](https://github.com/builderbook/builderbook/tree/master/book/6-start) folder is located at the root of the `book` directory inside the [builderbook repo](https://github.com/builderbook/builderbook).

*   If you haven't cloned the builderbook repo yet, clone it to your local machine with `git clone git@github.com:builderbook/builderbook.git`.
*   Inside the `6-start` folder, run `yarn` to install all packages.

These are the packages and their versions that we install specifically for Chapter 6:

*   `"body-parser": "^1.18.2"`
*   `"@octokit/rest": "14.0.5"`
*   `"qs": "^6.5.1"`
*   `"request": "^2.83.0"`

Remember to include your `.env` at the root of your app. By the end of Chapter 6, you will add `Github_Test_ClientID` and `Github_Test_SecretKey` environmental variables to your `.env` file.

* * *

In the previous chapter (Chapter 5), you built a complete internal API twice:

*   you rendered a list of books on the main Admin page (`pages/admin/index.js`) and
*   you rendered chapter content on the main Public page (`public/read-chapter.js`)

In this chapter, we will integrate our app with Github, add missing internal APIs for our Admin, and test out the _entire_ Admin experience in our web application. We will test adding a new book, editing it, and syncing its content with Github.

## Github integration
------------------------------------------------

This is the section where we finally integrate our app with Github. Let's quickly discuss why we chose Github as our content management system.

First, Github's markdown is familiar to most web developers, and we built our Builder Book app specifically for developers. Our app's Admin user is a web developer who can write, edit, and host chapter content using markdown on his/her favorite code editor or on Github. We prefer Visual Studio code editor (VS editor) for writing content. VS editor, unlike Github, has better scrolling, faster navigation, and lets you save your progress offline.

Second, Github comes with cloud storage for media files, such as images. Without Github, we would have to integrate with AWS S3 or another storage solution. In the final section of this chapter, I'll guide you through a Github integration that takes data from Github servers, saves it to our database, and fetches it inside our web app.

#### Set up server

To integrate our web app with Github, we have to achieve multiple things:

1.  When a user goes to `/auth/github`, we redirect the user to Github's authorize endpoint (see `AUTHORIZE_URI` below), where the user is asked to `Authorize application`.  
    We will follow the official API docs from Github. Check [this example](https://developer.github.com/v3/guides/basics-of-authentication/#accepting-user-authorization) in the basic authentication section. This official example provides the following URLs for `authorize` and `token` endpoints:
    
    https://github.com/login/oauth/authorize?scope=user:email&client\_id=<%= client\_id %>
    
    https://github.com/login/oauth/access_token
    
    Let's isolate the non-variable part (part without `scope`, `client_id`, etc.) of these URLs and point it to variables:
    
        const AUTHORIZE_URI = 'https://github.com/login/oauth/authorize';
        const TOKEN_URI = 'https://github.com/login/oauth/access_token';
        
    
    In step 1, we define an Express route: `server.get('/auth/github', (req, res) => { ... })`
    
    To get the complete `authorize` URL (with variables), we will stringify the non-variable part with variables by using the `qs.stringify()` [method](https://www.npmjs.com/package/qs#stringifying) from the `qs` package.
    
    In step 1, the `authorize` URL contains `client_id` and, in step 3, `request.post` requires `client_secret` \- so we have to define both before we use them:
    
        const dev = process.env.NODE_ENV !== 'production';
        
        const CLIENT_ID = dev ? process.env.Github_Test_ClientID : process.env.Github_Live_ClientID;
        const API_KEY = dev ? process.env.Github_Test_SecretKey : process.env.Github_Live_SecretKey;
        
    
    We will register our app on Github in the [Testing section](https://builderbook.org/books/builder-book/github-integration-admin-dashboard-testing-admin-ux-and-github-integration#testing) of this chapter.
    
2.  If the user gives permission, Github provides our app with a temporary authorization `code` value, and the user is redirected to `/auth/github/callback`.  
    Here, we define the Express route:
    
    server.get('/auth/github/callback', (req, res) => { ... })
    
3.  Our server sends a POST request with the authorization `code` to Github's server (at `TOKEN_URI`) and, in exchange, gets a result that contains either an `access_token` or error.  
    Since our Express server cannot send a request to Github's server (server to server request instead of server to client response), we use `request` from the `request` [package](https://www.npmjs.com/package/request) to send a POST request with `code` (to exchange it for `access_token`).
    
    Using `request` is straighforward, and we simply follow [this example](https://www.npmjs.com/package/request#forms):
    
    request.post({url:'value', form: {key:'value'}}, function(err, httpResponse, body){ /* ... */ })
    
    This POST request is sent (`request.post()`) from inside `server.get('/auth/github/callback', (req, res) => { ... })`, our Express route from step 2.
    
    Our Express routes from step 1 and step 2 will be combined in the `setupGithub({ server })` function. Later in this section, this function will be exported and imported to our main server code at `server/app.js` to initialize Github integration on the server.
    
4.  If the result has an `access_token`, then we update the user's document with:  
    `isGithubConnected: true, githubAccessToken: result.access_token`.  
    `result` comes back from Github in exchange for our POST request with an authorization `code`. If this `result` has an `access_token` \- we save it to the user's document. We'll use this `access_token` in step 5 when we need to access the user's data on Github, such as book content. And as you probaby guessed - we'll use `User.updateOne()` to update our user.
    
5.  We need to write a few API functions that return the user's repos, files inside these repos, and repo commits.  
    Here we define a `getAPI({ accessToken })` function that authenticates the user and sends a request to Github. We will _use_ this function inside:
    
    *   `getRepos({ accessToken })` (to get a list of repos),
    *   `getContent({ accessToken, repoName, path })` (to get content from repo's files) and
    *   `getCommits({ accessToken, repoName, limit })` (to get a list of commits).
    
    We will define `getAPI({ accessToken })` with the help of `GithubAPI` from the `github` package by closely following an [official example](https://www.npmjs.com/package/github#example). More on step 5 at the end of this subsection.
    

After putting code from steps 1-5 together, we get this code for setting up Github integration on our server:  
`server/github.js` :

    import qs from 'qs';
    import request from 'request';
    import GithubAPI from 'github';
    
    import User from './models/User';
    
    const AUTHORIZE_URI = 'https://github.com/login/oauth/authorize';
    const TOKEN_URI = 'https://github.com/login/oauth/access_token';
    
    export function setupGithub({ server }) {
      const dev = process.env.NODE_ENV !== 'production';
    
      const CLIENT_ID = dev ? process.env.Github_Test_ClientID : process.env.Github_Live_ClientID;
      const API_KEY = dev ? process.env.Github_Test_SecretKey : process.env.Github_Live_SecretKey;
    
      server.get('/auth/github', (req, res) => {
        // 1. check if user exists and user is Admin
        // If not, redirect to Login page, return undefined.
    
        // 2. Redirect to Github's OAuth endpoint (we will qs.stringify() here)
      });
    
      server.get('/auth/github/callback', (req, res) => {
        // 3. check if user exists and user is Admin
        // If not, redirect to Login page, return undefined.
        // (same as 1.) 
    
        // 4. return undefined if req.query has error
    
        const { code } = req.query;
    
        request.post(
          // 5. send request from our server to Github's server
    
          async (err, r, body) => {
          // 6. return undefined if result has error
    
    
            // 7. update User document on database
          },
        );
      });
    }
    
    function getAPI({ accessToken }) {
      const github = new GithubAPI({
        // 8. set parameters for new GithubAPI()
      });
    
      // 9. authenticate user by calling `github.authenticate()`
    
    }
    
    export function getRepos({ accessToken }) {
      // 10. function that gets list of repos for user
    }
    
    export function getContent({ accessToken, repoName, path }) {
      // 11. function that gets repo's content
    }
    
    export function getCommits({ accessToken, repoName, limit }) {
      // 12. function that gets list of repo's commits
    }
    

I've numbered the missing code snippets. We discuss them in detail below.

1.  Checking if a user exists and if the user is an Admin is straightforward:  
    
    if (!req.user || !req.user.isAdmin)
    
      
    If he user doesn't exist, let's redirect to the Login page: `res.redirect('/login');`. By now, you know how to return `undefined` with simple `return;`.  
    Put it all together:
    
        if (!req.user || !req.user.isAdmin) {
         res.redirect('/login');
         return;
        }
        
    
2.  Following [Github's official example](https://developer.github.com/v3/guides/basics-of-authentication/#accepting-user-authorization), we need to redirect the user (`res.redirect()`) to the `authorize` URL.
    
    However, before redirecting to this URL, we want to generate a full `authorize` URL by adding some parameters to the basic, non-variable part of the `authorize` URL (we called it `AUTHORIZE_URI`, see above).
    
    We create a full URL with `qs.stringify()`, which works like: `qs.stringify(object, [parameters]);`.  
    In our case:
    
    `${AUTHORIZE_URI}?${qs.stringify({
     // parameters we want to add to AUTHORIZE_URI
    })}`
    
    After adding `scope`, `state`, `client_id` parameters, we get:
    
    res.redirect(`${AUTHORIZE_URI}?${qs.stringify({
     scope: 'repo',
     state: req.session.state,
     client\_id: CLIENT\_ID,
    })}`);
    
3.  This code snippet is exactly the same as code snippet 1 (see above):
    
        if (!req.user || !req.user.isAdmin) {
         res.redirect('/login');
         return;
        }
        
    
4.  If the response from Github's server contains an error, we redirect the user and return undefined:
    
        if (req.query.error) {
         res.redirect(`/admin?error=${req.query.error_description}`);
         return;
        }
        
    
5.  _Else_, we send `request.post` by following `request`'s example:  
    `request.post({url:'value', form: {key:'value'}}, function(err, r, body){ /* ... */ })` (we renamed `httpResponse` to `response`).
    
    This POST request is sent to `TOKEN_URI` (see above) and contains three parameters: `client_id`, authorization `code` (taken from Github's initial response, `const { code } = req.query;`), and `client_secret`:
    
        {
         url: TOKEN_URI,
         headers: { Accept: 'application/json' },
         form: {
           client_id: CLIENT_ID,
           code,
           client_secret: API_KEY,
         },
        },
        
    
    The headers `{ Accept: 'application/json' }` tell Github's server to expect JSON-type data.
    
6.  If the response has an error, we will redirect the user and return undefined:
    
        if (err) {
         res.redirect(`/admin?error=${err.message || err.toString()}`);
         return;
        }
        
    
7.  _Else_, we will parse the response's `body` (which is a JSON string) with JavaScript's `JSON.parse()`. This will produce a JavaScript object. We will point the `result` variable to this JavaScript object. If the result has an error, we will redirect the user and return undefined:
    
        const result = JSON.parse(body);
        
        if (result.error) {
         res.redirect(`/admin?error=${result.error_description}`);
         return;
        }
        
    
8.  Here we follow an [example from the docs](https://github.com/octokit/rest.js#options). We specify some parameters for a `new GithubAPI()` instance.  
    `timeout` is the time for our server to acknowledge a request from Github. If the server does not respond, Github terminates the connection. The max timeout is 10 sec, and here we specify 10 seconds (10000 milliseconds).  
    `host` and `protocol` are self-explanatory.  
    `application/json` in `headers` informs Github's server that data is in JSON format.  
    `requestMedia` tells Github the data format our server wants to receive. Read [more](https://developer.github.com/v3/media).
    
    Pass the parameters above to `GithubAPI()`:
    
        const github = new GithubAPI({
         timeout: 10000,
         host: 'api.github.com', // should be api.github.com for GitHub
         protocol: 'https',
         headers: {
           accept: 'application/json',
         },
         requestMedia: 'application/json',
        });
        
    
9.  Again, we follow [an example](https://www.npmjs.com/package/github#authentication) from the docs:
    
        github.authenticate({
        type: 'oauth',
        token: process.env.AUTH_TOKEN,
        })
        
    
    Now we will use the `access_token` described above. Our server received this token from Github in exchange for the authorization `code` and saved the token to the user's document as `githubAccessToken` (see the Express route above for `/auth/github/callback`). `github.authenticate()` saves the type of authentication and token into our server's memory and uses them for subsequent API calls.
    
    For `accessToken` we get:
    
        github.authenticate({
         type: 'oauth',
         token: accessToken,
        });
        
    
10.  This method gets a list of the user's Github repos. The repo for the `github` package has a folder with examples. Check up [getRepos.js](https://github.com/octokit/node-github/blob/master/examples/getRepos.js):
    
        const GitHubApi = require('github')
        const github = new GitHubApi({
        debug: true,
        })
        
        github.authenticate({
        type: 'oauth',
        token: 'add-your-real-token-here',
        })
        
        github.repos.getAll({
        'affiliation': 'owner,organization_member',
        })
        
    
    We've already created a `new GitHubApi()` instance and called `github.authenticate()` inside `getAPI({ accessToken })`. The only thing left is to point `github` to `getAPI({ accessToken })` and call `github.repos.getAll()`:
    
        export function getRepos({ accessToken }) {
        const github = getAPI({ accessToken });
        
        return github.repos.getAll({ per_page: 100 });
        }
        
    
    We specified to show up to 100 repos per page by using the `per_page` parameter. `github.repos.getAll()` accepts seven parameters total. For a list of all parameters, search `repos.getAll()` in the [documentation](https://octokit.github.io/rest.js/#api-Search-repos).
    
11.  This method gets a repo's content by calling the `github.repos.getContent({ owner, repo, path })` API method. As you can see by searching `getContent` in [the docs](https://octokit.github.io/rest.js/#api-Search-repos), this method requires three parameters: `owner`, `repo` and `path`. The fourth parameter `ref` is optional.
    
    When we write the static method `syncContent()` for our Book and Chapter models, we will take `owner` and `repo` values from `repoName: book.githubRepo`. For example, if the `repoName` is `builderbook/book-1`, then `owner` is `builderbook` and `repo` is `book-1`. We reflect that by using ES6's destructuring and JavaScript's `split()` method:
    
    const \[owner, repo\] = repoName.split('/');
    
    Again, we point `github` to `getAPI({ accessToken })` and call `github.repos.getContent({ owner, repo, path })`:
    
        export function getContent({ accessToken, repoName, path }) {
        const github = getAPI({ accessToken });
        const [owner, repo] = repoName.split('/');
        
        return github.repos.getContent({ owner, repo, path });
        }
        
    
    Note, if the repo's root directory contains files with chapter content, then the `path` value is `'/'`.
    
12.  The `getCommits()` method is optional; however it's good practice to have. This method gets a list of repo commits. We take the latest commit and save it to our database. When we sync content between our database and the Github repo - we check if the latest commit id is the same. If it is, then the content in our database is up-to-date.  
    Check up the [list of parameters](https://octokit.github.io/rest.js/#api-Search-repos) for the `repos.getCommits()` method. Two required parameters are `owner` and `repo`. Again, we take the values of these parameters by splitting `repoName`:
    
    const \[owner, repo\] = repoName.split('/');
    
    And again, we point `github` to `getAPI({ accessToken })` and call `github.repos.getCommits({ owner, repo, per_page: limit })`:
    
        export function getCommits({ accessToken, repoName, limit }) {
        const github = getAPI({ accessToken });
        const [owner, repo] = repoName.split('/');
        
        return github.repos.getCommits({ owner, repo, per_page: limit });
        }
        
    
    We will specify `limit: 1` (in the static method for our Book model) to get only the one latest commit. Our static method will save the latest commit's hash to our database as `githubLastCommitSha` and compare it to Github's value every time the Admin user syncs content between the database and his/her Github repo.
    

Plug the twelve snippets of code above into our carcass for `server/github.js` :  
`server/github.js` :

    import qs from 'qs';
    import request from 'request';
    import GithubAPI from 'github';
    
    import User from './models/User';
    
    const TOKEN_URI = 'https://github.com/login/oauth/access_token';
    const AUTHORIZE_URI = 'https://github.com/login/oauth/authorize';
    
    export function setupGithub({ server }) {
      const dev = process.env.NODE_ENV !== 'production';
    
      const CLIENT_ID = dev ? process.env.Github_Test_ClientID : process.env.Github_Live_ClientID;
      const API_KEY = dev ? process.env.Github_Test_SecretKey : process.env.Github_Live_SecretKey;
    
      server.get('/auth/github', (req, res) => {
        if (!req.user || !req.user.isAdmin) {
          res.redirect('/login');
          return;
        }
    
        res.redirect(`${AUTHORIZE_URI}?${qs.stringify({
          scope: 'repo',
          state: req.session.state,
          client_id: CLIENT_ID,
        })}`);
      });
    
      server.get('/auth/github/callback', (req, res) => {
        if (!req.user || !req.user.isAdmin) {
          res.redirect('/login');
          return;
        }
    
        if (req.query.error) {
          res.redirect(`/admin?error=${req.query.error_description}`);
          return;
        }
    
        const { code } = req.query;
    
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
            if (err) {
              res.redirect(`/admin?error=${err.message || err.toString()}`);
              return;
            }
    
            const result = JSON.parse(body);
    
            if (result.error) {
              res.redirect(`/admin?error=${result.error_description}`);
              return;
            }
    
            try {
              await User.updateOne(
                { _id: req.user.id },
                { $set: { isGithubConnected: true, githubAccessToken: result.access_token } },
              );
              res.redirect('/admin');
            } catch (err2) {
              res.redirect(`/admin?error=${err2.message || err2.toString()}`);
            }
          },
        );
      });
    }
    
    function getAPI({ accessToken }) {
      const github = new GithubAPI({
        timeout: 10000,
        host: 'api.github.com', // should be api.github.com for GitHub
        protocol: 'https',
        headers: {
          accept: 'application/json',
        },
        requestMedia: 'application/json',
      });
    
      github.authenticate({
        type: 'oauth',
        token: accessToken,
      });
    
      return github;
    }
    
    export function getRepos({ accessToken }) {
      const github = getAPI({ accessToken });
    
      return github.repos.getAll({ per_page: 100 });
    }
    
    export function getContent({ accessToken, repoName, path }) {
      const github = getAPI({ accessToken });
      const [owner, repo] = repoName.split('/');
    
      return github.repos.getContent({ owner, repo, path });
    }
    
    export function getCommits({ accessToken, repoName, limit }) {
      const github = getAPI({ accessToken });
      const [owner, repo] = repoName.split('/');
    
      return github.repos.getCommits({ owner, repo, per_page: limit });
    }
    

The last three exported functions get user data such as repo, repo files, and repo commits from Github. The next step is to use these functions inside the static method `syncContent()` of our Book and Chapter models. As you may guess from the name, `Book.syncContent()` and `Chapter.syncContent()` static methods - with the help of `getRepos()`, `getContent()`, and `getCommits()` functions - will _get and sync data_ for our book and chapters from Github.

#### syncContent() for Book model

In the previous subsection, we wrote code for Github integration. We defined and exported a `setupGithub({ server })` function - this function has all necessary Express routes and will later be imported to our main server code at `server/app.js`. We defined and exported three API methods:

*   `getRepos({ accessToken })`,
*   `getContent({ accessToken, repoName, path })`,
*   `getCommits({ accessToken, repoName, limit })`.

But we are not done with Github integration yet. We have a few more tasks, which you should be familiar with:

*   update our Book and Chapter models with a static method that employs the three API methods above,
*   write an Express route with API endpoints (`server/api/admin.js`),
*   write API methods (`lib/api/admin.js`),
*   add missing pages to `pages/admin/*`.

Let's start with the first point. In this subsection, our goal is to _use_ our three API methods to define a static method `syncContent()` for our Book model. After an Admin user creates a book and decides to get content from Github, our app will execute `syncContent()` to get all necessary data and save that data to our database.

Take a look at our Book model at `server/models/Book.js`. We already defined a static method `add()` (`static async add({ name, price, githubRepo })`). Our Admin user sets a name + price and calls `add()` from `pages/admin/add-book.js` to create a new book. Our new static method `syncContent()` updates content for an _existing_ book. In other words, the Admin users calls `syncContent()` _after_ creating a book on his/her database.

`syncContent()` will be `async`. The method will find a book by its `id` and pass a user's `githubAccessToken` to Github's API methods defined earlier:  
`server/models/Book.js` :

    static async syncContent({ id, githubAccessToken }) {
      // 1. await find book by id
    
      // 2. throw error if there is no book
    
      // 3. get last commit from Github using `getCommits()` API method
    
      // 4. if there is no last commit on Github - no need to sync content, throw error
    
      // 5. if last commit's hash on Github's repo is the same as hash saved in database -
      // no need to extract content from repo, throw error
    
      // 6. define repo's main folder with `await` and `getContent()`
    
      await Promise.all(mainFolder.data.map(async (f) => {
        // 7. check if main folder has files, check title of files
        // 8. define `chapter` with `await` and `getContent()`
        // 9. Extract content from each qualifying file in repo
        // 10. For each file, run `Chapter.syncContent({ book, data })`
      }
    
      // 11. Return book with updated `githubLastCommitSha`
    }
    

The section above is a high-level structure (carcass) of the `syncContent()` static method. As always, before we write code, let's discuss the purpose of each code snippet.

1.  `syncContent()` is an async function. Inside it, we will find a book with Mongoose's `findById()` [method](http://mongoosejs.com/docs/api.html#model_Model.findById): `Model.findById(id, [projection])`.  
    The optional array `[projection]` is an array of parameter values that we want to return from a Model.  
    In this case, we want to return two book parameters: `githubRepo` and `githubLastCommitSha`:
    
    const book = await this.findById(id, 'githubRepo githubLastCommitSha');
    
2.  We did this one many times before. If there is no book (`if (!book)`), throw an error (`throw new Error('some informative text')`):
    
        if (!book) {
         throw new Error('Book not found');
        }
        
    
3.  Here we `await` for the `getCommits()` API method to get our repo's latest commit from Github. Remember this method takes three parameters `getCommits({ accessToken, repoName, limit })`. `accessToken` to authenticate the user, `repoName` to get and pass `owner` and `repo`, `limit` to limit the number of commits returned.
    
        const lastCommit = await getCommits({
         accessToken: githubAccessToken,
         repoName: book.githubRepo,
         limit: 1,
        });
        
    
    As discussed in the previous subsection, `getCommits()` returns a list of commits in reverse chronological order - that's why `limit: 1` ensures that we get the most recent commit.
4.  Here we are being overcautious and throw an error in the following three cases:
    *   if there is no list of commits for the repo (`if (!lastCommit)`) \_or\_
    *   if there are no elements inside the list of commits (`!lastCommit.data`) \_or\_
    *   if there is no first element in the list of commits (first element has index 0, `!lastCommit.data[0]`)
    *   then we won't extract any data from the repo; instead we will throw an error (`throw new Error('some informative text')`):
        
            if (!lastCommit || !lastCommit.data || !lastCommit.data[0]) {
            throw new Error('No change in content!');
            }
            
        
5.  First, we define `lastCommitSha` as `lastCommit.data[0].sha`. From code snippet 4, you know that `lastCommit.data[0]` is simply the first element in the list of commits - i.e. the last commit, since the list is ordered in reverse chronology.
    
    If the hash of the last commit in the Github repo `lastCommitSha` is the same as hash saved to the database `book.githubLastCommitSha`, then all content in the database is up-to-date. No need to extract data, so we throw an error:
    
         const lastCommitSha = lastCommit.data[0].sha;
         if (lastCommitSha === book.githubLastCommitSha) {
           throw new Error('No change in content!');
         }
        
    
6.  The main folder in a Github repo has `path: ''`. Let's defile the `mainFolder` using the `getContent()` API method. This method takes three parameters, `getContent({ accessToken, repoName, path })`:
    
         const mainFolder = await getContent({
           accessToken: githubAccessToken,
           repoName: book.githubRepo,
           path: '',
         });
        
    
7.  In our carcass, you may have noticed this construct:
    
        await Promise.all(mainFolder.data.map(async (f) => {
         // some code
        }
        
    
    As you already know, `await` pauses code until [Promise.all(iterable)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) returns a single resolved promise after _all_ promises inside [iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) have been resolved.
    
    In our case, iterable is [.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map). This JavaScript method iterates through all `.md` files with proper names inside mainFolder:
    
        if (f.type !== 'file') {
         return;
        }
        
        if (f.path !== 'introduction.md' && !/chapter-([0-9]+)\.md/.test(f.path)) {
         return;
        }
        
    
    First, our construct checks if the content inside `mainFolder.data` is a file. If not, the code returns undefined. Second, our construct checks if a file's path is `introduction.md` or `chapter-d.md`. If not, the code returns undefined. In this second construct, JavaScript's `.test(f.path)` tests if `f.path` equals `/chapter-([0-9]+)\.md/` and returns false if not. Read more about [.test()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test).
    
8.  Here we define `chapter` using the `getContent()` API method. You remember that this method takes three parameters `getContent({ accessToken, repoName, path })`. The method passes `accessToken` to `github.authenticate()`, splits `repoName` to extract `repo` and `owner`, and uses `path` to specify a repo file to get content from.
    
        const chapter = await getContent({
         accessToken: githubAccessToken,
         repoName: book.githubRepo,
         path: f.path,
        });
        
    
9.  After we define `chapter`, we need to extract content from the `.md` file. We use the [front-matter](https://www.npmjs.com/package/front-matter) package to extract data. Using `front-matter` is straightforward, check up an [official example](https://github.com/jxson/front-matter#example): `frontmatter(string)`. Below, we use this method to extract `data` from the `utf8` string:
    
    const data = frontmatter(Buffer.from(chapter.data.content, 'base64').toString('utf8'));
    
    You might get confused by the argument inside `frontmatter()`:
    
    Buffer.from(chapter.data.content, 'base64').toString('utf8')
    
    [Buffer](https://nodejs.org/api/buffer.html#buffer_buffer) is a class in Node designed for handling raw _binary_ data. Github API methods return _base64_ encoded content (see [docs](https://developer.github.com/v3/repos/contents/)). Thus, we use Buffer to handle base64-encoded `chapter.data.content` content from Github.
    
    We handle binary data from Github by using `Buffer.from(string[, encoding])`. This method creates a new Buffer that contains a copy of the provided `string` (see [Node docs](https://nodejs.org/api/buffer.html#buffer_buffer_from_buffer_alloc_and_buffer_allocunsafe)):
    
    Buffer.from(chapter.data.content, 'base64')
    
    Then we use the `.toString([encoding])` method to convert binary data to a `utf-8` string (see [Node docs](https://nodejs.org/api/buffer.html#buffer_buf_tostring_encoding_start_end)):
    
    .toString('utf8')
    
    Though not important for building this app, you are welcome to read more about [base64](https://en.wikipedia.org/wiki/Base64#Examples) and [utf-8](https://en.wikipedia.org/wiki/UTF-8#Examples).
    
10.  Here we pass `data` from code snippet 9 to the `syncContent()` static method inside our _Chapter model_: `Chapter.syncContent({ book, data })`. We pass `book` data as well. As you may guess, this particular `syncContent()` _creates_ a chapter document in the Chapter collection. This chapter document contains the proper `bookId` (from `book` data) and proper `content` (from `data`). Example of code that creates a chapter document:
    
        return this.create({
        bookId: book._id,
        githubFilePath: path,
        content: body,
        // more parameters
        });
    
    You see that the `githubFilePath` parameter is simply `path`, so we have to pass `path` to `data` with:
    
    data.path = f.path
    
    As always, let's use the `try/catch` construct:
    
        data.path = f.path;
        
        try {
        await Chapter.syncContent({ book, data });
        logger.info('Content is synced', { path: f.path });
        } catch (error) {
        logger.error('Content sync has error', { path: f.path, error });
        }
        
    
11.  We want `syncContent()` in our Book model to return a book with an _updated_ `githubLastCommitSha` parameter (this is the hash of the repo's latest commit from Github):
    
    return book.update({ githubLastCommitSha: lastCommitSha });
    

Good job, now the easy part - plug in these 11 code snippets into the `syncContent()` carcass for for our Book model:

    static async syncContent({ id, githubAccessToken }) {
      const book = await this.findById(id, 'githubRepo githubLastCommitSha');
    
      if (!book) {
        throw new Error('Not found');
      }
    
      const lastCommit = await getCommits({
        accessToken: githubAccessToken,
        repoName: book.githubRepo,
        limit: 1,
      });
    
      if (!lastCommit || !lastCommit.data || !lastCommit.data[0]) {
        throw new Error('No change!');
      }
    
      const lastCommitSha = lastCommit.data[0].sha;
      if (lastCommitSha === book.githubLastCommitSha) {
        throw new Error('No change!');
      }
    
      const mainFolder = await getContent({
        accessToken: githubAccessToken,
        repoName: book.githubRepo,
        path: '',
      });
    
      await Promise.all(mainFolder.data.map(async (f) => {
        if (f.type !== 'file') {
          return;
        }
    
        if (f.path !== 'introduction.md' && !/chapter-(\[0-9]+)\.md/.test(f.path)) {
        // not chapter content, skip
          return;
        }
    
        const chapter = await getContent({
          accessToken: githubAccessToken,
          repoName: book.githubRepo,
          path: f.path,
        });
    
        const data = frontmatter(Buffer.from(chapter.data.content, 'base64').toString('utf8'));
    
        data.path = f.path;
    
        try {
          await Chapter.syncContent({ book, data });
          logger.info('Content is synced', { path: f.path });
        } catch (error) {
          logger.error('Content sync has error', { path: f.path, error });
        }
      }));
    
      return book.update({ githubLastCommitSha: lastCommitSha });
    }
    

_Important_ \- remember to add this static method above to our Book model at `server/models/Book.js`. Add it after the `static async edit()` static method.

Make sure that you have all necessary imports for the `server/models/Book.js` file:

    import mongoose, { Schema } from 'mongoose';
    import frontmatter from 'front-matter';
    
    import generateSlug from '../utils/slugify';
    import Chapter from './Chapter';
    
    import { getCommits, getContent } from '../github';
    import logger from '../logs';
    

#### syncContent() for Chapter model

We passed `book` and `data` to our Chapter model with `Chapter.syncContent({ book, data });`. This method will create a chapter document in the Chapter collection \_or\_ if the document already exists, that document will be updated.

Before we continue, we need to understand the structure of data returned by the [front-matter](https://github.com/jxson/front-matter) package. For a Github `.md` file that looks like:  
![Builder Book](https://user-images.githubusercontent.com/10218864/34592422-ba17d474-f178-11e7-87af-916e8d9e9dfd.png)

`frontmatter()` method returns:  
![Builder Book](https://user-images.githubusercontent.com/10218864/34592474-29a2a3be-f179-11e7-8562-99239ba9425b.png)

Now that we know the structure, we use ES6 object destructuring for `data.attributes.title`, `data.attributes.excerpt`, `data.attributes.isFree`, `data.attributes.seoTitle`, `data.attributes.seoDescription`, `data.body`, and `data.path`:

    const {
      title,
      excerpt = '',
      isFree = false,
      seoTitle = '',
      seoDescription = '',
    } = data.attributes;
    
    const { body, path } = data;
    

Remember that we defined `data.path = f.path` in `syncContent()` of our Book model.

Next, let's assume the chapter document exists. In this case, we attempt to find it with Mongoose's `findOne()`. We search using two parameters: `bookId` and `githubFilePath`:

    const chapter = await this.findOne({
      bookId: book.id,
      githubFilePath: path,
    });
    

Remember that we passed the `book` object with `syncContent({ book, data })` and `bookId: book.id`. We defined `path` with `const { body, path } = data;` and passed it with `data.path = f.path`.

We also need a parameter to specify the order in which a chapter is displayed inside the Table of Contents. For example, we want a chapter with content from `introduction.md` to have `order = 1` and a chapter with content from `chapter-1.md` to have `order = 2`:

    let order;
    
    if (path === 'introduction.md') {
      order = 1;
    } else {
      order = parseInt(path.match(/[0-9]+/), 10) + 1;
    }    
    

We would like to find a number inside each chapter's path. For example, for path `chapter-3.md`, we want to return `order = 4` (introduction chapter with path `introduction.md` has `order = 1`). To do so, we use JavaScript's methods [str.match(regexp)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) and [parseInt(string, radix)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt).

The first JavaScript method finds `regexp` inside `str`. In our case, `regexp` or [regular expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions) is a digit, and `str` is a `path`. Regular expression for _digit_ is `[0-9]` or `/d`. In order to find multiple digits inside a string, we add `+`. Without `+`, we will not get `order = 14` for the path `chapter-13.md`, since only `1` will be found instead of `13`.

The second JavaScript method parses the resulting string and returns the integer that it finds. Radix is `10`, since we want to return a decimal system integer. If we don't use `parseInt()`, then instead of adding `1` to the number, we will join `1` to the string and return a joint string. For example, without `parseInt()`, the `order` for `path` `chapter-3.md` will be `31` instead of `4`. Moreover, the `order` will be a string, not a number.

Whenever possible, when using JavaScript methods, we like to test out code on our browser console. Go to Chrome's `Developer tools`, click `Console`, and paste the following code:

    path = 'chapter-3.md';
    order = parseInt(path.match(/[0-9]+/), 10) + 1;
    console.log(typeof(order), order)
    

Run the code by clicking `Enter`. As expected, the output is `number 4`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/35013813-6481cf98-fac3-11e7-8970-9811a3b1fa68.png)

Try removing `+` from `[0-9]+` and replacing `chapter-3.md` with `chapter-13.md`. Run the code. The `order` will be `2` instead of `14`:  
![Builder Book](https://user-images.githubusercontent.com/10218864/35013875-a7bc61a6-fac3-11e7-86d1-54c596ba1172.png)

Add `+` back and replace `chapter-13.md` with `chapter-3.md`. Remove the `parseInt()` function and run the code. The output is `string 31` instead of `number 4`, but this is hardly a surprise to us:  
![Builder Book](https://user-images.githubusercontent.com/10218864/35014056-3d72b6fa-fac4-11e7-8709-d66f6513ec40.png)

Let's put together everything we discussed about the `syncContent()` method for our Chapter model:  
`server/models/Chapter.js` :

    static async syncContent({ book, data }) {
      const {
        title,
        excerpt = '',
        isFree = false,
        seoTitle = '',
        seoDescription = '',
      } = data.attributes;
    
      const { body, path } = data;
    
      const chapter = await this.findOne({
        bookId: book.id,
        githubFilePath: path,
      });
    
      let order;
    
      if (path === 'introduction.md') {
        order = 1;
      } else {
        order = parseInt(path.match(/[0-9]+/), 10) + 1;
      }
    
      // 1. if chapter document does not exist - create slug and create document with all parameters
    
      // 2. else, define modifier for parameters: content, htmlContent, sections, excerpt, htmlExcerpt, isFree, order, seoTitle, seoDescription
    
      // 3. update existing document with modifier
    }
    

Let's discuss the missing code snippets.

1.  To create a new chapter document, we use Mongoose's `Model.create()` [method](http://mongoosejs.com/docs/api.html#model_Model.create). Check up how we did it for the User model at `server/models/User.js`. Before we call this method, we have to call and `await` for `generateSlug(Model, title)` to generate the chapter's `slug` from its `title`:
    
        if (!chapter) {
         const slug = await generateSlug(this, title, { bookId: book._id });
        
         return this.create({
           bookId: book._id,
           githubFilePath: path,
           title,
           slug,
           isFree,
           content,
           htmlContent,
           sections,
           excerpt,
           htmlExcerpt,
           order,
           seoTitle,
           seoDescription,
           createdAt: new Date(),
         });
        }
        
    
    Take a look at `server/utils/slugify.js` if you need to remember how the `generateSlug(Model, name, filter = {})` function works.
    
2.  When a chapter document already exists and our Admin user calls `syncContent()` on the Chapter model - we want to update (as in, overwrite) the chapter's parameters. Let's define a `modifier` object as:
    
        const modifier = {
         content,
         htmlContent,
         sections,
         excerpt,
         htmlExcerpt,
         isFree,
         order,
         seoTitle,
         seoDescription,
        };
        
    
    In case the book's `title` is changed, we should re-generate `slug` and extend our `modifier` object with `title` and `slug` parameters:
    
        if (title !== chapter.title) {
         modifier.title = title;
         modifier.slug = await generateSlug(this, title, {
           bookId: chapter.bookId,
         });
        }
        
    
3.  Mongoose's method [Model.updateOne()](http://mongoosejs.com/docs/api.html#model_Model.updateOne) updates a single chapter document that has a matching `_id`:
    
    return this.updateOne({ \_id: chapter.\_id }, { $set: modifier });
    
    As you know from writing the User model, the [$set](https://docs.mongodb.com/manual/reference/operator/update/set/index.html) operator replaces a parameter's value with a specified value.
    

Paste the three code snippets above, and we get `syncContent()` static method:

    static async syncContent({ book, data }) {
      const {
        title,
        excerpt = '',
        isFree = false,
        seoTitle = '',
        seoDescription = '',
      } = data.attributes;
    
      const { body, path } = data;
    
      const chapter = await this.findOne({
        bookId: book.id,
        githubFilePath: path,
      });
    
      let order;
    
      if (path === 'introduction.md') {
        order = 1;
      } else {
        order = parseInt(path.match(/[0-9]+/), 10) + 1;
      }
    
      if (!chapter) {
        const slug = await generateSlug(this, title, { bookId: book._id });
    
        return this.create({
          bookId: book._id,
          githubFilePath: path,
          title,
          slug,
          isFree,
          content,
          htmlContent,
          sections,
          excerpt,
          htmlExcerpt,
          order,
          seoTitle,
          seoDescription,
          createdAt: new Date(),
        });
      }
    
      const modifier = {
        content,
        htmlContent,
        sections,
        excerpt,
        htmlExcerpt
        isFree,
        order,
        seoTitle,
        seoDescription,
      };
    
      if (title !== chapter.title) {
        modifier.title = title;
        modifier.slug = await generateSlug(this, title, {
          bookId: chapter.bookId,
        });
      }
    
      return this.updateOne({ _id: chapter._id }, { $set: modifier });
    }
    

_Important_ \- remember to add this static method above to our Chapter model at `server/models/Chapter.js`. Add it after the `static async getBySlug()` static method.

Make sure you have all required imports for the `server/models/Chapter.js` file:

    import mongoose, { Schema } from 'mongoose';
    
    import generateSlug from '../utils/slugify';
    import Book from './Book';
    

In the code for the static method `syncContent()` of our Chapter model, we _have not_ defined markdown content `content`, HTML content `htmlContent`, and `sections`. Let's discuss these parameters in the next section.

## Markdown to HTML
--------------------------------------------

In the previous section, we defined:

const { body, path } = data;

`body` (defined as `data.body`) is the markdown content of `.md` file or markdown content of a chapter (`chapter.content`). In other words:

const content = body

Once we have `chapter.content`, we save it to our database with the `syncContent()` static method of our Chapter model. Markdown `content` is nice, and you probably like using [Github markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet). We use markdown `content` to compare content on our database with content on Github, then decide whether we should update the content on our database or not. However, we cannot render markdown `content` directly on the browser.

To render on the browser, we need to convert markdown `content` to HTML `htmlContent`. The [marked](https://www.npmjs.com/package/marked) package is a markdown parser and does exactly that. In other words, when we write `**some text**`, `marked` can convert it into `<b>some text</b>`, and a user will see **some text** on his/her browser.

Marked is straigforward to use. In our case, we will parse chapter content with:  
`marked(content)`

We can configure `marked` and modify rules that specify how `marked` renders some elements of markdown. You can [configure](https://www.npmjs.com/package/marked#overriding-renderer-methods) `marked` renderer with `new marked.Renderer()`. For example, we would like every external link in our app to have these attributes:

rel="noopener noreferrer" target="_blank"

Customizing `renderer` looks like:

    const renderer = new marked.Renderer();
    
    renderer.link = (href, title, text) => {
      const t = title ? ` title="${title}"` : '';
      return `<a target="_blank" href="${href}" rel="noopener noreferrer"${t}>${text}</a>`;
    };
    
    marked.setOptions({
      renderer,
    });
    

The `marked` package does not come with default highlighting of code. To highlight contents in the `<code>` tag, `marked` offers [multiple options](https://www.npmjs.com/package/marked#highlight). We will use the _synchronous_ example that uses the [`highlight.js` package](https://www.npmjs.com/package/highlight.js). This package works with any markup and detects language automatically.

After importing `hljs` from the `highlight.js` package, we set our `marked` options with `marked.setOptions()` (see [usage docs](https://www.npmjs.com/package/marked#usage)):

    marked.setOptions({
      renderer,
      breaks: true,
      highlight(code, lang) {
        if (!lang) {
          return hljs.highlightAuto(code).value;
        }
    
        return hljs.highlight(lang, code).value;
      },
    });
    

If a language (`lang`) is specified, we pass it to `hljs.highlightAuto()`. If not specified, we rely on automatic detection.

We also specified `breaks: true`, so `marked` recognizes and adds line breaks.

In [Chapter 5, section Testing](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#testing), we tested rendering of `htmlContent` on our `ReadChapter` page. We briefly discussed HTML elements with class names that start with `hljs`. Now you know where these class names come from - `marked` adds classes to text inside `<pre>` and `<code>` tags to highlight that text. In our case, `marked` recognizes code to be JavaScript and adds class name and highlights accordingly.

Besides customizing `link`, we would like to customize images with:

    renderer.image = href => `<img
      src="${href}"
      style="border: 1px solid #ddd;"
      width="100%"
      alt="Builder Book"
    >`;
    

We want all images to fit inside the page (`width="100%"`) and have a border around them (`style="border: 1px solid #ddd;"`).

Finally, we want to customize the conversion of headings, in particular `##` (`<h2>`) and `####` (`<h4>`):

    renderer.heading = (text, level) => {
      const escapedText = text
        .trim()
        .toLowerCase()
        .replace(/[^\w]+/g, '-');
    
      if (level === 2) {
        return `<h${level} class="chapter-section" style="color: #222; font-weight: 400;">
          <a
            class="section-anchor"
            name="${escapedText}"
            href="#${escapedText}"
            style="color: #222;"
          > 
            <i class="material-icons" style="vertical-align: middle; opacity: 0.5; cursor: pointer;">link</i>
          </a>
          ${text}
        </h${level}>`;
      }
    
      if (level === 4) {
        return `<h${level} style="color: #222;">
          <a
            name="${escapedText}"
            href="#${escapedText}"
            style="color: #222;"
          >
            <i class="material-icons" style="vertical-align: middle; opacity: 0.5; cursor: pointer;">link</i>
          </a>
          ${text}
        </h${level}>`;
      }
    
      return `<h${level} style="color: #222; font-weight: 400;">${text}</h${level}>`;
    };
    

Notice that we added a hyperlinked Material icon in front of the heading's text:

##

This icon loads from Google CDN, which you may recall from Chapter 1 when [customizing `<Document>`](https://builderbook.org/books/builder-book/app-structure-next-js-hoc-material-ui-server-side-rendering-styles#document). Open `pages/_document.js` \- we added the following `<link>` tag to the `<Head>` section of our custom document:

    <link
      rel="stylesheet"
      href="https://fonts.googleapis.com/icon?family=Material+Icons"
    />
    

We want this icon in front of the heading text to have a unique link, so users can share links to particular sections or subsections (`href="#${escapedText}")`) of a chapter. We also want the page to scroll to an anchor (`name="${escapedText}"`) when a user clicks the hyperlinked icon.

We've applied `class="chapter-section"` to our `<h2>` heading. We will use this class in Chapter 7 to detect an in-view section and highlight the corresponding section inside our `Table of Content`.

Converting markdown with `marked` works well with the exception of [HTML entities](https://developer.mozilla.org/en-US/docs/Glossary/Entity). For example, the _entity_ `"` stands for _character_ `"`. Github _encodes_ characters into entities. And we have to _decode_ entities _back_ into characters before we show content to the users in our web app. We will use the `he` package to achieve that. This package provides us with both `he.encode()` and `he.decode()` methods. We need to use the latter.

To convert markdown to HTML, we will use:

marked(he.decode(chapter.content))

instead of:

marked(content);

We've written a lot of code related to customization of `marked`. Let's put it all together inside a `markdownToHtml()` function. This function will take markdown content (`markdownToHtml(content)`) as an argument and output HTML content (`return marked(he.decode(content))`):

    function markdownToHtml(content) {
      const renderer = new marked.Renderer();
    
      renderer.link = (href, title, text) => {
        const t = title ? ` title="${title}"` : '';
        return `<a target="_blank" href="${href}" rel="noopener noreferrer"${t}>${text}</a>`;
      };
    
      renderer.image = href => `<img
        src="${href}"
        style="border: 1px solid #ddd;"
        width="100%"
        alt="Builder Book"
      >`;
    
      renderer.heading = (text, level) => {
        const escapedText = text
          .trim()
          .toLowerCase()
          .replace(/[^\w]+/g, '-');
    
        if (level === 2) {
          return `<h${level} class="chapter-section" style="color: #222; font-weight: 400;">
            <a
              name="${escapedText}"
              href="#${escapedText}"
              style="color: #222;"
            > 
              <i class="material-icons" style="vertical-align: middle; opacity: 0.5; cursor: pointer;">link</i>
            </a>
            <span class="section-anchor" name="${escapedText}">
              ${text}
            </span>
          </h${level}>`;
        }
    
        if (level === 4) {
          return `<h${level} style="color: #222;">
            <a
              name="${escapedText}"
              href="#${escapedText}"
              style="color: #222;"
            >
              <i class="material-icons" style="vertical-align: middle; opacity: 0.5; cursor: pointer;">link</i>
            </a>
            ${text}
          </h${level}>`;
        }
    
        return `<h${level} style="color: #222; font-weight: 400;">${text}</h${level}>`;
      };
    
      marked.setOptions({
        renderer,
        breaks: true,
        highlight(code, lang) {
          if (!lang) {
            return hljs.highlightAuto(code).value;
          }
    
          return hljs.highlight(lang, code).value;
        },
      });
    
      return marked(he.decode(content));
    }
    

To convert content from markdown to HTML:

const htmlContent = markdownToHtml(content)

To convert an excerpt from markdown to HTML:

const htmlExcerpt = markdownToHtml(excerpt)

Similar to `markdownToHtml(content)`, let's define a `getSections(content)` function. This function takes markdown content and outputs a `sections` array. This array contains sections for our Table of Contents - every `<h2>` tag inside the content becomes a section inside the Table of Contents:

    function getSections(content) {
      const renderer = new marked.Renderer();
    
      const sections = [];
    
      renderer.heading = (text, level) => {
        if (level !== 2) {
          return;
        }
    
        const escapedText = text
          .trim()
          .toLowerCase()
          .replace(/[^\w]+/g, '-');
    
        sections.push({ text, level, escapedText });
      };
    
      marked.setOptions({
        renderer,
      });
    
      marked(he.decode(content));
    
      return sections;
    }
    

We hyperlink sections on our Table of Contents using `escapedText`, but more on this in Chapter 7.

To make a `sections` array:

const sections = getSections(content)

At this point, we can update our Chapter model (`server/models/Chapter.js`):

1.  Define `markdownToHtml(content)` and `getSections(content)` functions before `const mongoSchema = new Schema()`
2.  Add the following snippet to the `syncContent()` static method:
    
        const content = body;
        const htmlContent = markdownToHtml(content);
        const htmlExcerpt = markdownToHtml(excerpt);
        const sections = getSections(content);
        
    
    right after:
    
        if (path === 'introduction.md') {
         order = 1;
        } else {
         order = parseInt(path.match(/[0-9]+/), 10) + 1;
        }
        
    
3.  Remember to import missing packages `marked`, `he` and `hljs`.

Follow steps 1-3 to update our Chapter model, and you should get:  
`server/models/Chapter.js` :

    import mongoose, { Schema } from 'mongoose';
    import marked from 'marked';
    import he from 'he';
    import hljs from 'highlight.js';
    import generateSlug from '../utils/slugify';
    import Book from './Book';
    
    function markdownToHtml(content) {
      const renderer = new marked.Renderer();
    
      renderer.link = (href, title, text) => {
        const t = title ? ` title="${title}"` : '';
        return `<a target="_blank" href="${href}" rel="noopener noreferrer"${t}>${text}</a>`;
      };
    
      renderer.image = href => `<img
        src="${href}"
        style="border: 1px solid #ddd;"
        width="100%"
        alt="Builder Book"
      >`;
    
      renderer.heading = (text, level) => {
        const escapedText = text
          .trim()
          .toLowerCase()
          .replace(/[^\w]+/g, '-');
    
        if (level === 2) {
          return `<h${level} class="chapter-section" style="color: #222; font-weight: 400;">
            <a
              name="${escapedText}"
              href="#${escapedText}"
              style="color: #222;"
            > 
              <i class="material-icons" style="vertical-align: middle; opacity: 0.5; cursor: pointer;">link</i>
            </a>
            <span class="section-anchor" name="${escapedText}">
              ${text}
            </span>
          </h${level}>`;
        }
    
        if (level === 4) {
          return `<h${level} style="color: #222;">
            <a
              name="${escapedText}"
              href="#${escapedText}"
              style="color: #222;"
            >
              <i class="material-icons" style="vertical-align: middle; opacity: 0.5; cursor: pointer;">link</i>
            </a>
            ${text}
          </h${level}>`;
        }
    
        return `<h${level} style="color: #222; font-weight: 400;">${text}</h${level}>`;
      };
    
      marked.setOptions({
        renderer,
        breaks: true,
        highlight(code, lang) {
          if (!lang) {
            return hljs.highlightAuto(code).value;
          }
    
          return hljs.highlight(lang, code).value;
        },
      });
    
      return marked(he.decode(content));
    }
    
    function getSections(content) {
      const renderer = new marked.Renderer();
    
      const sections = [];
      renderer.heading = (text, level) => {
        if (level !== 2) {
          return;
        }
    
        const escapedText = text
          .trim()
          .toLowerCase()
          .replace(/[^\w]+/g, '-');
    
        sections.push({ text, level, escapedText });
      };
    
      marked.setOptions({
        renderer,
      });
    
      marked(he.decode(content));
    
      return sections;
    }
    
    const mongoSchema = new Schema({
      bookId: {
        type: Schema.Types.ObjectId,
        required: true,
      },
      isFree: {
        type: Boolean,
        required: true,
        default: false,
      },
      githubFilePath: {
        type: String,
      },
      title: {
        type: String,
        required: true,
      },
      slug: {
        type: String,
        required: true,
      },
      excerpt: {
        type: String,
        default: '',
      },
      content: {
        type: String,
        default: '',
        required: true,
      },
      htmlContent: {
        type: String,
        default: '',
        required: true,
      },
      createdAt: {
        type: Date,
        required: true,
      },
      order: {
        type: Number,
        required: true,
      },
      seoTitle: String,
      seoDescription: String,
    });
    
    class ChapterClass {
      static async getBySlug({ bookSlug, chapterSlug, userId }) {
        const book = await Book.getBySlug({ slug: bookSlug, userId });
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
    
      static async syncContent({ book, data }) {
        const {
          title,
          excerpt = '',
          isFree = false,
          seoTitle = '',
          seoDescription = '',
        } = data.attributes;
    
        const { body, path } = data;
    
        const chapter = await this.findOne({
          bookId: book.id,
          githubFilePath: path,
        });
    
        let order;
    
        if (path === 'introduction.md') {
          order = 1;
        } else {
          order = parseInt(path.match(/[0-9]+/), 10) + 1;
        }
    
        const content = body;
        const htmlContent = markdownToHtml(content);
        const htmlExcerpt = markdownToHtml(excerpt);
        const sections = getSections(content);
    
        if (!chapter) {
          const slug = await generateSlug(this, title, { bookId: book._id });
    
          return this.create({
            bookId: book._id,
            githubFilePath: path,
            title,
            slug,
            isFree,
            content,
            htmlContent,
            sections,
            excerpt,
            htmlExcerpt,
            order,
            seoTitle,
            seoDescription,
            createdAt: new Date(),
          });
        }
    
        const modifier = {
          content,
          htmlContent,
          sections,
          excerpt,
          htmlExcerpt
          isFree,
          order,
          seoTitle,
          seoDescription,
        };
    
        if (title !== chapter.title) {
          modifier.title = title;
          modifier.slug = await generateSlug(this, title, {
            bookId: chapter.bookId,
          });
        }
    
        return this.updateOne({ _id: chapter._id }, { $set: modifier });
      }
    }
    
    mongoSchema.index({ bookId: 1, slug: 1 }, { unique: true });
    mongoSchema.index({ bookId: 1, githubFilePath: 1 }, { unique: true });
    
    mongoSchema.loadClass(ChapterClass);
    
    const Chapter = mongoose.model('Chapter', mongoSchema);
    
    export default Chapter;
    

We introduced `syncContent()` methods to Book and Chapter model. These methods use Github API methods to get or sync data from Github. So how do we trigger a sync event? We should let the Admin user initiate syncing with a button click. Each book should have some sort of _details_ page that has a _Sync_ button that an Admin clicks to sync content.

It's important to note that syncing content from Github is a slow process. First, our app calls `Book.syncContent()` method, which in turn loops through each chapter. For each chapter, app calls `Chapter.syncContent()` that sends server-to-server request to Github, waits for response, decodes content and saves it to our database. Since Node is single-threaded, any computationally intense task may block it for incoming requests. And this is exactly the case with syncing content from Github. You will notice that while app is syncing content, app will load pages with a noticeable delay.

It is possible to isolate slow requests (computationally intense tasks) into so called `forked` or `child` process which is a process that runs in parallel to `main` or `parent` Node process. That way main Node process stays _unblocked_ and available for incoming requests while forked process deals with slow request. We implemented forked process for syncing content in our [open source project](https://github.com/builderbook/builderbook/blob/master/server/api/admin.js), find the line with `const sync = fork()`. Check the code out if you want to dive deeper into Node scalability. Also stay tuned for upcoming [tutorials](https://builderbook.org/tutorials) about Node scalability.

In the next section, we introduce the remaining pages in our Admin dashboard: a page to create a book, a page to edit a book, and a detail page that has a button to sync content.

## Admin dashboard
------------------------------------------

At this point, our Book and Chapter models have all necessary static methods. Our Book model has `list()`, `getBySlug()`, `add()`, `edit()`, and `syncContent()` static methods. Our Chapter model has `getBySlug()` and `synContent`.

You'll notice that our Chapter model, unlike the Book model, does not have `add()` and `edit()` methods. That's because the Admin user creates and updates a _book_ directly inside our web app. However, the Admin creates and updates a _chapter_ on Github and syncs this content to our database with the `syncContent()` method. Thus, no need for `add()` and `edit()` methods in the Chapter model.

So far, we've only used the Book model's `list()` method to display a list of books on `pages/admin/index.js` and the Chapter model's `getBySlug()` method to display chapter data on `pages/public/read-chapter.js`. Remember that to test out these two pages, you had to _manually_ insert documents to MongoDB. In this and the following subsection, we will discuss and add three more Admin pages and one Admin component:

*   `pages/admin/add-book.js`
*   `pages/admin/edit-book.js`
*   `pages/admin/book-detail.js`
*   `components/admin/EditBook.js`

The names of these pages are self-explanatory. The first page will allow us to create a book; second to edit a book (for example, edit price or Github repo); third page will show our Admin user some book data and a `Sync` button that syncs book content between Github and our database. The `EditBook.js` component will render a list of repos and let the Admin user pick a repo from which our app will get content for the book's chapters.

After adding these missing pages and corresponding Express routes and methods, we will test out the entire Admin flow, from book creation to content syncing.

Let's discuss the Admin's initial action and subsequent data flow in detail.

1.  For adding a new book, the data flow is:
    
    *   Admin clicks button on page `add-book.js` (at `pages/admin/add-book.js`) =>
    *   API method `addBook` (at `lib/api/admin.js`) sends POST request to server. Request's `body` has the book's data. =>
    *   Express route `router.post('/books/add')` (at `server/api/admin.js`) =>
    *   Static method `static async add()` (at `server/models/Book.js`) \- _already done_
    
    `=>` stands for `calls or triggers`: clicking a button `calls` a method, the method `calls` an Express route, and the route `calls` a static method.
    
    We already wrote the static methods `list()`, `getBySlug`, `add()`, `edit()`, and `syncContent` for our Book and Chapter models. Thus we added the note _`already done`_ to the last step (static method step).
    
2.  The Admin user should be able to edit a book (for example, edit the price). Here is the initial Admin action and data flow:
    
    *   Admin clicks button on page `edit-book.js` (at `pages/admin/edit-book.js`) =>
    *   API method `editBook` (at `lib/api/admin.js`) sends POST request to server. Request's `body` has the book's data. =>
    *   Express route `router.post('/books/edit')` (at `server/api/admin.js`) =>
    *   Static method `static async edit()` (at `server/models/Book.js`) \- _already done_
3.  We want the Admin user to be able to see a book's parameters at `pages/admin/book-detail.js`. Instead of clicking a button, the Admin simply loads the page to call the API method `getBookDetail`:
    
    *   Admin loads page `book-detail.js` page (at `pages/admin/book-detail.js`) =>
    *   API method `getBookDetail` (at `lib/api/admin.js`) =>
    *   Express route `router.get('/books/detail/:slug')` (at `server/api/admin.js`) =>
    *   Static method `static async getBySlug()` (at `server/models/Book.js`) \- _already done_
4.  On the `book-detail.js` page, we will have a `Sync` button. The Admin triggers the `syncBookContent` API method by clicking this button.
    
    *   Admin clicks `Sync` button on `book-detail.js` page (at `pages/admin/book-detail.js`) =>
    *   API method `syncBookContent` (at `lib/api/admin.js`) =>
    *   Express route `router.post('/books/sync-content')` (at `server/api/admin.js`) =>
    *   Static method `static async syncContent()` (at `server/models/Book.js`) \- _already done_
5.  There is one more API method that we need to add: `getGithubRepos`. None of the Admin pages directly contain this method. In fact, we call it from the `EditBook.js` component at `components/admin/EditBook.js`. We import this component into two Admin pages: `add-book.js` and `edit-book.js`.
    
    From the method's name, `getGithubRepos`, you can understand that this method sends a request to our server, and in return, our server executes the `getRepos()` API method for Github. As a final outcome of this chain of events, our `EditBook.js` component receives a list of repos. Our Admin user is able to see this list on the `add-book.js` and `edit-book.js` pages. The Admin picks one repo from this list, thus passing a book's parameter `githubRepo` to the `addBook` and `editBook` API methods.
    
    *   Admin loads either `add-book.js` or `edit-book.js` page, this loads `EditBook.js` component (at `components/admin/EditBook.js`) =>
    *   API method `getGithubRepos` (at `lib/api/admin.js`) =>
    *   Express route `router.get('/github/repos')` (at `server/api/admin.js`) =>
    *   Github's API method `getRepos()` (at `server/github.js`) \- _already done_

#### Express routes

In this subsection, we will add the following five routes to our Admin code at `server/api/admin.js`:

1.  `router.post('/books/add')`
2.  `router.post('/books/edit')`
3.  `router.get('/books/detail/:slug')`
4.  `router.post('/books/sync-content')`
5.  `router.get('/github/repos')`

1.  The Express route `router.post('/books/add')` gets the book's data (`name`, `price`, `githubRepo`) from the request's `body`. This route calls the static method `add()` in our Book model to create a new book.  
    `server/api/admin.js` :
    
        router.post('/books/add', async (req, res) => {
         try {
           const book = await Book.add(Object.assign({ userId: req.user.id }, req.body));
           res.json(book);
         } catch (err) {
           logger.error(err);
           res.json({ error: err.message || err.toString() });
         }
        });
        
    
    The code inside this route does not have to return a `book` object to the client (browser). We use this route to create a new book with data stored in `req.body`. However, after a new book is created, we want to sync the book content with the `syncContent()` function that requires a `book._id` from a `book` object. Thus, let's return `book` object to the client (browser) with `res.json(book)`. In step 2, inside the Express route `router.post('/books/edit')`, we also don't need to return a `book` object. Thus, we return `res.json({ done: 1 });` to the client. See below for more detail.
    
    An important note on POST requests.
    
    Our server has to parse and decode a POST request's body `req.body`. We need to tell Express to use middleware that parses/decodes `application/json` format. We do so by using Express's package [body-parser](https://www.npmjs.com/package/body-parser).
    
    Import `bodyParser` to `server/app.js`:
    
    import bodyParser from 'body-parser';
    
    Add the following line to `server/app.js` above the `const MongoStore = mongoSessionStore(session);` line:
    
    server.use(bodyParser.json());
    
    An alternative to using the external bodyParser package is to use internal Express middleware. To do so, remove the import code for bodyParser and replace the above line of code with:
    
    server.use(express.json());
    
    Both `bodyParser.json()` and `express.json()` return middlware that parses and decodes data JSON format from request's `body` and saves output in `req.body`. To reduce number of external packages, let's use `express.json()`.
    
    To understand Express's body-parser in more detail, check out this [blog post](https://medium.com/@adamzerner/how-bodyparser-works-247897a93b90).
    
2.  The Express route `router.post('/books/edit')` is very similar to `router.post('/books/add')`. Instead of returning `res.json()`, it returns `res.json({ done: 1 })`.  
    `server/api/admin.js` :
    
        router.post('/books/edit', async (req, res) => {
         try {
           await Book.edit(req.body);
           res.json({ done: 1 });
         } catch (err) {
           res.json({ error: err.message || err.toString() });
         }
        });
        
    
    The reason we don't need to return a `book` object to the client is that the `book` object will _already_ be on the `EditBook` page. The Express route `router.get('/books/detail/:slug')` sends a `book` object to the `EditBook` page, so `router.post('/books/edit')` does not have to.
    
    Another note on POST requests.
    
    For POST requests that pass data to our server (to create/update data), the response typically _does not_ have to return any data to the client. For example, in the Express route above, we don't have to return a `book` object to the client (browser). However, the server must return a response in a `req`-`res` cycle. Thus, we decided to return `done: 1` (instead of returning any actual data). You can return whatever you want, for example `save: 1`.
    
3.  The Express route `router.get('/books/detail/:slug')` gets `slug` and is called by the `getBookDetail()` API method located in the `pages/admin/book-detail.js` page. `Book.getBySlug()`, inside this Express route, finds a book using `slug`.
    
    Express uses `req.params` ([discussed before in Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#page)) to extract a parameter from the route with `req.params.slug`.  
    `server/api/admin.js` :
    
        router.get('/books/detail/:slug', async (req, res) => {
         try {
           const book = await Book.getBySlug({ slug: req.params.slug });
           res.json(book);
         } catch (err) {
           res.json({ error: err.message || err.toString() });
         }
        });
        
    
4.  Inside the `router.post('/books/sync-content')` route, we want to do two things:
    
    *   check if our Admin user has connected Github to our app
    *   call the `syncContent()` static method from our Book model
    
    To check if the user has connected Github, we send the user's `_id` to our server as `req.user._id`. Then we use `req.user._id` to find this user with Mongoose's `Model.findById(id, [projection])` method. In `[projection]`, we specify values we'd like to return: `isGithubConnected` and `githubAccessToken`:  
    `const user = await User.findById(req.user._id, 'isGithubConnected githubAccessToken');`
    
    We check if `isGithubConnected` is true \_or\_ if `githubAccessToken` exists (not null). We throw an error if at least one of them is false or does not exist:
    
        if (!user.isGithubConnected || !user.githubAccessToken) {
           res.json({ error: 'Github is not connected' });
           return;
        }
        
    
    Finally, by using the `try/catch` construct (as you did many times already), our Express route calls the Book model's `syncContent()` static method. This method takes two parameters (check up `server/models/Book.js`):
    
        try {
         await Book.syncContent({ id: bookId, githubAccessToken: user.githubAccessToken });
         res.json({ done: 1 });
        } catch (err) {
         logger.error(err);
         res.json({ error: err.message || err.toString() });
        }
        
    
    Put it all together and you get:  
    `server/api/admin.js` :
    
        router.post('/books/sync-content', async (req, res) => {
         const { bookId } = req.body;
        
         const user = await User.findById(req.user._id, 'isGithubConnected githubAccessToken');
        
         if (!user.isGithubConnected || !user.githubAccessToken) {
           res.json({ error: 'Github not connected' });
           return;
         }
        
         try {
           await Book.syncContent({ id: bookId, githubAccessToken: user.githubAccessToken });
           res.json({ done: 1 });
         } catch (err) {
           logger.error(err);
           res.json({ error: err.message || err.toString() });
         }
        });
        
    
    One thing to note - `syncContent()` needs `bookId`. In our request that we send to the server, we pass `bookId` in the request's body as `req.body.bookId`. We use ES6 destructuring syntax:
    
    const { bookId } = req.body;
    
5.  Inside the `router.get('/github/repos')` Express route, our goals are:
    
    *   check if our Admin user has connected Github to our web app
    *   call the `getRepos` API method (defined in `server/github.js`), which returns a list of repos for a given user
    
    We just wrote code for checking if Github is connected in `router.post('/books/sync-content')`:
    
        const user = await User.findById(req.user._id, 'isGithubConnected githubAccessToken');
        
        if (!user.isGithubConnected || !user.githubAccessToken) {
         res.json({ error: 'Github is not connected' });
         return;
        }
        
    
    Calling `getRepos()` with `try/catch` will look very similar to how we called `Book.syncContent()` with that same construct. Keep in mind that unlike `Book.syncContent()`, `getRepos()` requires only one parameter, `accessToken: user.githubAccessToken`:
    
         try {
           const response = await getRepos({ accessToken: user.githubAccessToken });
           res.json({ repos: response.data });
         } catch (err) {
           logger.error(err);
           res.json({ error: err.message || err.toString() });
         }
        
    
    The only difference is that we wait (`await`) for a `response` with data (`response.data`) from the `getRepos()` API method. We send a response with data (list of repos) to the client:
    
    res.json({ repos: response.data });
    
    Put it together and you get:  
    `server/api/admin.js` :
    
        router.get('/github/repos', async (req, res) => {
         const user = await User.findById(req.user._id, 'isGithubConnected githubAccessToken');
        
         if (!user.isGithubConnected || !user.githubAccessToken) {
           res.json({ error: 'Github is not connected' });
           return;
         }
        
         try {
           const response = await getRepos({ accessToken: user.githubAccessToken });
           res.json({ repos: response.data });
         } catch (err) {
           logger.error(err);
           res.json({ error: err.message || err.toString() });
         }
        });
        
    

Add these five Express routes to `server/api/admin.js`, just below the `router.get('/books')` Express route.

Check your list of imports as well. It should be:

    import express from 'express';
    
    import Book from '../models/Book';
    import { getContent, getRepos } from '../github';
    import User from '../models/User';
    import logger from '../logs';
    

#### API methods

Alright, the static methods for our models and Express routes are done. Here we define five API methods:

1.  `addBook`
2.  `editBook`
3.  `getBookDetail`
4.  `syncBookContent`
5.  `getGithubRepos`

Let's discuss, write, and add these API methods to `lib/api/admin.js`.

Open `lib/api/admin.js`. Remember how we implemented the `getBookList()` API method:  
`lib/api/admin.js` :

    export const getBookList = () =>
      sendRequest(`${BASE_PATH}/books`, {
        method: 'GET',
      });
    

In Chapter 5, we defined the `sendRequest()` function at `lib/api/sendRequest.js`. By default, this method is POST unless we specify `method: 'GET'`.

1.  The API method `addBook` takes `name`, `price`, and `githubRepo` specified by our Admin user and sends a POST request to the server at `/api/v1/admin/books/add`. POST is the default method, so we don't need to specify it inside `sendRequest()`. We do add the three book parameters (necessary for new book creation) to our request's `body`.  
    `lib/api/admin.js` :
    
        export const addBook = ({ name, price, githubRepo }) =>
        sendRequest(`${BASE_PATH}/books/add`, {
         body: JSON.stringify({ name, price, githubRepo }),
        });
        
    
    Note that `${BASE_PATH}` for `lib/api/admin.js` is `/api/v1/admin`.
    
2.  The API method `editBook` is very similar to `addBook` \- it's a POST method that takes `name`, `price`, and `githubRepo`. In addition to these parameters, it takes a book's `id` to pass it to `findById` inside our static method `static async edit()` at `server/models/Book.js`.  
    `lib/api/admin.js` :
    
        export const editBook = ({
         id, name, price, githubRepo,
        }) =>
         sendRequest(`${BASE_PATH}/books/edit`, {
           body: JSON.stringify({
             id,
             name,
             price,
             githubRepo,
           }),
        });
        
    
3.  Unlike our `addBook` and `editBook` methods, the `getBookDetail` method sends a GET request. The server receives a `slug` parameter as part of the query string `/api/v1/admin/books/detail/${slug}`.  
    `lib/api/admin.js` :
    
        export const getBookDetail = ({ slug }) =>
        sendRequest(`${BASE_PATH}/books/detail/${slug}`, {
         method: 'GET',
        });
        
    
4.  The API method `syncBookContent` sends a POST request to the server. This method adds `bookId` to the request's `body`.  
    `lib/api/admin.js` :
    
        export const syncBookContent = ({ bookId }) =>
         sendRequest(`${BASE_PATH}/books/sync-content`, {
           body: JSON.stringify({ bookId }),
         });
        
    
5.  Finally, the API method `getGithubRepos` sends a GET request to the server. This method does not pass any of a book's parameters to the server. The HOC `withAuth.js` that wraps all Admin pages passes a user to the server, where our Express route `router.get('/github/repos')` uses `req.user._id` and `user.githubAccessToken` to find the user and get a list of his/her repos.
    
    Add the following snippet to `lib/api/admin.js` :
    
        export const getGithubRepos = () =>
         sendRequest(`${BASE_PATH}/github/repos`, {
           method: 'GET',
         });
        
    

Put the API methods from steps 1 to 5 into `lib/api/admin.js` as follows:  
`lib/api/admin.js` :

    import sendRequest from './sendRequest';
    
    const BASE_PATH = '/api/v1/admin';
    
    export const syncTOS = () => sendRequest(`${BASE_PATH}/sync-tos`);
    
    export const getBookList = () =>
      sendRequest(`${BASE_PATH}/books`, {
        method: 'GET',
      });
    
    export const getBookDetail = ({ slug }) =>
      sendRequest(`${BASE_PATH}/books/detail/${slug}`, {
        method: 'GET',
      });
    
    export const addBook = data =>
      sendRequest(`${BASE_PATH}/books/add`, {
        body: JSON.stringify(data),
      });
    
    export const editBook = data =>
      sendRequest(`${BASE_PATH}/books/edit`, {
        body: JSON.stringify(data),
      });
    
    export const syncBookContent = ({ bookId }) =>
      sendRequest(`${BASE_PATH}/books/sync-content`, {
        body: JSON.stringify({ bookId }),
      });
    
    export const getGithubRepos = () =>
      sendRequest(`${BASE_PATH}/github/repos`, {
        method: 'GET',
      });
    

#### Admin pages and components

At this point, we've implemented static methods inside our models, Express routes, and API methods. Here we work on our last task - adding API methods to our pages and components.

We will discuss components and pages in this order:

1.  Component `components/adminEditBook.js`
2.  Page `pages/admin/add-book.js`
3.  Page `pages/admin/edit-book.js`
4.  Page `pages/admin/book-detail.js`

1.  The component `EditBook` makes up most of the interface for our `add-book.js` and `edit-book.js` pages. Thus we should discuss how this component works before we go into pages.
    
    Note that we've already built multiple pages and components with React and Material-UI. We discussed how to use `propTypes`, `defaultProps`, `constructor`, `state`, `getInitialProps()`, `componentDidMount`, and more. We won't repeat what you learned - we'll primarily discuss how to add API methods to pages and components.
    
    The component `EditBook` is essentially a simple form with a `Save` button. When our Admin clicks `Save`, the form gets submitted, thereby triggering `onSubmit = (event) =>`. This event passes `name`, `price`, and `githubRepo` to `this.state.book` with ES6 destructuring:
    
    const { name, price, githubRepo } = this.state.book;
    
    If all three parameters exist, then they are passed to the `onSave` function as `this.state.book`, and we call the `onSave` prop function (i.e. parameter of `props` object, `this.props.onSave`) with:
    
    this.props.onSave(this.state.book);
    
    One important purpose of this component is to call `getGithubRepos()` to get a list of repos. Our Admin user will select one Github repo out of this list to create a book. As always, we call the method only after our component mounts, using our favorite `async/await` and `try/catch` combo:
    
        async componentDidMount() {
         try {
           const { repos } = await getGithubRepos();
           this.setState({ repos }); // eslint-disable-line
         } catch (err) {
           logger.error(err);
         }
        }
        
    
    `constructor` sets an initial state with `book` and `repos` props (we discussed `constructor` in detail in [Chapter 2](https://builderbook.org/books/builder-book/server-database-session-header-and-menudrop-components#menudrop-component) and [Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#readchapter-page)):
    
        constructor(props) {
         super(props);
        
         this.state = {
           book: props.book || {},
           repos: [],
         };
        }
        
    
    The form has three items: book name (`<TextField>`), book title (also `<TextField>`), and a dropdown list of repos (`<Select>` with `<MenuItem>`).
    
    Put all this information about our `EditBook` component together:  
    `components/admin/EditBook.js` :
    
        import React from 'react';
        import PropTypes from 'prop-types';
        import Button from 'material-ui/Button';
        import TextField from 'material-ui/TextField';
        import Input from 'material-ui/Input';
        import Select from 'material-ui/Select';
        import { MenuItem } from 'material-ui/Menu';
        
        import { getGithubRepos } from '../../lib/api/admin';
        import { styleTextField } from '../../components/SharedStyles';
        import notify from '../../lib/notifier';
        import logger from '../../server/logs';
        
        class EditBook extends React.Component {
         static propTypes = {
           book: PropTypes.shape({
             _id: PropTypes.string.isRequired,
           }),
           onSave: PropTypes.func.isRequired,
         };
        
         static defaultProps = {
           book: null,
         };
        
         constructor(props) {
           super(props);
        
           this.state = {
             book: props.book || {},
             repos: [],
           };
         }
        
         async componentDidMount() {
           try {
             const { repos } = await getGithubRepos();
             this.setState({ repos }); // eslint-disable-line
           } catch (err) {
             logger.error(err);
           }
         }
        
         onSubmit = (event) => {
           event.preventDefault();
           const { name, price, githubRepo } = this.state.book;
        
           if (!name) {
             notify('Name is required');
             return;
           }
        
           if (!price) {
             notify('Price is required');
             return;
           }
        
           if (!githubRepo) {
             notify('Github repo is required');
             return;
           }
        
           this.props.onSave(this.state.book);
         };
        
         render() {
           return (
             <div style={{ padding: '10px 45px' }}>
               <form onSubmit={this.onSubmit}>
                 <br />
                 <div>
                   <TextField
                     onChange={(event) => {
                       this.setState({
                         book: Object.assign({}, this.state.book, { name: event.target.value }),
                       });
                     }}
                     value={this.state.book.name}
                     type="text"
                     label="Book's title"
                     labelClassName="textFieldLabel"
                     style={styleTextField}
                     required
                   />
                 </div>
                 <br />
                 <br />
                 <TextField
                   onChange={(event) => {
                     this.setState({
                       book: Object.assign({}, this.state.book, { price: Number(event.target.value) }),
                     });
                   }}
                   value={this.state.book.price}
                   type="number"
                   label="Book's price"
                   className="textFieldInput"
                   style={styleTextField}
                   step="1"
                   required
                 />
                 <br />
                 <br />
                 <div>
                   <span>Github repo: </span>
                   <Select
                     value={this.state.book.githubRepo || ''}
                     input={<Input />}
                     onChange={(event) => {
                       this.setState({
                         book: Object.assign({}, this.state.book, { githubRepo: event.target.value }),
                       });
                     }}
                   >
                     <MenuItem value="">
                       <em>-- choose github repo --</em>
                     </MenuItem>
                     {this.state.repos.map(r => (
                       <MenuItem value={r.full_name} key={r.id}>
                         {r.full_name}
                       </MenuItem>
                     ))}
                   </Select>
                 </div>
                 <br />
                 <br />
                 <Button raised type="submit">
                   Save
                 </Button>
               </form>
             </div>
           );
         }
        }
        
        export default EditBook;
        
    
    We are done with the `EditBook` component. Now let's discuss pages.
    
2.  The page `add-book.js` is straightforward. Most of this page's interface comes from the `EditBook` component. We need to achieve this: Admin _clicks_ on the form's `Save` button to call the `addBook()` method and pass a book's `data` to this method. That's why we wrote an `addBookOnSave` function inside the `EditBook` component. Once our Admin clicks `Save`, the `EditBook` component does three things:
    
    *   submits the form
    *   passes the book's `name`, `price`, and `githubRepo` (as `this.state.book`) to the `onSave` function
    *   calls the `onSave` function
    
    We call `addBookOnSave()` with `<EditBook onSave={this.addBookOnSave} />`
    
    To create a new book, we want the `addBookOnSave` function to call the API method `addBook()`. The API method `addBook()` returns a `book` object.
    
    Then we wait for the API method `syncBookContent()` to sync content with Github. When done, we display success with `notify()` from [Chapter 4](https://builderbook.org/books/builder-book/testing-with-jest-debugging-with-winston-transactional-emails-in-app-notifications#in-app-notifications). Also we let the loading Nprogress bar finish with `NProgress.done()`.
    
    At the end, our app should redirect the Admin to the `BookDetail` page (`pages/admin/book-detail.js`) with Next.js's `Router.push()` ([read more](https://github.com/zeit/next.js#imperatively)):
    
        addBookOnSave = async (data) => {
         NProgress.start();
        
         try {
           const book = await addBook(data);
           notify('Saved');
           try {
             const bookId = book._id;
             await syncBookContent({ bookId });
             notify('Synced');
             NProgress.done();
             Router.push(`/admin/book-detail?slug=${book.slug}`, `/admin/book-detail/${book.slug}`);
           } catch (err) {
             notify(err);
             NProgress.done();
           }
         } catch (err) {
           notify(err);
           NProgress.done();
         }
        };
        
    
    We'll discuss and create the `BookDetail` page later in this chapter.
    
    Note that `data` inside the `addBookOnSave = async (data)` function is `this.state.book`. This is because we defined our `onSave()` function as `onSave(this.state.book)` (see the `EditBook` component above), and we pointed `onSave()` to `addBookOnSave()`: `onSave={this.addBookOnSave}` (see above snippet).
    
    Create an `AddBook` component and add the above code to it before `render()`:  
    `pages/admin/add-book.js` :
    
        import React from 'react';
        import Router from 'next/router';
        import NProgress from 'nprogress';
        
        import withLayout from '../../lib/withLayout';
        import withAuth from '../../lib/withAuth';
        import EditBook from '../../components/EditBook';
        import { addBook, syncBookContent } from '../../lib/api/admin';
        import notify from '../../lib/notifier';
        
        class AddBook extends React.Component {
         addBookOnSave = async (data) => {
           NProgress.start();
        
           try {
             const book = await addBook(data);
             notify('Saved');
             try {
               const bookId = book._id;
               await syncBookContent({ bookId });
               notify('Synced');
               NProgress.done();
               Router.push(`/admin/book-detail?slug=${book.slug}`, `/admin/book-detail/${book.slug}`);
             } catch (err) {
               notify(err);
               NProgress.done();
             }
           } catch (err) {
             notify(err);
             NProgress.done();
           }
         };
        
         render() {
           return (
             <div style={{ padding: '10px 45px' }}>
               <EditBook onSave={this.addBookOnSave} />
             </div>
           );
         }
        }
        
        export default withAuth(withLayout(AddBook));
        
    
    The Admin has to wait a bit for the API method to return a response (with data for GET requests, without data for POST requests). Thus we use `NProgress.start();` before we call `await addBook()`, and we call `NProgress.done();` after it.
    
3.  The page `edit-book.js` is a bit more complex than `add-book.js`. In addition to calling the API method `editBook()`, we have to display the book's current data with another API method, `getBookDetail()`. We do so:
    
    *   with Next.js's `getInitialProps()`:
        
            static getInitialProps({ query }) {
             return { slug: query.slug };
            }
            
        
    *   with our `getBookDetail()` API method inside the `componentDidMount` lifecycle hook:
        
            async componentDidMount() {
             NProgress.start();
            
             try {
               const book = await getBookDetail({ slug: this.props.slug });
               this.setState({ book }); // eslint-disable-line
               NProgress.done();
             } catch (err) {
               this.setState({ error: err.message || err.toString() }); // eslint-disable-line
               NProgress.done();
             }
            }
            
        
    *   and by passing the `book` prop to our `EditBook` component with `<EditBookComp book={book} />`
        
    
    Next, we need to make sure that when our Admin clicks the `Save` button, we call the API method `editBook()`. We make sure that after form submission, the `onSave` function points to the internal `editBookOnSave` function: `<EditBookComp onSave={this.editBookOnSave} book={book} />`
    
    We call the `editBook()` API method inside the `editBook()` function as follows:
    
        editBookOnSave = async (data) => {
         const { book } = this.state;
         NProgress.start();
        
         try {
           await editBook({ ...data, id: book._id });
           notify('Saved');
           NProgress.done();
           Router.push(`/admin/book-detail?slug=${book.slug}`, `/admin/book-detail/${book.slug}`);
         } catch (err) {
           notify(err);
           NProgress.done();
         }
        };
        
    
    Note that after `editBook()` finishes editing a book, we indicate this to user with `notify()` and `Nprogress.done()`.
    
    At the end, our app redirects the Admin user to the `BookDetail` page (`pages/admin/book-detail.js`) that we introduce later in this chapter.
    
    Again, `data` inside the `editBookOnSave = async (data)` function is `this.state.book`, because we defined our `onSave()` function as `onSave(this.state.book)` (in the `EditBook` component), and we pointed `onSave()` to `editBookOnSave()`: `onSave={this.editBookOnSave}`.
    
    You'll notice that unlike passing data without modification (`addBook(data);`), here we _add_ an `id` to our data. Thus the syntax is `editBook({ ...data, id: book._id });` instead of `editBook(data);`.
    
    In summary, we get:  
    `pages/admin/edit-book.js` :
    
        import React from 'react';
        import Router from 'next/router';
        import NProgress from 'nprogress';
        import PropTypes from 'prop-types';
        import Error from 'next/error';
        
        import EditBookComp from '../../components/admin/EditBook';
        import { getBookDetail, editBook } from '../../lib/api/admin';
        import withLayout from '../../lib/withLayout';
        import withAuth from '../../lib/withAuth';
        import notify from '../../lib/notifier';
        
        class EditBook extends React.Component {
         static propTypes = {
           slug: PropTypes.string.isRequired,
         };
        
         static getInitialProps({ query }) {
           return { slug: query.slug };
         }
        
         state = {
           error: null,
           book: null,
         };
        
         async componentDidMount() {
           NProgress.start();
        
           try {
             const book = await getBookDetail({ slug: this.props.slug });
             this.setState({ book }); // eslint-disable-line
             NProgress.done();
           } catch (err) {
             this.setState({ error: err.message || err.toString() }); // eslint-disable-line
             NProgress.done();
           }
         }
        
         editBookOnSave = async (data) => {
           const { book } = this.state;
           NProgress.start();
        
           try {
             await editBook({ ...data, id: book._id });
             notify('Saved');
             NProgress.done();
             Router.push(`/admin/book-detail?slug=${book.slug}`, `/admin/book-detail/${book.slug}`);
           } catch (err) {
             notify(err);
             NProgress.done();
           }
         };
        
         render() {
           const { book, error } = this.state;
        
           if (error) {
             notify(error);
             return <Error statusCode={500} />;
           }
        
           if (!book) {
             return null;
           }
        
           return (
             <div>
               <EditBookComp onSave={this.editBookOnSave} book={book} />
             </div>
           );
         }
        }
        
        export default withAuth(withLayout(EditBook));
        
    
4.  The `book-detail.js` page has two main purposes. The first is to show book data (such as `name`, `githubRepo`, `chapters`, and more) to the Admin user. The second is to sync content. This page will have a `Sync` button that our Admin clicks to get content from Github.
    
    Similar to our `edit-book.js` page, we need to display book data. We do it the same way as we did on `edit-book.js`.
    
    *   `getInitialProps()`:
        
             static getInitialProps({ query }) {
             return { slug: query.slug };
            }
            
        
    *   API method `getBookDetail()` inside lifecycle hook `componentDidMount`:
        
            async componentDidMount() {
             NProgress.start();
             try {
               const book = await getBookDetail({ slug: this.props.slug });
               this.setState({ book, loading: false }); // eslint-disable-line
               NProgress.done();
             } catch (err) {
               this.setState({ loading: false, error: err.message || err.toString() }); // eslint-disable-line
               NProgress.done();
             }
            }
            
        
    *   Passing props to component:
        
    
    To sync content on the button click, add the function `handleSyncContent()` to the `onClick` handler:
    
        <Button raised onClick={handleSyncContent(book._id)}>
         Sync with Github
        </Button>
        
    
    We call the API method `syncBookContent()` ([discussed in Admin Dashboard section](https://builderbook.org/books/builder-book/github-integration-admin-dashboard-testing-admin-ux-and-github-integration#api-methods)) inside the `handleSyncContent()` function:
    
        const handleSyncContent = bookId => async () => {
        try {
         await syncBookContent({ bookId });
         notify('Synced');
        } catch (err) {
         notify(err);
        }
        };
        
    
    Our `book-detail.js` page will now have a list of chapters with hyperlinked titles.
    
    Final code for this page:  
    `pages/admin/book-detail.js` :
    
        import React from 'react';
        import NProgress from 'nprogress';
        import PropTypes from 'prop-types';
        import Error from 'next/error';
        import Link from 'next/link';
        import Button from 'material-ui/Button';
        
        import { getBookDetail, syncBookContent } from '../../lib/api/admin';
        import withLayout from '../../lib/withLayout';
        import withAuth from '../../lib/withAuth';
        import notify from '../../lib/notifier';
        
        const handleSyncContent = bookId => async () => {
         try {
           await syncBookContent({ bookId });
           notify('Synced');
         } catch (err) {
           notify(err);
         }
        };
        
        const MyBook = ({ book, error }) => {
         if (error) {
           notify(error);
           return <Error statusCode={500} />;
         }
        
         if (!book) {
           return null;
         }
        
         const { chapters = [] } = book;
        
         return (
           <div style={{ padding: '10px 45px' }}>
             <h2>{book.name}</h2>
             <a target="_blank" rel="noopener noreferrer">
               Repo on Github
             </a>
             <p />
             <Button raised onClick={handleSyncContent(book._id)}>
               Sync with Github
             </Button>{' '}
             <Link as={`/admin/edit-book/${book.slug}`} href={`/admin/edit-book?slug=${book.slug}`}>
               <Button raised>Edit book</Button>
             </Link>
             <ul>
               {chapters.map(ch => (
                 <li key={ch._id}>
                   <Link
                     as={`/books/${book.slug}/${ch.slug}`}
                     href={`/public/read-chapter?bookSlug=${book.slug}&chapterSlug=${ch.slug}`}
                   >
                     <a>{ch.title}</a>
                   </Link>
                 </li>
               ))}
             </ul>
           </div>
         );
        };
        
        MyBook.propTypes = {
         book: PropTypes.shape({
           name: PropTypes.string.isRequired,
         }),
         error: PropTypes.string,
        };
        
        MyBook.defaultProps = {
         book: null,
         error: null,
        };
        
        class MyBookWithData extends React.Component {
         static propTypes = {
           slug: PropTypes.string.isRequired,
         };
        
         static getInitialProps({ query }) {
           return { slug: query.slug };
         }
        
         state = {
           loading: true,
           error: null,
           book: null,
         };
        
         async componentDidMount() {
           NProgress.start();
           try {
             const book = await getBookDetail({ slug: this.props.slug });
             this.setState({ book, loading: false }); // eslint-disable-line
             NProgress.done();
           } catch (err) {
             this.setState({ loading: false, error: err.message || err.toString() }); // eslint-disable-line
             NProgress.done();
           }
         }
        
         render() {
           return <MyBook {...this.props} {...this.state} />;
         }
        }
        
        export default withAuth(withLayout(MyBookWithData));
        
    

We are almost at the end of this section. Before we finish, let's make a small readability improvement.

Go back to `pages/public/read-chapter.js`. To pass `bookSLug` and `chapterSlug` parameters to the page and render this page on our server, we use `app.render()` ([discussed before in Chapter 5](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#page)):

    server.get('/books/:bookSlug/:chapterSlug', (req, res) => {
      const { bookSlug, chapterSlug } = req.params;
      app.render(req, res, '/public/read-chapter', { bookSlug, chapterSlug });
    });
    

When our app user navigates to the `/books/:bookSlug/:chapterSlug` route, we render the `/public/read-chapter` page on our server with `bookSLug` and `chapterSlug` parameters extracted from the route.

We have to do the same thing for our Admin's `edit-book.js` and `book-detail.js` pages. We need to extract a book's `slug` from the routes of these pages, pass this `slug` to the server, and then render `pages/admin/edit-book.js` and `pages/admin/book-detail.js`:

    server.get('/admin/book-detail/:slug', (req, res) => {
      const { slug } = req.params;
      app.render(req, res, '/admin/book-detail', { slug });
    });
    
    server.get('/admin/edit-book/:slug', (req, res) => {
      const { slug } = req.params;
      app.render(req, res, '/admin/edit-book', { slug });
    });
    

We can add the code snippet above to our main server code at `server/app.js` \- but that file is getting big.

Instead, let's move all three Express routes to a new file `routesWithSlug.js`:  
`server/routesWithSlug.js` :

    export default function routesWithSlug({ server, app }) {
      server.get('/books/:bookSlug/:chapterSlug', (req, res) => {
        const { bookSlug, chapterSlug } = req.params;
        app.render(req, res, '/public/read-chapter', { bookSlug, chapterSlug });
      });
    
      server.get('/admin/book-detail/:slug', (req, res) => {
        const { slug } = req.params;
        app.render(req, res, '/admin/book-detail', { slug });
      });
    
      server.get('/admin/edit-book/:slug', (req, res) => {
        const { slug } = req.params;
        app.render(req, res, '/admin/edit-book', { slug });
      });
    }
    

Go to `server/app.js`. Import this function with:

import routesWithSlug from './routesWithSlug';

Then initialize routes on the server with `routesWithSlug({ server, app })`. Add it like this::

    server.use(session(sess));
    
    auth({ server, ROOT_URL });
    api(server);
    routesWithSlug({ server, app });
    

We are ready for testing.

In the next section, we will improve our `Header` component. And in the section after that, we will test out entire Admin flow, which includes syncing content between our database and Github.

## Update Header component
----------------------------------------------------------

On our server, we added a redirect to the `/admin` page. Now we need to add Customer/Admin logic to our `Header` component.

For a logged-out user, the `Header` component looks the same, and this is how we want it to be.  
![Builder Book](https://user-images.githubusercontent.com/10218864/36345319-dd6ca3c4-13dc-11e8-9ec1-2b17d3c7d61f.png)

But let's change how the `Header` component looks to a logged-in user. The logged-in user can either be a Customer or Admin.

Currently, the `Header` component looks exactly same to Customer and Admin users:  
![Builder Book](https://user-images.githubusercontent.com/10218864/36345364-d4f466ae-13dd-11e8-9f94-13312e42087b.png)

We want to make 3 changes to our `Header` component:

1.  Remove the `Settings` link from the left
2.  Replace the `Got question?` link from the `MenuDrop` component with either `My books` link for a Customer user \_or\_ `Admin` link for an Admin user
3.  For an Admin user who did not connect Github to our app, we want to show the `Connect Github` button

Let's discuss each step.

1.  That's easy. Simply delete the `<div>` element containing the `Settings` link.
    
2.  This is where we need to use logic. Since a `user` object is available in the `Header` component (our `withLayout` HOC passes `user` to `Header`), we can check if a user is an Admin with `user.isAdmin`.
    
    For the Customer user, `{user && !user.isAdmin ? ... : ...}`.  
    For the Admin user, `{user && user.isAdmin ? ... : ...}`.
    
    Open `components/Header.js` and replace the following code snippet:
    
        <div style={{ whiteSpace: ' nowrap' }}>
         {user.avatarUrl ? (
           <MenuDrop options={optionsMenu} src={user.avatarUrl} alt={user.displayName} />
         ) : null}
        </div>
        
    
    with a snippet that checks if a user is an Admin:
    
        <div style={{ whiteSpace: ' nowrap' }}>
         {!user.isAdmin ? (
           <MenuDrop
             options={optionsMenuCustomer}
             src={user.avatarUrl}
             alt={user.displayName}
           />
         ) : null}
         {user.isAdmin ? (
           <MenuDrop
             options={optionsMenuAdmin}
             src={user.avatarUrl}
             alt={user.displayName}
           />
         ) : null}
        </div>
        
    
    Above, we simply used `{condition ? value1 : value2}`
    
    We need to define `optionsMenuCustomer` and `optionsMenuAdmin`. Replace the following code snippet:
    
        const optionsMenu = [
         {
           text: 'Got question?',
           href: 'https://github.com/builderbook/builderbook/issues',
         },
         {
           text: 'Log out',
           href: '/logout',
         },
        ];
        
    
    with:
    
        const optionsMenuCustomer = [
         {
           text: 'My books',
           href: '/customer/my-books',
           as: '/my-books',
         },
         {
           text: 'Log out',
           href: '/logout',
         },
        ];
        
        const optionsMenuAdmin = [
         {
           text: 'Admin',
           href: '/admin',
         },
         {
           text: 'Log out',
           href: '/logout',
         },
        ];
        
    
3.  For an Admin user who did not connect Github yet, `{user && user.isAdmin && !user.isGithubConnected ? ... : ...}`.  
    Add an extra grid column that contains the `Connect Github` button:
    
        <Grid item sm={2} xs={2} style={{ textAlign: 'right' }}>
         {user && user.isAdmin && !user.isGithubConnected ? (
           <Hidden smDown>
             <a href="/auth/github">
               <Button raised color="primary">
               Connect Github
               </Button>
             </a>
           </Hidden>
         ) : null}
        </Grid>
        
    

Combine code from steps 1 to 3, and the refactored `Header` component will look like:  
`components/Header.js` :

    import PropTypes from 'prop-types';
    import Link from 'next/link';
    import Router from 'next/router';
    import NProgress from 'nprogress';
    import Toolbar from 'material-ui/Toolbar';
    import Grid from 'material-ui/Grid';
    import Hidden from 'material-ui/Hidden';
    import Button from 'material-ui/Button';
    import Avatar from 'material-ui/Avatar';
    
    import MenuDrop from './MenuDrop';
    
    import { styleToolbar } from './SharedStyles';
    
    Router.onRouteChangeStart = () => {
      NProgress.start();
    };
    Router.onRouteChangeComplete = () => NProgress.done();
    Router.onRouteChangeError = () => NProgress.done();
    
    const optionsMenuCustomer = [
      {
        text: 'My books',
        href: '/customer/my-books',
        as: '/my-books',
      },
      {
        text: 'Log out',
        href: '/logout',
      },
    ];
    
    const optionsMenuAdmin = [
      {
        text: 'Admin',
        href: '/admin',
      },
      {
        text: 'Log out',
        href: '/logout',
      },
    ];
    
    function Header({ user }) {
      return (
        <div>
          <Toolbar style={styleToolbar}>
            <Grid container direction="row" justify="space-around" alignItems="center">
              <Grid item sm={9} xs={8} style={{ textAlign: 'left' }}>
                {!user ? (
                  <Link prefetch href="/">
                    <Avatar
                      src="https://storage.googleapis.com/builderbook/logo.svg"
                      alt="Builder Book logo"
                      style={{ margin: '0px auto 0px 20px', cursor: 'pointer' }}
                    />
                  </Link>
                ) : null}
              </Grid>
              <Grid item sm={2} xs={2} style={{ textAlign: 'right' }}>
                {user && user.isAdmin && !user.isGithubConnected ? (
                  <Hidden smDown>
                    <a href="/auth/github">
                      <Button variant="raised" color="primary">
                        Connect Github
                      </Button>
                    </a>
                  </Hidden>
                ) : null}
              </Grid>
              <Grid item sm={1} xs={2} style={{ textAlign: 'right' }}>
                {user ? (
                    <div style={{ whiteSpace: ' nowrap' }}>
                      {!user.isAdmin ? (
                        <MenuDrop
                          options={optionsMenuCustomer}
                          src={user.avatarUrl}
                          alt={user.displayName}
                        />
                      ) : null}
                      {user.isAdmin ? (
                        <MenuDrop
                          options={optionsMenuAdmin}
                          src={user.avatarUrl}
                          alt={user.displayName}
                        />
                      ) : null}
                    </div>
                  ) : (
                    <Link prefetch href="/public/login" as="/login">
                      <a style={{ margin: '0px 20px 0px auto' }}>Log in</a>
                    </Link>
                  )}
              </Grid>
            </Grid>
          </Toolbar>
        </div>
      );
    }
    
    Header.propTypes = {
      user: PropTypes.shape({
        avatarUrl: PropTypes.string,
        displayName: PropTypes.string,
      }),
    };
    
    Header.defaultProps = {
      user: null,
    };
    
    export default Header;
    

Testing time.

Go to mlab, navigate to your User collection, and find your user document.

*   To test an Admin user. Set `"isAdmin": true` and `"isGithubConnected": false` on the database at mLab.  
    Start the app (`yarn dev`), go to the `/login` page, and log in. You'll be redirected to the `/admin` page:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36345703-dcb0f360-13e4-11e8-9362-202426e20771.png)
    
*   The above is an Admin user who did not connect Github. Set `"isAdmin": true` and `"isGithubConnected": true` on the database at mLab.  
    Refresh the tab.  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36345658-49efd99c-13e4-11e8-9e7c-364c6d9e1e8b.png)
    
*   Testing the Customer user is tricky since our app has no `/my-books` page.  
    Let's add the bare minimum of code we need to render this page. Create a file at `pages/customer/my-books.js` with the following content:  
    `pages/customer/my-books.js` :
    
        import React from 'react';
        
        import withLayout from '../../lib/withLayout';
        import withAuth from '../../lib/withAuth';
        
        const MyBooks = () => (
          <div style={{ padding: '10px 45px' }}>
            <h3>Your books</h3>
          </div>
        );
        
        export default withAuth(withLayout(MyBooks));
        
    
    Also, since our server attempts to render the page from `/my-books` instead of `/customer/my-books`, we have to add some handler function to `server/app.js` to fix this.
    
    Go to `server/app.js`. Below the `'/login': '/public/login',` line of code, add a new line of code `'/my-books': '/customer/my-books',`:
    
        const URL_MAP = {
          '/login': '/public/login',
          '/my-books': '/customer/my-books',
        };
        
    
    Go to mLab. Edit your user on the database to make sure that `"isAdmin": false`.  
    Log out and log in. After logging in, you are automatically redirected to `/admin`.  
    Go to `/my-books` page:  
    ![Builder Book](https://user-images.githubusercontent.com/10218864/36345769-067fc40e-13e6-11e8-83a0-c7bb36d0375c.png)
    

Good job if you see the proper UI.

One problem is the UX for our Customer user. After a login event, we should redirect the Customer to `/my-books` instead of `/admin`.

Open `server/google.js` and find the following Express route:

    server.get(
      '/oauth2callback',
      passport.authenticate('google', {
        failureRedirect: '/login',
      }),
      (req, res) => {
        res.redirect('/admin');
      },
    );
    
We discussed this Express route in detail [in Chapter 3](https://builderbook.org/books/builder-book/authentication-hoc-promise-async-await-static-method-for-user-model-google-oauth#express-routes-for-auth-). Passport makes the `user` object available at `req.user`. Similar to above, use `req.user.isAdmin` to check if a user is Admin:

    server.get(
      '/oauth2callback',
      passport.authenticate('google', {
        failureRedirect: '/login',
      }),
      (req, res) => {
        if (req.user && req.user.isAdmin) {
          res.redirect('/admin');
        } else {
          res.redirect('/my-books');
        }
      },
    );
    
Make sure your user is a Customer, then log out and log back in. After logging in, you will be _automatically_ redirected to `/my-books`.

## Testing
--------------------------

In this section, we test out our _Admin_ flow. We plan to test:

*   connecting to Github
*   adding a new book
*   editing that new book
*   syncing content

As we test our Admin flow - make sure that your user document in the database has the following parameters: `"isAdmin": true` and `"isGithubConnected": false`.

#### Connecting Github

So far, so good. Here we continue testing our Admin flow. Make sure that your user document has `"isAdmin": true` and `"isGithubConnected": false`. Log out and log into the app if necessary.  
![Builder Book](https://user-images.githubusercontent.com/10218864/34746269-36eb4d5a-f548-11e7-8a21-e3bd4eef9d07.png)

Before we click the `Connect Github` button:

1.  Go to Github and follow these [instructions](https://developer.github.com/v3/guides/basics-of-authentication/#registering-your-app) on how to register your app and get `ClientID` and `SecretKey` API credentials. Add both credentials to your app's `.env` file as follows:
    
        Github_Test_ClientID="XXXXXX"
        Github_Test_SecretKey="XXXXXX"
    
    Important note, Github does not support multiple domains in one registered app. We suggest you register two apps on Github, one for `http://localhost:8000` and a second for your production domain (in our case, it is `https://builderbook.org`).
    
    For development, take the values of `ClientID` and `SecretKey` from your Github app registered with `http://localhost:8000`. Pass these values as `Github_Test_ClientID` and `Github_Test_SecretKey`.
    
    For production, take the values of `ClientID` and `SecretKey` from your Github app registered with `https://yourdomain.com`. Pass these values as `Github_Live_ClientID` and `Github_Live_SecretKey`.
    
    For both Github apps, the callback route is `/auth/github/callback`.
    
2.  Open `server/app.js` and import the `setupGithub` function as `github`:  
    `import { setupGithub as github } from './github';`  
    Initiate it on the server with `github({ server });`. Add it like this:
    
        auth({ server, ROOT_URL });
        github({ server });
        api(server);
        routesWithSlug({ server, app });
        
You should monitor what happens to your user document on the database after you click `Connect Github`.

Go back to the browser and click the dark blue `Connect Github` button.  
![Builder Book](https://user-images.githubusercontent.com/10218864/34748275-a460f940-f550-11e7-8338-deecc7303125.png)

The button disappears, because `isGithubConnected` becomes `true`.

Go to the database and check your user document - you successfully connected Github if your app received `githubAccessToken` and saved it to the database and if `"isGithubConnected": true`.

#### Adding new book

Before we create a new book, let's add an `Add book` button to `pages/admin/index.js`:

    <Link href="/admin/add-book">
      <Button variant="raised">Add book</Button>
    </Link>
    

Add this button _above_ the list of all books (`<ul>...</ul>`):  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624934-295b4bee-18cb-11e8-9e22-8b4616386367.png)

Now go to Github. To save time from creating your own new repo with content, fork our [demo-book repo](https://github.com/builderbook/demo-book).

Click the `Add book` button. You are now on the `/add-book` page:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624935-297abe84-18cb-11e8-876c-3fb4eaedae14.png)

Set a price, write a name, and pick a repo. We set `49`, `Demo Book`, and `builderbook/demo-book`, respectively:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624936-299a03ca-18cb-11e8-9688-211b5a3c7de9.png)

Click the `Save` button and you will see your newly added book at the top of the list:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624937-29b37d82-18cb-11e8-94d0-d938e278079d.png)

#### Editing existing book

Click `Demo Book` from the list of all books. You will go to the `pages/admin/book-detail.js` page for this book. We see the book's `name` and `githubRepo`, as well as two buttons that we are going to test:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36625802-7294dc3e-18db-11e8-90f5-9b0d6f3ee2e5.png)

Click the `Edit book` button and you will go to `pages/admin/edit-book.js`. This page looks very similar to `pages/admin/add-book.js`, since both pages are mainly the `EditBook` component:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624940-2dece24e-18cb-11e8-9088-b3a22622cf5e.png)

#### Syncing content

Go to Github and check out the `introduction.md` file in the `demo-book` repo that you forked:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624941-2e0b639a-18cb-11e8-8454-466ad6ce6cb7.png)

Notice that this file has metadata at the top: `title`, `seoTitle`, `seoDescription`, and `isFree`. We discussed these parameters, as well as many others, in the [Chapter Schema section](https://builderbook.org/books/builder-book/book-and-chapter-models-internal-api-render-chapter#schema-for-chapter) of Chapter 5.

Every chapter (`.md` file) that we create for our book needs to have this metadata section at the top. For chapters that are not free, replace the `isFree` parameter with `excerpt:""` and add some content between the quotes. This content will be a free preview, but all other content in the chapter will be hidden until a user buys the book. See [`chapter-1.md`](https://github.com/builderbook/demo-book/blob/master/chapter-1.md) in the `demo-book` repo for an example of `excerpt` content.

Now click the `Sync with Github` button on `pages/admin/book-detail.js`. You will see an in-app success message `Synced` at the top right. Refresh the page, and you will see a hyperlinked titles to the Introduction and Chapter 1 (titled `Example`):  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624942-2e27eab0-18cb-11e8-943e-038bde808059.png)

Click on the Introduction link, and you will go to the `books/:bookSlug/:chapterSlug` page (this page's code is at `pages/public/read-chapter.js`). For our demo book, the page's URL is `http://localhost:8000/books/demo-book/introduction`:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624944-3271dbb2-18cb-11e8-868c-ba93c4d3e033.png)

Go back to Github and edit your `introduction.md` file by adding a sentence: `Hello world!`. You can add anything you want - feel free to add markdown content.  
![Builder Book](https://user-images.githubusercontent.com/26158226/36624945-328b82f6-18cb-11e8-9115-f9f7326d6915.png)

Go back to your Admin dashboard, then click `Demo Book`, then `Sync with Github`, and then finally the `Introduction` hyperlink:  
![Builder Book](https://user-images.githubusercontent.com/26158226/36625832-0d975040-18dc-11e8-8823-3bd74f5496fa.png)

Our app has successfully synced content between our database and Github's!

In this chapter, we added and tested all Admin pages in our app. Now you, as an Admin, can write content on Github and display it in your web app.

In the next chapter (Chapter 7), we will work on the `ReadChapter` page. Among other things, we will add an interactive Table of Contents.


* * *

At the end of Chapter 6, your codebase should look like the codebase in `6-end`. The [6-end](https://github.com/builderbook/builderbook/tree/master/book/6-end) folder is located at the root of the `book` directory inside the [builderbook repo](https://github.com/builderbook/builderbook).

Compare your codebase and make edits if needed.

Enjoying the book so far? Please share a quick [review](https://goo.gl/forms/JdevtnCWsLwZTAio2). You can update your review at any time.

* * *
