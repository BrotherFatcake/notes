#  Mongoose Configuration

*   Import Mongoose
    *   ~~~ js
        const mongoose = require('mongoose');
        ~~~

*   Import DB URLs and Port values from 'config.js'
    *   ~~~ js
        const {PORT, DATABASE__URL} = require('./config');
        ~~~

*   'config.js' contains DB url and port details
    *   DATABASE_URL = PROD
    *   TEST_DATABASE_URL = Test
    *   ~~~ js
        exports.DATABASE_URL = process.env.DATABASE_URL ||
                      'mongodb://PRODUCTION_URL.COM/DB_DETAILS';
        exports.TEST_DATABASE_URL = process.env.TEST_DATABASE_URL ||
                      'mongodb://localhost/DB_DETAILS';
        exports.PORT = process.env.PORT || 8080;
        ~~~
    *   Other methods to set Environment variables
        *   Set one temporarily while the program runs by adding before the command used to run the program:
            *   ~~~ js
                PORT=5000 node server.js
                ~~~
        *   Set for the duration of a terminal session
            *   ~~~ js
                export PORT=5000
                node server.js
                ~~~

*   runServer is responsible for connecting to the DB and starting the HTTP server
    *   First Mongoose connects to the DB
        *   ~~~ js
            mongoose.connect(databaseUrl, ...
            ~~~
    *   Then listen for new connections on the specified port
    *   If ALL of the above was successful call a callback function to signal everything is up and running or return a Promise.

*   Useful bit of code added at bottom - if script run directly using 'node server.js' the then runServer function will be called.  If called from elsewhere 'require('./server')' then the function won't be called.  This allows the server to be started from different points. 
    *   *this will be useful when we need to execute the runServer function before each test in our test suites.
    *   ~~~ js
        if (require.main === module) {
        runServer(DATABASE_URL).catch(err => console.error(err));
        };
        ~~~

*   closeServer responsible for disconnecting from the DB and stopping the HTTP server.  
    *   If ALL of the above was successful call a callback function to signal everything is up and running or return a Promise.
    *   *Note that closeServer needs to access a server object, and server gets created in runServer. That's why we declare let server; outside both functions.


#  Data Modeling

*   For each model we define a schema.
    *   **SCHEMA** - specifies how all documents in a specific collection should look - specify the properties the document has along with the schema type, and other settings (required, default values, etc.)
    *   Example code from `models.js`: 
    *   ~~~ js
        const mongoose = require("mongoose");
        
        // this is our schema to represent a restaurant
        const restaurantSchema = mongoose.Schema({
        name: { type: String, required: true },
        borough: { type: String, required: true },
        cuisine: { type: String, required: true },
        address: {
            building: String,
            // coord will be an array of string values
            coord: [String],
            street: String,
            zipcode: String
        },
        // grades will be an array of objects
        grades: [
            {
            date: Date,
            grade: String,
            score: Number
            }
        ]
        });
        ~~~

*   To define a new model that uses the new schema that was defined
    *   ~~~ js
        const Restaurant = mongoose.model("restaurants", restaurantSchema);
        ~~~
    *   The first argument passed to .model; `restaurants` will be used to determine the collection in the DB that corresponds to the model.
    *   Mongo will convert the name as follows `'Restaurants' => 'restaurants'`.  This means that configuring Mongoose to work with 'Restaurants' Mongo is actually working with `db.restaurants`.
    *   The second argument is the name of the schema const that was created.
    *   Optional third argument - string indicating a collection name in the DB.  By default Mongoose will infer the collection name from the model name.

*   Queries run from `server.js` using the `Restaurant` model defined in `models.js` to `find, create, update, delete`
    *   Example of `find`:
    *   ~~~ js
        Restaurant
        .findOne()
        .then(restaurant => res.json({
            name: restaurant.name,
            cuisine: restaurant.cuisine
        }))
        .catch(err => {
            console.error(err)
            res.status(500).json({message: 'Something went wrong'})}
        );
        ~~~
*   `.findOne()` returns a promise, if function is successful the callback in `.then()` runs.

**Virtuals**

*   [**Virtuals**](http://mongoosejs.com/docs/guide.html#virtuals) provide ability to create derived properties on model instances.
    *   Address in example model is an object, can create a new property that is 'human readable' with **Virtuals**
    *   Would create a new property `addressString` that is available for use with `Restaurant` model
    *   ~~~ js
        restaurantSchema.virtual('addressString').get(function() {
        return `${this.address.building} ${this.address.street}`.trim()});
        ~~~

        *   The below example may log the following: `123 Main Street`
        *   ~~~ js
            console.log(myRestaurant.addressString);
            ~~~
    *   This example would create a new property `grade` and return the most recent Restaurant grade if present
        *   ~~~ js
            restaurantSchema.virtual('grade').get(function() {
                const gradeObj = this.grades.sort((a, b) => {return b.date - a.date})[0] || {};
                return gradeObj.grade;
            });
            ~~~
*   The example below gives each instance of the `Restaurant` model a `serialize` method.  
    *   Allows user to specify how data is represented outside of the application via the API.
    *   Items like passwords can be left out of the `serialize` method so that they are inaccessible from the API.
    *   Mongoose [Instanced Method](http://mongoosejs.com/docs/guide.html#methods) documentation
    *   ~~~ js
        restaurantSchema.methods.serialize = function() {
        return {
            id: this._id,
            name: this.name,
            cuisine: this.cuisine,
            borough: this.borough,
            grade: this.grade,
            address: this.addressString
        };
        };
        ~~~

#   Find, Create, Update, Delete via GET, POST, DELETE, PUT

##  Finding Documents

### Find all

*   User makes a `app.get('/endpointName')` request which calls `Restaurant.find()` to return all documents in the collection.
    *   If the query is successful object with property `restaurants` is returned that is an `array` of restaurant objects.
    *   For each restaurant returned `serialize()` is called to generate the object used to represent the restaurant to clients of the API.
    *   If query fails log and send `500 status` with message.
    *   ~~~ js
        //GET request that calls modelName.find()
        app.get('/restaurants', (req, res) => {
        Restaurant.find()
       
        //when query success .then() runs
            .then(restaurants => {
       
        //response json value 'restaurants' array is mapped  
            res.json({
                restaurants: restaurants.map(
        
        //From map method each restaurant value returned is serialized to generate object used to represent the data to users of the API
                (restaurant) => restaurant.serialize())
            });
            })
        
        //.catch() used to catch exceptions
            .catch(
       
        //When error occurs 
            err => {
       
        //log the error to console
                console.error(err);
       
        //Send status 500 with json message to user stating error ocurred.
                res.status(500).json({message: 'Internal server error'});
            });
        });
        ~~~


### Find by id

*   User makes a `app.get('/endpointName/:id')` request which calls `Restaurant.findById(req.prams.id)` to return a specific document in a collection.
    *   If the query is successful a json string with property `restaurants` is returned.
    *   `serialize()` is called to generate the object used to represent the restaurant to clients of the API.
    *   If query fails log and send `500 status` with message.
    *   ~~~ js
        //GET request to the /:id endpoint and calls modelName.findById()
        app.get('/restaurants/:id', (req, res) => {
       
        //Provide request parameter of id to modelName.findById(
            Restaurant.findById(req.params.id)
       
        //when query success .then() runs, json string response object serialized to generate object used to represent the data to users of the API
            .then(restaurant =>res.json(restaurant.serialize()))

        //.catch() used to catch exceptions
            .catch(err => {

        //log the error to console
                console.error(err);

        //Send status 500 with json message to user stating error ocurred.
                res.status(500).json({message: 'Internal server error'})
            });
        });
        ~~~

### Find by value

*   User makes a `app.get('/endpointName')` request which after identifying the fields queries makes a call to `Restaurant.find(filters)` to return documents in a collection that meet the search criteria specified in the URL params.
    *   If called without URL params the `.find()` call will return everything.
    *   If the query is successful a json string with property `restaurants` is returned.
    *   `serialize()` is called to generate the object used to represent the restaurant to clients of the API.
    *   If query fails log and send `500 status` with message.
    *   Example below requires specified queryableFields to be present in URL but the values can be empty
        *  `localhost:8080/restaurants?cuisine=Italian&borough=Manhattan`
        *  `localhost:8080/restaurants?cuisine=Italian&borough=`
        *  `localhost:8080/restaurants?cuisine=&borough=Manhattan `
    *   ~~~ js
        //GET request that calls modelName.find()
        app.get('/restaurants', (req, res) => {

        //create const object to assign URL param values to
        const filters = {};

        //create const defining the DB fields that can be queried
        const queryableFields = ['cuisine', 'borough'];

        //forEach queryableFields compare to fields in request
        queryableFields.forEach(field => {
        
        //IF field in request query matches field in queryableFields assign to filters object
            if (req.query[field]) {
                filters[field] = req.query[field];
            }
        });

        //call modelName.find(arg) passing the filters const as the argument
        Restaurant.find(filters)

        //when query success .then() is ran on modelName data => response json() passed to map    
            .then(Restaurants => res.json(
        
        //returned data from response goes through map method where each restaurant value returned is serialized to generate object used to represent the data to users of the API
                Restaurants.map(restaurant => restaurant.serialize())
            ))

        //.catch() used to catch exceptions
            .catch(err => {

        //log the error to console
                console.error(err);

        //Send status 500 with json message to user stating error ocurred.
                res.status(500).json({message: 'Internal server error'})
            });
        }); 
        ~~~

**Examples to add**
*   Also possible to:
    *   Check if a value is less than or greater than some number.
    *   Check for values in a nested field.
    *   Check if a value for a property is in a list of specific values.
    *   Use regular expressions to match (this would allow the cuisine and borough logic above, case insensitive).

*   For a full discussion of available query operators, consult the Mongo docs on [getting started with queries](https://docs.mongodb.com/getting-started/shell/query/) and on [query operators](https://docs.mongodb.com/manual/reference/operator/query/).


##  Create Documents

*   To create documents `app.post('/endpointName')` calls the `.create()` method
    *   The code first checks the request for the required fields, if any are missing an error is returned
    *   When success `.create()` is called and passed an object with key/value pairs that match up to the required fields (ie. the fields defined in the schema)
    *   When create succeeds, return object created by `modelName.serialize()`, if fails log an error and provide message


    *   ~~~ js

        //POST request that hands off response for verification of required fields
        app.post('/restaurants', (req, res) => {

        //create const array specifying required fields
        const requiredFields = ['name', 'borough', 'cuisine'];
        
        //loop through required field array to verify fields are in request
        for (let i=0; i<requiredFields.length; i++) {

        //const created to represent the required field in the loop at position [i]
            const field = requiredFields[i];

        //IF the field is **NOT** found in the request body
            if (!(field in req.body)) {
        
        //create const for missing field message that specifies specific missing field
            const message = `Missing \`${field}\` in request body`

        //log the error to the console displaying the message containing missing field
            console.error(message);

        //return status 400 along with message containing missing field
            return res.status(400).send(message);
            }
        }

        //call modelName.create() and pass key/val object params from request body as arg
        Restaurant.create({
            name: req.body.name,
            borough: req.body.borough,
            cuisine: req.body.cuisine,
            grades: req.body.grades,
            address: req.body.address})

        //when query success .then() runs, json string response object serialized to generate object used to represent the data to users of the API 
            .then(
        
        //data => response returns status 201 and json response is serialized
            restaurant => res.status(201).json(restaurant.serialize()))

        //.catch() used to catch exceptions
            .catch(err => {

        //log the error to console
                console.error(err);

        //Send status 500 with json message to user stating error ocurred.
                res.status(500).json({message: 'Internal server error'})
            });
        });



##  Update Documents

*   To update documents `app.put('/endpointName/:id')` calls the `.findByIdAndUpdate('arg1', 'arg2')` method.
    *   The code first checks that the `id` in the request params and request body are the same.
    *   Then a const object of fields to be updated is created.
    *   Then `.findByIdAndUpdate('arg1', 'arg2')` is called and passed `arg1 = 'id of the doc'` and `arg2 = 'object containing what to update'`
    

    *   ~~~ js

        //PUT request that hands off the data to verify that the body and params match
        app.put('/restaurants/:id', (req, res) => {

        //If request params id and request body id are missing and are not equal return an error
        if (!(req.params.id && req.body.id && req.params.id === req.body.id)) {

            //create const for error message containing the specific request param and body id's that should match
            const message = (`Request path id (${req.params.id}) and request body id (${req.body.id}) must match`);

            //log the error to console displaying the message
            console.error(message);

            // we return here to break out of this function
            //return response with status 400 and json message text
            return res.status(400).json({message: message});
        }

        //create const empty object of fields contained in the request to be updated
        const toUpdate = {};

        //create const array of fields that can be updated
        const updateableFields = ['name', 'borough', 'cuisine', 'address'];

        //for each item in the updateable array compare to fields in request
        updateableFields.forEach(field => {

        //IF field in request query matches field in updateableFields assign to toUpdate object
            if (field in req.body) {
            toUpdate[field] = req.body[field];
            }
        });

        //call modelName.findByIdAndUpdate, pass request param id and set values contained in toUpdate
        Restaurant.findByIdAndUpdate(req.params.id, {$set: toUpdate})

        //then data => response status 204 and end
            .then(restaurant => res.status(204).end())

        //use .catch() err => response status 500 with .json() message of server error
            .catch(err => res.status(500).json({message: 'Internal server error'}));
        });
        ~~~


##  Delete Documents

*   To delete documents `app.delete('/endpointName/:id')` calls the `.findByIdAndRemove('arg1')` method.
   
    *   ~~~ js
        //DELETE request calls the endpoint/:id to delete
        app.delete('/restaurants/:id', (req, res) => {
        
        //modelName.findByIdAndRemove called and passed request param id to delete
            Restaurant.findByIdAndRemove(req.params.id)
        //then when success => respond status 204 and end
            .then(() => res.status(204).end())
        
        //use .catch() err => response status 500 with .json() message of server error
            .catch(err => res.status(500).json({message: 'Internal server error'}));
        });
        ~~~

#   Key points

*   Configuring Mongoose: You'll always need to specify your database URL and then start your Node.js server. If you're connecting to a local database rather than mLab, be sure to start the local database server before starting Node.js.

*   Defining schemas: Each model you create needs a Mongoose schema, which specifies the fields that instances of the model will have and their data type. You can also add virtual properties and instance methods onto the schema.

*   CRUD operations: We use Mongoose's built-in model methods like `.create(), .find, .findById, findByIdAndUpdate, and findByIdAndRemove` to interact with the database layer.

*   Retrieving vs. presenting data: Just because our data looks one way in the database doesn't mean that our API needs to pass along the raw data. If you want to have a standard way of representing your documents in your API, adding a serialize() instance method can be a good way to achieve that (note that there's nothing magical about the name serialize, we just chose that name because it describes how the object needs to be turned into a JSON string when sent via our REST API).


 
#Deploy to Heroku, mLab and Travis CI

## mLab - Mongo DB

*   Create DB on mLab

*   Create DB User for new DB and gather connection info required for Heroku or accessing from your PC
~~~ js
//Seed/Import data into local DB
mongoimport --db <dbName> --collection <collectionName> --drop --file ../seed-data.json

//Connect to mlab DB
mongo ds123456.mlab.com:29621/<dbName> -u <dbuser> -p <dbpassword>

//Seed/Import data into mlab DB
mongoimport -h ds123456.mlab.com:29621 -d <dbName> -c <collectionName> -u <dbuser> -p <dbpassword> --file seed-data.json

//Heroku config for DB
mongodb://<dbuser>:<dbpassword>@ds123456.mlab.com:29621/<dbName>
~~~

## Heroku

1.  Go to project directory and run `heroku create` and document the name of your app presented when complete.

2.  Run `git push heroku master`.

3.  Run `heroku ps:scale web=1` to start up a free web dyno on the server.

4.  Navigate to [Heroku](https://id.heroku.com/login)

5.  Once on your dashboard locate the app name documented in `Step 1` and open it.  Then access `Setting`.

6.  Click `Reveal Config Vars`, ADD `KEY = DATABASE_URL` and `VALUE = mongodb://<dbuser>:<dbpassword>@ds123456.mlab.com:29621/<dbName>`

7.  Click `Open app`, app can now receive CRUD requests from Postman


## Travis CI

~~~
To add
~~~ 




# Example Comments for Practice
-------------------------------------------

## Setup server.js

~~~ js

//Use strict
"use strict";

//Create a constant to import(require) Express

//Create a constant to import(require) Mongoose

//Create a constant to import/require Morgan for logging

//mongoose Promise to use global ES6 Promises

//Create const of PORT, DATABASE_URL to import(require) config.js

//Create const of {model} to import(require) models.js

//Create constant that creates a new app instance by calling top level Express function

//tell app to use express.json

//tell app to use morgan for common logging

**If using Express Routing by putting CRUD code in separate JS file**

//Create const for new `constNameRoute` (can name it anything) to import(require) routeFile.js (can name it anything)

//tell app to use args of '/endPointName' and const specified for 'constNameRoute'

**End Express Routing specific**

/*	Catch-all and Server Start/Stop		*/

//catch-all endpoint
//tell app to use args `"*"` and `function with req/res` args

//response status 404 and json message 



//Server Start
//declare empty `server` variable - this is needed for stopServer


//function named startServer takes args `databaseUrl` and `port = PORT`

//return a new promise with args `resolve` and `reject =>`

//tell mongoose to connect with args `databaseUrl` and `err =>`

//if err return reject err

//assign `server` = tell app to listen with args `port` and `() =>`

//log to console the app is listening to port `${port}`

//resolve for outstanding promise


//on args `error` and `err =>`

//mongoose disconnect

//reject with err for outstanding promise


//Server Stop

//function named stopServer has no args

//return mongoose disconnect with no args then `=>`

//return a new promise with args `resolve` and `reject =>`

//log to console the server is being stopped

//tell `server` to close with arg `err =>`

//if err return reject with err for outstanding promise

//resolve for outstanding promise



//Code to allow server to be called directly or via tests
//if require main is strictly equal to module

//runServer with arg `DATABASE_URL` and `catch err =>` log `error err` to console 


//export the modules created add, runServer and closeServer

~~~


## Setup config.js

~~~ js

//Use strict
"use strict";

//exports DATABASE_URL equals process env DATABASE_URL || "mongodb://databaseURL/dbName...";

//exports TEST_DATABASE_URL equals process env TEST_DATABASE_URL || "mongodb://databaseURL/test_dbName...";

//exports PORT equals process env PORT || <portNum>;

~~~


## Setup models.js

~~~ js

//Use strict
"use strict";

//Create a constant to import(require) Mongoose

//Create const for SCHEMA
//schemaName is mongoose Schema ({schemaObject})

//Within schemaObject specify fieldName: {type: <type>, required: true/false/'do not include'}
//Example: fieldName: {type: String, required: true}, fieldName2: {type: String}

//Create **VIRTUALS** to return more human readable sub-values
//Example personName may have sub-values of first and last
//schemaName virtual with arg 'virtualName' get function with no args

//return this.personName.first this.personName.last trim with no args

//Create cleanUp function to specif what should be returned via the API
//schemaName methods methodName is function with no args

//return {object}
//Within object {key1: this.val1, key2: this.val2, key3: this.virtualName}

//Create const modelName is mongoose model with args '<DB collectionName>' and schemaName

//export module modelName

~~~


## Setup routeFile.js OR CRUD requests within server.js

~~~ js
//Use strict
"use strict";

**Only needed if using Express Routing**
//Create a constant to import(require) Express

//Create a constant to import(require) express Router

//Create const of {model} to import(require) models.js


//At bottom of file
//export module of router;

**End Express Routing specific**



**Needed for all CRUD**
**If part of server.js use 'app.' instead of 'router.' and '/endPointName' and '/endPointName/:id' instead of '/' and '/:id'**

//GET - ALL

//call router or app get from '/' with args request response =>

//modelName find with no args

//then with arg '<DB collectionName>' =>

//respond with json object key/val pair, val is map of <DB collectionName>

//New map (arrayName) => send arrayName through cleanUp method 

//ERROR CATCHER
//catch err =>

//log error err to console

//respond status 500 with json message stating error ocurred


//GET by ID

//call router or app get from '/:id' with args request response =>

//modelName findById with args request params id

//then dataName => respond with json with arg of 'send dataName through cleanUp method
    //where data name (ex. student or 'blog post') is the object being returned


//ERROR CATCHER
//catch err =>

//log error err to console

//respond status 500 with json message stating error ocurred


//POST

//call router or app post from '/' with args request response =>

//create const array that specifies required fields
    //If required field has sub-fields they do not need to be specified here it would be specified 
    //in the POST request sent from postman

//loop through requiredFields

//create const field to assign requiredField position i during loop


//IF field is not in the request body

//create const for error message naming missing field from body

//log the error to console with arg errMessage

//return response status 400 and send errMessage


//CREATE NEW DB OBJECT
//modelName create {object}

    //Within object {key1: req.body.val1, ley2: req.body.val2}
    //If key/val has sub-key/val they do not need to be specified here it would be specified 
        //in the POST request sent from postman


//then dataName => respond status 201 and json dataName send through cleanUp
    //where data name (ex. student or 'post') is the object being returned


//ERROR CATCHER
//catch err =>

//log error err to console

//respond status 500 with json message stating error ocurred


//PUT

//call router or app put from '/:id' with args request response =>

//IF NOT request params id AND request body id AND request params id strict equal request body id 

//create const for error message stating request params id AND request body id must match

//log the error to console with arg errMessage

//return response status 400 and send errMessage

//create const of empty object for fields to update

//create const array of fields that are allowed to be updated

//for each canUpdate with arg dataArg =>

//if dataArg is in request body

//assign the request body[dataArg] to makeUpdate[dataArg] object


//modelName call findByAndUpdate with args request params id and object with $set/val pair where val is makeUpdate

//then dataName => respond with json message naming id has been updated, status 204, and end
    //where data name (ex. student or 'post') is the object being returned


//ERROR CATCHER
//catch err =>

//log error err to console

//respond status 500 with json message stating error ocurred


//DELETE

//call router or app delete from '/:id' with args request response =>

//modelName findByIdAndRemove with arg request params id

//then dataName => respond with json message naming id has been removed, status 204, and end
    //where data name (ex. student or 'post') is the object being returned


//ERROR CATCHER
//catch err =>

//log error err to console

//respond status 500 with json message stating error ocurred
~~~

