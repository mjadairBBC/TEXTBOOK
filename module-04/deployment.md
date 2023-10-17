# Deploying a Python / React app on Heroku

Heroku is a web hosting platform and deployment pipeline that uses git. It is free in the most part, but does have usage bands, so if your site is receiving a lot of traffic you may be charged.

## Setting up Heroku

1. First sign up to [Heroku](https://heroku.com)

  You should set your language to _Node.js_ and your position to _student_. You do not have to supply a company name.

1. You should now [download the Heroku toolbelt](https://devcenter.heroku.com/articles/heroku-cli#download-and-install), which is a command line tool for deployment. Follow the instructions for installation with _Homebrew_.

1. Once the CLI has installed, you need to log in to Heroku in the terminal:

  ```sh
  heroku login
  ```

  A browser window will open, allowing you to log in. Once you have logged in via the browser, you should see a success message **in the terminal**.

1. Navigate to the root of your project. This should be a git repo. If not, you need to initialise it with `git init`

1. Ceate a Heroku app with the following command:

  ```sh
  heroku create --region=eu project-name
  ```

  > **Note** replace `project-name` with the name of your project. This will become part of the website's URL.

1. We need to add two buildpacks to this project, one for Node.js and one for the Python app. We want to build our software with webpack using Node.js first, then launch the Flask app with Python second, so we need to add the build packs in that order:

  ```sh
  heroku buildpacks:add heroku/nodejs
  heroku buildpacks:add heroku/python
  ```

1. To check the buildpack run `heroku buildpacks`. You should see the following output:

  ```sh
  === project-name Buildpack URLs
  1. heroku/nodejs
  2. heroku/python
  ```

### Adding a database

If you are using a database for your app, you will also need to create a Postgres database instance on Heroku.

1. Add a credit/debit card to your Heroku account. _Don't worry, you will not be charged, unless you have large volumes of traffic to your site._

1. In the terminal create a PostgreSQL instance for your Heroku app:

  ```sh
  heroku addons:create heroku-postgresql:hobby-dev
  ```

1. Tell Heroku about all your `.env` variables. E.g.:
  ```sh
  heroku config:set FLASK_APP=backend/app.py
  ```

## Setting up your app for deployment

### Flask app

1. Add an `config/evironment.py` file:

  ```py
  import os

  db_uri = os.getenv('DATABASE_URL', 'postgres://localhost:5432/<YOUR-DATABASE-NAME>')
  secret = os.getenv('SECRET', 'a suitable secret')
  ```

1. Update the `app.py` file to use the now environment variables:

  ```py
  app.config['SQLALCHEMY_DATABASE_URI'] = db_uri
  app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
  ```

1. Set up a static folder in `app.py`:

  ```py
  app = Flask(__name__, static_folder='dist')
  ```

  > **Note** this assumes you are building your code to a `dist` folder

1. Add a `Procfile` at the root of your project, that will be used by Heroku to launch your Python app:

  ```sh
  web: flask run --port $PORT --host 0.0.0.0
  ```

1. Update your `app.py` (or `server.py`) to serve files from your static folder, or the `index.html` if no other endpoints are matched. Add this code at the bottom of the file (You'll also need to import `os` at the top, like you did in your environment config file):

  ```py
  ## registering your blueprints...

  @app.route('/', defaults={'path': ''}) # homepage
  @app.route('/<path:path>') # any other path
  def catch_all(path):
      dirname = os.path.dirname(__file__)
      filename = os.path.join(dirname, 'dist/' + path)

      if os.path.isfile(filename): # if path is a file, send it back
          return app.send_static_file(path)

      return app.send_static_file('index.html') # otherwise send back the index.html file
  ```

### React app

1. Add the following scripts to your `package.json`:

  ```json
  "scripts": {
    "build": "pipenv install; npm i; webpack -p; pipenv run python backend/seeds.py",
    "serve:react": "webpack-dev-server",
    "serve:flask": "pipenv run flask run"
  }
  ```

1. Add a `.gitignore` file in your project root, to ignore your `dist` folder:

  ```sh
  dist
  ```

1. Make sure your `webpack.config.js` is building dist to the right place...
  ```
  output: {
    filename: 'bundle.js',
    path: path.resolve('backend/dist')
  },
  ```


1. Test your app locally by running `npm run build`, then `npm run serve:flask`. Naviagte to `http://localhost:5000`. Fix any bugs or issues.

1. Commit your code

  ```sh
  git add . && git commit -m "Ready for deployment"
  ```

1. Deploy to Heroku

  ```sh
  git push heroku master
  ```

1. Once your code has finished deploying, open your app with `heroku open`

1. If you get an Application Error, check the logs on heroku with `heroku logs --tail`, and debug!
