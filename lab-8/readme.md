## Lab-8: Middleware


Each step of a middleware chain is a function that takes three arguments: req, res, and next, in that order.

**req**

This argument is short for Request. It contains information about the server request that initiated execution. The req object contains user input information, such as the content-type that the user accepts, the user's login and locale information, or session information. The req argument parses query string parameters and assigns them to the req.querystring object.

**res**

This argument is short for Response. It contains functionality for outputting data back to the client. For example:

- res.cacheExpiration(24): Sets cache expiration to 24 hours from now.
- res.render(templateName, data): Outputs an ISML template back to the client and assigns data to pdict.
- res.json(data): Prints a JSON object back to the screen. It's helpful in creating AJAX service endpoints that you want to execute from the client-side scripts.
- res.setViewData(data): Doesn't render anything, but sets the output object. This behavior can be helpful if you want to add multiple objects to the pdict of the template. The pdict contains the information for rendering that is passed to the template. setViewData merges all the data that you passed into a single object, so you can call it at every step of the middleware chain. For example, you can create a separate middleware function that retrieves information about a user's locale to render a language switch on the page. The output object of the ISML template or JSON is set after every step of the middleware chain is complete.

You can also use the ViewData object to extend the data created in a controller that you are extending. You don't have to duplicate the logic used in the original controller to get the data. You only have to add the additional data to the ViewData object and render it.

**next()**

Executing the next function notifies the server that you are done with a middleware step so that it can execute the next step in the chain.

By chaining multiple middleware functions, you can compartmentalize your code and extend or modify routes without having to rewrite them.

**Example 1: Conditionally Executing a Middleware Step**

This example shows a main function that conditionally executes next()or next(new Error()) depending on whether an Apple Pay order is being placed.
```javascript
server.post('Submit', function (req, res, next) {
    var order = OrderMgr.getOrder(req.querystring.order_id);

    if (!order && req.querystring.order_token !== order.getOrderToken()) {
        return next(new Error('Order token does not match'));
    }

    var orderPlacementStatus = orderHelpers.placeOrder(order);

    if (orderPlacementStatus.error) {
        return next(new Error('Could not place order'));
    }

    var orderModel = orderHelpers.buildOrderModel(order);
    res.render('checkout/confirmation/confirmation', { order: orderModel });
    return next();
});
```

The code executed between the first and last parameter is referred to as middleware and the entire process is called chaining. You can create middleware functions to limit route access, add information to the data object passed to the template for rendering, or for any other purpose. One limitation to this approach is that you must call the next function at the end of every step in the chain. Otherwise, the next function in the chain is not executed.


**Event Emitters**

The server module emits events at every step of execution and you can subscribe and unsubscribe to events from a given route. Use an event emitter to override the middleware chain by removing the event listener and creating a new one. However, if you have to change individual steps in a middleware chain, we recommend that you replace a route. While SFRA does supply removeListener and removeAllListener functions, they don't recognize named event emitters. For this reason, it isn't possible to use Step event emitters to override a specific step in the middleware chain.

The following is a list of currently supported events:

- route:BeforeComplete is emitted before the route:Complete event but after all middleware functions. Used to store user submitted data to the database; most commonly in forms.
- route:Complete is emitted after all steps in the chain finish execution. Subscribed to by the server to render ISML or JSON back to the client.
- route:Redirect is emitted before res.redirect execution.
- route:Start is emitted as before middleware chain execution.
- route:Step is emitted before execution of each step in the middleware chain.
All events provide both req and res as parameters to all handlers.

Subscribing or unsubscribing to an event lets you do complex and interesting things. For example, the server subscribes to the route:Complete event to render ISML back to the client. If you want to use something other than ISML to render the content of your template, you can unsubscribe from the route:Complete event. You can subscribe to it again with a function that uses your own rendering engine instead of ISML, without modifying any of the existing controllers.

**OnRequest and OnSession Event Handlers**

The OnRequest and OnSession event handlers that were implemented as pipelines in SiteGenesis Pipeline Processor (SGPP) and as controllers in SGJC are not used in SFRA. You still have access to request and session data using the middleware req (request) and res (response) objects. However, SFRA avoids using OnRequest and OnSession anywhere in our code outside of the server module.

If you want to implement OnRequest and OnSession, they must be implemented through hooks. B2C Commerce looks for OnSession as the controller name, but the new architecture doesn't do that. The only difference between a hook and controller is that the hook doesn't have access to the req and res objects.


**Inheriting Functionality from Another Controller and Extending It**

It's important to understand when to extend a controller and when to override it, because this decision can significantly impact functionality and performance.

**When do I want to Override?**

It's best to override if you want to avoid executing the middleware of the controller or script you're modifying.

When extending a controller, you first execute the original middleware, and then execute the additional steps included in your extension. If the original middleware steps include interaction with a third-party system, that interaction is still executed. If your extension also includes the interaction, the interaction is executed twice. Similarly, if the original middleware includes one or more steps that are computationally expensive, you can avoid executing the original middleware.

**When do I want to Extend?**

If the middleware you want to override is looking up a string or performing inexpensive operations, you can extend the controller or module.

**How do I extend or Override?**

Use the module.superModule mechanism to import the functionality from a controller and then override or add to it.


**Example: Adding Product Reviews to Product.Js**

This example customizes the product detail page to include product review information. The code for this example is available in the Plugin_reviews demo cartridge.

In this example, the Product.js controller uses the following APIs for customization:
- module.superModule: Imports functionality from the first controller with the same name and location found to the right of the current cartridge on the cartridge path.
- server.extend: Inherits the existing server object and extends it with a list of new routes from the super module. In this case, it adds the routes from the module.superModule Project.js file.
- server.append: Modifies the Show route by appending middleware that adds properties to the viewData object for rendering. Using server.append causes a route to execute both the original middleware chain and any additional steps. If you're interacting with a web service or third-party system, don’t use server.append.
- res.getViewData: Gets the current viewData object from the response object.
- res.setViewData: Updates the viewData object used for rendering the template. Create your customized template in the same location and with the same name as the template rendered by the superModule controller. For example, if this controller inherits functionality from app_storefront_base, the rendering template depends on the product type being rendered. The rendering template can be either product/productDetails, product/bundleDetails, or product/setDetails.


```javascript
// Product.js

'use strict';

var server = require('server');
var page = module.superModule;        //inherits functionality from next Product.js found to the right on the cartridge path
server.extend(page);                  //extends existing server object with a list of new routes from the Product.js found by module.superModule

server.append('Show', function (req, res, next) { //adds additional middleware
    var viewData = res.getViewData();
    viewData.product.reviews = [{
        text: 'Lorem ipsum dolor sit amet, cibo utroque ne vis, has no sumo graece.' +
          ' Dicta persius his id. Ea maluisset scripserit contentiones quo, est ne movet dicam.' +
          ' Equidem scriptorem vis no. Civibus tacimates interpretaris has et,' +
          ' ei offendit ocurreret vis, eos purto pertinax eleifend ea.',
        rating: 3.5
    }, {
        text: 'Very short review',
        rating: 5
    }, {
        text: 'Lorem ipsum dolor sit amet, cibo utroque ne vis, has no sumo graece.',
        rating: 1.5
    }];

    res.setViewData(viewData);
    next();
});

module.exports = server.exports();
```


**Replacing or Adding a Route** 

If you want to completely replace a route, rather than append it, use module.superModule to inherit the functionality of the controller and route you want to replace. Then register the functions you want the route to use.

**Example: replacing the Product-Varation route**

In your custom cartridge, create a Product.js file in the same location as the Product.js file in the base cartridge. Use the following code to import the functionality of Product.js and redefine it.

```javascript
var page = require('app_storefront_base/cartridge/controller/Product');
var server = require('server);

server.extend(page);

server.replace('Show', server.middleware.get, function(req, res, next){
    res.render('myNewTemplate');
    next();
});

```


**Overriding Instead of Replacing a Step in the Middleware Chain**

You can use the middleware functions provided by Commerce Cloud or create your own. We recommend that you replace a route when changing access.

These middleware filtering functions are provided by Commerce Cloud:

 - get: Filter for get requests
 - htt: Filter for http requests
 - https: Filter for https requests
 - include: Filter for remote includes
 - post: Filter for post requests

If the request doesn't match the filtering condition, the function returns an Error with the text Params do not match route.


**Discovering deeply 'cartridge/scripts/middleware'**

**1. https access**
   
You can enhance this code by adding the server.middleware.https parameter after Show, to limit this route to only allow HTTPS requests. This example restricts the Account-Show route to HTTPS.

```javascript

'use strict';

var server = require('server');    //the server module is used by all controllers
var cache = require('*/cartridge/scripts/middleware/cache');

server.get('Show', server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

**2. cache**

2.1 cache.applyDefaultCache - Applies the default expiration value for the page cache.
```javascript

'use strict';

var server = require('server');    //the server module is used by all controllers
var cache = require('*/cartridge/scripts/middleware/cache');

server.get('Show', cache.applyDefaultCache, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

2.2 cache.applyPromotionSensitiveCache - Applies the default price promotion page cache.
```javascript

'use strict';

var server = require('server');    //the server module is used by all controllers
var cache = require('*/cartridge/scripts/middleware/cache');

server.get('Show', cache.applyPromotionSensitiveCache, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

2.3 cache.applyShortPromotionSensitiveCache - Applies the default price promotion page cache.
```javascript

'use strict';

var server = require('server');    //the server module is used by all controllers
var cache = require('*/cartridge/scripts/middleware/cache');

server.get('Show', cache.applyShortPromotionSensitiveCache, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

2.4 cache.applyInventorySensitiveCache -  Applies the inventory sensitive page cache.
```javascript

'use strict';

var server = require('server');    //the server module is used by all controllers
var cache = require('*/cartridge/scripts/middleware/cache');

server.get('Show', cache.applyInventorySensitiveCache, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

3. consentTracking

Middleware to use consent tracking check

```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var consentTracking = require('*/cartridge/scripts/middleware/consentTracking');

server.get('Show', consentTracking.consent, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

4. pageMetaData - Middleware to compute request pageMetaData object

```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var pageMetaData = require('*/cartridge/scripts/middleware/pageMetaData');

server.get('Show', pageMetaData.computedPageMetaData, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

```html
<iscontent type="text/html" charset="UTF-8" compact="true" />

<html>
    <head>
        <title>${ Resource.msg('Home') }</title>
    </head>
    <body>
        <span> ${pdict.CurrentPageMetaData.title }</span>
        <span> ${pdict.CurrentPageMetaData.description }</span>
        <span> ${pdict.CurrentPageMetaData.keywords }</span>
        <span> ${pdict.CurrentPageMetaData.pageMetaTags }</span>
    </body>
</html>
```

5. userLoggedIn 

5.1 validateLoggedIn - Middleware validating if user logged in
```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var userLoggedIn = require('*/cartridge/scripts/middleware/userLoggedIn');

server.get('Show', userLoggedIn.validateLoggedIn, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

5.2 validateLoggedInAjax - Middleware validating if user logged in from ajax request

```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var userLoggedIn = require('*/cartridge/scripts/middleware/userLoggedIn');

server.get('Show', userLoggedIn.validateLoggedInAjax, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

6. csrf

6.1 validateRequest - Middleware validating CSRF token
```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var csrf = require('*/cartridge/scripts/middleware/csrf');

server.get('Show', csrf.validateRequest, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```
6.2 validateAjaxRequest - Middleware validating CSRF token from ajax request
```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var csrf = require('*/cartridge/scripts/middleware/csrf');

server.get('Show', csrf.validateAjaxRequest, server.middleware.https, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

6.3 generateToken - Middleware generating a csrf token and setting the view data
```javascript
'use strict';

var server = require('server');    //the server module is used by all controllers
var csrf = require('*/cartridge/scripts/middleware/csrf');

server.get('Show', csrf.generateToken, function (req, res, next) {  //registers the Show route for the Home module
    res.render('/home/homepage');      //renders the hompage template
    next();            //notifies middleware chain that it can move to the next step or terminate if this is the last step.
});

module.exports = server.exports();
```

```html
<iscontent type="text/html" charset="UTF-8" compact="true" />

<html>
    <head>
        <title>${ Resource.msg('Home') }</title>
    </head>
    <body>
        <span> ${pdict.csrf.tokenName }</span>
        <span> ${pdict.csrf.token }</span>
       
    </body>
</html>
```

[**back to labs**](../README.md) | [next](../lab-9/readme.md)
