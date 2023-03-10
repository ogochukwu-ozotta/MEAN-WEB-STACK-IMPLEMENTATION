# MEAN-WEB-STACK-IMPLEMENTATION

# PROJECT 4 - MEAN WEB STACK IMPLEMENTATION

### MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

Now, when you have already learned how to deploy LAMP, LEMP and MERN Web stacks – it is time to get yourself familiar with MEAN stack and deploy it to Ubuntu server.

#### MEAN Stack is a combination of following components:
1. MongoDB (Document database) – Stores and allows to retrieve data.
2. Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
3. Angular (Front-end application framework) – Handles Client and Server Requests
4. Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user

Instructions On How To Submit Your Work For Review And Feedback
To submit your work for review and feedback – follow this instruction.

Step 0 – Preparing prerequisites
In order to complete this project you will need an AWS account and a virtual server with Ubuntu Server OS.

If you do not have an AWS account – go back to Project 1 Step 0 to sign in to AWS free tier account and create a new EC2 Instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM) image. 
Remember, you can have multiple EC2 instances, but make sure you STOP the ones you are not working with at 
the moment to save available free hours.


## Step 1: Install NodeJs
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

SSH into the EC2 instance (GitBash)

```
ssh -i <key pem file> ubuntu@<public-IP-Address>
```

Update ubuntu

```
sudo apt update
```

Upgrade ubuntu

```
sudo apt upgrade
```

Install NodeJS

```
sudo apt install -y nodejs
```


Step 2: Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. 
For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
mages/WebConsole.gif


```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```

<img width="454" alt="image" src="https://user-images.githubusercontent.com/88560609/197116322-b284aa11-5a79-4029-8e84-b61e0ce6f2d5.png">

```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

<img width="457" alt="image" src="https://user-images.githubusercontent.com/88560609/197116405-cc386968-203c-44ec-b1b9-5f57d256bf31.png">


### Install MongoDB


```
sudo apt install -y mongodb
```

Start The server

```
sudo service mongodb start
```

Verify that the service is up and running

```
sudo systemctl status mongodb
```

<img width="458" alt="image" src="https://user-images.githubusercontent.com/88560609/197116629-fe66204e-3bf2-4d48-a8cc-36e6338a58d7.png">

Install npm – Node package manager.

```
sudo apt install -y npm
```

Install body-parser package

We need ‘body-parser’ package to help us process JSON files passed in requests to the server.


```
sudo npm install body-parser
```

<img width="443" alt="image" src="https://user-images.githubusercontent.com/88560609/197116980-29e4da72-7ae0-4683-89e9-15e112e6eca4.png">

Create a folder named ‘Books’

```
mkdir Books && cd Books
```

In the Books directory, Initialize npm project

```
npm init
```

<img width="454" alt="image" src="https://user-images.githubusercontent.com/88560609/197117185-ef55986a-e6c4-4588-93c2-45b381beba1f.png">

Enter the details in the image below 

<img width="525" alt="image" src="https://user-images.githubusercontent.com/88560609/197117725-646e449e-22a3-489d-be7b-c22eafbfde4f.png">


Add a file to it named server.js

```
vi server.js
```

Copy and paste the web server code below into the server.js file.


```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});

```

<img width="521" alt="image" src="https://user-images.githubusercontent.com/88560609/197117950-731b2a48-3911-479d-998c-fdfc49d8a242.png">


### INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER

Step : Install Express and set up routes to the server
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications.
We will use Express in to pass book information to and from our MongoDB database.

We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. 
We will use Mongoose to establish a schema for the database to store data of our book register.


```
sudo npm install express mongoose
```

<img width="529" alt="image" src="https://user-images.githubusercontent.com/88560609/197118175-037a3ddc-247a-48f0-bcd4-37613c0970eb.png">


In ‘Books’ folder, create a folder named apps

```
mkdir apps && cd apps
```

Create a file named routes.js

```
vi routes.js
```

Copy and paste the code below into routes.js


```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};

```

In the ‘apps’ folder, create a folder named models

```
mkdir models && cd models
```

Create a file named book.js

```
vi book.js
```

Copy and paste the code below into ‘book.js’


```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);

```

### Step 4 – Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express 
and perform actions on our book register.

Change the directory back to ‘Books’

```
cd ../..
```

Create a folder named public

```
mkdir public && cd public
```

Add a file named script.js


```
vi script.js
```

Copy and paste the Code below (controller configuration defined) into the script.js file.

```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});

```

<img width="527" alt="image" src="https://user-images.githubusercontent.com/88560609/197118931-3ceeeef9-b871-47c8-abc6-22cb987c2123.png">


In public folder, create a file named index.html;

```
vi index.html
```

Copy and paste the code below into index.html file.


```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

```

Change the directory back up to Books


```
cd ..
```

Start the server by running this command:


```
node server.js
```

The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally.

```
curl -s http://localhost:3300
```


It shall return an HTML page, it is hardly readable in the CLI, but we can also try and access it from the Internet.

For this – you need to open TCP port 3300 in your AWS Web Console for your EC2 Instance.

You are supposed to know how to do it, if you have forgotten – refer to Project 1 (Step 1 — Installing Apache and Updating the Firewall)

Your Sercurity group shall look like this:



Now you can access our Book Register web application from the Internet with a browser using Public IP address or Public DNS name.

Quick reminder how to get your server’s Public IP and public DNS name:

1. You can find it in your AWS web console in EC2 details
2. Run curl -s http://169.254.169.254/latest/meta-data/public-ipv4 for Public IP address or curl -s http://169.254.169.254/latest/meta-data/public-hostname for Public DNS name.
This is how your Web Book Register Application will look like in browser:

