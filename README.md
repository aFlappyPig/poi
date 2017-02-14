# vbuild

[![NPM version](https://img.shields.io/npm/v/vbuild.svg?style=flat)](https://npmjs.com/package/vbuild) [![NPM downloads](https://img.shields.io/npm/dm/vbuild.svg?style=flat)](https://npmjs.com/package/vbuild) [![Build Status](https://img.shields.io/circleci/project/egoist/vbuild/master.svg?style=flat)](https://circleci.com/gh/egoist/vbuild) [![donate](https://img.shields.io/badge/$-donate-ff69b4.svg?maxAge=2592000&style=flat)](https://github.com/egoist/donate)

Install once, Build everywhere.

<!-- toc -->

- [Install](#install)
- [How to use](#how-to-use)
- [Config file](#config-file)
  * [Arguments](#arguments)
    + [options](#options)
    + [req](#req)
  * [Babel](#babel)
  * [PostCSS](#postcss)
  * [Code splitting](#code-splitting)
  * [Webpack](#webpack)
  * [Custom build process](#custom-build-process)
  * [Custom output filename](#custom-output-filename)
- [FAQ](#faq)
- [Contributing](#contributing)
- [Author](#author)

<!-- tocstop -->

## Install

It works with both Yarn(>=0.17) and npm(>=3):

```bash
yarn global add vbuild
# You can also install it locally
# yarn add vbuild --dev
```

## How to use

<details><summary>The simple way</summary>

```bash
vbuild init <folder>
```

Then follow the instructions in terminal.
</details>

Or manually, populate an entry file, let's say `index.js`:

```js
import Vue from 'vue'

new Vue({
  el: '#app',
  render: h => <h1>Hello World!</h1>
})
```

Run app in dev mode:

```bash
vbuild index.js --dev
```

So far we get:

- Automatic transpilation and bundling (with webpack and babel/postcss)
- Hot code reloading
- Static file in `./static/` will coppied to `./dist/`

To see how simple it is, check out [our official website](https://github.com/egoist/vbuild.js.org) which is built with vbuild itself.

Build app in production mode (default mode):

```bash
vbuild index.js
```

## Config file

All CLI options and advanced options can be set here:

```js
module.exports = (options, req) => ({
  port: 5000
  // Other options
})

// Note that you can directly export an object too:
// module.exports = {port: 5000}
```

To use it, you can add `--config [path]` in CLI arguments. If no path was speified, it defaults to `vbuild.config.js`.

### Arguments

#### options

CLI options.

#### req

The `require` function but context directory is the path to `node_modules/vbuild`, which means you can use it to load vbuild's dependencies, like `webpack`.

### Babel

JS files and `script` tags in single-file components are transpiled by Babel. We only use one preset by default: [babel-preset-vue-app](https://github.com/egoist/babel-preset-vue-app).

You can provide custom babel config by setting `babel` in config file or using `.babelrc` or setting `babel` field in `package.json`.

### PostCSS

Standalone `.css` files and `style` tags in single-file components are transpiled by PostCSS, the only plugin we add by default is `autoprefixer`, and you can use `autoprefixer` option in config file to adjust it, the default value is:

```js
{
  browsers: ['ie > 8', 'last 4 versions']
}
```

You can also set `postcss` option in config file, this way the `autoprefixer` plugin we added will be overriden.

If you want to use PostCSS config file like `postcss.config.js` or whatever [postcss-load-config](https://github.com/michael-ciniawsky/postcss-load-config) supports, please set `postcss` option in config file to `undefined` first.

### Webpack entry

Type: `string` `Array` `Object`

You can set webpack entry from CLI option or `entry` property in config file. If it's an array or string, we add it into `webpackConfig.entry.client` entry, otherwise it will totally override `webpackConfig.entry`

### Hot reloading

By default we add HMR client to `client` entry, you can change it by:

```js
module.exports = {
  hmrEntry: ['other-name']
}
```

### Code splitting

To split vendor code and app code, you can set vendor dependencies using `vendor` option:

```js

module.exports = {
  vendor: ['vue']
}
```

### Webpack

Mutate webpack config as you wish:

```js
module.exports = options => ({
  webpack(webpackConfig) {
    if (options.dev) {
      // Apply some changes to webpackConfig
    }
    return webpackConfig
  }
})
```

The value of `webpack` could also be a plain object, this way it will be merged into default webpack config using [webpack-merge](https://github.com/survivejs/webpack-merge).

### Custom HTML output

Type: `Object` `Array` `boolean`

[html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) options, use this option to customize `index.html` output, default value:

```js
module.exports = {
  html: {
    title: 'VBuild',
    template: path.join(__dirname, '../lib/template.html')
  }
}
```

Check out the [default template](/lib/template.html) file we use. To disable generating html file, you can set `html` to `false`.

### Custom output filename

Set custom filename for js css static files:

```js
module.exports = {
  filename: {
    js: 'index.js',
    css: 'style.css',
    static: 'static/[name].[ext]'  
  }
}
```

### Proxy API request

To tell the development server to serve any `/api/*` request to your API server in development, use the `proxy` options:

```js
module.exports = {
  proxy: 'http://localhost:8080/api'
}
```

This way, when you fetch `/api/todos` in your Vue app, the development server will proxy your request to `http://localhost:8080/api/todos`.

We use [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) under the hood, so the `proxy` option can also be an object:

```js
module.exports = {
  proxy: {
    '/api/foo': 'http://localhost:8080/api',
    '/api/fake-data': {
      target: 'http://jsonplaceholder.typicode.com',
      changeOrigin: true,
      pathRewrite: {
        '^/api/fake-data': ''
      }
    }
  }
}
```

Keep in mind that proxy only has effect in development.

### Dev server

#### port

Type: `number`<br>
Default: `4000`

Port of dev server.

#### host

Type: `string`<br>
Default: `localhost`

Host of dev server.

### Custom server logic

Perform some custom logic to development server:

```js
module.exports = {
  setup(app) {
    app.get('/api', (req, res) => {
      res.end('This is the API')
    })
  }
}
```

### Custom build process

Insead of letting vbuild run webpack as the build process, you can perform a custom one by using `run` function in config file:

```js
// For example, run tests with Karma
const Server = require('karma').Server

module.exports = {
  run(webpackConfig) {
    const server = new Server({
      webpack: webpackConfig,
      // ...Other karma options
    })
    server.start()
  }
}
```

## FAQ

<details><summary>Is it like Next.js or Nuxt.js?</summary>

Yes and no, yes is because they all simplified the process of building a complete web app, while `vbuild` is more focusing on building single-page app without the server-side, at least it is for now.
</details>

<details><summary>What is this inspired by?</summary>

Despiting that `vbuild` predates `Next.js` `create-react-app` `nwb` `vue-cli`, we're heavily inspired by these projects.
</details>

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D


## Author

**vbuild** © [EGOIST](https://github.com/egoist), Released under the [MIT](./LICENSE) License.<br>
Authored and maintained by egoist with help from contributors ([list](https://github.com/egoist/vbuild/contributors)).

> [egoistian.com](https://egoistian.com) · GitHub [@egoist](https://github.com/egoist) · Twitter [@rem_rin_rin](https://twitter.com/rem_rin_rin)
