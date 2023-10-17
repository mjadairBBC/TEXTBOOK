# Heroku and express

Heroku is a web hosting platform and deployment tool. It uses git to upload files and folders from your computer to a shared server in the cloud.

## Sign up to Heroku

If you haven't already, create an account with at https://www.heroku.com/​

1. Click on **SIGN UP** then fill our the registration form. Since you are creating a personal account you can comfortably leave Company name blank. Set Role to Student, and Primary development language to Node.js

1. Once you have created your account, click on the ninja icon in the top right hand corner and select Account settings

  <img src="https://media.git.generalassemb.ly/user/15120/files/92aaaa4e-89d3-11e8-859b-5155b9350851" style="max-width:300px">

1. Now select Billing from the tabs on the left hand side. Add a credit card.

  > **Note**: Using Heroku during the course will not cost you any money. However some features like databases if heavily used will incur running costs. In order to use these features we need to add a credit card in either case

1. Once you have added a card you need to install the Heroku toolbelt which adds heroku commands to the terminal. Type the following in the terminal:

  ```
  brew install heroku/brew/heroku
  ```

Once the installation has finished, type `heroku login` and enter the credentials that you used to create your Heroku account.

## Deployment

Before we deploy an Express app to Heroku we need to do some things:

1. **Merging to master**

	You've been working off the `development` branch until now. Now it's time to merge your work to the master branch. Make sure you have the latest from development first.
	
	**Have one team member:**
	
	```
	> git pull origin development
	> gco master
	> git merge development
	> git push origin master
	
	```
	**Have all team members who didn't create the original repo:**
	
	- Fork the repo to their personal github
	- `git clone` the repo to their macbook.
	- `cd` into that repo. 
	- `npm i`
	

2. **Have express serve our index.html and bundle.js**

	So far, we've been using `webpack-dev-server` to develop our front-ends, and proxying through our requests to express, for just our `/api` endpoints. But now it's time to give express the power to serve our react app.

	First, let's import some packages to help us with this in our `app.js`

	```
	const path = require('path')
	const dist = path.join(__dirname, 'dist');
	```

	Now let's add some 'catch-all' routes to handle requests for static files:

	```
	app.use('/', express.static(dist));

	app.get('*', function(req, res) {
	  res.sendFile(path.join(dist, 'index.html'));
	});
	
	```
	Note: this is now expecting us to have a `dist` folder wherever our backend `app.js` is located. So we need to make sure that webpack bundles our files into this location now...


3. **Tell webpack to bundle our files to where express needs them**

	If your frontend and backend are in separate directores, then you'll need to update webpack to have the following output destination (make sure its being built to wherever express expects it to be, based on your own project structure):
	
	```
	output: {
	    filename: 'bundle.js',
	    path: path.resolve('./backend/dist'),
	    publicPath: '/'
	},
	
	```
	
4. **Update config/environment.js file:**

  ```
  const port = process.env.PORT || 4000;
  const dbURI = process.env.MONGODB_URI || 'mongodb://localhost:27017/database-name'
  ```

  This will ensure that when deployed Heroku can set our port and database URI.


5. **Update the package.json:**

  Make sure you have _at least_ a `build` script and a `start` script in your package.json. Heroku will run these itself.

  ```json
  "scripts": {
    	"build": "npm run seed; webpack -p",
    	"start": "node backend/app.js"
  }
  ```
  In `build`, we've added seed first, so that your database gets seeded by heroku. It'll then use webpack to bundle your frontend.
  
  `start` should be to the entrypoint for your express backend (`app.js`)
  

6. **Create a Heroku app:**

  Type the following command in the terminal:
  ```
  heroku create --region=eu <project-name>
  ```

  `<project-name>` is the name of your project. If you leave it blank Heroku will create a random name for you.

7. **Add a database:**

  Type the following command in the terminal:
  ```
  heroku addons:create mongolab
  ```

8. **Add any environment variables from your `.zshrc` file, if needed:**

  For example, if you are using Dark Sky API, you would type the following command in your terminal:

  ```
  heroku config:set DARKSKY_API_KEY="b7a8aba2c0429ef0616a86a0ba2c64c2"
  ```
  
9. **Check that this all works locally:**

	Before deploying to Heroku, better to check that your setup works locally first. To do this, just run your express app with `npm run start`, and visit `localhost:PORT` in Chrome, where `PORT` is the one your app is running on. Test that your app works as expected here.


10. **Now you can deploy the app with the following command:**

  ```
  git push heroku master
  ```

Once the deployment is complete, open the site using:

```
heroku open
```

If you see a screen that says Application Error, you can check the logs with:

```
heroku logs
```

## Further reading

* [Deploying with Git - Heroku](https://devcenter.heroku.com/articles/git)