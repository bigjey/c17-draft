### New project

When working in team, it's better to use one package manager, either **npm** or **yarn**. In this guide we will use **npm**.

`npm init`
or
`npm init --yes` to answer all as 'default'.

Command above will create a **package.json** file in current folder.
All package installations in this folder will go to this **package.json** file.

### Folder structure

```
client/
server/
.eslint.json (OPTIONAL)
.gitignore
.prettierrc.json (OPTIONAL)
package-lock.json
package.json
```

**client** - source code for front-end
**server** - source code for back-end
**.eslint.json** - linter config
**.gitignore** - list of files and folders ignored by git
**.prettierrc.json** - prettier config
**package-lock.json** - autogenerated file, result of `npm install` commands
**package.json** - all the data about our node project

### Base node app

Let's add following file

`server/index.js`

Most important part of back-end application would be a http-server. We can use nodejs api to create one.

```js
// server/index.js
var http = require('http');

var server = http.createServer(function requestHandler(req, res) {
    res.writeHead(200);
    res.end('Hi everybody!');
});

server.listen(8080);
```

> **Note**: http is [one](https://nodejs.org/docs/latest-v8.x/api/http.html#http_http) of many built-in nodejs packages, they are available once you have installed nodejs software. For full list and more information you can check out [nodejs api documentation](https://nodejs.org/docs/latest-v8.x/api/index.html).

Run `node server/index.js` and open `http://localhost:8080` in brower, you should see 'Hi everybody!' message.

That's it, now our small http-server is running. Every request that comes to `http://localhost:8080` is being processed by `requestHandler` function, which results in sending 'Hi everybody!' back as a response.
Try requesting different url like `http://localhost:8080/somethingelse` -- response will not change. To make it act differently based on requested url, we would need to write more code. But it is possible without frameworks.

We will use **expressjs** framework for Node.js, which will help us implementing more features faster.

Let's write same http-server with express. First, install the npm package:

`npm install express`

Then replace content of `server/index.js` file with following code:

```js
// server/index.js
const express = require('express');

const app = express();

app.use(function requestHandler(req, res) {
    res.send('Hi everybody! Now it is expressjs server');
});

app.listen(8080);
```

> **Note** - express utilizes http package behind the scenes.

Run script `node server/index.js` to start http-server again.

Now we should see exactly same result with slightly different message: 'Hi everybody! Now it is expressjs server' in browser.

We can change logic of http-server to respond only to "index url" -- `http://localhost:8080/`, and generate 404 error for all other requests.

Replace

```js
app.use(function requestHandler(req, res) {
    res.send('Hi everybody! Now it is expressjs server');
});
```

with

```js
app.get('/', function requestHandler(req, res) {
    res.send('Hi everybody! Now it is expressjs server');
});
```

Restart server with `node server/index.js`.
Now visit two different urls: `http://localhost:8080/` and `http://localhost:8080/givemesomething`. Only index page will show us some response. All other requests will end up with 404 [HTTP status](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

> **Note** - in order to avoid restarting server every time we change source code we can use [nodemon](https://nodemon.io/) package. You can install it globally and use it instead of node executable to run script with node and restart it once source code changes.
> `npm install --global nodemon`
> Now you can use `nodemon server/index.js` to run and watch node script.

Now we can "teach" our server to handle different urls and [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) which together construct routes.

> _Routing_ refers to determining how an application responds to a client request to a particular endpoint, which is a URI (or path) and a specific HTTP request method (GET, POST, and so on). Each route can have one or more handler functions, which are executed when the route is matched.

Route definition takes the following structure:

```js
router.METHOD(PATH, HANDLER, [HANDLER2, [HANDLER3,...]])
```

> **Note** - app implements route functionality itself, that's why it is possible to create routes like this:
>
> ```js
> app.get('/data', function(req, res) {});
> ```

Routers can be nested. You can define router as a handler:

```js
const apiRouter = express().Router();

apiRouter.get('/', function(req, res) {
    res.send('triggered by GET /api/ path');
});

apiRouter.post('/add', function(req, res) {
    res.send('triggered by POST /api/add path');
});

app.use('/api', apiRouter);

app.get('/', function(req, res) {
    res.send('index page, triggered by GET /');
});
```

Code above will create 3 routes which can be triggered. Any other route will generate 404 error.

```
GET /api
POST /api/add
GET /
```

> **Note 1** - order is important. Routes defined first in code will trigger when match
> **Note 2** - once route handler was triggered, it will not proceed to another matching route handler, unless you use **next()** function. Next function is being provided as 3rd argument to handler functions.
>
> ```js
> app.get('*', function logGetRequests(req, res, next) {
>     // if this is first declared route - you will see the following message on every GET request
>     console.log('someone made a request with GET method');
>     next(); // this function will pass execution to next matching route handler
> });
> ```
>
> **Note 3** - to understand better how express works and order of execution of your handlers it's advised to read more about [writing](https://expressjs.com/en/guide/writing-middleware.html) and [using](https://expressjs.com/en/guide/using-middleware.html) middleware functions.

Let's recap how our `server/index.js` file content might look like:

```js
const express = require('express');

const app = express();

const apiRouter = express.Router();

app.get('*', function logGetRequests(req, res, next) {
    console.log('someone made a request with GET method');
    next();
});

apiRouter.get('/', function(req, res) {
    res.send('triggered by GET /api/ path');
});

apiRouter.post('/add', function(req, res) {
    res.send('triggered by POST /api/add path');
});

app.use('/api', apiRouter);

app.get('/', function(req, res) {
    res.send('index page, triggered by GET /');
});

app.listen(8080);
```

If you open `http://localhost:8080/` url in brower, you will see "index page, triggered by GET /" message in browser and `someone made a request with GET method` message in terminal. You can also access following routes:

-   `GET` `http://localhost:8080/api/`
-   `POST` `http://localhost:8080/api/add`

### Splitting app code

Although we can write all code in one file, but eventually it will become bigger and bigger.
Let's try to split it to make each piece to be responsible for specific part.

We will change folder structure into something like this:

```
server/
	api/
		index.js - api routes
	app.js - here we will setup our app with all the middlewares
	index.js - script we will run to start our back-end app
```

```js
// server/index.js
const app = require('./app');

app.listen(8080);
```

```js
// server/app.js
const express = require('express');

const apiRouter = require('./api');

const app = express();

app.get('*', function logGetRequests(req, res, next) {
    console.log('someone made a request with GET method');
    next();
});

app.get('/api', apiRouter);

app.get('/', function(req, res) {
    res.send('index page, triggered by GET /');
});

module.exports = app;
```

```js
// server/api/index.js
const apiRouter = require('express').Router();

apiRouter.get('/', function(req, res) {
    res.send('triggered by GET /api/ path');
});

apiRouter.post('/add', function(req, res) {
    res.send('triggered by POST /api/add path');
});

module.exports = apiRouter;
```

### Serving static assets

Express provides [special middleware](https://expressjs.com/en/starter/static-files.html) which helps to serve static assets requested by client.

```javascript
express.static(root, [options]);
```

We can use static middleware to allow our http-server to send content of any requested static file from `root` folder.
For example, if we create `public/index.html` and `public/style.css` files we will be able to request them from browser directly. Let's try it:

Add this middleware before any route in `server/app.js`

```javascript
// server/app.js
app.use(express.static(path.join(__dirname, 'public')));
```

> **Note** - path is another default nodejs package.

Create new `public/index.html` file

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Node App</title>
        <link rel="stylesheet" href="/style.css" />
    </head>
    <body>
        <h1>Home page</h1>
    </body>
</html>
```

Create new `public/style.css` file

```css
body {
    background: #f0f0f0;
    color: #444;
    font: normal 18px Arial, sans-serif;
}

h1 {
    color: red;
}
```

Now, accessing `http://localhost:8080/` will trigger static middleware to serve `public/index.html` (if index.html file is available it gets served automatically) file which will request `http://localhost:8080/style.css` (check network tab) and get content of
`public/style.css`.

Note that our `app.get('/', ...);` route is not triggered anymore. Reason for that is, if express.static could find a file by requested name, next() function will NOT be called. If there is no such file, next() function will be executed. You can try it by accessing not existing asset, like `http://localhost:8080/app.js` and changing `app.get('/', ...);` to `app.get('*', ...);` for example. You will see "index page, triggered by GET /" message in browser.

---

### (OPTIONAL) Lint and format js/jsx

[eslint](https://eslint.org/) - js code linter

> Code linting is a type of static analysis that is frequently used to
> find problematic patterns or code that doesn’t adhere to certain style
> guidelines.

[prettier](https://prettier.io/) - an opinionated code formatter

Eslint will help us to find and fix bad code patterns.
Prettier helps to focus on writing code, not styling it.

Both eslint and prettier have integration with most IDEs and code editors.

Run this command to install all packages needed for eslint and prettier setup.

```
npm istall --save-dev eslint eslint-config-airbnb-base eslint-config-prettier eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-prettier eslint-plugin-react prettier
```

Add the following content to **.eslintrc.json** file

```json
// .eslintrc.json
{
    "extends": [
        "airbnb-base",
        "plugin:react/recommended",
        "plugin:prettier/recommended"
    ],
    "rules": {
        "prettier/prettier": "error",
        "func-names": 0,
        "react/jsx-filename-extension": [
            1,
            {
                "extensions": [".js", ".jsx"]
            }
        ]
    },
    "env": {
        "browser": true,
        "node": true,
        "jasmine": true
    },
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        }
    }
}
```

and this code for **.prettierrc.json**

```json
// .prettierrc.json
{
    "trailingComma": "es5",
    "tabWidth": 4,
    "semi": true,
    "singleQuote": true
}
```

Next step would be to set up your code editor. You can download eslint and prettier plugins for it. Most likely those plugins will find and use config files we just created.

### Lint files automatically before commit.

To achieve this we can use [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks). **pre-commit** hook in particular.

Let's install following packages: **husky** and **lint-staged**

With **husky** we can specify commands we want to run on git hooks.
**lint-staged** helps to perform commands only on staged files (the ones that were staged with `git add` command.

```
npm install --save-dev husky lint-staged
```

To use husky, we can add following section in our **package.json** file:

```json
// package.json
"husky": {
    "hooks": {
        "pre-commit": "lint-staged"
    }
},
```

Similar for lint-staged:

```json
// package.json
"lint-staged": {
    "*.{js,jsx}": [
        "eslint",
        "git add"
    ]
},
```

Combination of these two sections will run eslint on all staged files, whenever we try to commit our changes.
If eslint finds errors, commit process will be aborted. In that case you would need to fix problems first and commit again.

This setup will allow you work in team with more confidence, have same code linting and formatting for every team member, and won't allow "bad code" to go to remote repo.
