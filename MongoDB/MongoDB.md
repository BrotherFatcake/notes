# Mongo Basics

*   Launch Mongo Shell
    *   ~~~ js
        mongo
        ~~~

    
*   List Available DBs
    *   ~~~ js
        show dbs
        ~~~

*   Switch to different DB
    *   ~~~ js
        use <dbName>
        ~~~

*   Retrieve list of collections in DB
    *   ~~~ js
        db.getCollectionNames()
        ~~~

*   Find and return all records
    *   ~~~ js
        db.<collectionName>.find()
        ~~~

*   Find and return records meeting specific criteria
    *   ~~~ js
        db.<collectionName>.find({key1: "property1", key2: "property2"})
        ~~~

*   Find and return records meeting specific criteria and return only certain fields.
    *   ~~~ js
        db.<collectionName>.find({key1: "property1", key2: "property2"}, {keyA:1, keyB:1})
        ~~~

*   Find and return records meeting specific criteria, certain fields, sort by a key, and limit number of records returned
    *   ~~~ js
        db.<collectionName>.find({key1: "property1", key2: "property2"}, {keyA:1, keyB:1}).sort({sortKey:1}).limit(5)
        ~~~

*   Import a DB from json file
    *   ~~~ js
        mongoimport --db <dbName> --collection <CollectionName> --drop --file ~/<path-to-unzipped-data-file>
        ~~~


# Create documents

*   Create as a var:
    *   ~~~ js
        var varName =   {
            key1:   "data as string testtest",
            key2:   "data as number 12345",
            key3:   {"data": "as", "an": "object"},
            key4:   [{"Data": "as", "arrayOf": "objects"}]
        }

        db.<collectionName>.insert(varName)
        ~~~
    
    *   Example
        ~~~ js
        var myRestaurant = {
        address: {},
        borough: "Queens",
        cuisine: "Colombian",
        grades: [],
        name: "Seba Seba",
        restaurant_id: "373737373737373737"
        }

        db.restaurants.insertOne(myRestaurant);
        ~~~


# Modify documents

*   Create var to identify object to update
    *   ~~~ js
        var updateVar = db.<collectionName>.find({}, {_id: 1})._id
        ~~~

*   Create updated data set
    *   ~~~ js
        db.<collectionName>.update({_id: updateVar}, {$set: {key1: "What to update"}});
        ~~~

    *   Example
        *   ~~~ js
            // update name on this single restaurant

            var objectId = db.restaurants.findOne({}, {_id: 1})._id

            // note that we're not using method chaining here to get
            // the object id. the code above is equivalent to this:
            // var myObject = db.restaurants.findOne({}, {_id: 1});
            // var objectId = myObject._id

            db.restaurants.updateOne(
            {_id: objectId},
            {$set: {name: "Foo Bar Bizz Bang"}}
            );
            
            /*see our changes*/
            
            db.restaurants.findOne({_id: objectId});
            ~~~

*   Push new **ARRAY** content to an object
    *   Determine attribute to update object/s
    *   If array does not yet exist as part of the object Mongo will add it!
    ~~~ js
    db.<collectionName>.update({keyToId: keyToIdVal}, {$push: {arrayName: {"arrayKey": "arrayKeyVal or string"}}})
    ~~~

    *   Example
        ~~~ js
        db.blogposts.update({_id: ObjectId("5b3ff56e6021f920da673593")}, {$push: {comments: {"content": "wubba dubba lub dub!!"}}})
        ~~~


# Replacing documents

*   **Be careful!** If you really want to replace, this situation is fine. But if you write your code like the code demonstrated below, thinking it will only update a subset of fields, know that it will actually replace your existing document with whatever object you supply here.

*   Create the replacement data set and replace existing document
    *   ~~~ js
        db.<collectionName>.replaceOne({key1: "key Used to ID document ex: name"}, {keyA: "key used to replace existing document"});~~~

    *   ~~~ js
        db.<collectionName>,find({keyA: "Replaced Data"});

*   Create replacement document with var
    *   Create the document by assigning to a var
        ~~~ js
        var replaceVar = {
            key1:   "data as string testtest",
            key2:   "data as number 12345",
            key3:   {"data": "as", "an": "object"},
            key4:   [{"Data": "as", "arrayOf": "objects"}]
        }
        ~~~
    *   Replace the existing document with the document assigned to the var
        ~~~ js
        db.<collectionName>.replaceOne({key1: "key Used to ID document ex: ObjectId"}, replaceVar);
        ~~~

    *   Example-
        ~~~ js
        var myRestaurant5 = {
            address: {"building": "123", "street": "South 32 Ave"},
            borough: "Queens",
            cuisine: "Colombian",
            grades: [],
            name: "Test5",
            restaurant_id: "1234567890"
        }

        db.restaurants.replaceOne({_id: ObjectId("59074c7c057aaffaafb0da6d")}, myRestaurant5)
        ~~~

# Deleting documents

*   This method deletes a single record identified by unique ObjectID
    *   Search and identify the record to verify accuracy:
        ~~~js
        db.<collectionName>.find(uniqueId("1234567890"));
        ~~~
    *   If needed can verify total count of objects:
        ~~~ js
        db.<collectionName>.find(uniqueId("1234567890")).count();
        ~~~
    *   Delete the document
        ~~~ js
        db.<collectionName>.remove(uniqueId("1234567890"));
        ~~~

*   This method can delete many records that hold a specific key/value pair
    *   Search and identify the record to verify accuracy:
        ~~~js
        db.<collectionName>.find({keyA: "keyValue"});
        ~~~
    *   If needed can verify total count of objects:
        ~~~ js
        db.<collectionName>.find({keyA: "keyValue"}).count();
        ~~~
    *   Delete the documents
        ~~~ js
        db.<collectionName>.remove({keyA: "keyValue"})
        ~~~

Examples-

*   This example deletes a single record identified by unique ObjectID
    *   Search and identify the record to verify accuracy:
        ~~~js
        db.restaurants.find(ObjectId("59074c7c057aaffaafb0da6d"));
        ~~~
    *   If needed can verify total count of objects:
        ~~~ js
        db.restaurants.find(ObjectId("59074c7c057aaffaafb0da6d")).count();
        ~~~
    *   Delete the document
        ~~~ js
        db.restaurants.remove(ObjectId("59074c7c057aaffaafb0da6d"));
        ~~~

*   This example can delete many records that hold a specific key/value pair
    *   Search and identify the record to verify accuracy:
        ~~~js
        db.restaurants.find({borough: "Queens"});
        ~~~
    *   If needed can verify total count of objects:
        ~~~ js
        db.restaurants.find({borough: "Queens"}).count();
        ~~~
    *   Delete the documents
        ~~~ js
        db.restaurants.remove({borough: "Queens"})
        ~~~


   