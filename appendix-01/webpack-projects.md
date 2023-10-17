# The Ultimate Webpack

## React and Node

- `Dontenv` allows us to access our API keys stored in the .env file. We can then call upon those API keys using this syntax `process.env`.
- We need the `env` const for deployment on heroku as without it, our API keys would be returned as null - in turn breaking your website!
- For this configuration, we told webpack to create the dist folder in the backend directory and build bundle.js with `frontend/src/app.js`.
- In `module.exports`, within module rules, we've added all the standard loaders required for us to render a website with a bit of styling (babel, style, css, sass). However, you'll find with React you aren't able to add an image locally and then display it when supplying the image path. In order to render these images we need to tell webpack.config.js how to do so. We add both `file-loader` and `url-loader` to make this happen where it looks for files ending with `jpg`, `png` and `gif`.
- The devServer remains mostly the same however, in order to run our full stack app we need to add a `proxy` server. The proxy provides us the run both backend and frontend at the same time. More specifically, it allows us to run our app on the `port` specified (i.e. localhost: 8001) whilst running our backend API on a different `port` (i.e localhost: 8000). These ports can be any number but **must not** be the same.

```js
const path = require('path')
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const Dotenv = require('dotenv-webpack')
const env = process.env.NODE_ENV === 'production' ? (
  new webpack.EnvironmentPlugin({ ...process.env })
) : (
  new Dotenv()
)
module.exports = {
  entry: './frontend/src/app.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve('./backend/dist'),
    publicPath: '/'
  },
  module: {
    rules: [
      { test: /\.js$/, loader: 'babel-loader', exclude: /node_modules/ },
      { test: /\.css$/, loader: ['style-loader', 'css-loader'] },
      { test: /\.s(a|c)ss$/, loader: ['style-loader', 'css-loader', 'sass-loader'] },
      { test: /\.(jpg|png|gif)/, use: [{
        loader: 'url-loader'
      }] },
      { test: /\.woff2?$/, loader: 'file-loader' },
      { test: /\.(jpg|png|gif)$/, loader: 'file-loader' }
    ]
  },
  devServer: {
    contentBase: path.resolve('./frontend/src'),
    hot: true,
    open: true,
    port: 8001,
    watchContentBase: true,
    historyApiFallback: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        secure: false
      }
    }
  },
  plugins: [
    new Dotenv(),
    new webpack.HotModuleReplacementPlugin(),
    new HtmlWebpackPlugin({
      template: 'frontend/src/index.html',
      filename: 'index.html',
      inject: 'body'
    }),
    env
  ]
}

```

## Summary
- The config above should handle most cases for your frontend and backend however, you probably won't have to do this level of config when you move into your first job!