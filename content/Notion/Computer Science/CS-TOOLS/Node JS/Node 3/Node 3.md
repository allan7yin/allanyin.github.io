## Asynchronous NodeJS

  

We can learn this through creating a weather app. So, what do we mean when we say that Node is asynch? Node is asynch in that it does not execute lines of code in the order that it reads it. So, it is not like C++, where it reads each line and executes it. Consider the code below:

```JavaScript
console.log('starting');
 
setTimeout(()=>{ // this makes this function execute after 2000 miliseconds. 
    console.log('2 seconds later');
}, 2000)


console.log('stopping');
```

  

As expected, it will output

```JavaScript
starting
stopping
2 seconds later
```

  

Now, what about this code snippet?

```JavaScript
console.log('starting');
 
setTimeout(()=>{ // this makes this function execute after 2000 miliseconds. 
    console.log('2 seconds later');
}, 2000)

setTimeout(()=>{ // this makes this function execute after 0 miliseconds. 
    console.log('0 seconds later');
}, 0)

console.log('stopping');
```

  

So, we should expect something like below right?

```JavaScript
0 seconds later
starting
stopping
2 seconds later
```

  

Nope, thats’ now this works. The actual output will be:

```JavaScript
starting
stopping
0 seconds later
2 seconds later
```

Let’s learn why this is the case. The reason is very simple and is due to how NodeJS is designed. When a function is called, it is added to call stack. So, like the below:

  

![[IMG_3832.jpg]]

So, as seen above, the entire code snippet is first run under `main()`. When we reach `console.log(’Starting up’)`, that is a function and is added to the call stack. There, it is executed and so, on the console, we see the words “starting up”. Then, we read in the two `setTimeout` , which are set to execute after a set period of time, so they are added to the Node API. Then, we go down the javascript code, and reach `console.log(’Finishing up’)`, which is then added to the call stack. Then, the main() is popped from the stack, and so, now out call stack is empty. So, when ready, we move the executables in Node API’s into the Callback Queue, where, once the call stack is empty, they are popped off the queue into the call stack to be executed. Hence, this is why we see the ‘zero seconds’ logged into the console, although more than “0 seconds” has passed.

  

![[IMG_3833.jpg]]

  

### Making HTTP Requests + Customizing HTTP Requests

  

To learn this, we will access a weather API to build a weather application. Before we make any HTTP requests from node, let’s see how we do in the browser. For this demonstration, we will use weatherstack api with the following account details:

![[Screen_Shot_2022-09-21_at_9.42.53_AM.png]]

Now, let’s say we wanted to call this api through the browser to retrieve some sort of data. So, to make a HTTP Request, we of course start off with the HTTP. Basically, we start with the [`http://api.weatherstack.com`](http://api.weatherstack.com) and we append things we want to the end.

  

Say we wanted to retrieve current weather data. So, we need to add a query string to the URL. A query string always starts with a ?, and is in key, value pairs.

```JavaScript
http://api.weatherstack.com/current?key=value
```

  

So, in this case, the final query in the browser will look something like:

```JavaScript
http://api.weatherstack.com/current?access_key=bcfa648008114776bf42df4bcf972b53&query=37.8267,-122.4233 

// we are entering coordinates for the weather of that location, the way the API 
// is designed is like this
```

  

This will return a JSON in the browser. As seen before, this JSON is easily parsed, and then, we can easily access select data we want from the data. Hence, we have successfully called on the weatherstack api. This is the response JSON:

```JavaScript
{"request":{"type":"LatLon","query":"Lat 37.83 and Lon -122.42","language":"en","unit":"m"},"location":{"name":"North Beach","country":"United States of America","region":"California","lat":"37.806","lon":"-122.411","timezone_id":"America\/Los_Angeles","localtime":"2022-09-21 07:09","localtime_epoch":1663744140,"utc_offset":"-7.0"},"current":{"observation_time":"02:09 PM","temperature":17,"weather_code":296,"weather_icons":["https:\/\/assets.weatherstack.com\/images\/wsymbols01_png_64\/wsymbol_0017_cloudy_with_light_rain.png"],"weather_descriptions":["Light Rain"],"wind_speed":9,"wind_degree":220,"wind_dir":"SW","pressure":1014,"precip":0,"humidity":90,"cloudcover":100,"feelslike":17,"uv_index":1,"visibility":13,"is_day":"yes"}} 
```

  

Here is the code that makes the HTTP request, and analyzes the returned data:

```JavaScript
const request = require('request');
const url = 'http://api.weatherstack.com/current?access_key=bcfa648008114776bf42df4bcf972b53&query=37.8267,-122.4233';


// so how do we use the request function? The first parameter is an options object that outlines what we would like to do
// such as the url and other information. The second parameter is the function to execute once the data has been retrieved. 

/*
request({ url: url}, (error, response) => {
    const data = JSON.parse(response.body);
    console.log(data.current);
})

*/

// Now, we can simplify the above way by passing an additional parameter into the response, we can get the request module to parse for us

request({url: url, json: true}, (error, response) => { // json: true parses it for us, no need to parse then
    const currTemp = response.body.current.temperature;
    const feelsTemp = response.body.current.feelslike;

    console.log('It is currently ' + currTemp + ' degrees out. It feels like ' + feelsTemp + ' degrees out.');
})

// YOU NEED TO FIX THE ISSUES WITH CHROME, HTTP SITE IS BEING REDIRECTED TO HTTPS, so can't access the the JSON response from the weatherstack api 

// As a pratcice excercise, print out the current temperature and what the temperature "feels like" (DONE, ABOVE)
```

  

Moving forward, we’ll being building our weather application. To do so, we’ll use another API, which is position-stack.

  

![[Screen_Shot_2022-09-21_at_12.29.54_PM.png]]

  

So, using this API, we can obtain information on a certain location via an API call:

```JavaScript
// Entering this in the browser:
http://api.positionstack.com/v1/forward?access_key=472f8191838dbe4f50b9d56228ffd19b&query=Oakville

// This returns a large JSON
{
  "data": [
    {
      "latitude": 43.409235,
      "longitude": -79.651457,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Ontario",
      "region_code": "ON",
      "county": null,
      "locality": "Oakville",
      "administrative_area": null,
      "neighbourhood": null,
      "country": "Canada",
      "country_code": "CAN",
      "continent": "North America",
      "label": "Oakville, ON, Canada"
    },
    {
      "latitude": 41.597559,
      "longitude": -73.086231,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Connecticut",
      "region_code": "CT",
      "county": "Litchfield County",
      "locality": "Oakville",
      "administrative_area": "Watertown",
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, CT, USA"
    },
    {
      "latitude": 38.453155,
      "longitude": -90.310083,
      "type": "localadmin",
      "name": "Oakville Township",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Missouri",
      "region_code": "MO",
      "county": "St. Louis County",
      "locality": null,
      "administrative_area": "Oakville Township",
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville Township, MO, USA"
    },
    {
      "latitude": 31.85266,
      "longitude": -84.46408,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Georgia",
      "region_code": "GA",
      "county": "Terrell County",
      "locality": "Oakville",
      "administrative_area": null,
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, GA, USA"
    },
    {
      "latitude": 36.15368,
      "longitude": -89.89648,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Missouri",
      "region_code": "MO",
      "county": "Pemiscot County",
      "locality": "Oakville",
      "administrative_area": "Braggadocio Township",
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, MO, USA"
    },
    {
      "latitude": 38.39124,
      "longitude": -76.64357,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Maryland",
      "region_code": "MD",
      "county": "St. Mary\u0027s County",
      "locality": "Oakville",
      "administrative_area": null,
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, MD, USA"
    },
    {
      "latitude": 40.79839,
      "longitude": -80.65924,
      "type": "locality",
      "name": "Signal",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Ohio",
      "region_code": "OH",
      "county": "Columbiana County",
      "locality": "Signal",
      "administrative_area": "Elkrun Township",
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Signal, OH, USA"
    },
    {
      "latitude": 36.74893,
      "longitude": -86.87555,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Kentucky",
      "region_code": "KY",
      "county": "Logan County",
      "locality": "Oakville",
      "administrative_area": null,
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, KY, USA"
    },
    {
      "latitude": 36.49765,
      "longitude": -78.10055,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "North Carolina",
      "region_code": "NC",
      "county": "Warren County",
      "locality": "Oakville",
      "administrative_area": null,
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, NC, USA"
    },
    {
      "latitude": 38.22151,
      "longitude": -75.62048,
      "type": "locality",
      "name": "Oakville",
      "number": null,
      "postal_code": null,
      "street": null,
      "confidence": 1,
      "region": "Maryland",
      "region_code": "MD",
      "county": "Somerset County",
      "locality": "Oakville",
      "administrative_area": null,
      "neighbourhood": null,
      "country": "United States",
      "country_code": "USA",
      "continent": "North America",
      "label": "Oakville, MD, USA"
    }
  ]
}
```

  

From this, we can retrieve the data we need to build are application as it returns latitude and longitude, which we can then use to retrieve data from the weather stack API.

  

### Handling Errors

  

Essentially, as we are able to take in an error and a response object, we can first determine if there is some sort of error we need to handle before we move forward.

  

Error handing notes are contained in the code below:

```JavaScript
const request = require('request');
// const url = 'http://api.weatherstack.com/current?access_key=bcfa648008114776bf42df4bcf972b53&query=37.8267,-122.4233';
const geocodeurl = 'http://api.positionstack.com/v1/forward?access_key=472f8191838dbe4f50b9d56228ffd19b&query=Los Angeles';


// so how do we use the request function? The first parameter is an options object that outlines what we would like to do
// such as the url and other information. The second parameter is the function to execute once the data has been retrieved. 

/*
request({ url: url}, (error, response) => {
    if (error) { // error handling 
        console.log('Unale to connect to weather service');
    } else if (response.body.error) {
        console.log('Unable to find location'); // we could also log some other important details about the type of error contained in response. 
    } else {
        const data = JSON.parse(response.body);
        console.log(data.current);
    }
})

*/

// Now, we can simplify the above way by passing an additional parameter into the response, we can get the request module to parse for us

/*
request({url: url, json: true}, (error, response) => { // json: true parses it for us, no need to parse then
    const currTemp = response.body.current.temperature;
    const feelsTemp = response.body.current.feelslike;

    // console.log('It is currently ' + currTemp + ' degrees out. It feels like ' + feelsTemp + ' degrees out.');
})
*/

// YOU NEED TO FIX THE ISSUES WITH CHROME, HTTP SITE IS BEING REDIRECTED TO HTTPS, so can't access the the JSON response from the weatherstack api 

// As a pratcice excercise, print out the current temperature and what the temperature "feels like"


// Now, we'll be using a differnet api, called Geocoding. So, here the jist of how this weather application is going to work:

// Address -> Lat/long -> weatherstack Api 

// First exercise, retrieve the lat and long of LA

request({url: geocodeurl, json: true}, (error, response)  => {
    // console.log(error); when there is no error, the error object is null 
    if (error) {
        console.log('Unable to connect to position service');

    } else if (response.body.error) {
        console.log('Error! ' + response.body.error.context.query.type + ': ' + response.body.error.context.query.message);
    } else {
        const lat = response.body.data[0].latitude;
        const long = response.body.data[0].longitude;
        console.log('Los Angeles is at latitude: ' + lat + ', and longitude: ' + long);
    }
    
})

// ok, so the above request successfully calls the API from positionstack
```

  

### The Callback Function

  

Here is a very clear illustration on the way call back works in Node. Consider the following:

```JavaScript
function add(a, b, callback) {
  callback(a + b);
}


console.log('before');
add(1, 2, function(result) {
  console.log('Result: ' + result);
});
console.log('after');
```

This is a very simple piece of synchronous code that outputs the following:

```JavaScript
before
Result: 3
after
```

  

To make this asynchronous, we can do the following:

```JavaScript
function addAsync(a, b, callback) {
  setTimeout(function() {
    callback(a + b);
  }, 100);
}

console.log('before');
addAsync(1, 2, function(result) {
  console.log('Result: ' + result);
});
console.log('after');
```

  

This will output:

```JavaScript
before
after
Result: 3
```

  

Now, why is this the case? Since `setTimeout()` triggers an asynchronous operation, it will not wait anymore for the callback to be executed, but instead, it returns immediately giving the control back to `addAsync()`, and then back to its caller. This property in Node.js is crucial, as it allows the stack to unwind, and the control to be given back to the event loop as soon as an asynchronous request is sent, thus allowing a new event from the queue to be processed.

  

So, here is the code for geocode, that you enter a place, and it retrieves data from the place:

```JavaScript
// this in geocode.js file
const request = require('request');

const geocode = (address, callback) => {
    const url = 'http://api.positionstack.com/v1/forward?access_key=472f8191838dbe4f50b9d56228ffd19b&query=' + encodeURIComponent(address);
    // encodeURIcomponent prevents complications such as crahses if the user eneters special characters that mean something in a URL structure, such as ?
    request({url: url, json: true}, (error, response) => {
        if (error) {
            callback('Unable to connect to location services', udefined);
            // we return this to the callback. Before, we directly logged the error, but, to make a more maintainable and reusable module, we let the 
            // callback function decide what to do if an error occurs 
        } else if (response.body.error) {
            callback('Unable to find location. Try another search', undefined);
        } else {
            callback(undefined, {
                city: response.body.data[0].name,
                latitude: response.body.data[0].latitude,
                longitude: response.body.data[0].longitude,
                region: response.body.data[0].region,
                country: response.body.data[0].country
            })
        }
    })

}
// now that this is so modularized, we can place it into another file, and import it from there
module.exports = geocode;
```

Now, in app.js, we can use this function to retrieve the data for any location we want:

```JavaScript
geocode('Shanghai', (error, data) => { // this is the convention for callback, we have two parametrs, error and data, same as request above 
    console.log('Error', error);
    console.log('Data', data);
})

geocode('Shibuya', (error, data) => { // this is the convention for callback, we have two parametrs, error and data, same as request above 
    console.log('Error', error);
    console.log('Data', data);
})

///// THESE GIVE:
Error undefined
Data {
  city: 'Shanghai',
  latitude: 31.246027,
  longitude: 121.483385,
  region: 'Shanghai',
  country: 'China'
}

Error undefined
Data {
  city: 'Shibuya',
  latitude: 35.670289,
  longitude: 139.696816,
  region: 'Tokyo',
  country: 'Japan'
}
```

Pretty cool, so now I’m able to make some simple API calls to fetch data. So, in conclusion, the callback pattern is very useful when using NodeJS to create asynchronous systems.

  

### Callback Chaining

Chaining callback is a way we chain together some functions via callbacks, this is so that the first function can use the non-blocking I/O call the other functions are dependent on. Each function in tasks run only after the previous function is completed. If any of the functions throw an error, the subsequent functions are not executed and the callback is fired with an error value.

  

  

### HTTP Requests Without a Library

Now, the request module we downloaded through npm is optional. In fact, many of the npm modules downloaded are not needed, they only serve to make many processes easier to use. NodeJS has its own built in request module. We will take a look at that here:

  

```JavaScript
const http = require('http');
// we also have https core module, but since this api doesnt have https, we use the http module 
const url = 'http://api.weatherstack.com/current?access_key=bcfa648008114776bf42df4bcf972b53&query=';

// low level api like this can get a bit confusing 
const request = http.request(url, (response) => {
    // will return something from http.request, save it into request 

    // unlike the request module we get from npm, in this callback, we do not have access to the complete response
    // we can grab inidivdual chunks since http data can be streamed through muiltiple parts

    // we need to listen for these chunks, and listen for when all chunks have arrived, and request is done 

    let data = ''; // using let since the value of variable will be changing
    // response.on allows us to register a handler 
    response.on('data', (chunk) => {
        // chunk is a chunk, and we pass it to a callback function, chunk of response
        data += chunk.toString(); // need to convert chunk from buffer to string
        console.log(data);


    })

    // so the above response.on may be fired once, or multiple times, depends on the number of chunks that come in, whcih usually
    // depends on how the server is set up

    response.on('end', () => {
        const body = JSON.parse(data);
        console.log(body);
    })
})

request.on('error', (error) => {
    console.log('An error:', error);
})

request.end();

// so this file wraps up how we would make a http request without the request np module. It is worth noting that most people
// do not use this, and it is confusing and unessacry.
```