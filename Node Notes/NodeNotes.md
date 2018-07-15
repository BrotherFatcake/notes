# Express setup
-------------------------------------------

## server.js setup

1.  Create a constant to require Express
    
        const express = require('express');

2.  Create a constant to import/require Morgan for logging
    
        const morgan = require('morgan');

3.  Create constant that creates a new app instance by calling top level Express function
    
        const app = express();

4. **When using Express Routing-** 
    
    Create const values to import/require module JS files (moduleOneRouter.js & moduleTwoRouter.js - the .js is not used to setup the params)

        const moduleOneRouter = require('./moduleOneRouter');
        const moduleTwoRouter = require('./moduleTwoRouter');

5.  Use Morgan for logging http layer

        app.use(morgan('common'));

6.  **When static assets will be served-**
    
        app.use(express.static('public')); *standard location is public, can name whatever I want "ex. /static/index.html"

7.  **When using Express Routing-**

    Tell app to route requests at specific endpoints to the appropriate /moduleOne and /moduleTwo routers

        app.use('/endpointOne', moduleOneRouter);
        app.use('/endpointTwo', moduleTwoRouter);

8.  listen for requests and log when server started

        app.listen(process.env.PORT || 8080, () => {
            console.log(`Your app is listening on port ${process.env.PORT || 8080}`);
        });


## Route file setups

1.  Create a constant to require Express
    
        const express = require('express');

2.  Create a constant named router to call Express Router
    
        const router = express.Router(); 

3.  Create const of bodyParser and require body-parser to parse POST/PUT values
        
        const bodyParser = require('body-parser');

4.  Create const to use call bodyParser and parse incoming json requests
        
        const jsonParser = bodyParser.json();

5.  Create const of imported/required object array in model file

        const {objArray} = require('./models');

6.  Module exports router  - this goes at END of file

        module.exports = router;

**THINGS TO NOTE**
    
- When calling **GET, POST, PATCH, PUT, and DELETE** call **router.get, router.post, etc.**  No longer call app.get, etc in this setup.
- Include below to create example objects to GET immediately **this is not needed when using a DB**
            
        objArray.create('key value goes here (ex. name)', ['val1', 'val2', 'val3']); -  (an array of values can be provided)
        objArray.create('key value goes here (ex. name)', 'key value goes here (ex. description)'); - (a single string/num/etc can be provided)



# Express Basics
-------------------------------------------

The role of a web server is to respond to HTTP requests from clients with HTTP responses. Express provides a request and a response object for representing and interacting with HTTP requests and responses.

Express supports a large number of HTTP methods, but the ones you'll most commonly use are **GET, POST, PATCH, PUT, and DELETE**. These correspond to the Express methods app.get(), app.post(), app.patch(), app.put(), and app.delete(), respectively.

* GET is used to read or retrieve resources
* POST is used to create new resources
* PATCH is used to update part of an existing resource
* PUT is used to replace an existing resource
* DELETE is used to delete resources

The request method along with the request path are used to route the request to the right request handler, which is a function that knows how to supply the requested resource. 
HTTP responses default to 200. If the response contains content, that content goes in the response body.
![Alt](https://tf-curricula-prod.s3.amazonaws.com/curricula/dd41a329d3f0aee6e10c87c70b2895e1/NODE-001/v5/assets2/1.1.4/RequestReponseThinksheet.png "URL Parts and Req/Res")


# POSTMAN SETUP
-------------------------------------------

### GET
Call the endpoint:

    All data = endpoint.com:8080/endpoint

    Parameterized data = endpoint.com:8080/endpoint/123456789

### POST

Set headers to:

    Key: Content-Type
    Value: application/json

Must provide json object in BODY:

    {"key1": "key1Val",
        "key2": "key2Val",
        "key3": "key3Val"
        "key4": ["val1", "val2", "val3"]
    }

Call the endpoint:

    endpoint.com:8080/endpoint
 
### DELETE

Set headers to:

    Key: Content-Type
    Value: application/json

Call the endpoint with id of object to delete:

    Parameterized data = endpoint.com:8080/endpoint/123456789

### PUT

Set headers to:

    Key: Content-Type
    Value: application/json

Must provide json object in BODY with updated values, identifying key must match existing object identifying key:

    {"key1": "key1Val",
        "key2": "key2Val",
        "key3": "key3Val"
        "key4": ["val1", "val2", "val3"]
    }

Call the endpoint with id of object to update:

    Parameterized data = endpoint.com:8080/endpoint/123456789


# REQUEST EXAMPLES
-------------------------------------------

## GET

### Simple GET JSON response

`res.json` converts JavaScript objects to JSON and appropriately sets the Content-Type header to application/json; charset=utf-8 . By default, we'll get a 200 HTTP status code.

~~~ js
app.get('/', (req, res) => res.json({foo: 'bar'}));
~~~

What this does-

app[our previous const]

.get[GET method]

('/'[calls root of server]

, (req, res) => res.json({foo:bar}));[converts JS object to JSON, sets header to application/json and returns object]





### Path variable examples**

~~~ js
// this would normally be stored in DB

const exampleData = [
  {
    id: '1546906',
    name: 'Sally Student',
    grade: 'A',
  },
  {
    id: '2300457',
    name: 'Thaddeus Think',
    grade: 'A',
  },
  {
    id: '9920711',
    name: 'Jason Javascript',
    grade: 'B',
  }
];
~~~

~~~ js
//GET request that returns all array items as json object

app.get('/endpoint', (req, res) => {
  res.status(200).json(exampleData)
})

~~~

~~~ js
// GET request to 'url.com/endpoint/123456789' to retrieve specific item at endpoint + :id
app.get('/endpoint/:id', (req, res)  =>  {

  // create const that assigns request param id to new const
  const searchID = req.params.id;

  // create unassigned variable to hold matching search data and to send in response
  let itemID;

  // loop through data array to find the item ID that matches URL request param id
  for(let i=0; i<recipeList.length; i++)  {

    // if searchID (request param id) is found to match item key value ID, set the item object to 
      //previous unassigned variable
    if(searchID === recipeList[i].id)  {
        itemID = recipeList[i];
    }
   }

   // responds with the requested object data values
   res.status(200).json(itemID);

});

~~~



### Conditional logic and status example**
~~~ js
// this is stubbed. random-ishly returns true or false
const canAccessEndpoint = (req) => Boolean(Math.floor(Math.random() * 2));

app.get('/', (req, res) => {
// if canAccessEndpoint is false return console message.
  if (!canAccessEndpoint(req)) {
    console.log(
      `unauthorized request for ${req.originalUrl} by an anonymous intruder`);
    res.status(401).send("i won't let you");
  }
// if canAccessEndpoint is true return 'ok'
  else {
    res.send('ok');
  }
});

~~~



## POST


//Add at head of server.js
//Create const of bodyParser and require body-parser to parse POST values

const bodyParser = require('body-parser');

//create const to use call bodyParser and parse incoming json requests
const jsonParser = bodyParser.json();


~~~ js
//create post function to listen at named endpoint
    //identify endpoint, call jsonParser before req, res args **If using routing may only need to specify '/'**
app.post('/endpoint', jsonParser, (req, res) =>  {

    //create const for array of required POST fields
    const requiredFields = ['name', 'ingredients'];

    //loop through json object keys presented in request the length of requiredFields
  for(let i=0; i<requiredFields.length; i++) {
    
    //create const to check for field existing using IF statement at current array position i
    const field = requiredFields[i];
    
    //if not field in request body
    if(!(field in req.body))  { 
      //send message if field/s missing
      //log error message to console
      
       console.error(`${field} is missing`);
      //respond with 400 status and send error message in json response
        res.status(400)
    }

      else {
        //if field in request 
        //create const to create item with the request name and ingredient in request by passing it to endpoint.create
        const item = endpoint.create(req.body.name, req.body.ingredients);

        //respond with status 200 and json object item
        res.status(200).json(item);

      };
  };
~~~


## DELETE

Like GET and POST, DELETE will call similar endpoint but in example calling path variable for id of item to be deleted

~~~ js
    localhost:8080/endpoint/:id
~~~

Will provide the id to endpoint.delete();

~~~ js
//create delete function to listen at named endpoint + :id
app.delete('/endpoint/:id', (req, res)  =>  {
    
    //call delete operation and include the id from the request params (not request body)
    endpoint.delete(req.params.id);
    
    //log a message stating item was deleted and include the id in the request params (not request body)
    console.log(`\'${req.params.id}\' was removed`);

    //respond with 200 status and end
    res.status(200).end();
});
~~~

## PUT (update)

Like DELETE we'll call similar endpoint but in example calling path variable for id of item to be updated

Like POST will need to parse data sent by client and validate that required fields are present, if correct will call update operation


//Add at head of server.js if doesn't already exist
//Create const of bodyParser and require body-parser to parse PUT values

const bodyParser = require('body-parser');

//create const to use call bodyParser and parse incoming json requests
const jsonParser = bodyParser.json();

~~~ js
//create put function to listen at named endpoint + :id
        //identify endpoint, call jsonParser before req, res args
    app.put('/endpoint/:id', jasonParser,  (req, res)  =>  {
    
    //create constant to identify array of requiredFields
    const = requiredFields = ['name', 'ingredients', 'id'];

    //loop through json object keys presented in request the length of requiredFields
    for (let i=0; i < requiredFields; i++)  {

    //create const of field of requiredFields[i] to be used to check field at array[i] exists in IF statement
        const field = requiredFields[i];

    //if not field in request body
        if(!(field in req.body))    {

      //log error message to console with message that field is missing
        console.log(`${field} is not in the request`);

      //respond with 400 status and send error message in json response
        res.status(400);

        }  
    }

    //check if the request param ID is NOT equal to the request body ID log an error and return 400 status
        if(req.prams.id !== req.body.id)    {
            console.error(`${req.params.id} and ${req.body.id} do not match`);
            return res.status(400);
        }
        
        //when ID matches log message of update to request param ID
            console.log('${req.params.id} has been updated.`);
        
    //call update operation with object of request param id and request body object key/values
            operation.update({
                id: req.params.id,
                name: req.body.name,
                ingredients: req.body.ingredients
            });
        
        //respond with status 204 and end
            res.status(204).end();

});

 ~~~   

# Example Comments for Practice
-------------------------------------------

## Setup


//Create a constant to require Express

//Create a constant to import/require Morgan for logging

//Create constant that creates a new app instance by calling top level Express function

//**When using Express Routing-** Create const values to import/require module JS files (moduleOneRouter.js & moduleTwoRouter.js - the .js is not used to setup the params)

//Tell app to use morgan for common http logging

//Tell app to use express static location of public for static assets

//**When using Express Routing-** Tell app to use specific endpoints for the appropriate /moduleOne and /moduleTwo routers

//Listen for requests and log when server started
/*
    app.listen(process.env.PORT || 8080, () => {
        console.log(`Your app is listening on port ${process.env.PORT || 8080}`);
    });
*/

**Route file setups**

//Create a constant to require Express

//const express = require('express');

//Create a constant named router to call Express Router

//Create const of bodyParser and require body-parser to parse POST/PUT values

//Create const to call bodyParser and parse json requests

//Create const object of imported/required object array in models file

//module exports router - this goes at END of file



## GET

### Simple GET JSON response

`res.json` converts JavaScript objects to JSON and appropriately sets the Content-Type header to application/json; charset=utf-8 . By default, we'll get a 200 HTTP status code.

~~~ js
app.get('/', (req, res) => res.json({foo: 'bar'}));
~~~

What this does-

app[our previous const]

.get[GET method]

('/'[calls root of server]

, (req, res) => res.json({foo:bar}));[converts JS object to JSON, sets header to application/json and returns object]





### Path variable examples**

**Example Data**
~~~ js
// this would normally be stored in DB

const exampleData = [
  {
    id: '1546906',
    name: 'Sally Student',
    grade: 'A',
  },
  {
    id: '2300457',
    name: 'Thaddeus Think',
    grade: 'A',
  },
  {
    id: '9920711',
    name: 'Jason Javascript',
    grade: 'B',
  }
];
~~~


## GET

// GET request to 'url.com/endpoint' to retrieve all items at endpoint

OR

// GET request to 'url.com/endpoint/123456789' to retrieve specific item at endpoint + :id

// create const that assigns request param id to new const

// create unassigned variable to hold matching search data and to send in response

// loop through data array to find the item ID that matches URL request param id

// if searchID (request param id) is found to match item key value ID, set the item object to 
    //previous unassigned variable

// responds with the requested object data values




## POST


//Add at head of server.js
//Create const of bodyParser and require body-parser to parse POST values

//create const to use call bodyParser and parse incoming json requests


//create post function to listen at named endpoint
    //identify endpoint, call jsonParser before req, res args  **If using routing may only need to specify '/'**

//create const for array of required POST fields

//loop through json object keys presented in request the length of requiredFields

//create const to check for field existing using IF statement at current array position i
    
//if not field in request body
    //send message if field/s missing
    //log error message to console
      
//respond with 400 status and send error message in json response

//if field in request 
    //create const to create item with the request name and ingredient in request by passing it to endpoint.create

//respond with status 200 and json object item



## DELETE

Like GET and POST, DELETE will call similar endpoint but in example calling path variable for id of item to be deleted

//Will provide the id to endpoint.delete();
    //localhost:8080/endpoint/:id


//create delete function to listen at named endpoint + :id
    
//call delete operation and include the id from the request params (not request body)

//log a message stating item was deleted and include the id in the request params (not request body)

//respond with 200 status and end


## PUT (update)

Like DELETE we'll call similar endpoint but in example calling path variable for id of item to be updated

Like POST will need to parse data sent by client and validate that required fields are present, if correct will call update operation


//Add at head of server.js if doesn't already exist
    //Create const of bodyParser and require body-parser to parse PUT values

//create const to use call bodyParser and parse incoming json requests


//create put function to listen at named endpoint + :id
    //identify endpoint, call jsonParser before req, res args
    
//create constant to identify array of requiredFields

//loop through json object keys presented in request the length of requiredFields

//create const of field of requiredFields[i] to be used to check field at array[i] exists in IF statement

//if not field in request body

//log error message to console with message that field is missing

//respond with 400 status and send error message in json response


//check if the request param ID is NOT equal to the request body ID log an error and return 400 status

//when ID matches log message of update to request param ID
        
//call update operation with object of request param id and request body object key/values

             
//respond with status 204 and end


 