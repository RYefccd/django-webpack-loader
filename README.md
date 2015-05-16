# django-webpack-loader

[![Join the chat at https://gitter.im/owais/django-webpack-loader](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/owais/django-webpack-loader?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<br>

**NOTE:** *This is very very new. Do not use in proudction yet.*

<br>

Use webpack to generate your static bundles without django's staticfiles or opaque wrappers. 


Django webpack loader consumes the output generated by [webpack-bundle-tracker](https://github.com/owais/webpack-bundle-tracker) and lets you use the generated bundles in django.


<br>

## Install

```bash
npm install --save-dev webpack-bundle-tracker

pip install django-webpack-loader
```

<br>
## Configuration

<br>
### Assumptions

Assuming `BASE_DIR` in settings refers to the root of your django app.
```python
import sys
import os

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
```

<br>
Assuming `assets/` is in `settings.STATICFILES_DIRS` like
```python
STATICFILES_DIRS = (
	os.path.join(BASE_DIR, 'assets'),
)
```

<br>
Assuming your webpack config lives at `./assets/webpack.config.js` and looks like this
```javascript
module.exports = {
  context: __dirname,
  entry: {
    app: './js/index'
    output: {
      path: require('path').resolve('./assets/bundles/'),
      filename: '[name]-[hash].js',
  },

  plugins: [
    new BundleTracker({filename: './assets/webpack-stats.json'})
  ]
}

```


<br>
### Default Configuration
```python
WEBPACK_LOADER = {
    'BASE_URL': 'webpack_bundles/',
    'STATS_FILE': 'webpack-stats.json',
    'POLL_DELAY': 0.2,
    'IGNORE': ['.+\.hot-update.js', '.+\.map']
}
```

<br>

#### WEBPACK_BUNDLE_URL
```python
WEBPACK_LOADER = {
	'BASE_URL': STATIC_URL + 'bundles/'
}
```

`BASE_URL` is used to build the complete public url to the bundle that is used in `<script>` or `<link>` tags.

If the bundle generates a file called `main-cf4b5fab6e00a404e0c7.js`, then with the above config, the `<script>` tag will look like this

```html
<script type="text/javascript" src="/assets/bundles/main-cf4b5fab6e00a404e0c7.js"/>
```

<br>

#### STATS_FILE
```python
WEBPACK_LOADER = {
	'STATS_FILE': os.path.join(BASE_DIR, 'assets/webpack-stats.json')
}
```

`STATS_FILE` is the filesystem path to the file generated by `webpack-bundle-tracker` plugin. If you initialize `webpack-bundle-tracker` plugin like this

```javascript
new BundleTracker({filename: './assets/webpack-stats.json'})
```

and your webpack config is located at `/home/src/assets/webpack.config.js`, then the value of `STATS_FILE` should be `/home/src/assets/webpack-stats.json`

<br>

#### IGNORE
`IGNORE` is a list of regular expressions. If a file generated by webpack matches one of the expressions, the file will not be included in the template.

<br>

#### POLL_INTERVAL

`POLL_INTERVAL` is the number of seconds webpack_loader should wait between polling the stats file. The stats file is polled every 200 miliseconds by default and any requests to are blocked while webpack compiles the bundles. You can reduce this if your bundles take shorter to compile.

**NOTE:** Stats file is not polled when in production (DEBUG=False).

<br>


## Usage
<br>

#### Templates

```HTML+Django
{% load render_bundle from webpack_loader %}

{{ render_bundle 'main' }}
```

`render_bundle` will render the proper `<script>` and `<link>` tags needed in your template.

<br>

## How to use in Production

**It is up to you**. There are a few ways to handle this. I like to have slightly separate configs for production and local. I tell git to ignore my local stats + bundle file but track the ones for production. Before pushing out newer version to production, I generate a new bundle using production config and commit the new stats file and bundle. I store the stats file and bundles in a directory that is added to the `STATICFILES_DIR`. This gives me integration with collectstatic for free. The generated bundles are automatically collected to the target directory and synched to S3.


`./assets/webpack_production.config.js`
```javascript
config = require('./webpack.config.js');

config.output.path = require('path').resolve('./assets/dist');

config.plugins = [
    new BundleTracker({filename: './assets/webpack-stats-prod.json'})
]

// override any other settings here like using Uglify or other things that make sense for production environments.

module.exports = config;
```

`settings.py`
```python
if not DEBUG:
	WEBPACK_LOADER['BASE_URL'] = STATIC_URL + '/dist'
	WEBPACK_LOADER.update({
		'BASE_URL': STATIC_URL + '/dist',
		'STATS_FILE': os.path.join(BASE_DIR, 'assets/webpack-stats-prod.json'
	})
```

<br><br>


You can also simply generate the bundles on the server before running collectstatic if that works for you. 


--------------------
<br>


Enjoy your webpack with django :)
