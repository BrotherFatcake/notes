<x id='top'></x>
[General Mongoose](#genmongoo) | [Data Model](#gendatamod) | [Virtuals and Methods](#virtualsmethod) | [CRUD](#crud) | [CRUD-Find/GET](#findget) | [CRUD-Create/POST](#createpost) | [CRUD-Update/PUT](#updateput) | [CRUD-Delete](#delete) | [CRUD-Key Points](#keys) | [Heroku, mLab, Travis CI](#hermlabtrav) | [Practice Comments](#practice) | [server.js](#server) | [config.js](#config) | [models.js](#models) | [test.js](#testfile) | [Route File](#route)

-------------------------------------------

#  Mongoose Configuration
<x id='genmongoo'></x>

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

[Top](#top) 
#  Data Modeling
<x id='gendatamod'></x>

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

[Top](#top) 
**Virtuals**
<x id='virtualsmethod'></x>

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

[Top](#top) 
#   Find, Create, Update, Delete via GET, POST, DELETE, PUT
<x id='crud'></x>

##  Finding Documents

### Find all
<x id='findget'></x>

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


[Top](#top) 
##  Create Documents
<x id='createpost'></x>

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


[Top](#top) 
##  Update Documents
<x id='updateput'></x>

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

[Top](#top) 
##  Delete Documents
<x id='delete'></x>

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

[Top](#top) 
#   Key points
<x id='keys'></x>

*   Configuring Mongoose: You'll always need to specify your database URL and then start your Node.js server. If you're connecting to a local database rather than mLab, be sure to start the local database server before starting Node.js.

*   Defining schemas: Each model you create needs a Mongoose schema, which specifies the fields that instances of the model will have and their data type. You can also add virtual properties and instance methods onto the schema.

*   CRUD operations: We use Mongoose's built-in model methods like `.create(), .find, .findById, findByIdAndUpdate, and findByIdAndRemove` to interact with the database layer.

*   Retrieving vs. presenting data: Just because our data looks one way in the database doesn't mean that our API needs to pass along the raw data. If you want to have a standard way of representing your documents in your API, adding a serialize() instance method can be a good way to achieve that (note that there's nothing magical about the name serialize, we just chose that name because it describes how the object needs to be turned into a JSON string when sent via our REST API).


[Top](#top) 
# Deploy to Heroku, mLab and Travis CI
<x id='hermlabtrav'></x>

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


## Mocha Chai package.json config

1.  Go to project directory and run:

2.  `npm install mocha`

3.  `npm install chai`

4.  `npm install chai-http`

5.  `npm install faker`

6.  `vi package.json`

7.  Update `"test"` key value to `./path/to/test.js`

8.  Save and quit


## Travis CI & Heroku deploy

**TRAVIS CI**
1.  Go to project directory and `touch .travis.yml && vi .travis.yml` to create and open the file

2.  Add the below text to the .travis.yml file:
        ~~~ js
        language: node_js
        node_js: node
        services:
        - mongodb
        ~~~ 

3.  Create/Access Repo on GitHub and open `Settings > Integrations & Services`

4.  Select `Add service` and search for and select `Travis CI`, then click `Add service`

5.  Open [Travis CI](https://travis-ci.org/) and access your profile

6.  Search for the project and enable the switch

7.  To trigger a build commit changes to git repo or perform `git commit --allow-empty`.  Return to OR refresh the main Travis CI page to view build status

**HEROKU**

8.  If needed create new DB and DB User on mLab (Instructions above)

9.  If needed seed data on new DB

10. Check Travis CI and Ruby verions
    ~~~ js
    termPrompt$ ruby -v

    termPrompt$ travis version

    ~~~
    *   Verify versions against install instructions OR if needed install [Travis CI CLI](https://github.com/travis-ci/travis.rb#installation)

11. Run `travis login` and enter GitHub `UserName` and `Password`

12. Run `travis setup heroku`, if changes are not needed hit `Enter` to continue through the prompts

13. Verify Travis changes by entering `git diff`

14. Enter `heroku create` and document the name of your app presented when complete.


15. Edit .travis.yml and update `deploy:app:` with the Heroku name - `app: random-words-89873`

16.  Navigate to [Heroku](https://id.heroku.com/login)

17.  Once on your dashboard locate the app name documented in `Step 1` and open it.  Then access `Setting`.

18.  Click `Reveal Config Vars`, ADD `KEY = DATABASE_URL` and `VALUE = mongodb://<dbuser>:<dbpassword>@ds123456.mlab.com:29621/<dbName>`

19. Trigger a build: commit changes to git repo or perform `git commit --allow-empty`.  Return to OR refresh the main Travis CI page to view build status.  Travis CI will build, test, and deploy to Heroku.

20.  Run `heroku ps:scale web=1` to start up a free web dyno on the server.

21. After Travis CI builds, tests and deploys to Heroku return to Heroku `settings` and click `Open app`, app can now receive CRUD requests from Postman





[Top](#top) 

# Example Comments for Practice
<x id='practice'></x>

-------------------------------------------

[Top](#top) 
## Setup server.js
<x id='server'></x>

~~~ js

//Use strict
'use strict';

//if using .env fila
require('dotenv').config();


//Create a constant to require express

//Create a constant to require mongoose

//Create a constant to require morgan for logging

//mongoose Promise equals global ES6 Promises

//Create const of PORT, DATABASE_URL to require config.js

//Create const of modelName or {modelName} to require models.js - use {} if there are multiple models

//Create constant that creates a new app instance by calling top level express with no args

//tell app to use with arg express.json with no arg

//tell app to use morgan with arg 'common' logging

**If using Express Routing by putting CRUD code in separate JS file**

//Create const for new `constNameRoute` (can name it anything) to require routeFile.js (can name it anything)

//tell app to use args of '/endPointName' and const specified for 'constNameRoute'

**End Express Routing specific**

/*	Catch-all and Server Start/Stop		*/

//catch-all endpoint
//tell app to use args `"*"` and `function with req/res` args

//response status 404 and json message 



//Server Start
//declare empty `server` variable - this is needed for stopServer

//function named startServer takes args databaseUrl and port = PORT

//return a new Promise( with args (resolve and reject) =>

    //tell mongoose to connect with open args (databaseUrl and err =>

        //if err return reject err

            //assign `server` = app listen with args port and () =>

            //log to console the app is listening to port ${port}

                //resolve with no args for outstanding promise


        //chain on with args `error` and err =>

        //mongoose disconnect with no args

        //reject with arg err for outstanding promise


//Server Stop

//function named stopServer has no args

    //return mongoose disconnect with no args then( with no args() =>

    //return a new promise with args (resolve and reject) =>

    //log to console the server is being stopped

    //tell server to close( with arg err =>

        //if err return reject with err for outstanding promise

        //resolve with no args for outstanding promise



//Code to allow server to be called directly or via tests
//if require main is strictly equal to module

//startServer with arg DATABASE_URL, catch with arg err => error to console with arg err to console not a message 


//module exports created {app, startServer, stopServer}

~~~

[Top](#top) 
## Setup config.js
<x id='config'></x>

~~~ js

//Use strict
'use strict';

//exports DATABASE_URL equals process env DATABASE_URL || "mongodb://databaseURL/dbName...";
    //For dev environment may need to swap positions - 'mongodb://databaseURL/dbName...' || process env DATABASE_URL


//exports TEST_DATABASE_URL equals process env TEST_DATABASE_URL || "mongodb://databaseURL/test_dbName...";

//exports PORT equals process env PORT || <portNum>;

~~~

[Top](#top) 
## Setup models.js - Single DB Collection
<x id='models'></x>

~~~ js

//Use strict
'use strict';

//Create a constant to require mongoose

//Create const for the schemaName equals mongoose Schema ({schemaObject})

    //Within schemaObject specify fieldName: {type: <type>, required: true/false/'do not include'}
        //Example: fieldName: {type: String, required: true}, fieldName2: {type: String}, include nested fields

//Create **VIRTUAL** to return more human readable sub-values
    //Example personName may have sub-values of first and last
    //schemaName virtual with arg 'virtualName' get function with no args

    //return template literal placeholders this.personName.first and this.personName.last, trim with no args

//Create cleanUp function to specify what should be returned via the API
//schemaName methods methodName equal function with no args

    //return {object}
    //Within object {key1: this.val1, key2: this.val2, key3: this.virtualName}


//Create const modelName is mongoose model with args '<DB collectionName>' and schemaName

//module exports equal {modelName}

~~~

## Setup models.js - Multiple DB Collection
~~~ js

//Use strict
'use strict';

//Create a constant to require mongoose

//Create const for the schemaName equals mongoose Schema ({schemaObject})

    //Within schemaObject specify fieldName: {type: <type>, required: true/false/'do not include'}
        //Example: fieldName: {type: String, required: true}, fieldName2: {type: String}, include nested fields

//Create **VIRTUAL** to return more human readable sub-values
    //Example personName may have sub-values of first and last
    //schemaName virtual with arg 'virtualName' get function with no args

    //return template literal placeholders this.personName.first and this.personName.last, trim with no args

//Create cleanUp function to specify what should be returned via the API
//schemaName methods methodName equal function with no args

    //return {object}
    //Within object {key1: this.val1, key2: this.val2, key3: this.virtualName}


//Create const modelName is mongoose model with args '<DB collectionName>' and schemaName

//module exports equal {modelName}

~~~

[Top](#top) 
## Setup test file
<x id='testfile'></x>

~~~ js
//use strict
'use strict';

//create const to require chai

//create const to require chai-http


//create const to require faker

//create const to require mongoose

//create const expect to assert chai expect

//Create const of {model} to require models.js.  {model} = model name defined in models.js

//create const of object app, runServer, closeServer to require server.js

//Create const of TEST_DATABASE_URL to require config.js

//tell chai to use chaiHttp


//SEED TEST DATA

//create seed data function with no args

    //log info with message to console that data is seeding

    //create const of empty array for seed data

    //loop through x number of times and push seed data with arg of generate data function with no args

    //return modelName and insertMany with arg of seed data


//create generate data function with no args

// return {object}
	//{object} contains fields needed to create DB test data
	// {key1: faker.dataSet.dataType(), key2: generateDataFunction2(), key3: {key3a: faker.dataSet.dataType(), key3b: generateDataFunction3()}, key4: [array of faker or function calls]}



//**ADDITIONAL OPTIONAL FUNCTIONS USED TO CREATE DATA FAKER ISN'T ABLE TO PRODUCE - ADD AS NEEDED**

//create generate data function2 with no args

//create const of <DB fieldName> array ['data1','data2','data3']

//return <DB fieldName> as array position of 'x'
	//'x' = Math floor of Math random * <DB fieldName> array length
	//can also return object containing value of above and generated faker data
	// {key5: faker.dataSet.dataType(), key6: <DB fieldName> array const

//**END OPTIONAL AS NEEDED FUNCTIONS



//DB TEAR DOWN and BEFORE/AFTER FUNCTIONS

//create tear down function with no args

    //warn to console that DB is being deleted

    //return mongoose connection dropDatabase with no args


//describe with args of 'description' and function with no args 

    //BEFORE - RUNS ONCE TO START SERVER
    //before with arg of function with no args

        //return runServer with arg of test DB url


    //BEFOREEACH - RUNS WITH EACH TEST TO SEED DATA
    //beforeEach with arg of function with no args

        //return seed data function with no args


    //AFTEREACH - RUNS AT THE END OF EACH TEST TO CLEAR THE DB
    //afterEach with arg of function with no args

        //return tear down function with no args

    //AFTER - RUNS ONCE TO STOP SERVER WHEN TESTING IS COMPLETE
    //after with arg of function with no args

        //return closeServer with no args
		

//**TESTS**

//GET TESTS

//describe with args of 'description of test block' and function with no args



//Test 1
      // strategy:
      //    1. get back all objects returned by by GET request to `/endpoint`
      //    2. prove res has right status, data type
      //    3. prove the number of objects we got back is equal to number
      //       in db.


//it with args 'description of test' and function with no args

    //create 1stEmpty variable to save future response value

        //return chai request with arg app

        //.get with arg of '/endpoint'

        //then with arg of function with arg of _res

            //set 1stEmpty variable to _res

            //expect arg res to have a status 200

            //expect arg response body collectionName to have a lengthof at least 1

            //return modelName count with no args

        //then with arg of function with arg of count

            //expect arg response body collectionName to have length of arg count


//Test 2
        // Strategy: 
        // Get back all objects, and ensure they have expected keys

//it with args 'description of test' and function with no args

        //create 2ndEmpty variable (do not reuse 1st) to save future response value

        //return chai request with arg app

        //.get with arg of '/endpoint'

        //then with arg of function with arg of _res

            //expect arg response to have status 200

            //expect arg response to be json

            //expect arg response body collectionName to have lengthof at least 1

                //response body collectionName for each with arg function arg dataName
                //expect arg dataName to be a arg 'object'
                //expect arg dataName to include keys args 'key1', 'key2', etc

                //set 2ndEmpty to response body collectionName position 0
                //return modelName findById arg id of 2ndEmpty

        //then arg function with arg dataName

            //expect 2ndEmpty id to equal dataName id

            //expect 2ndEmpty key1 to equal dataName key1

            //expect 2ndEmpty key2 to equal dataName key2

            //expect 2ndEmpty key3 to contain dataName key3.key3a or key3.key3b

            //expect 2ndEmpty key4 to equal dataName key4



//POST TESTS
        // strategy: 
        // make a POST request with data,
        // then prove that the object we get back has
        // right keys, and that `id` is there (which means
        // the data was inserted into db)

//describe with args of 'description of test block' and function with no args

//it with args 'description of test' and function with no args

    //create const 1stConst that equals generate data function with no args

    //create 1stOptional variable that is empty --Create only if needed to perform validation of specific items

    //return chai request with arg app

        //.post with arg of '/endpoint'

            //.send with arg of new 1stConst

        //then with arg of function with arg of res

            //expect arg response to have status 201

            //expect arg response to be json

            //expect arg response body to be a arg 'object'

            //expect arg response body to include specific keys args 'key1', 'key2', etc

            //expect arg response body key1 to equal 1stConst key1

            //expect response body id to not be null

            //expect response body key2 to be equal to 1stConst key2

            //expect response body key3 to be equal to 1stConst key3


                //**FOR USE WITH OPTIONAL VARIABLE
                //set 1stOptional equal to 1stConst key4 to sort with arg (a, b) => b.valB - a.valA)[0].someDataName;

                //expect response body someDataName to equal 1stOptional

                //**END OPTIONAL VARIABLE BLOCK


            //return modelName findById response body id


        //then arg function with arg dataName

            //expect dataName key1 to equal 1stConst key1

            //expect dataName key2 to equal 1stConst key2

            //expect dataName key3.key3a to equal 1stConst key3.key3a

            //expect dataName key3.key3b to equal 1stConst key3.key3b



//PUT TESTS
        // strategy:
        //  1. Get an existing object from db
        //  2. Make a PUT request to update that object
        //  3. Prove object returned by request contains data we sent
        //  4. Prove object in db is correctly updated

//describe with args of 'description of test block' and function with no args

//it with args 'description of test' and function with no args

    //create const 2ndConst {object}
        //object is test data {key1: newKey1Val, key2: newKey2Val}

    //return modelName and findOne with no args

            //then with arg of function with arg dataName 
            
            //2ndConst id equal dataName id

            //return chai request with arg app

            //.put with arg of '/endpoint/${dataName id}'

            //.send with arg of new 2ndConst

        //then with arg of function with arg of res

            //expect arg response to have status 204

            //return modelName findByID with arg 2ndConst id

        //then with arg function arg dataName

            //expect dataName key1 to equal 2ndConst key1

            //expect dataName key2 to equal 2ndConst key2


//DELETE TESTS

        // strategy:
        //  1. get a DB object
        //  2. make a DELETE request for that object id
        //  3. assert that response has right status code
        //  4. prove that object with the id doesn't exist in db anymore


//describe with args of 'description of test block' and function with no args

//it with args 'description of test' and function with no args

    //create empty variable 1stVar

    //return modelName and findOne with no args


        //then arg function with arg _1stVar

            //set 1stVar equal to _1stVar

            //return chai request with arg app chain delete with arg /endpoint/${1stVar id}

        //then arg function arg response

            //expect arg response to have status 204

            //return modelName findById arg 1stVar id

        //then arg function arg _1stVar

            //expect _1stVar to be null




~~~

[Top](#top) 
## Setup routeFile.js OR CRUD requests within server.js
<x id='route'></x>

~~~ js
//Use strict
'use strict';

**Only needed if using Express Routing**
//Create a constant to require express

//Create a constant of router equal express Router with no args

//Create const of modelName or {modelName} to require models.js - use {} if there are multiple models


//At bottom of file
//module exports the const router

**End Express Routing specific**



**Needed for all CRUD**
**If part of server.js use 'app.' instead of 'router.' and '/endPointName' and '/endPointName/:id' instead of '/' and '/:id'**

//GET - ALL

//call router or app get from '/' with args request response =>

    //modelName find with no args

        //then with arg '<DB collectionName>' =>

            //respond with json object key/val pair, key is <DB collectionName> and val is <DB collectionName> map

                //New map (arrayName) => (arrayName) through cleanUp method with no args 

    //ERROR CATCHER
    //catch err =>

        //log error err to console

        //respond status 500 and json object message stating error ocurred


//GET by ID

//call router or app get from '/:id' with args request response =>

    //modelName findById with args request params id

    //then dataName => respond json non-object with arg of dataName cleanUp method
        //where data name (ex. student or post or data) is the object being returned


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

        //create const field and assign requiredField position i during loop


    //IF field is not in the request body

        //create const for error message naming missing field from body

        //log the error to console with arg errMessage

        //return response status 400 and send errMessage


//CREATE NEW DB OBJECT
    //modelName create {object} that contains the key req body pair values

    //Within object {key1: req.body.val1, ley2: req.body.val2}
    //If key/val has sub-key/val they do not need to be specified here it would be specified 
        //in the POST request sent from postman


//then dataName => respond status 201 and json dataName send through cleanUp
    //where data name (ex. student or post or data) is the object being returned


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

            //assign the request body[dataArg] to toUpdate[dataArg] object


        //modelName call findByAndUpdate with args request params id, {object} with $set/val pair where val is toUpdate, {object} of new: true, and function w/ no args
            //respond with json message non-object 
                //ex -> res.json(`${req.body.id} has been updated`)
            
            //OR

            //respond with json object message
                //ex -> res.json({ "code": "200", "reason": "SUCCESS", "location": "", "message": `User record has been updated` })


//ERROR CATCHER
//catch err =>

    //log error err to console

    //respond status 500 with json message stating error ocurred


//DELETE

//call router or app delete from '/:id' with args request response =>

    //modelName call findByIdAndRemove with args request params id, {object} with $set/val pair where val is toUpdate, and function w/ no args
        //respond with json message non-object 
            //ex -> res.json(`${req.body.id} has been updated`)
        
        //OR

        //respond with json object message
            //ex -> res.json({ "code": "200", "reason": "SUCCESS", "location": "", "message": `User record has been updated` })


//ERROR CATCHER
    //catch err =>

    //log error err to console

    //respond status 500 with json message stating error ocurred
~~~

[Top](#top) 
