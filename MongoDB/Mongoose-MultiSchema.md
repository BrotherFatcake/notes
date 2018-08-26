<x id='top'></x>
[server.js](#server) | [config.js](#config) | [models.js](#models) | [test.js](#testfile) | [primaryRoute.js](#primaryroute) | [secondRoute.js](#secondroute)

-------------------------------------------


# Example Comments for Practice
-------------------------------------------
<x id='server'></x>
## Setup server.js

~~~ js

//Use strict
"use strict";

//Create a constant to require express

//Create a constant to require mongoose

//Create a constant to require morgan for logging

//Create a constant to require passport for authentication

//mongoose Promise equals global ES6 Promises

//Create const of PORT, DATABASE_URL to require config.js

//create const of Primary {modelName}  OR {modelName1, modelName2} to require models file

//Create constant that creates a new app instance by calling top level express with no args

//tell app to use with arg express.json with no arg

//tell app to use morgan with arg 'common' logging

**If using Express Routing by putting CRUD code in separate JS file**

//Create const for new `constNameRoute` (can name it anything) to import(require) routeFile.js (can name it anything) *duplicate for multiple route files

//tell app to use args of '/endPointName' and const specified for 'constNameRoute' *duplicate for multiple route files

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
"use strict";

//exports DATABASE_URL equals process env DATABASE_URL || "mongodb://databaseURL/dbName...";
    //For dev environment may need to swap positions - 'mongodb://databaseURL/dbName...' || process env DATABASE_URL


//exports TEST_DATABASE_URL equals process env TEST_DATABASE_URL || "mongodb://databaseURL/test_dbName...";

//exports PORT equals process env PORT || <portNum>;

~~~

[Top](#top)
## Setup models.js - Multiple DB Collection
<x id='models'></x>

~~~ js

//Use strict
"use strict";

//Create a constant to import(require) mongoose

//Create const for "Primary/Identifier" SCHEMA (this could be a collection of users that the secondary schema can reference ONE-to-MANY)
//schemaName equals mongoose Schema ({schemaObject})
    
    //Within schemaObject specify fieldName: {type: <type>, required: true/false/'do not include'}
    //Example: fieldName: {type: String, required: true}, fieldName2: {type: String}, fieldName3: {type: Array}, 
    //fieldName4: {type: String, unique: true} include nested fields


//Create const for "Other" SCHEMA (this could be used for appending data ("blog comments") associated with the Primary and Secondary schemas)
//schemaName equals mongoose Schema ({schemaObject})
//Within schemaObject specify fieldName: {type: <type>, required: true/false/'do not include'}
//Example: fieldName: {type: String}, fieldName2: {type: String, required: true}, fieldName3: {type: Array}, use nested field/s names if schema for nested items

//Create const for the "Secondary" SCHEMA (This could be a collection of user data that can be referenced against the primary schema ONE-to-MANY)
//schemaName equals mongoose Schema ({schemaObject})
    
    //Within schemaObject specify fieldName: {type: <type>, required: true/false/'do not include'}
    //Example: fieldName: {type: String, required: true}, fieldName2: {type: String}, fieldName3: {type: Array}, key: [otherSchemaName] include nested fields

    
    //within the object reference data from Primary schema
    //key: {type: mongoose Scheema Types ObjectId, ref: '<DB CollectionName>'} 

    //within the object reference data from Other schema
    //key: [otherSchemaName]



//Create Pre-Hook that can ben used by Virtuals when data is being retrieved from a different collection using a findOne()
//SecondarySchemaName pre with args of findOne and function with arg next
    
    //this populate with arg of secondary schema 'keyName' that needs to be populated

    //next with no args


//Create Pre-Hook that can ben used by Virtuals when data is being retrieved from a different collection using find()
//SecondarySchemaName pre with args of find and function with arg next
    
    //this populate with arg of secondary schema 'keyName' that needs to be populated

    //next with no args


//Create **VIRTUAL** to return more human readable sub-values
    //Example personName may have sub-values of first and last
    //schemaName virtual with arg 'virtualName' get with arg function with no args

    //return template literal placeholders this.personName.first and this.personName.last, trim with no args


//Create cleanUp function for primary schema/collection to specify what should be returned via the API
//schemaName methods methodName equal function with no args


    //return {object}
    //Within object {key1: this.val1, key2: this.val2, key3: this.virtualName}


    //Create cleanUp function for secondary schema/collection to specify what should be returned via the API
    //schemaName methods methodName equal function with no args

    //return {object}
    //Within object {key1: this.val1, key2: this.val2, key3: this.virtualName}



//Create const modelName equals mongoose model with args '<DB collectionName>' and Primary schemaName

//Create const modelName2 equals mongoose model with args '<DB collectionName>' and Secondary schemaName

//module export equal {modelName} OR {modelName1, modelName2}
~~~


[Top](#top)
## Setup PRIMARY routeFile.js OR CRUD requests within server.js
<x id='primaryroute'></x>

~~~ js
//Use strict
"use strict";

**Only needed if using Express Routing**
//Create a constant to require express

//Create a constant of router equal express Router with no args

//create const of Primary {modelName}  OR {modelName1, modelName2} to require models file
//Create const of modelName or {modelName} to require models.js - use {} if there are multiple models


//At bottom of file
//module exports equals the const router

**End Express Routing specific**



**Needed for all CRUD**
**If part of server.js use 'app.' instead of 'router.' and '/endPointName' and '/endPointName/:id' instead of '/' and '/:id'**

//GET - ALL

//call router or app get from '/' with args request response =>
    //modelName find with no args
    //then( with arg <DB collectionName> =>
        //respond with json object key/val pair, key is <DB collectionName> and val is <DB collectionName> map( with arg
        //(dataName) => (dataName) through cleanUp method with no args 
    
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
    
        //create const field and assign requiredField position i inside loop

        //IF field is not in the request body
            //create const for error message naming missing field from body
            //log the error to console with arg errMessage
            //return response status 400 and send errMessage

    
    //DETERMINE IF KEY VAL ALREADY EXISTS IN DB - this is needed when there is a UNIQUE value that must be created
    //modelName and find one {object pair}
        //Within object is {key: val} or {key: req.body.val}

    //then( dataName => 

        //IF dataName exists

            //create const for error message naming missing field from body
            //log the error to console with arg errMessage
            //return response status 400 and send errMessage
        
        
        //ELSE when dataName does not already exist continue with creation

            //CREATE NEW DB OBJECT
            //modelName create {object} that contains the key req body pair values
                //Within object {key1: req.body.val1, ley2: req.body.val2}
                //If key/val has sub-key/val they do not need to be specified here it would be specified 
                //in the POST request sent from postman

            
            //then dataName => respond status 201 and json dataName send through cleanUp
            //where data name (ex. student or post or data) is the object being returned

            //END if/else flow for unique check
            
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


    //DETERMINE IF KEY VAL ALREADY EXISTS IN DB - this is needed when there is a UNIQUE value that can be updated
    //call modelName and find one {object pair}
        //Within object is {key: val} or {key: req.body.val}

    //then dataName => 
        
        //IF dataName exists
            
            //create const for error message naming missing field from body
            //log the error to console with arg errMessage
            //return response status 400 and send errMessage
                    
        //ELSE when dataName does not already exist continue with update
    
        //modelName call findByAndUpdate with args request params id, {object} with $set/val pair where val is toUpdate, {object} of new: true, and function w/ no args
            //respond with json message non-object 
                //ex -> res.json(`${req.body.id} has been updated`)
            
            //OR

            //respond with json object message
                //ex -> res.json({ "code": "200", "reason": "SUCCESS", "location": "", "message": `User record has been updated` })
            

        //END flow for unique check



        //ERROR CATCHER
        //catch err =>
    
            //log error err to console

            //respond status 500 with json message stating error ocurred

    //DELETE
    
//call router or app delete from '/:id' with args request response =>

    //DELETE MULTIPLE SHARED VALUES IN DB FROM SECOND Collection - this is needed when there is a UNIQUE Primary Collection value being deleted in a one to many 
    //set the request ID to a parameter

    //Call secondary Model and deleteMany based on shared {key/val pair}
    
    //then dataName => respond status 204

    //END flow for unique check

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
## Setup SECONDARY routeFile.js OR CRUD requests within server.js
<x id='secondroute'></x>

~~~ js
//Use strict
"use strict";

**Only needed if using Express Routing**
//Create a constant to require express

//Create a constant of router equal express Router with no args

//create const of Primary {modelName}  OR {modelName1, modelName2} to require models file
//Create const of modelName or {modelName} to require models.js - use {} if there are multiple models


//At bottom of file
//module exports the const router

**End Express Routing specific**



**Needed for all CRUD**
**If part of server.js use 'app.' instead of 'router.' and '/endPointName' and '/endPointName/:id' instead of '/' and '/:id'**

//GET - ALL

//call router or app get from '/' with args request response =>
    //modelName find with no args
    //then with arg <DB collectionName> =>
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
 

    //Verify unique post value exists in Primary Collection
    //primaryModel find by id with args {req.body.key} and (callback) function with args err and <<callbackVarName>>

        //set value of {req.body.key} = <<callbackVarName>>

    //then dataName => 
         
            //IF not dataName 
            //create const for error message naming missing field from body
            //log the error to console with arg errMessage
            //return response status 400 and send errMessage


        //ELSE when dataName does not already exist continue with creation

        //CREATE NEW DB OBJECT
        //modelName create {object} that contains the key req body pair values

            //Within object {key1: req.body.val1, ley2: req.body.val2}
            //If key/val has sub-key/val they do not need to be specified here it would be specified 
            //in the POST request sent from postman


        //then dataName =>

            //then dataName => respond status 201 and json dataName send through cleanUp
            //where data name (ex. student or post or data) is the object being returned

                
        //ERROR CATCHER
        //catch err =>
                    //log error err to console
                    //respond status 500 with json message stating error ocurred


    //END flow for unique check

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

    //modelName findByIdAndRemove with arg request params id

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