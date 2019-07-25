---
layout: post
title:  "Angular 8 - Creating a Webpack Configuration from Scratch"
description: "Learn what it takes to write a custom Webpack configuration for Angular projects. This post will cover the configurations for both development and production."
image: assets/images/electron-builder.png
slug: angular-webpack
date:   2018-08-06
---

The Angular CLI has recently been significantly overhauled with v6, utilizing Webpack 4 for additional speed improvements and smaller bundle sizes. The CLI is perfect for a large majority of projects, and is only getting better, but there are occasions where you just need more freedom, or perhaps you just want to use some of Webpacks great loaders, plugins or features that aren't currently supported from the CLI workspace file.

In Angular CLI v1.x there was the option to eject, which generated a Webpack configuration that could be changed or extended to suit your needs, however this feature has been temporarily disabled. We also used to have a section on the Angular documentation site dedicated to creating a Webpack config for your applications, however this has since been removed also leaving you somewhat on your own to create it.

In this post I will demonstrate how to create a custom Webpack configuration for both development and production that will provide largely the same setup as the CLI does for you. We will also go over some of the fantastic, yet largely unknown Webpack plugins and loaders that the Angular team have developed that aren't necessarily limited to just an Angular application. The complete configurations can be found at the bottom of the post.


##### Development Configuration

Before we begin, I generated a simple project using the Angular CLI to create a familiar project structure and provide the same starting point for anyone following along, however we will not be using the CLI to build or serve our application. If your application is structured differently than a CLI project some paths may differ.

Let's start by installing all the dependencies we need to get started:

{% highlight bash %}
{% raw %}
npm install webpack webpack-cli webpack-dev-server raw-loader file-loader url-loader to-string-loader style-loader css-loader @angular-devkit/build-angular @ngtools/webpack copy-webpack-plugin @angular/compiler @angular/compiler-cli circular-dependency-plugin source-map-loader mini-css-extract-plugin @angular-devkit/build-optimizer uglifyjs-webpack-plugin --save-dev
{% endraw %}
{% endhighlight %}

Next we want to create a new file in the root of our project called `webpack.config.js`. This will contain our development configuration, in other words, this configuration will prioritize rebuild speed over bundle size.

Lets go through each part of the configuration and briefly explain its purpose:

###### **Mode**

This setting informs Webpack of the purpose of this configuration file. It will provide some sensible defaults for us automatically. The two main modes are `development` and `production`. Development mode will disable features like code minification, as this is slow and unnecessary when developing. If no mode is specified `production` will be used by default.

{% highlight javascript %}
{% raw %}
mode: 'development'
{% endraw %}
{% endhighlight %}

###### **Devtool**

This will define whether we want to produce sourcemaps or not, and also allow us to specify how our sourcemaps should be included, for example they may be inline or in separate `.js.map` files. Source maps are a way to help us debug code that is transformed during the build process. For example, we write TypeScript code, but this is compiled to JavaScript. A sourcemap would appear to let us debug the TypeScript code in the browser. The process of generating sourcemaps can be slow, so when running in development mode we want to use a less expensive sourcemap like `eval-source-map`, which will often not be as accurate as other types, but is usually enough to be useful most of the time.

{% highlight javascript %}
{% raw %}
devtool: 'eval-source-map'
{% endraw %}
{% endhighlight %}

###### **Entry**

This defines several files that Webpack can start building a dependency graph from and begin bundling. The three entry points we want to specify are the `main.ts` file, which is the starting point of our application code, the `polyfills.ts` file, which contains all the code that will ensure our application runs in older browsers (...cough... IE...) and finally the `styles.css` file, which is our global stylesheet.

{% highlight javascript %}
{% raw %}
entry: {
    main: './src/main.ts',
    polyfills: './src/polyfills.ts',
    styles: './src/styles.css'
}
{% endraw %}
{% endhighlight %}

###### **Output**

This option allows us to define what directory our application should be built to and also specify how we want out files no be named. In the case below `[name]` will be replaced with the corresponding entrypoint name:

{% highlight javascript %}
{% raw %}
output: {
    path: resolve('./dist'),
    filename: '[name].js',
}
{% endraw %}
{% endhighlight %}

###### **Resolve**

By default Webpack only knows about JavaScript files when looking at import statements, however we are writing our code using TypeScript so we need to tell Webpack our imports may be TypeScript files too. We also want to ensure our RxJS paths are all resolved correctly so we can add a helper function here too.

{% highlight javascript %}
{% raw %}
resolve: {
    extensions: ['.ts', '.js'],
    alias: rxPaths()
}
{% endraw %}
{% endhighlight %}

###### **Optimizations**

This section allows us to configure how our application should be optimized, for example, through minification and code splitting. In the example below we indicate we want a file containing the Webpack runtime code (the boilerplate that handles module loading) to prevent unneccesary duplication. In our production configuration we will also specify some plugins to minify our code here too, but we will omit that in our development configuration as it can make rebuilt times very slow.

In our development build we also want a vendors bundle. Vendor in ths case refers to any third party package, which is unlikely to change often, so we can increase rebuild time by having it in a separate chunk so it doesn't get processed when we make changes to our application source code.

{% highlight javascript %}
{% raw %}
optimization: {
  runtimeChunk: 'single',
  splitChunks: {
    maxAsyncRequests: Infinity,
    cacheGroups: {
      default: {
        chunks: 'async',
        minChunks: 2,
        priority: 10,
      },
      common: {
        name: 'common',
        chunks: 'async',
        minChunks: 2,
        enforce: true,
        priority: 5,
      },
      vendors: false,
      vendor: {
        name: 'vendor',
        chunks: 'initial',
        enforce: true,
        test: (module, chunks) => {
          const moduleName = module.nameForCondition ? module.nameForCondition() : '';
          return /[\\/]node_modules[\\/]/.test(moduleName) &&
            !chunks.some(({ name }) => name === 'polyfills' || 'styles.css' === name);
        },
      },
    }
  }
},
{% endraw %}
{% endhighlight %}

###### **Loaders**

We can define what should happen when Webpack encounters a specific file type. There are comments above each loader indicating what it does.

{% highlight javascript %}
{% raw %}
module: {
  rules: [
    // process the TypeScript files and detects any templates and stylesheets in Component decorators
    {
      test: /\.ts$/,
      use: '@ngtools/webpack'
    },
    // extract source maps
    {
      test: /\.js$/,
      exclude: /(ngfactory|ngstyle).js$/,
      enforce: 'pre',
      use: 'source-map-loader'
    },
    // Process component templates
    {
      test: /\.html$/,
      use: 'raw-loader'
    },
    // Process component stylesheets
    {
      test: /\.css$/,
      use: ['to-string-loader', 'css-loader'],
      exclude: [resolve('./src/styles.css')]
    },
    // Process the global stylesheet
    {
      test: /\.css$/,
      use: ['style-loader', 'css-loader'],
      include: [resolve('./src/styles.css')]
    },
    // Process file assets that are encountered and ensure the url is correct when moved to output folder
    {
      test: /\.(eot|svg|cur)$/,
      loader: 'file-loader',
      options: {
        name: `[name].[ext]`,
        limit: 10000
      }
    },
    // Inline small assets in base64 form - assets too large will be processed by the file-loader
    {
      test: /\.(jpg|png|webp|gif|otf|ttf|woff|woff2|ani)$/,
      loader: 'url-loader',
      options: {
        name: `[name].[ext]`,
        limit: 10000
      }
    },
    // This hides some deprecation warnings that Webpack throws
    {
      test: /[\/\\]@angular[\/\\]core[\/\\].+\.js$/,
      parser: { system: true },
    }
  ]
}
{% endraw %}
{% endhighlight %}

###### **Plugins**

Plugins allow use to perform additional actions that aren't limited to just a specific file like a loader. We can use the following plugins to enhance our build:

###### IndexHtmlWebpackPlugin

This is a plugin by the Angular team, similar to the `HtmlWebpackPlugin`. It allows us to produce an `index.html` file that has the appropriate scripts and stylesheets included automatically.

{% highlight javascript %}
{% raw %}
new IndexHtmlWebpackPlugin({
  input: './src/index.html',
  output: 'index.html',
  entrypoints: [
    'styles',
    'polyfills',
    'main'
  ]
}),
{% endraw %}
{% endhighlight %}

###### AngularCompilerPlugin

This plugin is used along with the `@ngtools/webpack` loader to compile our components using the Angular compiler, commonly known as Ahead of Time compilation. In development mode we want to run in JIT mode for faster rebuilds, so we set `skipCodeGeneration` to true. We can also state we want to name lazy loaded files which can be very useful when debugging.

{% highlight javascript %}
{% raw %}
new AngularCompilerPlugin({
  mainPath: resolve('./src/main.ts'),
  sourceMap: true,
  nameLazyFiles: true,
  tsConfigPath: resolve('./src/tsconfig.app.json'),
  skipCodeGeneration: true
}), 

new SuppressExtractedTextChunksWebpackPlugin()
{% endraw %}
{% endhighlight %}

###### ProgressPlugin

This plugin simply allows Webpack to report the progress of the build.

{% highlight javascript %}
{% raw %}
new ProgressPlugin()
{% endraw %}
{% endhighlight %}

###### CircularDependencyPlugin

Circular dependencies can often occur in large applications, and sometimes this can cause bugs. This plugin can help detect these circular dependencies automatically.

{% highlight javascript %}
{% raw %}
new CircularDependencyPlugin({ exclude: /[\\\/]node_modules[\\\/]/ })
{% endraw %}
{% endhighlight %}

###### CopyWebpackPlugin

There are some files we may want to include in our dist folder that would not be included by the `file-loader` or `url-loader` such as the favicon. We want to copy these files to our output directory and this plugin makes it easy to do so.

{% highlight javascript %}
{% raw %}
new CopyWebpackPlugin([
  {
    from: 'src/assets',
    to: 'assets'
  },
  {
    from: 'src/favicon.ico'
  }
])
{% endraw %}
{% endhighlight %}

###### Building our Application

Now that we have our configuration file complete we can build our application by running:

{% highlight javascript %}
{% raw %}
webpack
{% endraw %}
{% endhighlight %}

Additionally we can run a local development serve that will rebuild automatically when we make any changes. To do this we can use the webpack-dev-server:

{% highlight javascript %}
{% raw %}
webpack-dev-server
{% endraw %}
{% endhighlight %}

_Note: You must have webpack, webpack-cli and webpack-dev-server installed globally to run these directly from the command line._

##### Production Configuration

In production mode we want to make a few optimizations. We want to minify our JavaScript code and our stylesheets to reduce the amount of code a user has to download. We also want to move our global stylesheet to a CSS file, rather than using the `style-loader` which simply places the code in style tags.

We want to begin by creating a separate Webpack config file, for example `webpack.production.config.js`. To run Webpack with this configuration we can do:

{% highlight javascript %}
{% raw %}
webpack --config webpack.production.config.js
{% endraw %}
{% endhighlight %}

###### **Mode**

We want to ensure we set the mode to `production`.

{% highlight javascript %}
{% raw %}
mode: 'production'
{% endraw %}
{% endhighlight %}

###### **Devtool**

We want to produce a more accurate sourcemap as part of our production build. This is slower but is a significant help when trying to debug minified code.

{% highlight javascript %}
{% raw %}
devtool: 'source-map'
{% endraw %}
{% endhighlight %}

###### **Loaders**

We want to make a few changes to the list of loaders for further optimizations.

We want to ensure we use the Angular Build Optimizer loader to reduce our bundle size quite significantly.

{% highlight javascript %}
{% raw %}
{
  test: /\.js$/,
  loader: '@angular-devkit/build-optimizer/webpack-loader',
  options: { sourceMap: true }
},
{% endraw %}
{% endhighlight %}

We also no longer want to use the `style-loader` and instead want to extract it to a separate global stylesheet. We can do this using the `MiniCssExtractPlugin`.

{% highlight javascript %}
{% raw %}
{
  test: /\.css$/,
  use: [MiniCssExtractPlugin.loader, 'css-loader'],
  include: [resolve('./src/styles.css')]
},
{% endraw %}
{% endhighlight %}

###### **Optimizations**

We will make some changes to our production build that result in smaller bundle sizes, largely by adding plugins to minify our code.

The `HashedModuleIdsPlugin` will produce hashes of our files and use this as the file names. This will result in files getting different names when the contents change preventing cache issues.

The `UglifyJSPlugin` will minify our JavaScript code. We want it to run on multiple processes for faster build times. We will also enable caching to speed up build times also, however it will increase memory usage.

The `CleanCssWebpackPlugin` is another plugin by the Angular team that will allow us to easily minify our stylesheets.

{% highlight javascript %}
{% raw %}
optimization: {
  noEmitOnErrors: true,
  runtimeChunk: 'single',
  splitChunks: {
    cacheGroups: {
      default: {
        chunks: 'async',
        minChunks: 2,
        priority: 10
      },
      common: {
        name: 'common',
        chunks: 'async',
        minChunks: 2,
        enforce: true,
        priority: 5
      },
      vendors: false,
      vendor: false
    }
  },
  minimizer: [
    new HashedModuleIdsPlugin(),
    new UglifyJSPlugin({
      sourceMap: true,
      cache: true,
      parallel: true,
      uglifyOptions: {
        safari10: true,
        output: {
          ascii_only: true,
          comments: false,
          webkit: true,
        },
        compress: {
          pure_getters: true,
          passes: 3,
          inline: 3,
        }
      }
    }),
    new CleanCssWebpackPlugin({
      sourceMap: true,
      test: (file) => /\.(?:css)$/.test(file),
    })
  ]
}
{% endraw %}
{% endhighlight %}

###### **Plugins**

We will make a few changes to our plugins section in our production config.

###### AngularCompilerPlugin

We want to alter this plugin's options to ensure we enable Ahead of Time compilation, turn of named lazy files and replace the environment file with the production version.

{% highlight javascript %}
{% raw %}
new AngularCompilerPlugin({
  mainPath: resolve('./src/main.ts'),
  sourceMap: true,
  nameLazyFiles: false,
  tsConfigPath: resolve('./src/tsconfig.app.json'),
  skipCodeGeneration: false,
  hostReplacementPaths: {
    [resolve('src/environments/environment.ts')]: resolve('src/environments/environment.prod.ts')
  }
})
{% endraw %}
{% endhighlight %}

###### MiniCssExtractPlugin

This plugin will simply produce a global stylesheet for us:

{% highlight javascript %}
{% raw %}
new MiniCssExtractPlugin({ filename: '[name].css' })
{% endraw %}
{% endhighlight %}

###### SuppressExtractedTextChunksWebpackPlugin

This is another useful plugin by the Angular team that allows us to remove any entrypoints that produce CSS files. Webpack by default would produce a `styles.js` file if we have a `styles.css` entrypoint and this file would be completely unnecessary.

##### Additional Plugins

There are a few additional plugins created by the Angular team which are worth mentioning, however I have not included them in these configuration files as they would not necessarily be required by every application.

*   **ScriptsWebpackPlugin** - This allows you to easily include global scripts in your application. This is great if your application utilizes a package like jQuery or Bootstrap.js that can often be a little bit tricky to include using import statements.
*   **BundleBudgetPlugin** - This allows you to specify some bundle size limits. This can either warn you when a bundle exceed the specified limit or will emit an error when the limit is exceeded. This is a great way to ensure the bundle size stays within an acceptable limit to give your users the best possible experience.

The complete range of plugins created by the Angular team can be [found here](https://github.com/angular/angular-cli/tree/master/packages/angular_devkit/build_angular/src/angular-cli-files/plugins).

##### Final Configurations

The complete configurations can be found here:

###### Development

{% highlight javascript %}
{% raw %}
const { resolve } = require('path');
const rxPaths = require('rxjs/_esm5/path-mapping');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const ProgressPlugin = require('webpack/lib/ProgressPlugin');
const CircularDependencyPlugin = require('circular-dependency-plugin');
const { AngularCompilerPlugin } = require('@ngtools/webpack');
const { IndexHtmlWebpackPlugin } = require('@angular-devkit/build-angular/src/angular-cli-files/plugins/index-html-webpack-plugin');

module.exports = {

  mode: 'development',

  devtool: 'eval',

  entry: {
    main: './src/main.ts',
    polyfills: './src/polyfills.ts',
    styles: './src/styles.css'
  },

  output: {
    path: resolve('./dist'),
    filename: '[name].js',
  },

  resolve: {
    extensions: ['.ts', '.js'],
    alias: rxPaths()
  },

  node: false,

  performance: {
    hints: false,
  },

  module: {
    rules: [
      {
        test: /\.ts$/,
        use: '@ngtools/webpack'
      },
      {
        test: /\.js$/,
        exclude: /(ngfactory|ngstyle).js$/,
        enforce: 'pre',
        use: 'source-map-loader'
      },
      {
        test: /\.html$/,
        use: 'raw-loader'
      },
      {
        test: /\.css$/,
        use: ['to-string-loader', 'css-loader'],
        exclude: [resolve('./src/styles.css')]
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
        include: [resolve('./src/styles.css')]
      },
      {
        test: /\.(eot|svg|cur)$/,
        loader: 'file-loader',
        options: {
          name: `[name].[ext]`,
          limit: 10000
        }
      },
      {
        test: /\.(jpg|png|webp|gif|otf|ttf|woff|woff2|ani)$/,
        loader: 'url-loader',
        options: {
          name: `[name].[ext]`,
          limit: 10000
        }
      },

      // This hides some deprecation warnings that Webpack throws
      {
        test: /[\/\\]@angular[\/\\]core[\/\\].+\.js$/,
        parser: { system: true },
      }
    ]
  },

  plugins: [
    new IndexHtmlWebpackPlugin({
      input: './src/index.html',
      output: 'index.html',
      entrypoints: [
        'styles',
        'polyfills',
        'main'
      ]
    }),

    new AngularCompilerPlugin({
      mainPath: resolve('./src/main.ts'),
      sourceMap: true,
      nameLazyFiles: true,
      tsConfigPath: resolve('./src/tsconfig.app.json'),
      skipCodeGeneration: true
    }),

    new ProgressPlugin(),

    new CircularDependencyPlugin({
      exclude: /[\\\/]node_modules[\\\/]/
    }),

    new CopyWebpackPlugin([
      {
        from: 'src/assets',
        to: 'assets'
      },
      {
        from: 'src/favicon.ico'
      }
    ])
  ]
};
{% endraw %}
{% endhighlight%}



###### Production

{% highlight javascript %}
{% raw %}
const { resolve } = require('path');
const rxPaths = require('rxjs/_esm5/path-mapping');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const ProgressPlugin = require('webpack/lib/ProgressPlugin');
const CircularDependencyPlugin = require('circular-dependency-plugin');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
const { CleanCssWebpackPlugin } = require('@angular-devkit/build-angular/src/angular-cli-files/plugins/cleancss-webpack-plugin');
const { AngularCompilerPlugin } = require('@ngtools/webpack');
const { IndexHtmlWebpackPlugin } = require('@angular-devkit/build-angular/src/angular-cli-files/plugins/index-html-webpack-plugin');
const { SuppressExtractedTextChunksWebpackPlugin } = require('@angular-devkit/build-angular/src/angular-cli-files/plugins/suppress-entry-chunks-webpack-plugin');
const { HashedModuleIdsPlugin } = require('webpack');

module.exports = {

  mode: 'production',

  devtool: 'source-map',

  entry: {
    main: './src/main.ts',
    polyfills: './src/polyfills.ts',
    styles: './src/styles.css'
  },

  output: {
    path: resolve('./dist'),
    filename: '[name].js',
  },

  resolve: {
    extensions: ['.ts', '.js'],
    alias: rxPaths()
  },

  node: false,

  performance: {
    hints: false,
  },

  module: {
    rules: [
      {
        test: /\.ts$/,
        use: '@ngtools/webpack'
      },
      {
        test: /\.js$/,
        loader: '@angular-devkit/build-optimizer/webpack-loader',
        options: { sourceMap: true }
      },
      {
        test: /\.js$/,
        exclude: /(ngfactory|ngstyle).js$/,
        enforce: 'pre',
        use: 'source-map-loader'
      },
      {
        test: /\.html$/,
        use: 'raw-loader'
      },
      {
        test: /\.css$/,
        use: ['to-string-loader', 'css-loader'],
        exclude: [resolve('./src/styles.css')]
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
        include: [resolve('./src/styles.css')]
      },
      {
        test: /\.(eot|svg|cur)$/,
        loader: 'file-loader',
        options: {
          name: `[name].[ext]`,
          limit: 10000
        }
      },
      {
        test: /\.(jpg|png|webp|gif|otf|ttf|woff|woff2|ani)$/,
        loader: 'url-loader',
        options: {
          name: `[name].[ext]`,
          limit: 10000
        }
      },

      // This hides some deprecation warnings that Webpack throws
      {
        test: /[\/\\]@angular[\/\\]core[\/\\].+\.js$/,
        parser: { system: true },
      }
    ]
  },

  optimization: {
    noEmitOnErrors: true,
    runtimeChunk: 'single',
    splitChunks: {
      cacheGroups: {
        default: {
          chunks: 'async',
          minChunks: 2,
          priority: 10
        },
        common: {
          name: 'common',
          chunks: 'async',
          minChunks: 2,
          enforce: true,
          priority: 5
        },
        vendors: false,
        vendor: false
      }
    },
    minimizer: [
      new HashedModuleIdsPlugin(),
      new UglifyJSPlugin({
        sourceMap: true,
        cache: true,
        parallel: true,
        uglifyOptions: {
          safari10: true,
          output: {
            ascii_only: true,
            comments: false,
            webkit: true,
          },
          compress: {
            pure_getters: true,
            passes: 3,
            inline: 3,
          }
        }
      }),
      new CleanCssWebpackPlugin({
        sourceMap: true,
        test: (file) => /\.(?:css)$/.test(file),
      })
    ]
  },

  plugins: [
    new IndexHtmlWebpackPlugin({
      input: resolve('./src/index.html'),
      output: 'index.html',
      entrypoints: [
        'styles',
        'polyfills',
        'main',
      ]
    }),

    new AngularCompilerPlugin({
      mainPath: resolve('./src/main.ts'),
      sourceMap: true,
      nameLazyFiles: false,
      tsConfigPath: resolve('./src/tsconfig.app.json'),
      skipCodeGeneration: false,
      hostReplacementPaths: {
        [resolve('src/environments/environment.ts')]: resolve('src/environments/environment.prod.ts')
      }
    }),

    new MiniCssExtractPlugin({ filename: '[name].css' }),

    new SuppressExtractedTextChunksWebpackPlugin(),

    new ProgressPlugin(),

    new CircularDependencyPlugin({
      exclude: /[\\\/]node_modules[\\\/]/
    }),

    new CopyWebpackPlugin([
      {
        from: 'src/assets',
        to: 'assets'
      },
      {
        from: 'src/favicon.ico'
      }
    ])
  ]
};
{% endraw %}
{% endhighlight%}