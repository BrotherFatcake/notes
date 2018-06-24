# Install and Setup Mocha/Chai

1.  Create npm project.

        npm init

2.  Install Mocha using --save-dev so that Mocha is not made a dependency in Prod
    
        npm install mocha --save-dev

3.  Update package.json file of npm project to use mocha for testing

        "scripts":  {
            "test": "mocha --exit"
        },

    Once saved the command 'npm test' can be used

4.  Install Chai using --save-dev so that Chai is not made a dependency in Prod

        npm install chai --save-dev

# Unit testing

1.  Update the testing file

        //create const to require/import the under test
        const fileToTest = require('../fileToTest');

        //create const that can be used as 'expect' variable to require/import Chai
        const expect = require('chai).expect;

2.  Create the tests

        //describe the function and call the fileToTest and function call
        describe('fileToTest', function()   {

        //what IT does - description of the test and function call 
            it('should give the correct result', function() {

                //const of data array to be passed to function under test.  Ex, if adding 2 numbers
                    //provide 2 numbers and expected result.
                const testInputs = [TestDataArray];

                //POS/NEG - for each value of array pass as input into a function, within function create const to recv answer when input passed to test file.
                testInputs.forEach(function(input)  {
                
                //set result to const named answer
                    const answer = fileToTest(input.a, input.b);
                
                //assert that const answer is expected to be equal to the expected result key/value in TestInputs
                    expect(answer).to.equal(input.expected);

                //ERROR Tests - Expect function with block to run test file with inputs, and to expect Error 
                    expect(function()    {
                        fileToTest(input);
                    }).to.throw(Error);

                });
            });
        });


# Integration Testing

1.  Create constants to require 'chai' and 'chai-http'

        const chai = require('chai');
        const chaiHttp = require('chai-http');

2.  Import server.js and use destructuring assignment to create variables for server.app, server.runServer, server.closeServer

        const {app, runServer, closeServer} = require('../server');

3.  Declare a const for expect from the chai import

        const expect = chai.expect;

4.  Tell the app to use chaiHttp

        chai.use(chaiHttp);


5.  Setup the function to Start/Stop the server

        let server;

        function runServer() {
        const port = process.env.PORT || 8080;
        return new Promise((resolve, reject) => {
            server = app.listen(port, () => {
            console.log(`Your app is listening on port ${port}`);
            resolve(server);
            })
            .on('error', err => {
            reject(err);
            });
        });
        }

        //... closeServer defined here

        if (require.main === module) {
        runServer().catch(err => console.error(err));
        };



6.  Create the RUN/CLOSE Server functions in the test file

        describe('testOperation', function()    {

            //Starts the server before the tests run.  `runServer` function returns a promise, and we return the promise by doing `return runServer`.

            before(function()   {
                return runServer();
            });
            
            //Stops the server when the tests are complete.
            after(function()    {
                return closeServer();
            });
        });
    



7.  describe the test/s

    **GET Example**

        //What IT does - description of the test and function call 
            it('should test something', function()  {
            //return/call chai.request on variable app - this allows to make server requests
                return chai.request(app)
            //make get request to the endpoint
                .get('/endPoint')
            
            //then run the assertions on the response and verify expected results of the test
                //expect correct status, to be json, to be ('array/object'), to be above specific length
                    //if initial response is an array of objects can look into array with forEach function
                    //and assert expected result items to be object and to have all keys.
                .then(function(res) {
                    expect(res).to.have.status(200);
                    expect(res).to.be.jason;
                    expect(res.body).to.be.a('array');
                    expect(res.body.length).to.be.above(0);
                    res.body.forEach(function(item) {
                        expect(item).to.be.a('object');
                        expect(item).to.have.all.keys(
                            'key1', 'key2', 'key3');
                        });
                    });
                });
            });

    **POST Example**

        //What IT does - description of the test and function call 
        it('should add an item on POST', function() {
        
        //Create constant containing data (object/array/etc) to send to function/endpoint under test
        const newItem = {name: 'coffee', checked: false};

        //return/call chai.request on variable app - this allows to make server requests
        return chai.request(app)
        
        //make get request to the endpoint
            .post('/shopping-list')
        //send the test data
            .send(newItem)

        //then run the assertions on the response and verify expected results of the test
                //expect correct status, to be json, to be ('array/object'), to be above specific length
                    //if initial response is an array of objects can look into array with forEach function
                    //and assert expected result items to be object and to have all keys.
            .then(function(res) {
            expect(res).to.have.status(201);
            expect(res).to.be.json;
            expect(res.body).to.be.a('object');
            expect(res.body).to.include.keys('id', 'name', 'checked');
            expect(res.body.id).to.not.equal(null);
            // response should be deep equal to `newItem` from above if we assign
                // `id` to it from `res.body.id`
                //deep equal compares the input test data to the response data
            expect(res.body).to.deep.equal(Object.assign(newItem, {id: res.body.id}));
            });
        });

    
    **PUT Example**

        //What IT does - description of the test and function call 
        it('should update items on PUT', function() {

        //Create constant containing data (object/array/etc) to send to function/endpoint under test
        const updateData = {
        name: 'foo',
        checked: true
        };

        //return/call chai.request on variable app - this allows to make server requests
        return chai.request(app)
     
        // first have to GET so we have an idea of object to update
        .get('/shopping-list')
     
        .then(function(res) {
        //After GET returns set updateData key/val pair of id to the response body id 
            updateData.id = res.body[0].id;

        //return/call chai.request on variable app - this allows to make server requests
            return chai.request(app)

        //make the PUT call to the endpoint with key of object to update
            .put(`/shopping-list/${updateData.id}`)
            .send(updateData)
        })

        //then run the assertions on the response and verify expected results of the test
                //expect correct status, to be json, to be ('array/object'), to be above specific length

        .then(function(res) {
            expect(res).to.have.status(200);
            expect(res).to.be.json;
            expect(res.body).to.be.a('object');

            // response should be deep equal to `newItem` from above if we assign
                // `id` to it from `res.body.id`
                //deep equal compares the input test data to the response data
            expect(res.body).to.deep.equal(updateData);
            });
        });


    **DELETE Example**

        //What IT does - description of the test and function call 
        it('should delete items on DELETE', function() {

        //return/call chai.request on variable app - this allows to make server requests
            return chai.request(app)
            // first have to GET so we have an `id` of item
            // to delete
            .get('/shopping-list')

        //After GET completes then call chai request and send DELETE to endpoint with key of object to delete
            .then(function(res) {
                return chai.request(app)
                .delete(`/shopping-list/${res.body[0].id}`);
            })

        //then run the assertions on the response and verify response status to have 204
            .then(function(res) {
                expect(res).to.have.status(204);
            });
        });