**Initial Express setup**

1. Create a constant to import(require) Express
    const express = require('express');

2. Create constant that creates a new app instance by calling top level Express function
    const app = express();

3. If static assets to serve
    app.use(express.static('public')); *standard location is public, can name whatever I want "ex. /static/index.html"

4. listen for requests and log when server started
```
    app.listen(process.env.PORT || 8080, () => {
        console.log(`Your app is listening on port ${process.env.PORT || 8080}`);
    });
```
**Express Basics**

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

**Simple GET JSON response**

`res.json` converts JavaScript objects to JSON and appropriately sets the Content-Type header to application/json; charset=utf-8 . By default, we'll get a 200 HTTP status code.

```
app.get('/', (req, res) => res.json({foo: 'bar'}));
```

What this does-

app[our previous const]

.get[GET method]

('/'[calls root of server]

, (req, res) => res.json({foo:bar}));[converts JS object to JSON, sets header to application/json and returns object]

**Path variable example**

```
// this would normally be stored in DB
const studentData = [
  {
    studentId: '1546906',
    studentName: 'Sally Student',
    currentGrade: 'A',
  },
  {
    studentId: '2300457',
    studentName: 'Thaddeus Think',
    currentGrade: 'A',
  },
  {
    studentId: '9920711',
    studentName: 'Jason Javascript',
    currentGrade: 'B',
  }
];

// user makes a request to 'url.com/123456789'

app.get('/:studentID', (req, res) => {

// create const that assigns 'req.params.studentID'
    const {studentID} = req.params;
// create unassigned variable to send requested data in response later
    let requestedData;

// loop through StudentData array to find studentID that matches URL 'req.params.studentID'
    for (let i = 0; i<studentData.length; i++)  {
        if (studentData[i].studentID === studentId) {
// sets the matching studentData key value to requestedData
            requestedData = studentData[i]
            }
        };
// returns the requested studentData values to user
        res.json(requestedData);
    });

```



**Conditional logic and status example**
```
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

```



**POST**


//Add at head of server.js
//Create const of bodyParser and require body-parser to parses POST values

const bodyParser = require('body-parser');

//create const to use call bodyParser and parse incoming json requests
const jsonParse = bodyParser.json();


//create post function to listen for the post at junkData endpoint
    //identify endpoint, call jsonParse before req, res args
app.post('/junkData', jsonParse, (req, res) =>  {

    //create const for required POST fields
    const requiredFields = ['name', 'location'];

    //loop through json object keys presented in POST
    for (let i=0; i<requiredFields.length; i++) {

    //create const to check for field existing using IF
    //if not field in request body
    if (!(field in req.body))   {

        //send message if field/s missing
        const message = `Missing \'${field}\' in request`
        //log error message to console
        console.error(message);

        //return 400 status and send error message in json response
        return res.status(400).send(message);
        
        }

    }

    //if field in response 
    //create const to create item with the request name and location in request by passing it to functionName.create
    const item = functionName.create(req.body.name, req.body.location);

    //respond status 200 and json object item
    res.status(200).json(item);
})


platform named Jekyl that is good for blogging w/ GitHub pages

