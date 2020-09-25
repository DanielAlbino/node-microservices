# node-microservices

we’re going to create a microservice using Node.js which connects to an external API.
The requirement for this service is to accept two Zip Codes of two stores and return the distance between them in miles.

**INITIAL STEPS**

1. Have Node.js installed
2. Run npm init in the root folder for the project. This will create a package.json file that will prompt some questions about the package, if you are not sure how to answer you can use the default.
3. We are going to use two packages, Express and Require that can be installed like this:

```bash
    npm install express request --save
```

The primary file in our project is named server.js.
Then we create two folders, api for files that will support the API, and service for the logic to connect to a third-party API.
Inside the api folder we are going to create two files, controller.js and routes.js.
In the service folder we are going to create the file distance.js

**CREATING A SERVER TO ACCESS REQUESTS**

Open the serve.s file and add the following code:

```bash
    const express = require('express')
    const app = express();
    const port = process.env.PORT || 3000;

    const routes = require('./api/routes');
    routes(app);
    app.listen(port, function() {
        console.log('Server started on port: ' + port);
    });
```

First we require express into the file, and use it to create a new app object const app = express(); then we specify the port, in this case, we use the environment variable called PORT, and if the variable isn’t defined, it will use the default port: 3000.

Then we bring the routes object from the routes.js file in the api folder. We’ll pass the app to the routes object, and that sets the routes for our application. Finally, we’ll tell the app to start listening on the port we defined and to display a message to the console when this process is complete.

**DEFINING THE ROUTES**

The next step is to define the routes for the microservices and then assign each to a target in the controller object (that will control the flow of data in the application). We’ll have two endpoints. One endpoint called “about” that returns information about the application. And a “distance” endpoint that includes two path parameters, both Zip Codes of the stores. This endpoint returns the distance, in miles, between these two Zip Codes.

routes.js

```bash
    'use strict';

    const controller = require('./controller');

    module.exports = function(app) {
    app.route('/about')
        .get(controller.about);
    app.route('/distance/:zipcode1/:zipcode2')
        .get(controller.getDistance);
    };
```

**ADDING CONTROLLER LOGIC**

Within the controller file, we’re going to create a controller object with two properties. Those properties are the functions to handle the requests we defined in the routes module.

```bash
    'use strict';

    var properties = require('../package.json')
    var distance = require('../service/distance');

    var controllers = {
    about: function(req, res) {
        var aboutInfo = {
            name: properties.name,
            version: properties.version
        }
        res.json(aboutInfo);
    },
    getDistance: function(req, res) {
            distance.find(req, res, function(err, dist) {
                if (err)
                    res.send(err);
                res.json(dist);
            });
        },
    };

    module.exports = controllers;
```

**MAKING AN EXTERNAL CALL**

We’re ready for the final piece of the puzzle. This file handles the call to a third-party API. We’ll use the distance API provided by ZipCodeAPI.com. (You need an API key to use this, and it is free if you register.
I set my key as an environment variable on my system and named it ZIPCODE_API_KEY.

distance.js

```bash
    var request = require('request');

    const apiKey = process.env.ZIPCODE_API_KEY || "UcROKGLFeLnue77m4SgRuUHlrNYpgDl8UvOfdIWO0BTSNfoqz19zpK3w6HlLTTGC";
    const zipCodeURL = 'https://www.zipcodeapi.com/rest/';

    var distance = {
    find: function(req, res, next) {
        request(zipCodeURL + apiKey
                + '/distance.json/' + req.params.zipcode1 + '/'
                + req.params.zipcode2 + '/mile',
        function (error, response, body) {
            if (!error && response.statusCode == 200) {
                response = JSON.parse(body);
                res.send(response);
            } else {
                console.log(response.statusCode + response.body);
                res.send({distance: -1});
            }
        });

    }
    };

    module.exports = distance;
```

** EXECUTION **

Assuming there aren’t any typos, your application should be ready to execute. Open a console window and run the following command:

```bash
    npm start

```

Assuming it starts correctly, and the port you define is 3000, you can now open your browser and navigate to:

http://localhost:3000/about when you will see the name of the app and the version.

Now if you add two parameters, the two zip codes, you will the distance between them.

http://localhost:3000/distance/84010/97229
