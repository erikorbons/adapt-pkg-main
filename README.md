## adapt-pkg-main

This module helps consume modules installed as package directories without needing to configure a module loader to load them.

The target are AMD and likely ES6 module projects that do not want to have a loader config entry for every dependency.

## Background

Any front-end (browser based) module loader that allows dynamic loading needs to know how to map a module ID to a file path. The basic convention that works without any configuration, besides perhaps configuring a baseUrl for path lookups, is `baseUrl + moduleId + '.js'`.

However for modules delivered in packages (like from package managers like npm and bower), they normally have an extra bit of knowledge that comes in to play: they normally have some sort of config JSON file (like package.json or bower.json), and they specify a "main" value, for the main module ID that should be used by outside code if they ask for the 'packageName' dependency.

For front end loaders, there are two ways to deal with communicating this information to the runtime loader:

1) **Config Rewriting**: The loader has a config API where you can tell it the main module and location for each dependency. If using bower and requirejs, [yeoman/bower-requirejs](https://github.com/yeoman/bower-requirejs) is an example of this type of approach.

Pros:

* It can avoid some HTTP requests as compared to Main Adapter Writing.

Cons:

* Leads to larger config blocks, some of which may not be needed on initial load.
* Requires modifying the app file that has the config in it.

2) **Main Adapter Writing**: When installing the `packageName` directory, also write out a `packageName.js` file as a sibling to the `packageName` directory, and just have it be a module that depends on the "main" module specified in the config file that was in `packageName`.

This module takes the Main Adapter Writing approach. Ideally package managers would allow this natively as an install option, particularly once looking forward to ES 6 modules.

In the meantime, you can use this module to do the work, as something you trigger after using your package manager, as a post install step.

You do not need to use this module for every browser load of your project, just whenever there is change to the installed dependencies for your project.

Pros:

* No more giant loader config blocks
* Distributes the "config" to when the modules are actually needed.

Cons:

* In a non-build scenario, an extra HTTP request to fetch the adapter module.

## Example

xxx


Mention when to run it here.


## API

This module can be used as require()'d module in another node module, or on the command line.

### Script API

```javascript
// Synchronously does the work,
//throws error if there is an error.
require('adapt-pkg-main')(dir, options);
```

**dir** is the directory to scan. This is likely the directory your package manager has installed dependencies, like `bower_components` or `node_modules`. It should contain a directories which are the actual packages of JS code that have been installed by a package manager.

Relative directories are relative to whatever the current working directory is when this module is called.

Only one level of directory scanning is done by this module. It does not try to recursively scan for other directories containing module packages, like nested `node_modules`.

For those nested cases, do the subdirectory scan outside of this module, and call this module with each directory (although that case likely entails setting up some other loader config, like the [requirejs map config](http://requirejs.org/docs/api.html#config-map) manually or using some other tool).

So if the directory structure is like this:

* my_packages
  * alpha
  * beta

Then pass `my_packages` as the dir value, and alpha and beta will be scanned for "main" package config, and a `my_packages/alpha.js` and `my_packages/beta.js` will be generated by this module.

The **options** object is optional, and can have the following properties:

**configFileNames**: An array of JSON-formated file names to check for a "main" property in each package. The default is `['package.json', 'bower.json']`. The order of the entries is the order this module uses to find a "main" entry.

**adapterText**: A string that is used for the module body of the adapter. Default is `define(['./{id}'], function(m) { return m; });`. The `{id}` part is replaced by this module with the name of the package's main module ID.

If wanting to write out the adapters in Node's module format, you can pass the string `module.exports = require('./{id}');`


### Command line API

x