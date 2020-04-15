# express-dynamodb-rest-api

Node.js RESTful API with DynamoDB Local
James Hamann
James Hamann
Follow
Feb 1, 2018 · 9 min read

Node is usually used along side MongoDB in the MEAN stack. However, using Amazon’s DynamoDB has it’s own benefits, not least from speed, scalability, affordability and freeing your time up from configuring database clusters/updates. Through this post I’ll discuss how to setup DynamoDB with your Node project locally.
Versions
Node 9.2.1
Express 4.15.5
DynamoDB Local — Latest
JRE (Java Runtime Environment) 6.x or newer
Setting up your Node Project
To get things moving quickly, we’ll use the express generator to scaffold a project for us.
# bash
$ express node-dynamo-db

   create : node-dynamo-db
   create : node-dynamo-db/package.json
   create : node-dynamo-db/app.js
   create : node-dynamo-db/public
   create : node-dynamo-db/routes
   create : node-dynamo-db/routes/index.js
   create : node-dynamo-db/routes/users.js
   create : node-dynamo-db/views
   create : node-dynamo-db/views/index.jade
   create : node-dynamo-db/views/layout.jade
   create : node-dynamo-db/views/error.jade
   create : node-dynamo-db/bin
   create : node-dynamo-db/bin/www
   create : node-dynamo-db/public/javascripts
   create : node-dynamo-db/public/images
   create : node-dynamo-db/public/stylesheets
   create : node-dynamo-db/public/stylesheets/style.css
install dependencies:
     $ cd node-dynamo-db && npm install
run the app:
     $ DEBUG=node-dynamo-db:* npm start
$ cd node-dynamo-db
$ npm install
Fire up your server to ensure it’s all working as intended.
$ npm start
Navigate to <http://localhost:3000> and you’ll see the welcome page from express, like below.

Generic Express Welcome Page
Next, as there’s no live-reloading, we’ll install Nodemon to watch our files and whenever a change is made, it’ll restart the server for us. Without Nodemon, you’re gonna get frustrated real fast. Once installed, we’ll update our start command within the package.json to run the nodemon command as opposed to node.
# bash
$ npm install -g nodemon
--------------------------------------------------------------------

# package.json
{
  "name": "node-dynamo-db",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "nodemon  ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.18.2",
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.9",
    "express": "~4.15.5",
    "jade": "~1.11.0",
    "morgan": "~1.9.0",
    "serve-favicon": "~2.4.5"
  }
}
Setting up DynamoDB
First download the file from the link above, unpack it and navigate into the directory. You’ll notice DynamoDB is provided as an executable .jar file. In order to start the database up, we need to run the following command within the directory the .jar file is located.
# bash
$ java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb
Initializing DynamoDB Local with the following configuration:
Port: 8000
InMemory: false
DbPath: null
SharedDb: true
shouldDelayTransientStatuses: false
CorsParams: *
Boom, you’ve got a local instance of DynamoDB running! Problem is, unless you’re gifted with photographic memory, you’re probably not going to rememeber the above command and even if you do, it’s ballache to write out each time. Lets speed things up and create an alias command within our .bashrc or .zshrc, depending on what you use. Mine looks like this.
# bash .zshrc or .bashrc
alias ddb="cd path/to/dynamodb_local_latest && java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb"
I’ve named my alias ddb, it navigates to the directory and then executes the .jar, simple as that. Now when reloading my terminal window and running ddb, DynamoDB should spin up.
# bash
$ ddb
Initializing DynamoDB Local with the following configuration:
Port: 8000
InMemory: false
DbPath: null
SharedDb: true
shouldDelayTransientStatuses: false
CorsParams: *
Now we’re all set to start creating our table and to begin seeding some data into our table. For the purpose of this demo, I’ll be making a database revolving around cars.
Before moving forward, let’s just update our package.json to automate some of the commands we’ll be running fairly frequently.
{
  "name": "crafty-api",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "nodemon app.js",
    "create-db": "cd dynamodb && node createCarsTable.js && cd ..",
    "delete-db": "cd dynamodb && node deleteCarsTable.js && cd ..",
    "load-data": "cd dynamodb && node loadCarData.js && cd ..",
    "read-data": "cd dynamodb && node readDataTest.js && cd .."
  },
  "dependencies": {
    "aws-sdk": "^2.176.0",
    "body-parser": "~1.18.2",
    "cookie-parser": "~1.4.3",
    "cors": "^2.8.4",
    "debug": "~2.6.9",
    "ejs": "^2.5.7",
    "express": "~4.15.5",
    "jade": "~1.11.0",
    "morgan": "~1.9.0",
    "newman": "^3.9.1",
    "node-uuid": "^1.4.8",
    "serve-favicon": "~2.4.5",
    "uuid": "^3.2.1"
  }
}
This is what my current one looks like and it just speeds things up so much, so consider adding your own to speed up your workflow.
First things first, we’re gonna need to create a table and choose a partition key. Amazon provided pretty good advice here on what constitutes as a good key. Reason we need a key is because Dynamo DB partitions our data across multiple storage units and uses that key to both store and read the data. Therefore, the partition key must be a unique value. Good examples are user_ids and devices_ids.
For my table I’ve chosen car_id.
# JavaScript - createCarsTable.js
var AWS = require("aws-sdk");
AWS.config.update({
  region: "eu-west-2",
  endpoint: "<http://localhost:8000">
});
var dynamodb = new AWS.DynamoDB();
var params = {
    TableName : "Cars",
    KeySchema: [
        { AttributeName: "id", KeyType: "HASH"},  //Partition key
],
    AttributeDefinitions: [
        { AttributeName: "id", AttributeType: "N" },
],
    ProvisionedThroughput: {
        ReadCapacityUnits: 5,
        WriteCapacityUnits: 5
    }
};
dynamodb.createTable(params, function(err, data) {
    if (err) {
        console.error("Unable to create table. Error JSON:", JSON.stringify(err, null, 2));
    } else {
        console.log("Created table. Table description JSON:", JSON.stringify(data, null, 2));
    }
});
Now run you’re create-db command, making sure Dynamo DB is running in the background on another terminal window, on port 8000.
# bash
yarn create-db
yarn run v1.3.2
$ cd dynamodb && node createCarsTable.js && cd ..
Created table. Table description JSON: {
  "TableDescription": {
    "AttributeDefinitions": [
      {
        "AttributeName": "id",
        "AttributeType": "N"
      }
    ],
    "TableName": "Cars",
    "KeySchema": [
      {
        "AttributeName": "id",
        "KeyType": "HASH"
      }
    ],
    "TableStatus": "ACTIVE",
    "CreationDateTime": "2018-02-01T16:08:25.308Z",
    "ProvisionedThroughput": {
      "LastIncreaseDateTime": "1970-01-01T00:00:00.000Z",
      "LastDecreaseDateTime": "1970-01-01T00:00:00.000Z",
      "NumberOfDecreasesToday": 0,
      "ReadCapacityUnits": 5,
      "WriteCapacityUnits": 5
    },
    "TableSizeBytes": 0,
    "ItemCount": 0,
    "TableArn": "arn:aws:dynamodb:ddblocal:000000000000:table/Cars"
  }
}
✨  Done in 0.47s.
Now you’re table is setup and ready to seed data into.
In this example, we’re using Dynamo DB’s PutItem method to seed some data into our Database.
# JSON - carData.json
[
  { "id": 1,
    "type" : "Automatic",
    "name" : "Toyota Yaris",
    "manufacturer" : "Toyota",
    "fuel_type" : "Petrol",
    "description" : "A smooth ride"
  },
  { "id": 2,
    "type" : "Manual",
    "name" : "Volkswagen Golf",
    "manufacturer" : "Volkswagen",
    "fuel_type" : "Petrol",
    "description" : "Good Value"
  }
]
------------------------------------------------------------------

# JavaScript - loadCarData.js
var AWS = require("aws-sdk");
var fs = require('fs');
AWS.config.update({
    region: "eu-west-2",
    endpoint: "<http://localhost:8000">
});
var docClient = new AWS.DynamoDB.DocumentClient();
console.log("Importing Cars into DynamoDB. Please wait.");
var cars = JSON.parse(fs.readFileSync('carData.json', 'utf8'));
cars.forEach(function(car) {
  console.log(car)
var params = {
        TableName: "Cars",
        Item: {
            "id": car.id,
            "type": car.type,
            "name": car.name,
            "manufacturer": car.manufacturer,
            "fuel_type": car.fuel_type,
            "description": car.description
        }
    };
docClient.put(params, function(err, data) {
       if (err) {
           console.error("Unable to add Car", car.name, ". Error JSON:", JSON.stringify(err, null, 2));
       } else {
           console.log("PutItem succeeded:", car.name);
       }
    });
});
If you run your load-data command, it should seed in the two items in our carData.json file and log the names back in the console, like below.
# bash
yarn load-data
yarn run v1.3.2
$ cd dynamodb && node loadCarData.js && cd ..
Importing Cars into DynamoDB. Please wait.
{ id: 1,
  type: 'Automatic',
  name: 'Toyota Yaris',
  manufacturer: 'Toyota',
  fuel_type: 'Petrol',
  description: 'A smooth ride' }
{ id: 2,
  type: 'Manual',
  name: 'Volkswagen Golf',
  manufacturer: 'Volkswagen',
  fuel_type: 'Petrol',
  description: 'Good Value' }
PutItem succeeded: Toyota Yaris
PutItem succeeded: Volkswagen Golf
✨  Done in 0.46s.
Now our datas in there, but how do we know? Let’s run a quick test using Dynamo DBs DocumentClient .get method. DocumentClient is just a class that simplifies working with DynamoDB Items.
# JavaScript - readDataTest.js
var AWS = require("aws-sdk");
AWS.config.update({
  region: "eu-west-2",
  endpoint: "<http://localhost:8000">
});
var docClient = new AWS.DynamoDB.DocumentClient()
var table = "Cars";
var id = 1;
var params = {
    TableName: table,
    Key:{
        "id": id
    }
};
docClient.get(params, function(err, data) {
    if (err) {
        console.error("Unable to read item. Error JSON:", JSON.stringify(err, null, 2));
    } else {
        console.log("GetItem succeeded:", JSON.stringify(data, null, 2));
    }
});
Remembering our JSON file, we should expect the Toyota Yaris to be returned to the console…
# bash
$ yarn read-data
yarn run v1.3.2
$ cd dynamodb && node readDataTest.js && cd ..
GetItem succeeded: {
  "Item": {
    "name": "Toyota Yaris",
    "description": "A smooth ride",
    "id": 1,
    "type": "Automatic",
    "fuel_type": "Petrol",
    "manufacturer": "Toyota"
  }
}
✨  Done in 0.56s.
BAM! DynamoDB is setup and seeded with data, now we just need to bring all the elements together.
Bringing it all together
At the moment, our Node backend isn’t actually talking to Dynamo DB at all, lets change that by incorporating some of the methods we’ve used above and create a route that returns all cars.
To do this we’re going to using DynamoDBs DocClient scan method.
# Javascript app.js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var AWS = require("aws-sdk");
var app = express();
app.listen(3000, () => console.log('Cars API listening on port 3000!'))
AWS.config.update({
  region: "eu-west-2",
  endpoint: "<http://localhost:8000">
});
var docClient = new AWS.DynamoDB.DocumentClient();
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.set('view engine', 'jade');
app.get('/', function (req, res) {
  res.send({ title: "Cars API Entry Point" })
})
app.get('/cars', function (req, res) {
var params = {
    TableName: "Cars",
    ProjectionExpression: "#id, #name, #type, #manufacturer, #fuel_type, #description",
    ExpressionAttributeNames: {
        "#id": "id",
        "#name": "name",
        "#type": "type",
        "#manufacturer": "manufacturer",
        "#fuel_type": "fuel_type",
        "#description": "description"
    }
};
console.log("Scanning Cars table.");
docClient.scan(params, onScan);
function onScan(err, data) {
    if (err) {
        console.error("Unable to scan the table. Error JSON:", JSON.stringify(err, null, 2));
    } else {
        res.send(data)
        // print all the Cars
        console.log("Scan succeeded.");
        data.Items.forEach(function(car) {
           console.log(car.id, car.type, car.name)
        });
if (typeof data.LastEvaluatedKey != "undefined") {
            console.log("Scanning for more...");
            params.ExclusiveStartKey = data.LastEvaluatedKey;
            docClient.scan(params, onScan);
        }
    }
  }
})
This is what you want your app.js file to look like. I know we can refactor this and move some code to the routes folder, however for the purposes of keeping this article as to the point as possible, I’ll leave that to you.
As the file shows, we create a new route called /cars and create a params variable, which contains the name of the table and what we want to be returned from our scan. We then create a function called onScan which sends our data to the client and logs our results to console. This also contains some error catching, should there be any issues with your request.
Now, if you navigate to <http://localhost:3000/cars> you should see something resembling the below.
# JSON - response from <http://localhost:3000/cars>
{"Items":[{"name":"Volkswagen Golf","description":"Good Value","id":2,"fuel_type":"Petrol","type":"Manual","manufacturer":"Volkswagen"},{"name":"Toyota Yaris","description":"A smooth ride","id":1,"fuel_type":"Petrol","type":"Automatic","manufacturer":"Toyota"}],"Count":2,"ScannedCount":2}
Great job! Now you’ve got building blocks of a Node.js RESTful API using AWS DynamoDB.
Let’s do one more route where we ask DynamoDB to return a car, by id.
Let’s call our route /cars/:id. We’ll pass the ID in via our request url. We’ll then use the ID to query the table and return us the correct car. We get the id value by slicing the string to return us only the number.
Remember, however, when we created our table we specified that the id was a number type. Therefore if we try to pass the value, as it is, to DynamoDB, it’ll spit back an error. We first need to convert our id value from string to integer using parseInt().
# JavaScript - app.js
[...]
app.get('/cars/:id', function (req, res) {
var carID = parseInt(req.url.slice(6));
  console.log(req.url)
  console.log(carID)
var params = {
      TableName : "Cars",
      KeyConditionExpression: "#id = :id",
      ExpressionAttributeNames:{
          "#id": "id"
      },
      ExpressionAttributeValues: {
          ":id": carID
      }
  };
docClient.query(params, function(err, data) {
    if (err) {
        console.error("Unable to query. Error:", JSON.stringify(err, null, 2));
    } else {
        console.log("Query succeeded.");
        res.send(data.Items)
        data.Items.forEach(function(car) {
            console.log(car.id, car.name, car.type);
        });
    }
});
});
We save our converted carID value in a variable and use this in our params object. We then use the query method to gather and return the data to the client. If all is setup correctly, you should be able to navigate to <http://localhost:3000/cars/1> and see that the Yaris is returned as JSON. If you check your terminal you’ll see the id, name and type of the car queried.
# JSON - <http://localhost:3000/cars/1>
[{"name":"Toyota Yaris","description":"A smooth ride","id":1,"type":"Automatic","fuel_type":"Petrol","manufacturer":"Toyota"}]
# bash
$ yarn start
[nodemon] starting `node app.js`
Cars API listening on port 3000!
/cars/1
1
Query succeeded.
1 'Toyota Yaris' 'Automatic'
GET /cars/1 200 47.279 ms - 126
From here you can add additional routes to search by car name, car type and look to implement POSTing to the DB. Hint: this will be similar to our loadCarData.js file, using DynamoDB’s PutItem function.
Next time I’ll look to deploy our sample app to AWS Elastic Beanstalk along with AWS DynamoDB and implement a build pipeline with CircleCI and testing using Postman.
If you wish, you can check all the code out here, at the example Github Repo.
As always, thanks for reading, hit 👏 if you like what you read and be sure to follow to keep up to date with future posts.

Quick Code
Find the best tutorials and courses for the web, mobile…
Follow
597

Nodejs
AWS
Dynamodb
API
Web Development
597 claps

James Hamann
WRITTEN BY

James Hamann
Follow
Software Developer <https://jameshamann.com>
Quick Code
Quick Code
Follow
Find the best tutorials and courses for the web, mobile, chatbot, AR/VR development, database management, data science, web design and cryptocurrency. Practice in JavaScript, Java, Python, R, Android, Swift, Objective-C, React, Node Js, Ember, C++, SQL & more.
See responses (5)
More From Medium
More from Quick Code
Local Notifications with Swift 4
Ali Fakih in Quick Code
Jan 15, 2019 · 8 min read
1K

More from Quick Code
Advanced Python made easy
Ravindra Parmar in Quick Code
Sep 6, 2018 · 5 min read
2.2K

More from Quick Code
10 Reasons Why Python Beats PHP for Web Development
myTectra in Quick Code
Feb 27, 2019 · 4 min read
1.5K

Discover Medium
Welcome to a place where words matter. On Medium, smart voices and original ideas take center stage - with no ads in sight. Watch
Make Medium yours
Follow all the topics you care about, and we’ll deliver the best stories for you to your homepage and inbox. Explore
Become a member
Get unlimited access to the best stories on Medium — and support writers while you’re at it. Just $5/month. Upgrade
About
Help
Legal
