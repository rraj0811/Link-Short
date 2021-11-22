# Introduction to the project
    We will be using the following technologies:

Mongoose as the ORM
=>MongoDB as the backend database
=>Node.js as the backend
=>A simple embedded JS file as the frontend

We will complete this project in 7 steps

Part 1: Setting up the Express server

First let's set up our Node server. We'll use Express as the framework for this part as it is easy to work with.

The solution could look like this:

***************************************************************
// Initialize express server on PORT 1337
const express = require('express')
const app = express()

app.get('/', (req, res) => {
 res.send('Hello World! - from codedamn')
})

app.get('/short', (req, res) => {
 res.send('Hello from short')
})

app.listen(process.env.PUBLIC_PORT, () => {
 console.log('Server started')
})
***********************************************************************************************************

Simple and easy. We create another GET route using app.get, and it should get the job done.



Part 2: Setting up our view engine

Now that we're familiar with Express installation, let's take a look at the .ejs template we have. 

The EJS engine allows you to pass variables down with the Node.js code to your HTML and iterate or display them before you send an actual response to the server.

Take a quick look at the views/index.ejs file. It'll look similar to how a regular HTML file looks, except that you can use variables.

Here's our current index.js file:


Now, you can see that in the index.js file we have the line app.set('view engine', 'ejs') . It tells Express to use ejs as its default templating engine.

Finally, see that we are using res.render and only passing the file's name, not the full path. This is because Express will automatically look inside the views folder 
for available .ejs templates.

We pass variables as the second argument, which we can then access in the EJS file. We'll use this file later, but for now let's go through a quick challenge.

To pass this challenge, view the index.ejs file first and then update your name to anything else you like. Here's a good solution:

************************************************************************************************************************************************

const express = require('express')
const app = express()

app.set('view engine', 'ejs')

app.get('/', (req, res) => {
 res.render('index', { myVariable: 'My name is Sharath!' })
})

app.listen(process.env.PUBLIC_PORT, () => {
 console.log('Server started')
})
***********************************************************************************************************************************************

Part 3: Setting up MongoDB
Now that we have a bit of frontend and backend understanding, let's go ahead and setup MongoDB.

We'll use Mongoose for connecting to MongoDB. Mongoose is an ORM for MongoDB.

Simply speaking, MongoDB is a very loose database, and it allows all sorts of operations on anything.

While it is good for unstructured data, most of the time we are actually aware of what the data will be (like user records or payment records). Thus, we can define a 
schema for MongoDB using Mongoose. This makes a lot of functions easy for us.

For example, once we have a schema, we can be assured that data validation and any necessary checks will be handled by Mongoose automatically. Mongoose also gives us a 
bunch of helper functions to make our lives easier. Let’s now set it up.

To complete this part, we have to take care of the following points:

Mongoose NPM package has already been installed for you. You can directly require it.
Connect to the mongodb://localhost:27017/codedamn URL using the mongoose.connect method.
Here's our current index.js file:

*************************************************************************************************************************************************
const express = require('express')
const app = express()
const mongoose = require('mongoose')

app.set('view engine', 'ejs')

app.get('/', (req, res) => {
 res.render('index')
})

app.post('/short', (req, res) => {
 const db = mongoose.connection.db
 // insert the record in 'test' collection

 res.json({ ok: 1 })
})

// Setup your mongodb connection here
// mongoose.connect(...)

// Wait for mongodb connection before server starts
app.listen(process.env.PUBLIC_PORT, () => {
 console.log('Server started')
})
***********************************************************************************************************

Let's fill in the appropriate placeholders with the relevant code:

*********************************************************************************************************************
const express = require('express')
const app = express()
const mongoose = require('mongoose')

app.set('view engine', 'ejs')

app.get('/', (req, res) => {
 res.render('index')
})

app.post('/short', (req, res) => {
 const db = mongoose.connection.db
 // insert the record in 'test' collection
 db.collection('test').insertOne({ testCompleted: 1 })

 res.json({ ok: 1 })
})

// Setup your mongodb connection here
mongoose.connect('mongodb://localhost/codedamn', {
 useNewUrlParser: true,
 useUnifiedTopology: true
})
mongoose.connection.on('open', () => {
 // Wait for mongodb connection before server starts
 app.listen(process.env.PUBLIC_PORT, () => {
  console.log('Server started')
 })
})
*****************************************************************************************************************************************************************

Notice how we start our HTTP server only when our connection with MongoDB is open. This is fine because we don't want users to hit our routes before our database is 
ready.

We finally use the db.collection method here to insert a simple record, but we'll have a better way soon to interact with the database using Mongoose models.

Part 4: Setting up a Mongoose schema
Now that we have had our hands-on experience with the MongoDB implementation in the last section, let’s draw out the schema for our URL shortener. 

A Mongoose schema allows us to interact with the Mongo collections in an abstract way. Mongoose's rich documents also expose helper functions like .save which are 
enough to perform a full DB query to update changes in your document.

Here's how our schema for the URL shortener will look:

****************************************************************************************************************************************
const mongoose = require('mongoose')
const shortId = require('shortid')

const shortUrlSchema = new mongoose.Schema({
  full: {
    type: String,
    required: true
  },
  short: {
    type: String,
    required: true,
    default: shortId.generate
  },
  clicks: {
    type: Number,
    required: true,
    default: 0
  }
})

module.exports = mongoose.model('ShortUrl', shortUrlSchema)
************************************************************************************************************************************************************


We'll store this file in the models/url.js file. Once we have the schema, we can pass this part of the exercise. We have to do the following two things:

Create this model in the models/url.js file. (We did that.)
A POST request to /short should add something to the database to this model.
In order to do that, we can generate a new record using the following code:

****************************************************************************************************************************
app.post('/short', async (req, res) => {
 // insert the record using the model
 const record = new ShortURL({
  full: 'test'
 })
 await record.save()
 res.json({ ok: 1 })
})
***************************************************************************************************************************************

You'll see that we can omit the clicks and short field because they already have a default value in the schema. This means Mongoose will populate them automatically 
when the query runs.

Our final index.js file to pass this challenge should look like this:

*********************************************************************************************************************************
const express = require('express')
const app = express()
const mongoose = require('mongoose')
// import the model here
const ShortURL = require('./models/url')

app.set('view engine', 'ejs')

app.get('/', (req, res) => {
 res.render('index', { myVariable: 'My name is John!' })
})

app.post('/short', async (req, res) => {
 // insert the record using the model
 const record = new ShortURL({
  full: 'test'
 })
 await record.save()
 res.json({ ok: 1 })
})

// Setup your mongodb connection here
mongoose.connect('mongodb://localhost/codedamn')

mongoose.connection.on('open', () => {
 // Wait for mongodb connection before server starts
 app.listen(process.env.PUBLIC_PORT, () => {
  console.log('Server started')
 })
})
*****************************************************************************************************************

Part 5: Linking the frontend, backend, + MongoDB
Now that we have a handle on the backend part, let’s get back to the frontend and setup our webpage. There we can use the Shrink button to actually add some records to 
the database. 

If you look inside the views/index.ejs file, you’ll see that we have already passed the form data on the backend /short route. But right now we are not grabbing it.

You can see that there’s a new line called app.use(express.urlencoded({ extended: false })) on line 8, which allows us to read the response of the user from the form.
In the index.ejs file, you can see that we set name=”fullURL” which is how we can receive the URL on the backend.
Here's our index.ejs file:

*******************************************************************************************************************************************************************
<!DOCTYPE html>
<html lang="en">
 <head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <meta http-equiv="X-UA-Compatible" content="ie=edge" />
  <link
   rel="stylesheet"
   href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css"
  />
  <title>codedamn URL Shortner Project</title>
 </head>
 <body>
  <div class="container">
   <h1>URL Shrinker</h1>
   <form action="/short" method="POST" class="my-4 form-inline">
    <label for="fullUrl" class="sr-only">URL</label>
    <input
     required
     placeholder="URL"
     type="url"
     name="fullUrl"
     id="fullUrl"
     class="form-control col mr-2"
    />
    <button class="btn btn-success" type="submit">Shrink This!</button>
   </form>

   <table class="table table-striped table-responsive">
    <thead>
     <tr>
      <th>Full URL</th>
      <th>Short URL</th>
      <th>Clicks</th>
     </tr>
    </thead>
    <tbody>
     <% shortUrls.forEach(shortUrl => { %>
     <tr>
      <td><a href="<%= shortUrl.full %>"><%= shortUrl.full %></a></td>
      <td><a href="<%= shortUrl.short %>"><%= shortUrl.short %></a></td>
      <td><%= shortUrl.clicks %></td>
     </tr>
     <% }) %>
    </tbody>
   </table>
  </div>
 </body>
</html>
*****************************************************************************************************************************************************

This is a simple challenge, because we just have to put this code in to complete it:

***************************************************************************************************************************************************************
app.use(express.urlencoded({ extended: false }))

app.post('/short', async (req, res) => {
 // Grab the fullUrl parameter from the req.body
 const fullUrl = req.body.fullUrl
 console.log('URL requested: ', fullUrl)

 // insert and wait for the record to be inserted using the model
 const record = new ShortURL({
  full: fullUrl
 })

 await record.save()

 res.redirect('/')
})
*******************************************************************************************************************************************************************

First of all, we grab the sent URL by HTML using the req.body.fullUrl. To enable this, we also have app.use(express.urlencoded({ extended: false })) which allows us to 
get the form data.

Then we create and save our record just like we did the last time. Finally, we redirect the user back to the homepage so that the user can see the new links.

Tip: You can make this application more interesting by performing an Ajax request to the backend API instead of typical form submission. But we'll leave it here as it 
focuses more on MongoDB + Node setup instead of JavaScript.


Part 6: Displaying short URLs on the frontend
Now that we’re storing shortened URLs in MongoDB, let’s go ahead and show them on the frontend as well.

Remember our variables passed down to the ejs template from before? Now we’ll be using them.

The template loop for ejs has been done for you in the index.ejs file (you can see that loop above). However, we have to write the Mongoose query to extract the data 
in this section.

If we see the template, we'll see that in index.js we have the following code:

****************************************************************************************************************************
app.get('/', (req, res) => {
 const allData = [] // write a mongoose query to get all URLs from here
 res.render('index', { shortUrls: allData })
})
*******************************************************************************************************************************************************************

We already have a model defined with us to query data from Mongoose. Let's use it to get everything we need.

Here's our solution file:

*******************************************************************************************************************************************************************
const express = require('express')
const app = express()
const mongoose = require('mongoose')
// import the model here
const ShortURL = require('./models/url')

app.set('view engine', 'ejs')
app.use(express.urlencoded({ extended: false }))

app.get('/', async (req, res) => {
 const allData = await ShortURL.find()
 res.render('index', { shortUrls: allData })
})

app.post('/short', async (req, res) => {
 // Grab the fullUrl parameter from the req.body
 const fullUrl = req.body.fullUrl
 console.log('URL requested: ', fullUrl)

 // insert and wait for the record to be inserted using the model
 const record = new ShortURL({
  full: fullUrl
 })

 await record.save()

 res.redirect('/')
})

// Setup your mongodb connection here
mongoose.connect('mongodb://localhost/codedamn', {
 useNewUrlParser: true,
 useUnifiedTopology: true
})

mongoose.connection.on('open', async () => {
 // Wait for mongodb connection before server starts

 // Just 2 URLs for testing purpose
 await ShortURL.create({ full: 'http://google.com' })
 await ShortURL.create({ full: 'http://codedamn.com' })

 app.listen(process.env.PUBLIC_PORT, () => {
  console.log('Server started')
 })
})
********************************************************************************************************************************************************************

You can see that it was as easy as doing await ShortURL.find() in the allData variable. The next part is where things get a bit tricky.

Part 7: Making the redirection work
We’re almost done! We have the full URL and short URL stored in the database now, and we show them on the frontend too.

But you’ll notice that the redirection does not work right now and we get an Express error.

Let’s fix that. You can see in the index.js file there’s a new dynamic route added at the end which handles these redirects:

***********************************************************************************************************************************************************************
app.get('/:shortid', async (req, res) => {
 // grab the :shortid param
 const shortid = ''

 // perform the mongoose call to find the long URL

 // if null, set status to 404 (res.sendStatus(404))

 // if not null, increment the click count in database

 // redirect the user to original link
})
************************************************************************************************************************************************

Our challenges for this part looks like this:


Alright. First things first, we have to extract out the full URL when we visit a short URL. Here's how we'll do that:

*****************************************************************************************************************************************************
app.get('/:shortid', async (req, res) => {
 // grab the :shortid param
 const shortid = req.params.shortid

 // perform the mongoose call to find the long URL
 const rec = await ShortURL.findOne({ short: shortid })

 // ...
})
************************************************************************************************************************************************

Now, if we see that our result is null, we'll send a 404 status:

*****************************************************************************************************************************************
app.get('/:shortid', async (req, res) => {
 // grab the :shortid param
 const shortid = req.params.shortid

 // perform the mongoose call to find the long URL
 const rec = await ShortURL.findOne({ short: shortid })

 // if null, set status to 404 (res.sendStatus(404))
 if (!rec) return res.sendStatus(404)

 res.sendStatus(200) 
})
***************************************************************************************************************************************************************

This passes our first challenge. Next, if we in fact have a link, let's redirect the user and increment the click count too in the database.

**********************************************************************************************************************************************************
app.get('/:shortid', async (req, res) => {
 // grab the :shortid param
 const shortid = req.params.shortid

 // perform the mongoose call to find the long URL
 const rec = await ShortURL.findOne({ short: shortid })

 // if null, set status to 404 (res.sendStatus(404))
 if (!rec) return res.sendStatus(404)

 // if not null, increment the click count in database
 rec.clicks++
 await rec.save()

 // redirect the user to original link
 res.redirect(rec.full)
})
***************************************************************************************************************************************************************

This way, we can increment and store the result in the database again. And that should pass all of our challenges.


Conclusion:
      In this way we can built a full working URL shortener by using EXPRESS + NODE + MONGODB.

##################################################################################THE END ####################################################################################
