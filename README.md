# Express Router

## Lesson Objectives

1. Explain what `express.Router` does for us
1. Create an external controller file for routes
1. Move `server.js` routes to external controller file
1. Use controller file in `server.js`
1. Remove references to base of controller's URLs

## Explain what `express.Router` does for us

Our `server.js` file is getting rather bloated again, and `express.Router` will let us put our routes in a separate file.

## Create external controller file for routes

1. `mkdir controllers`
2. `touch controllers/fruits_controllers.js`
3. In `controllers/fruits_controllers.js`:

```javascript
const express = require('express');
const router = express.Router();

module.exports = router;
```

## Move `server.js` routes to external controller file

First, let's cut all the routes from `server.js` and paste them in `controllers/fruits_controllers.js`.

Once they're in there, replace `app` with `router`, the variable we declared at the top.

```js
const express = require('express');
const router = express.Router();

// create route
// this route will catch GET requests to /fruits/newForm
// and respond with a form for creating new fruits
router.get('/fruits/newForm', (req, res) => {
    res.render('new.ejs');
});

// create route
// this route will catch POST requests to /fruits
// and, after creating new data, respond by redirecting
// the user to the index route
router.post('/fruits', (req, res)=>{
    if(req.body.readyToEat === 'on'){
        req.body.readyToEat = true;
    } else {
        req.body.readyToEat = false;
    }
    fruits.push(req.body);
     // redirect the user to the index route
     // since the index route is listening for GET requests
     // with a URL path of '/fruits', we just need to include
     // the URL path as the argument since the .redirect() method
     // has a default HTTP verb of GET.
    res.redirect('/fruits');
});

// show route
// this route will catch GET requests to /fruits/anyValue
// and respond with a single the fruit
router.get('/fruits/:fruitIndex', (req, res) => {
    res.send(fruits[req.params.fruitIndex]);
});

// index route
// this route will catch GET requests to /fruits
// and respond with all the fruits
router.get('/fruits', (req, res) => {
    res.send(fruits);
});

module.exports = router;
```

Because `router` is an object, we can add routes to it and then just export the entire object at once instead of having to export each route.


## Require fruit model in controller file

We no longer need to import the `fruits` data into `server.js` because our references to it are in the routes. So, let's move that import statement from `server.js` to `controllers/fruits_controllers.js`.

```js
const express = require('express');
const router = express.Router();
const fruits = require('../models/fruit_model.js');
```

Notice how we had to modify the path to `fruit_model.js` because we have to go up a level before going into the `models` directory.


## Use the controller file in `server.js`

Now that the routes are no longer in `server.js`, we have to import the controller file into `server.js` and then `.use()` it to direct all requests to it.

```js
const fruitsController = require('./controllers/fruits_controllers.js');
app.use('/fruits', fruitsController);
```

Notice that we identified the route as `/fruits`. This is because all of our routes so far include `/fruits` as the first part of the URL path. So, we can DRY up our code by identifying the first part of the URL path here, and removing it from all of the routes, leaving the parts of the URL paths that make them unique.

## Remove references to base of controller's URLs

The middleware in `server.js` is now taking care of the first part of the URL path that all the routes have in common, so we can remove them from the routes in `controllers/fruits_controllers.js`.

(For those routes that **only** have `/fruits` in their route, we just leave a simple `'/'` to indicate that nothing follows `/fruits` in the URL path. We're essentially saying that `/fruits` is the same thing as `/fruits/`, which is true.)

```js
const express = require('express');
const router = express.Router();

// create route
// this route will catch GET requests to /fruits/newForm
// and respond with a form for creating new fruits
router.get('/newForm', (req, res) => {
    res.render('new.ejs');
});

// create route
// this route will catch POST requests to /fruits
// and, after creating new data, respond by redirecting
// the user to the index route
router.post('/', (req, res)=>{
    if(req.body.readyToEat === 'on'){
        req.body.readyToEat = true;
    } else {
        req.body.readyToEat = false;
    }
    fruits.push(req.body);
     // redirect the user to the index route
     // since the index route is listening for GET requests
     // with a URL path of '/fruits', we just need to include
     // the URL path as the argument since the .redirect() method
     // has a default HTTP verb of GET.
    res.redirect('/fruits');
});

// show route
// this route will catch GET requests to /fruits/anyValue
// and respond with a single the fruit
router.get('/:fruitIndex', (req, res) => {
    res.send(fruits[req.params.fruitIndex]);
});

// index route
// this route will catch GET requests to /fruits
// and respond with all the fruits
router.get('/', (req, res) => {
    res.send(fruits);
});

module.exports = router;
```
