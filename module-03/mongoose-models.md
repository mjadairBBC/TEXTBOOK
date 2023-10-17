# Mongoose Models

Rather than using MongoDB directly in the command line, we need a way to interface with it from our Express app. To do this we will use an _Object-relational Mapping_ application or \(ORM\). Although this sounds complex it is simply a piece of software which interacts with the database on our behalf.

There are ORMs for all sorts of databases and programming languages, but for a node app interacting with a mongo database, the _de facto_ choice is Mongoose.

We can use Mongoose to create models which perform two fuctions:

1. They perform CRUD actions on the relevant collection in our database
2. They validate the data that we send to the database

## Creating a model

Before we create a model, we need to establish a _schema_. A schema is simply a design for the data we want store. Think _class_ or _constructor function_.

Let's take cheese as an example. We want to create a collection of cheese records in our database. Each record needs to have specific properties, and each of those properties should have a specific data type.

A cheese model might look something like this:

```js
const mongoose = require('mongoose');

const schema = new mongoose.Schema({
  name: { type: String, required: true },
  origin: { type: String, required: true },
  strength: { type: Number, required: true }
});

module.exports = mongoose.model('Cheese', schema);
```

So here we are saying that when we create a cheese record, it should have:

* a `name` which must be a string, and must be present
* a `origin` which must be a string, and must be present
* a `strength` must be a number, and must be present

Anything else will be removed by mongoose, to ensure that our data is consistent.

## Using a model

Once we've created a model, we can use it to perform CRUD actions on our `cheeses` collection. The mongoose model will interact with the database for us, running the correct Mongo commands. Whenever we use Mongoose models we need to pass a callback function which will be called _once the database interaction has been completed_.

It's essential that we connect Mongoose to the database in our `index.js` file, _before_ we try to perform any CRUD actions. When we connect we pass a special URL which will connect us to our local MongoDB instance. The final part of the URL specifies the database we are connecting to.

Inside `index.js`:

```js
const router = require('router');
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/cheese-app'); // in this case cheese-app would be the name of the database

// other middlewares, loggers, body-parser etc 

// router
app.use(router);
app.listen(port, (err) => console.log(`Up and running on port ${port}`));
```

### Create

You can create a new record like so:

```js
Cheese
  .create({ name: 'Gorgonzola', origin: 'Italy', strength: 5 })
  .then(cheese => console.log(cheese))
  .catch(err => console.log(err))
```

### Read

```js
Cheese  // get all the cheeses
	.find() 
	.then(cheeses => console.log(cheeses))
	
Cheese	// get all the cheeses that have a strength of 4
	.find({ strength: 4 })
	.then(cheeses => console.log(cheeses)

Cheese // get the cheese with a specific ID
	.findById('5a82d0ba2f65c8c6cb4d77f1')
	.then(cheese => console.log(cheese)) 
```

### Update

Again there are several ways to update a record with Mongoose.

The preferred method is the two stage process of finding a record, updating it, then saving it again.

```js
// This is the preferred method
Cheese
	.findById('5a82d0ba2f65c8c6cb4d77f1') // find the cheese to edit
	.then(cheese => {
	    cheese.name = 'Cheddar' // update the cheese's properties
	    return cheese.save() // save the cheese
	})
});
```

### Delete

Similar to update. Again the preferred method is to find a record, then delete it.

```js
// This is the preferred method
Cheese
	.findById('5a82d0ba2f65c8c6cb4d77f1') // find the cheese to edit
	.then(cheese =>  cheese.remove()) // save the cheese
});
```

## Further reading

* [Easily Develop Node.js and MongoDB Apps with Mongoose](https://scotch.io/tutorials/using-mongoosejs-in-node-js-and-mongodb-applications)
* [Mongoose CRUD](https://coursework.vschool.io/mongoose-crud/)
* [Mongoose API Docs](http://mongoosejs.com/docs/api.html)
