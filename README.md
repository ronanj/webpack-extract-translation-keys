# Webpack Extract Translation Keys Plugin

Webpack provides an official plugin for managing translation using [i18n-webpack-plugin](https://github.com/webpack/i18n-webpack-plugin), but in only allows for build-time translations by replacing strings in the source code.

This plugin serves a similar purposes, but instead of replacing translation keys with actual string values it just collects translation keys allowing you to know exactly which translations are necessary for your client side.

Approach like this also allows to provide dynamically generated translation bundles to the client allowing you to get real-time updates to translation without regenerating whole client side bundle.

This plugin also compatible with Webpack 5.

## Usage

### Configuration

First you need to install plugin:

```bash
npm install --save-dev webpack-extract-translation-keys-plugin
```

And then include it in your configuration:

```javascript
// webpack.config.js

var ExtractTranslationKeysPlugin = require('webpack-extract-translation-keys-plugin');
module.exports = {
    plugins: [
        new ExtractTranslationKeysPlugin({
            functionName: '_TR_',
            output: path.join(__dirname, 'dist', 'translation-keys.json')
        })
    ]

    // rest of your configuration...
}
```

Now inside your module you can write something like this:

```js
console.log(_TR_('translation-key-1'));
console.log(_TR_('translation-key-2'));
```

If you run `webpack` now, you should get `dist/translation-keys.json` file with following content:

```json
{
    "translation-key-1": "translation-key-1",
    "translation-key-2": "translation-key-2"
}
```

It may seems like a waste to output a map with the keys and values being the same thing, the purpose is to keep the output format consistent with the times when the `mangle` option is enabled.

## Output

if output string contains [name], one output file will be created per entry key at corresponding output


```js
// ...
    plugins: [
        new ExtractTranslationKeysPlugin({
            output: path.join(__dirname, 'dist', '[name]/translation-keys.json')
        })
    ]
// ...
```

### Key Mangling

In some applications translation keys are quite long, so for the situations where you want to save some additional bytes in your application, you can enable mangling during the plugin initialization:

```js
// ...
    plugins: [
        new ExtractTranslationKeysPlugin({
            mangle: true,
            functionName: '_TR_',
            output: path.join(__dirname, 'dist', 'translation-keys.json')
        })
    ]
// ...
```

This setting changes the behavior of the plugin to replace the key name with a minimal ascii-readable string.

In order to be able to map back to the original translation key, the plugin outputs mapping object with keys being original keys and the values being the mangled ones:

```json
{ "translation-key-1": " ", "translation-key-2": "!" }
```

> It's recommended to only enable mangling for production builds, as it makes the debugging harder and also may break hot reloading, depending on your setup.

Starting from version 6.0, it is also possible use custom keys instead of the minimal ascii-readable string. You just need to define `mangle` as a function instead of `true`. For example, this code will use the sha-1 as the key:

```js
var crypto = require('crypto')

// ...
    plugins: [
        new ExtractTranslationKeysPlugin({
            mangle: function(string) {
                return crypto.createHash('sha1').update(string, 'binary').digest('hex')
            },
            functionName: '_TR_',
            output: path.join(__dirname, 'dist', 'translation-keys.json')
        })
    ]
// ...
```

### Runtime

Since this plugin doesn't replace function with something else it's up to you to provide function that will actually handle translation in the runtime. It can be a globally defined function or you can use `webpack.ProvidePlugin` inside your configuration:

```js
module.exports = {
    plugins: [
        new ExtractTranslationKeysPlugin({
            functionName: '__',
            output: path.join(__dirname, 'dist', 'translation-keys.json')
        }),
        new webpack.ProvidePlugin({
            '__': 'path/to/module/with/translation/function.js'
        })
    ]

    // rest of your configuration...
}
```

### Default options

* `functionName` : `__`
* `done` : `function (result) {}`
* `output` : false
* `mangle` : false

### Error handling

Plugin throw an error if you try to call the translation function without any arguments or with a non-string argument (e.g. a variable).

## Release Notes

### 6.0.0

Support for Webpack 5, if you are using Webpack 4, please install 5.x.x version of this plugin. This can be done by running:

```bash
npm install --save-dev webpack-extract-translation-keys-plugin@5
```

### 5.0.0

Support for multiple output for multiple entries with [name] inside the output string. If [name] is not present, only one output file will be created for all entries

### 4.0.0

Support for Webpack 4, if you are using Webpack 3, please install 3.x.x version of this plugin. This can be done by running:

```bash
npm install --save-dev webpack-extract-translation-keys-plugin@3
```

### 3.0.0

Support for Webpack 2, if you are using Webpack 1, please install 2.x.x version of this plugin. This can be done by running:

```bash
npm install --save-dev webpack-extract-translation-keys-plugin@2
```

### 2.0.0

Support for key mangling. The format of the output without mangling has changed from array to a map. If you want to have old behavior, you can implement it using `done` callback option.

## License

Copyright 2015 Dmitriy Kubyshkin

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
