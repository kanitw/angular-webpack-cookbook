We talked about how you could use the minified versions of your dependencies in development to make the rebundling go as fast as possible. Let us look at a small helper you can implement to make this a bit easier to handle.

*webpack.config.js*
```javascript
var webpack = require('webpack');
var path = require('path');
var nodeModulesDir = path.join(__dirname, '../node_modules');

var deps = [
  'angular/angular.min.js',
  'moment/min/moment.min.js',
  'underscore/underscore-min.js',
];

var config = {
  entry: [
    'webpack/hot/dev-server',
    '../app/main.js'
  ],
  output: {
    path: path.resolve(__dirname, '../build'),
    filename: 'bundle.js'
  },
  resolve: {
    alias: {}
  },
  module: {
    noParse: [],
    loaders: []
  }
};

// Run through deps and extract the first part of the path, 
// as that is what you use to require the actual node modules 
// in your code. Then use the complete path to point to the correct
// file and make sure webpack does not try to parse it
deps.forEach(function (dep) {
  var depPath = path.resolve(node_modules_dir, dep);
  config.resolve.alias[dep.split(path.sep)[0]] = depPath;
  config.module.noParse.push(depPath);
});

module.exports = config;
```
Not all modules include a minified distributed version of the lib, but most do. Especially with large libraries like Angular JS you will get a significant improvement.

## Exposing Angular to the global scope
You might be using distributed versions that requires Angular JS on the global scope. To fix that you can install the expose-loader by `npm install expose-loader --save-dev` and set up the following config, focusing on the *module* property:

```javascript
var webpack = require('webpack');
var path = require('path');
var nodeModulesDir = path.join(__dirname, '../node_modules');

var deps = [
  'angular/angular.min.js',
  'moment/min/moment.min.js',
  'underscore/underscore-min.js',
];

var config = {
  entry: ['webpack/hot/dev-server', '../app/main.js'],
  output: {
    path: path.resolve(__dirname, '../build'),
    filename: 'bundle.js'
  },
  resolve: {
    alias: {}
  },
  module: {
    noParse: [],

    // Use the expose loader to expose the minified Angular JS
    // distribution.
    loaders: [{
      test: path.resolve(nodeModulesDir, deps[0]),
      loader: "expose?angular"
    }]
  }
};

deps.forEach(function (dep) {
  var depPath = path.resolve(nodeModulesDir, dep);
  config.resolve.alias[dep.split(path.sep)[0]] = depPath;
  config.module.noParse.push(depPath);
});

module.exports = config;
```