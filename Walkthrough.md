## Walk through

####Part 1 - Installation and Configuration
Disclaimer: I am using Mac, so all the below steps are written with respect to Mac.  

Step 1: Install Node.js  
If you have Homebrew installed, install using  
`
brew install node
`

Step 2: Install Express Generator  
`
npm install -g express-generator
`

Step 3: Create an Express project  
`
express learning-nodeJS
`

Step 4: Edit dependencies  
Add MongoDB and Monk dependencies.
This is the list of dependencies in package.json

```
"dependencies": {  
    "body-parser": "~1.12.4",
    "cookie-parser": "~1.3.5",
    "debug": "~2.2.0",
    "express": "~4.12.4",
    "jade": "~1.9.2",
    "morgan": "~1.5.3",
    "serve-favicon": "~2.2.1",
    "mongodb": "^1.4.4",
    "monk": "^1.0.1"
}
```

Step 5: Install Dependencies  
`
npm install
`

Make a directory called *data/* which will be used to house mongodb data.  
`
mkdir data
`

Start the web server:  
`
npm start
`

Head over to *http://localhost:3000* where you will see a Welcome to Express page.

####Part 2 - Getting to "Hello, World!"

Create a bunch of basic JS variables and tie them to certain packages, dependencies, node functionality and routes.   

```
//app.js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var routes = require('./routes/index');
var users = require('./routes/users');

```

Instantiate Express:  
`
var app = express();
`

Configuring some Express stuff:  

```
// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

// uncomment after placing your favicon in /public
//app.use(favicon(__dirname + '/public/favicon.ico'));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', routes);
app.use('/users', users);
```

Setup Error Handlers:  

```
/// catch 404 and forwarding to error handler
app.use(function(req, res, next) {
    var err = new Error('Not Found');
    err.status = 404;
    next(err);
});

/// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
    app.use(function(err, req, res, next) {
        res.status(err.status || 500);
        res.render('error', {
            message: err.message,
            error: err
        });
    });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
        message: err.message,
        error: {}
    });
});
```

Export an object from the modules so that it can be easily called elsewhere in the code  
`
module.exports = app;
`

Routing in app.js  

```
app.use('/', routes);
app.use('/users', users);
```

Adding a new route

```
/* routes\index.js */
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res) {
    res.render('index', { title: 'Express' });
});

/* NEW ROUTE - GET Hello World page. */
router.get('/helloworld', function(req, res) {
    res.render('helloworld', { title: 'Hello, World!' });
});

module.exports = router;
```

Build a template

```
// views/helloworld.jade
extends layout

block content
	h1= title
	p Hello, World! Welcome to #{title}
```

Restart the server  
Press `Ctrl`+`C` to kill the already running server  
`npm start` to restart the server

Go to *http://localhost:3000/helloworld* to check out the hello world page

####Part 3 - Create our Database and read stuff from it

Step 1: Install Mongo.   
Using Homebrew, type in `brew install mongodb`

Step 2: Run MongoD and Mongo
We will now use the *data/* we creating in Step 1.

Start Mongo server pointing to that data/ directory  
`mongod --dbpath <path>\learning-nodeJs\data\`

Step 3: Create a Database  
Inside the mongo console, point to the correct database:
`use learning-nodeJS`

Step 4: Add some Data  
Run this command inside the mongo client

```
db.usercollection.insert({ "username" : "testuser1", "email" : "testuser1@testdomain.com" })
```

Find all records in the database table

```
db.usercollection.find().pretty()
```

Adding data in bulk

```
newstuff = [{ "username" : "testuser2", "email" : "testuser2@testdomain.com" }, { "username" : "testuser3", "email" : "testuser3@testdomain.com" }]
db.usercollection.insert(newstuff);
```

Check if the inserts were successful  

```
db.usercollection.find().pretty()
```

Step 5: Hooking Mongo up to Node  

```
// app.js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

// New Code
var mongo = require('mongodb');
var monk = require('monk');
var db = monk('localhost:27017/learning-nodeJs');
```

Make DB accessible to the router

```
// app.js
// Make our db accessible to our router
app.use(function(req,res,next){
    req.db = db;
    next();
});
```

Step 6: Retrieving data from Mongo and display it

```
/* routes\index.js */
/* GET Userlist page. */
router.get('/userlist', function(req, res) {
    var db = req.db;
    var collection = db.get('usercollection');
    collection.find({},{},function(e,docs){
        res.render('userlist', {
            "userlist" : docs
        });
    });
});
```

Create a template

```
// views/userlist.jade

block content
    h1.
        User List
    ul
        each user, i in userlist
            li
                a(href="mailto:#{user.email}")= user.username
```


Restart the server  
Press `Ctrl`+`C` to kill the already running server  
`npm start` to restart the server

####Part 4 - Writing to the DB

Step 1: Create Data Input

Create a new route that builds the page to add a new user

```
/* routes/index.js */
/* GET New User page. */
router.get('/newuser', function(req, res) {
    res.render('newuser', { title: 'Add New User' });
});
```

Create a template for adding a new user

```
// views/index.jade
extends layout

block content
    h1= title
    form#formAddUser(name="adduser",method="post",action="/adduser")
        input#inputUserName(type="text", placeholder="username", name="username")
        input#inputUserEmail(type="text", placeholder="useremail", name="useremail")
        button#btnSubmit(type="submit") submit
```

Step 2: Create DB functions

```
/* routes/index.js */
/* POST to Add User Service */
router.post('/adduser', function(req, res) {

    // Set our internal DB variable
    var db = req.db;

    // Get our form values. These rely on the "name" attributes
    var userName = req.body.username;
    var userEmail = req.body.useremail;

    // Set our collection
    var collection = db.get('usercollection');

    // Submit to the DB
    collection.insert({
        "username" : userName,
        "email" : userEmail
    }, function (err, doc) {
        if (err) {
            // If it failed, return error
            res.send("There was a problem adding the information to the database.");
        }
        else {
            // And forward to success page
            res.redirect("userlist");
        }
    });
});
```

Restart the server  
Press `Ctrl`+`C` to kill the already running server  
`npm start` to restart the server

    
