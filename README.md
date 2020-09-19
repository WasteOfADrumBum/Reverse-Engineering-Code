# Unit-14-Sequelize-Homework-Reverse-Engineering-Code

![GitHub license](https://img.shields.io/badge/Made%20by-%40WasteOfADrumBum-green)

---

## PASSWORD AUTHENTICATION

This app allows users to create an account, log into the account and sign back out securely. All user data is stored in a mysql
database.

---

## USER STORY

As a user who wants to safely log in to a website, I want to know my personal details are safely stored saftly.

---

## TABLE OF CONTENTS

- [Languages Used](#LANGUAGES-USED)
- [Instructions](#INSTRUCTIONS)
- [Installation](#INSTALLATION)
- [Directory of Files](#DIRECTORY-OF-FILES)
- [Directory of Files Explained](#DIRECTORY-OF-FILES-EXPLAINED)
- [Example Img](#EXAMPLE-IMAGE)
- [Repository Link](#Repository)
- [Test](#Test)
- [Licence](#Licence)
- [GitHub Info](#GitHub)

---

## LANGUAGES USED

- HTML5/CSS3
- JAVASCRIPT
- BCRYPTJS
- EXPRESS
- EXPRESS-SESSION
- MYSQL2
- PASSPORT
- PASSPORT-LOCAL
- SEQUELIZE

---

## INSTRUCTIONS

After cloning this repository into your local storage proceed to preform the following steps;

1. Create a MySQL database called `passport_demo`
2. Open `config.js` and insert your personal username and password for MySQL
3. Open terminal in current root folder and install all node packages [Installation](#INSTALLATION)
4. Run `node server.js` to successfully connect to the server
5. Open browser and put `http://localhost:8080` in search bar
6. Begin using the app

---

## INSTALLATION

Open terminal in the root directory and install the following:

```bash
npm install
npm install express
npm install express-session
npm install mysql
npm install passport
npm install passport-local
npm install sequelize
```

---

## DIRECTORY OF FILES

- [config](<#config-(root/config)>) ↓
  - → [middleware](<#MIDDLEWARE-(root/config/middleware)>) ↓
    - → [isAuthenticated.js](#isAuthenticated.js)
  - → [passport.js](#passport.js)
  - → [config.json](#config.json)
- [models](#) ↓
  - → [index.js](#index.js)
  - → [user.js](#user.js)
- [node_modules](<#NODE_MODULES-(root/node_modules)>) ↓
  - → installed node files/folders
- [public](<#PUBLIC-(root/public)>) ↓
  - → [js](<#JS-(root/public/js)>) ↓
    - → [login.js](#login.js)
    - → [members.js](#members.js)
    - → [signup.js](#signup.js)
  - → [stylesheet](<#STYLESHEETS-(root/public/stylesheets)>) ↓
    - → [style.css](#style.css)
  - → [login.html](#login.html)
  - → [members.html](#members.html)
  - → [signup.html](#signup.html)
- [routes](<#ROUTES-(root/routes)>) ↓
  - → [api-routes.js](#api-routes.js)
  - → [html-routes.js](#html-routes.js)
- [server.js](#server.js)
- [package.json](#package.json)

---

## DIRECTORY OF FILES EXPLAINED

- ### CONFIG (root/config)

  - ### MIDDLEWARE (root/config/middleware)

    - #### isAuthenticated.js

      - This is middleware for restricting routes a user is not allowed to visit. If the user is logged in, continue with the request to the restricted route. If the user isn't logged in, redirect them to the login page

      ```javascript
      // This is middleware for restricting routes a user is not allowed to visit
      if not logged in module.exports = function(req, res, next) {
      // If the user is logged in, continue with the request to the restricted route
      if (req.user) {return next();}
      // If the user isn't logged in, redirect them to the login page
      return res.redirect("/");
      };
      ```

  - #### passport.js

    - Contains javascript logic that tells passport we want to log in with an email address and password. Tell passwort we want to login using a username and password. Set the Local Strategy to use a an email as the username. Once tan email is given with the matching password then login with given credentials by authenticating the http request by using Sequelize to serialize and deserialize the user and exporting the reuired passport.

    ```javascript
    // Require passport, passport-local and /models path
    var passport = require("passport");
    var LocalStrategy = require("passport-local").Strategy;
    var db = require("../models");
    // Telling passport we want to use a Local Strategy. In other words, we want login with a username/email and password
    passport.use(
    	new LocalStrategy(
    		// Our user will sign in using an email, rather than a "username"
    		{ usernameField: "email" },
    		function (email, password, done) {
    			// When a user tries to sign in this code runs
    			db.User.findOne({
    				where: {
    					email: email,
    				},
    			}).then(function (dbUser) {
    				// If there's no user with the given email
    				if (!dbUser) {
    					return done(null, false, {
    						message: "Incorrect email.",
    					});
    				}
    				// If there is a user with the given email, but the password the user gives us is incorrect
    				else if (!dbUser.validPassword(password)) {
    					return done(null, false, {
    						message: "Incorrect password.",
    					});
    				}
    				// If none of the above, return the user
    				return done(null, dbUser);
    			});
    		},
    	),
    );
    // In order to help keep authentication state across HTTP requests,
    // Sequelize needs to serialize and deserialize the user
    passport.serializeUser(function (user, cb) {
    	cb(null, user);
    });
    passport.deserializeUser(function (obj, cb) {
    	cb(null, obj);
    });
    // Exporting our configured passport
    module.exports = passport;
    ```

  - #### config.json

    - Connection configuration to connect to server

    ```javascript
    {
        "development": {"username": "root", "password": null, "database": passport_demo", "host": "127.0.0.1", "dialect": "mysql"},

        "test": {"username": "root", "password": null, "database": "database_test", "host": "127.0.0.1", "dialect": "mysql"},

        "production": {"username": "root", "password": null, "database": "database_production", "host": "127.0.0.1", "dialect": "mysql"}
    }
    ```

- ### MODELS (root/models)

  - #### index.js

    - Connects to database and imports users log in data

    ```javascript
    'use strict';
    // Require fs, path, sequelize
    var fs        = require('fs');
    var path      = require('path');
    var Sequelize = require('sequelize');
    var basename  = path.basename(module.filename);
    var env       = process.env.NODE_ENV || 'development';
    var config    = require(__dirname + '/../config/config.json')[env];
    // set db to an empty array
    var db        = {};
    if (config.use_env_variable) {
        var sequelize = new Sequelize(process.env[config.use_env_variable]);
    } else {
        var sequelize = new Sequelize(config.database, config.username, config.password, config);
    }
    fs
    .readdirSync(**dirname)
    .filter(function(file) {
        return (file.indexOf('.') !== 0) && (file !== basename) && (file.slice(-3) === '.js');
        })
        .forEach(function(file) {
            var model = sequelize['import'](path.join(**dirname, file));
            db[model.name] = model;
        });
    Object.keys(db).forEach(function(modelName) {
        if (db[modelName].associate) {
            db[modelName].associate(db);
        }});
    db.sequelize = sequelize;
    db.Sequelize = Sequelize;
    // export to database
    module.exports = db;
    ```

  - #### user.js

    - Requires "bcrypt" for password hashing. This makes the database secure even if compromised. JavaScript is used to define what is stored on our database when creating a user model.

    ```javascript
    // Requiring bcrypt for password hashing. Using the bcryptjs version as the regular bcrypt module sometimes causes errors on Windows machines
    var bcrypt = require("bcryptjs");
    // Creating our User model
    module.exports = function (sequelize, DataTypes) {
    	var User = sequelize.define("User", {
    		// The email cannot be null, and must be a proper email before creation
    		email: {
    			type: DataTypes.STRING,
    			allowNull: false,
    			unique: true,
    			validate: {
    				isEmail: true,
    			},
    		},
    		// The password cannot be null
    		password: {
    			type: DataTypes.STRING,
    			allowNull: false,
    		},
    	});
    	// Creating a custom method for our User model. This will check if an unhashed password entered by the user can be compared to the hashed password stored in our database
    	User.prototype.validPassword = function (password) {
    		return bcrypt.compareSync(password, this.password);
    	};
    	// Hooks are automatic methods that run during various phases of the User Model lifecycle
    	// In this case, before a User is created, we will automatically hash their password
    	User.addHook("beforeCreate", function (user) {
    		user.password = bcrypt.hashSync(
    			user.password,
    			bcrypt.genSaltSync(10),
    			null,
    		);
    	});
    	return User;
    };
    ```

- ### NODE_MODULES (root/node_modules)

  - A collection of files/folders installed when running npm install from package.json and other required applications

- ### PUBLIC (root/public)

  - ### JS (root/public/js)

    - #### login.js

      - Gets references to the form and inputs. When the form is sumbitted, the email and passowrd entered are validated and then the loginUser function is run and the form cleared. loginUser posts to the api/login route and if successful it redirects to the members page.

      ```javascript
      $(document).ready(function () {
      	// Getting references to our form and inputs
      	var loginForm = $("form.login");
      	var emailInput = $("input#email-input");
      	var passwordInput = $("input#password-input");
      	// When the form is submitted, we validate there's an email and password entered
      	loginForm.on("submit", function (event) {
      		event.preventDefault();
      		var userData = {
      			email: emailInput.val().trim(),
      			password: passwordInput.val().trim(),
      		};
      		if (!userData.email || !userData.password) {
      			return;
      		}
      		// If we have an email and password we run the loginUser function and clear the form
      		loginUser(userData.email, userData.password);
      		emailInput.val("");
      		passwordInput.val("");
      	});
      	// loginUser does a post to our "api/login" route and if successful, redirects us the the members page
      	function loginUser(email, password) {
      		$.post("/api/login", {
      			email: email,
      			password: password,
      		})
      			.then(function () {
      				window.location.replace("/members");
      				// If there's an error, log the error
      			})
      			.catch(function (err) {
      				console.log(err);
      			});
      	}
      });
      ```

    - #### members.js

      - This file just does a GET request to figure out which user is logged in and updates the HTML on the page

      ```javascript
      $(document).ready(function () {
      	// This file just does a GET request to figure out which user is logged in
      	// and updates the HTML on the page
      	$.get("/api/user_data").then(function (data) {
      		$(".member-name").text(data.email);
      	});
      });
      ```

    - #### signup.js

      - Get the refernces from the form and input. When the signup button is clicked, the email and password are checked to make sure they're not blank. If an email and password is inputed then it runs the signUpUser function. Posts to the signup route redirecting to page to the member's page.

      ```javascript
      $(document).ready(function () {
      	// Getting references to our form and input
      	var signUpForm = $("form.signup");
      	var emailInput = $("input#email-input");
      	var passwordInput = $("input#password-input");
      	// When the signup button is clicked, we validate the email and password are not blank
      	signUpForm.on("submit", function (event) {
      		event.preventDefault();
      		var userData = {
      			email: emailInput.val().trim(),
      			password: passwordInput.val().trim(),
      		};

      		if (!userData.email || !userData.password) {
      			return;
      		}
      		// If we have an email and password, run the signUpUser function
      		signUpUser(userData.email, userData.password);
      		emailInput.val("");
      		passwordInput.val("");
      	});

      	// Does a post to the signup route. If successful, we are redirected to the members page
      	// Otherwise we log any errors
      	function signUpUser(email, password) {
      		$.post("/api/signup", {
      			email: email,
      			password: password,
      		})
      			.then(function (data) {
      				window.location.replace("/members");
      				// If there's an error, handle it by throwing up a bootstrap alert
      			})
      			.catch(handleLoginErr);
      	}

      	function handleLoginErr(err) {
      		$("#alert .msg").text(err.responseJSON);
      		$("#alert").fadeIn(500);
      	}
      });
      ```

  - ### STYLESHEETS (root/public/stylesheets)

    - #### style.css

      - A stylesheet for designing custome CSS. This stylesheet only applies a top margin of 50px for the form.signup and form.login html references.

  - #### login.html

    - This file is used to create the html content for the login page. It includes a Bootswatch style sheet to utilize the Lumen Bootstrap template as well as linking the style.css stylesheet, jQuery's JavaScript library and the login.js script. It renders a navbar and login form.

  - #### members.html

    - This file is used to create the html content for the members page. It includes a Bootswatch style sheet to utilize the Lumen Bootstrap template as well as linking the style.css stylesheet, jQuery's JavaScript library and the login.js script. It renders a navbar with a logout option and a member's welcom page.

  - #### signup.html

    - This file is used to create the html content for the members page. It includes a Bootswatch style sheet to utilize the Lumen Bootstrap template as well as linking the style.css stylesheet, jQuery's JavaScript library and the login.js script. It renders a navbar and a signup page with a form with the option to register a user name and password or to retun to the login page to use already registered credentials.

- ### ROUTES (root/routes)

  - #### api-routes.js

    - Contains routes for signing in, logging out and getting users specific data to be displayed client side. It requirs models and passport as configured.

    ```javascript
    // Requiring our models and passport as we've configured it
    var db = require("../models");
    var passport = require("../config/passport");
    module.exports = function (app) {
    ```

    - `/api/login` uses the passport.authenticate middleware with local strategy to validate the users login credentials and proceeds to send them to the members page.

    ```javascript
    // Using the passport.authenticate middleware with our local strategy.
    // If the user has valid login credentials, send them to the members page.
    // Otherwise the user will be sent an error
    app.post("/api/login", passport.authenticate("local"), function (req, res) {
    	res.json(req.user);
    });
    ```

    - `/api/setup` is used to sign up a user by automatically hashing and storing the users password using the configured Sequelize User Model. Once the user's credentials are created it logs the user in using the `/api/login` route.

    ```javascript
    // Route for signing up a user. The user's password is automatically hashed and stored securely thanks to
    // how we configured our Sequelize User Model. If the user is created successfully, proceed to log the user in,
    // otherwise send back an error
    app.post("/api/signup", function (req, res) {
    	db.User.create({
    		email: req.body.email,
    		password: req.body.password,
    	})
    		.then(function () {
    			res.redirect(307, "/api/login");
    		})
    		.catch(function (err) {
    			res.status(401).json(err);
    		});
    });
    ```

    - `/logout` is used for loggin the user out then redirects them back to the main page.

    ```javascript
    // Route for logging user out
    app.get("/logout", function (req, res) {
    	req.logout();
    	res.redirect("/");
    });
    ```

    - `/api/user_data` is used for getting user data to be used client side

    ```javascript
        // Route for getting some data about our user to be used client side
    	app.get("/api/user_data", function (req, res) {
    		if (!req.user) {
    			// The user is not logged in, send back an empty object
    			res.json({});
    		} else {
    			// Otherwise send back the user's email and id
    			// Sending back a password, even a hashed password, isn't a good idea
    			res.json({
    				email: req.user.email,
    				id: req.user.id,
    			});
    		}
    	});
    };
    ```

  - #### html-routes.js

    - Routes that check whether user is signed in and if the user already has account then sends them to the correct html page. It requires path to use relative routes to our html files. It requires custom middleware for checking if a user is logged in. If the user already has an account send them to the members page by using isAuthenticated middleware. Else if a user who is not logged in tries to access this route they will be redirected to the signup page.

    ```javascript
    // Requiring path to so we can use relative routes to our HTML files
    var path = require("path");
    // Requiring our custom middleware for checking if a user is logged in
    var isAuthenticated = require("../config/middleware/isAuthenticated");
    module.exports = function (app) {
    	app.get("/", function (req, res) {
    		// If the user already has an account send them to the members page
    		if (req.user) {
    			res.redirect("/members");
    		}
    		res.sendFile(path.join(__dirname, "../public/signup.html"));
    	});
    	app.get("/login", function (req, res) {
    		// If the user already has an account send them to the members page
    		if (req.user) {
    			res.redirect("/members");
    		}
    		res.sendFile(path.join(__dirname, "../public/login.html"));
    	});
    	// Here we've add our isAuthenticated middleware to this route.
    	// If a user who is not logged in tries to access this route they will be redirected to the signup page
    	app.get("/members", isAuthenticated, function (req, res) {
    		res.sendFile(path.join(__dirname, "../public/members.html"));
    	});
    };
    ```

- #### server.js

  - Declares required npm packages. Requires passport as configured. Sets up port and requires models for syncing. Creates an express application and configures middleware needed for authentication. Requires html and api routes. Syncs the database and logs a message if successful.

  ```javascript
  // Requiring necessary npm packages
  var express = require("express");
  var session = require("express-session");
  // Requiring passport as we've configured it
  var passport = require("./config/passport");
  // Setting up port and requiring models for syncing
  var PORT = process.env.PORT || 8080;
  var db = require("./models");
  // Creating express app and configuring middleware needed for authentication
  var app = express();
  app.use(express.urlencoded({ extended: true }));
  app.use(express.json());
  app.use(express.static("public"));
  // We need to use sessions to keep track of our user's login status
  app.use(
  	session({ secret: "keyboard cat", resave: true, saveUninitialized: true }),
  );
  app.use(passport.initialize());
  app.use(passport.session());
  // Requiring our routes
  require("./routes/html-routes.js")(app);
  require("./routes/api-routes.js")(app);
  // Syncing our database and logging a message to the user upon success
  db.sequelize.sync().then(function () {
  	app.listen(PORT, function () {
  		console.log(
  			"==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.",
  			PORT,
  			PORT,
  		);
  	});
  });
  ```

- #### package.json

  - Contains all package info, node modules used and version info.
    - See [Installation](#INSTALLATION) information for list of installed dependencies.

  ```javascript
  {
  "name": "1-Passport-Example", "version": "1.0.0", "description": "", "main": "server.js",

  "scripts": { "test": "echo \"Error: no test specified\" && exit 1", "start": "node server.js", "watch": "nodemon server.js"},

  "keywords": [], "author": "", "license": "ISC",

  "dependencies": { "bcryptjs": "2.4.3", "express": "^4.17.1", "express-session":  ^1.17.1", "mysql2": "^1.6.5", "passport": "^0.4.1", "passport-local": "^1.0.0", "sequelize": "^5.8.6"}
  }
  ```

---

↑ Return to [Directory of Files](#DIRECTORY-OF-FILES) ↑

---

## EXAMPLE IMAGE

<img src="public\images\process.png" width="800" />

---

## REPOSITORY

![GitHub repo size](https://img.shields.io/github/repo-size/WasteOfADrumBum/Reverse-Engineering-Code?logo=github)

![GitHub Commit Activity](https://img.shields.io/github/commit-activity/m/WasteOfADrumBum/Reverse-Engineering-Code)
![GitHub Last Commit](https://img.shields.io/github/last-commit/WasteOfADrumBum/Reverse-Engineering-Code)

![GitHubopen pull request](https://img.shields.io/github/issues-pr/WasteOfADrumBum/Reverse-Engineering-Code)
![GitHub closed pull request](https://img.shields.io/github/issues-pr-closed/WasteOfADrumBum/Reverse-Engineering-Code)

![GitHub Stars](https://img.shields.io/github/stars/WasteOfADrumBum/Reverse-Engineering-Code?style=social)

- [Project Repo](https://github.com/WasteOfADrumBum/Reverse-Engineering-Code)

---

## CONTRIBUTORS

![GitHub contributors](https://img.shields.io/github/contributors/WasteOfADrumBum/Reverse-Engineering-Code)
![GitHub Forks](https://img.shields.io/github/forks/WasteOfADrumBum/Reverse-Engineering-Code?label=Fork)
![GitHub Watchers](https://img.shields.io/github/watchers/WasteOfADrumBum/Reverse-Engineering-Code?label=Watch)

---

## TEST

![GitHub test](https://img.shields.io/badge/test-100%25-success)

![GitHub open issues](https://img.shields.io/github/issues/WasteOfADrumBum/Reverse-Engineering-Code)
![GitHub closed issues](https://img.shields.io/github/issues-closed/WasteOfADrumBum/Reverse-Engineering-Code)

---

## LICENSE

![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)

---

## GITHUB

<img src="https://avatars0.githubusercontent.com/u/66432859?v=4" width="250" />

### Joshua M. Small

![GitHub Followers](https://img.shields.io/github/followers/WasteOfADrumBum?label=Follow)

[GitHub Profile](https://github.com/WasteOfADrumBum)

E-Mail: <JMSmall89@gmail.com>
