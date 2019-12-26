## Lab-4: Controllers extend


Use the module.superModule mechanism to import the functionality from a controller and then override or add to it.

In this example, the Product.js controller uses the following APIs for customization:

module.superModule: Imports functionality from the first controller with the same name and location found to the right of the current cartridge on the cartridge path.

server.extend: Inherits the existing server object and extends it with a list of new routes from the super module. In this case, it adds the routes from the module.superModule Project.js file.

server.append: Modifies the Show route by appending middleware that adds properties to the viewData object for rendering. Using server.append causes a route to execute both the original middleware chain and any additional steps. If you're interacting with a web service or third-party system, don’t use server.append.

res.getViewData: Gets the current viewData object from the response object.

res.setViewData: Updates the viewData object used for rendering the template. Create your customized template in the same location and with the same name as the template rendered by the superModule controller. For example, if this controller inherits functionality from app_storefront_base, the rendering template depends on the product type being rendered. The rendering template can be either product/productDetails, product/bundleDetails, or product/setDetails.

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



[**back to labs**](../README.md) | [next](../lab-5/readme.md)
