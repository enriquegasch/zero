<p align="center">
  <h1 align="center">Zero Server</h1>
  <p align="center">Zero configuration web application framework.</p>
</p>


<p align="center">
  <a href="#features">Features</a> |
  <a href="#installation">Installation</a> |
  <a href="#getting-started">Getting Started</a> |
  <a href="examples/">Examples</a> |
  <a href="docs/">Docs</a>
</p>

---

**Zero** is a web framework. It allows you to write your code in a mix of [Node.js](docs/nodejs/README.md), [React](docs/react/README.md), [HTML](docs/html/README.md), [MDX](docs/mdx/README.md), and [static files](docs/static/README.md). Zero will serve them all and requires no configuration. Zero makes it easier to get started.

*An example project with different types of pages, all in one folder:*

![A basic mono-repo](docs/images/header.gif?raw=true "A basic mono-repo")

## Features
**No Configuration Required**: Your project folder doesn't need any config files. You just place your code and it's automatically compiled, bundled and served.

**File-system Routing**: If your code resides in `./api/login.js` it's exposed at `http://<SERVER>/api/login`. Inspired by good ol' PHP days.

**Automatically Resolves Dependencies**: If a file does `require('underscore')`, it is automatically installed and resolved. You can always create your own `package.json` file to install a specific version of a package.

**Multiple Languages**: Zero is designed to support code written in many languages all under a single project. Imagine this:
1) Exposing your Tensorflow model as a python API.
2) Using React pages to consume it. 
3) Writing the user login code in Node.js. 
4) Your landing pages in a mix of HTML or Markdown/MDX.

All under a single project folder as a single web application.

**Improved Error Handling**: Each endpoints run in their own process. So if `/api/login` crashes for some reason, it doesn't affect `/chatroom` page or `/api/chat` API. Crashed endpoints are restarted automatically when the next user visits them.

## Installation

You can install `zero` globally by:

```sh
npm install -g zeroserver
```

## Getting Started

Let's start by making a website that tells us server time.

First we need to create an API endpoint in Node.js to tell us time in JSON.

Create a new folder and add a new file `time.js` in that folder. In this file, export a function that accepts `Request` and `Response` objects ([like Express](https://expressjs.com/en/4x/api.html#res)):

```js
// time.js
const moment = require("moment")

module.exports = (req, res) => {
  var time = moment().format('LT');   // 11:51 AM
  res.send({time: time })
}

```

Once saved, you can `cd` into that folder and start the server like this:

```sh
zero
```

Running this command will automatically install any dependencies (like *momentjs* here) and start the web server.

Open this URL in the browser: [`http://localhost:3000/time`](http://localhost:3000/time)

You just created an API endpoint 🎉:

![Time API](docs/images/timeapi.png?raw=true "Time API")

Keep the server running. Now let's consume our API from a React page, create a new file `index.jsx` and add the following code:

```jsx
// index.jsx
export default class extends React.Component {
  static async getInitialProps(){
    var json = await fetch("/time").then((resp) => resp.json())
    return {time: json.time}
  }

  render() {
    return <p>Current time is: {this.props.time}</p>
  }
}
```

This is a standard React component. With one additional hook for initial data population:

`getInitialProps` is an `async` static method which is called by `zero` when the page loads. This method can return a plain object which populates `props`.

Now go to this URL: `http://localhost:3000/` and you should see the current server time rendered by React while `fetch`-ing an API endpoint you created earlier:

![Time In React](docs/images/timejsx.png?raw=true "Time In React")

`zero` automatically bundles your code and supports server-side rendering. You don't need to fiddle with webpack anymore.

That's it! You just created a web application.

## Routing
### File-system Routing
Zero serves routes based on file structure. If you write a function that resides in `./api/login.js` it's exposed at `http://<SERVER>/api/login`. Similarly if you put a React page under `./about.jsx` it will be served at `http://<SERVER>/about`

### Route Rewrites
Sometimes you would want to change `/user?id=luke` to `/user/luke`. To cater this type of routes, **all requests sent to a route that doesn't exist are passed on to the closest parent function**. 

So if you visit `/user/luke` and there is no `./user/luke.js` but there is `./user.js`. Zero will send the request to `/user` and set `req.params` to `['luke']`. Code for this:

```js
// user.js
module.exports = function(req, res) {
  console.log(req.params) // ['luke'] when user visits /user/luke
  res.send({params: req.params})
}
```

Another example: if you visit `/user/luke/messages`. Zero will also forward this to `./user.js` and set `req.params` to `['luke', 'messages']`

## Supported Languages
- [Node.js](docs/nodejs/README.md)
- [React](docs/react/README.md)
- [HTML](docs/html/README.md)
- [Markdown / MDX](docs/mdx/README.md)
- Python *(soon)*

## Running on Cloud
A zero app is a regular Node.js server. But zero doesn't create `package.json` in your project folder. For most clouds (Heroku, EC2, etc) you probably need `package.json`. You can create one similar to this:

```json
{
  "name": "my-zero-app",
  "scripts": {
    "start": "NODE_ENV=production zero"
  },
  "dependencies": {
    "zeroserver": "*"
  }
}
```

- We add dependency `zeroserver`, so the cloud builder can install `zero` on your server.
- Add a `"start"` command and also set `NODE_ENV` to `production` so zero generates minified builds.

After this, you can follow the instructions from your cloud provider for deploying a Node.js app.

## Contributing
*TODO*

## License

Zero is [Apache-2.0 licensed](LICENSE.md).