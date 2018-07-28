**NEED TO FORMAT**


//First create initial function call and place call at EOF ex - $(rmSearch);

//Create constant for API URL
const RMAPI = 'https://rickandmortyapi.com/api/character/';




//First function that is in the EOF call
function rmSearch()  {
//Submit listener function to listen to submit or click input events on specified HTML doc class
    $('.js-rmsearch').submit(function(event)    {

    //prevent default behaviors from form submit
        event.preventDefault();

    //create variable by listening for value entered into search box
        let searchTerm = $('.js-searchBox').val()

        console.log('searchTerm: ', searchTerm);

    //function call to next function and name of (call to) render function.  
        //Data returned from callback will send to next function
        rmApiCAll(searchTerm, rmRender);

    });

};

//API function has 2 args - searchTerm from first and callback to send data BACK to first function
function rmApiCAll(searchTerm, callback)    {

    //create variable object for search string - this example could be handled without if const URL included name params
    let query = {
        name: `${searchTerm}`
        }

    //jQuery getJSON call that includes API URL const, query, and callback to return data to first function
    $.getJSON(RMAPI,query, callback);

};

//render results function, single arg of data
function rmRender(data) {
    
    //create const for new array after mapping data arg
    const rmChar = data.results.map(function(data) {
      //  console.log('map data: ', data)

      //return params present in data as HTML to pass to lower jQuery html display
        return ` 
        <div class='rickANDmorty'>
        <ul>
            <li>Name: ${data.name}</li>
            <li>Origin: ${data.origin.name}</li>
            <li><img src='${data.image}' alt='RM Image'></li>
        </ul>
    
        </div>
        `;

    })

    
    console.log('rmChar: ', rmChar)

    //renter returned HTML to screen at appropriate class object
    $('.js-rmInfo').html(rmChar)

}






$(rmSearch);




//First create initial function call and place call at EOF ex - $(rmSearch);

//Create constant for API URL


//First function that is in the EOF call
function rmSearch()  {
//Submit listener function to listen to submit or click input events on specified HTML doc class
    $('.js-rmsearch').submit(function(event)    {

    //prevent default behaviors from form submit
        event.preventDefault();

    //create variable by listening for value entered into search box
        let searchTerm = $('.js-searchBox').val()

        console.log('searchTerm: ', searchTerm);

    //function call to next function and name of (call to) render function.  
        //Data returned from callback will send to next function
        rmApiCAll(searchTerm, rmRender);

    });

};

//API function has 2 args - searchTerm from first and callback to send data BACK to first function
function rmApiCAll(searchTerm, callback)    {

    //create variable object for search string - this example could be handled without if const URL included name params
    let query = {
        name: `${searchTerm}`
        }

    //jQuery getJSON call that includes API URL const, query, and callback to return data to first function
    $.getJSON(RMAPI,query, callback);

};

//render results function, single arg of data
function rmRender(data) {
    
    //create const for new array after mapping data arg
    const rmChar = data.results.map(function(data) {
      //  console.log('map data: ', data)

      //return params present in data as HTML to pass to lower jQuery html display
        return ` 
        <div class='rickANDmorty'>
        <ul>
            <li>Name: ${data.name}</li>
            <li>Origin: ${data.origin.name}</li>
            <li><img src='${data.image}' alt='RM Image'></li>
        </ul>
    
        </div>
        `;

    })

    
    console.log('rmChar: ', rmChar)

    //renter returned HTML to screen at appropriate class object
    $('.js-rmInfo').html(rmChar)
