## Lab-9: Models and js decorators

Models are serializable JSON objects. A model takes the object as a parameter and construct itself by retrieving neccesery information from the object. The model applies bussiness logic while it's doing the convertion to represent information in a usable format. Some models are complicated, for example Product is the most complicated model in the SFRA. There are several varities of products such as Product, ProductBundle, ProductLineItem,  ProductSet etc. To make it easier for you to customize the features, that are available we use decorator patterns. A decorator pattern is a standart development approach when you have to decorate an object with the set of attributes and/or actions. Each attribute is a separate decorator and is applied to the object to add functionality. We use a decorator pattern for more complicated models to separate functionality and allow you to easily supstitude it for different functionality where needed. Different models such as Product, ProductBundle,  ProductSet share the same decorators, for example the productImages decorator is shared amongst Product, ProductBundle,  ProductSet models. If you need to change how the images are retrieved from the system, for example if you want to store images on an external server instead of using the bussiness manager you only have to change imagedecorator and all models that are also using it will have the new information.
        ![](../assets/img/Screenshot_28.png)

To see how it works go to: */cartridge/models/product/decorators/index.js where are imporrted all decorator in the folder. Discover some of them, for example base.js decorator.

```javascript
'use strict';

module.exports = function (object, apiProduct, type) {
    Object.defineProperty(object, 'uuid', {
        enumerable: true,
        value: apiProduct.UUID
    });

    Object.defineProperty(object, 'id', {
        enumerable: true,
        value: apiProduct.ID
    });

    Object.defineProperty(object, 'productName', {
        enumerable: true,
        value: apiProduct.name
    });

    Object.defineProperty(object, 'productType', {
        enumerable: true,
        value: type
    });

    Object.defineProperty(object, 'brand', {
        enumerable: true,
        value: apiProduct.brand
    });
};
```

The go to fullProduct.js in product folder. And check the usage out.

```javascript
'use strict';

var decorators = require('*/cartridge/models/product/decorators/index');

/**
 * Decorate product with full product information
 * @param {Object} product - Product Model to be decorated
 * @param {dw.catalog.Product} apiProduct - Product information returned by the script API
 * @param {Object} options - Options passed in from the factory
 * @property {dw.catalog.ProductVarationModel} options.variationModel - Variation model returned by the API
 * @property {Object} options.options - Options provided on the query string
 * @property {dw.catalog.ProductOptionModel} options.optionModel - Options model returned by the API
 * @property {dw.util.Collection} options.promotions - Active promotions for a given product
 * @property {number} options.quantity - Current selected quantity
 * @property {Object} options.variables - Variables passed in on the query string
 *
 * @returns {Object} - Decorated product model
 */
module.exports = function fullProduct(product, apiProduct, options) {
    decorators.base(product, apiProduct, options.productType);
    decorators.price(product, apiProduct, options.promotions, false, options.optionModel);

    if (options.variationModel) {
        decorators.images(product, options.variationModel, { types: ['large', 'small'], quantity: 'all' });
    } else {
        decorators.images(product, apiProduct, { types: ['large', 'small'], quantity: 'all' });
    }

    decorators.quantity(product, apiProduct, options.quantity);
    decorators.variationAttributes(product, options.variationModel, {
        attributes: '*',
        endPoint: 'Variation'
    });
    decorators.description(product, apiProduct);
    decorators.ratings(product);
    decorators.promotions(product, options.promotions);
    decorators.attributes(product, apiProduct.attributeModel);
    decorators.availability(product, options.quantity, apiProduct.minOrderQuantity.value, apiProduct.availabilityModel);
    decorators.options(product, options.optionModel, options.variables, options.quantity);
    decorators.quantitySelector(product, apiProduct.stepQuantity.value, options.variables, options.options);

    var category = apiProduct.getPrimaryCategory();
    if (!category && options.productType !== 'master') {
        category = apiProduct.getMasterProduct().getPrimaryCategory();
    }

    if (category) {
        decorators.sizeChart(product, category.custom.sizeChartID);
    }

    decorators.currentUrl(product, options.variationModel, options.optionModel, 'Product-Show', apiProduct.ID, options.quantity);
    decorators.readyToOrder(product, options.variationModel);
    decorators.online(product, apiProduct);
    decorators.raw(product, apiProduct);
    decorators.pageMetaData(product, apiProduct);
    decorators.template(product, apiProduct);

    return product;
};

```

Then this model is imported and used in scripts: 

```javascript
'use strict';

var ProductMgr = require('dw/catalog/ProductMgr');
var PromotionMgr = require('dw/campaign/PromotionMgr');
var productHelper = require('*/cartridge/scripts/helpers/productHelpers');
var productTile = require('*/cartridge/models/product/productTile');
var bonusProduct = require('*/cartridge/models/product/bonusProduct');
var fullProduct = require('*/cartridge/models/product/fullProduct');
var productSet = require('*/cartridge/models/product/productSet');
var productBundle = require('*/cartridge/models/product/productBundle');
var productLineItem = require('*/cartridge/models/productLineItem/productLineItem');
var bonusProductLineItem = require('*/cartridge/models/productLineItem/bonusProductLineItem');
var bundleProductLineItem = require('*/cartridge/models/productLineItem/bundleLineItem');
var orderLineItem = require('*/cartridge/models/productLineItem/orderLineItem');
var bonusOrderLineItem = require('*/cartridge/models/productLineItem/bonusOrderLineItem');
var bundleOrderLineItem = require('*/cartridge/models/productLineItem/bundleOrderLineItem');

module.exports = {
    get: function (params) {
        var productId = params.pid;
        var product = Object.create(null);
        var apiProduct = ProductMgr.getProduct(productId);
        if (apiProduct === null) {
            return product;
        }
        var productType = productHelper.getProductType(apiProduct);
        var options = null;
        var promotions;

        switch (params.pview) {
            case 'tile':
                product = productTile(product, apiProduct, productType);
                break;
            case 'bonusProductLineItem':
                promotions = PromotionMgr.activeCustomerPromotions.getProductPromotions(apiProduct);
                options = {
                    promotions: promotions,
                    quantity: params.quantity,
                    variables: params.variables,
                    lineItem: params.lineItem,
                    productType: productType
                };

                switch (productType) {
                    case 'bundle':
                        // product = bundleProductLineItem(product, apiProduct, options, this);
                        break;
                    default:
                        var variationsBundle = productHelper.getVariationModel(apiProduct, params.variables);
                        if (variationsBundle) {
                            apiProduct = variationsBundle.getSelectedVariant() || apiProduct; // eslint-disable-line
                        }

                        var optionModelBundle = apiProduct.optionModel;
                        var optionLineItemsBundle = params.lineItem.optionProductLineItems;
                        var currentOptionModelBundle = productHelper.getCurrentOptionModel(
                            optionModelBundle,
                            productHelper.getLineItemOptions(optionLineItemsBundle, productId)
                        );
                        var lineItemOptionsBundle = optionLineItemsBundle.length
                            ? productHelper.getLineItemOptionNames(optionLineItemsBundle)
                            : productHelper.getDefaultOptions(optionModelBundle, optionModelBundle.options);


                        options.variationModel = variationsBundle;
                        options.lineItemOptions = lineItemOptionsBundle;
                        options.currentOptionModel = currentOptionModelBundle;

                        if (params.containerView === 'order') {
                            product = bonusOrderLineItem(product, apiProduct, options);
                        } else {
                            product = bonusProductLineItem(product, apiProduct, options);
                        }

                        break;
                }

                break;
            case 'productLineItem':
                promotions = PromotionMgr.activeCustomerPromotions.getProductPromotions(apiProduct);
                options = {
                    promotions: promotions,
                    quantity: params.quantity,
                    variables: params.variables,
                    lineItem: params.lineItem,
                    productType: productType
                };

                switch (productType) {
                    case 'bundle':

                        if (params.containerView === 'order') {
                            product = bundleOrderLineItem(product, apiProduct, options, this);
                        } else {
                            product = bundleProductLineItem(product, apiProduct, options, this);
                        }
                        break;
                    default:
                        var variationsPLI = productHelper.getVariationModel(apiProduct, params.variables);
                        if (variationsPLI) {
                            apiProduct = variationsPLI.getSelectedVariant() || apiProduct; // eslint-disable-line
                        }

                        var optionModelPLI = apiProduct.optionModel;
                        var optionLineItemsPLI = params.lineItem.optionProductLineItems;
                        var currentOptionModelPLI = productHelper.getCurrentOptionModel(
                            optionModelPLI,
                            productHelper.getLineItemOptions(optionLineItemsPLI, productId)
                        );
                        var lineItemOptionsPLI = optionLineItemsPLI.length
                            ? productHelper.getLineItemOptionNames(optionLineItemsPLI)
                            : productHelper.getDefaultOptions(optionModelPLI, optionModelPLI.options);


                        options.variationModel = variationsPLI;
                        options.lineItemOptions = lineItemOptionsPLI;
                        options.currentOptionModel = currentOptionModelPLI;

                        if (params.containerView === 'order') {
                            product = orderLineItem(product, apiProduct, options);
                        } else {
                            product = productLineItem(product, apiProduct, options);
                        }

                        break;
                }

                break;
            case 'bonus':
                options = productHelper.getConfig(apiProduct, params);

                switch (productType) {
                    case 'set':
                        break;
                    case 'bundle':
                        break;
                    default:
                        product = bonusProduct(product, options.apiProduct, options, params.duuid);
                        break;
                }

                break;
            default: // PDP
                options = productHelper.getConfig(apiProduct, params);

                switch (productType) {
                    case 'set':
                        product = productSet(product, options.apiProduct, options, this);
                        break;
                    case 'bundle':
                        product = productBundle(product, options.apiProduct, options, this);
                        break;
                    default:
                        product = fullProduct(product, options.apiProduct, options);
                        break;
                }
        }

        return product;
    }
};

```
 
After that it's imported to controller which for example shows us quick view of the product
```javascript
server.get('ShowQuickView', cache.applyPromotionSensitiveCache, function (req, res, next) {
    var URLUtils = require('dw/web/URLUtils');
    var productHelper = require('*/cartridge/scripts/helpers/productHelpers');
    var ProductFactory = require('*/cartridge/scripts/factories/product');
    var renderTemplateHelper = require('*/cartridge/scripts/renderTemplateHelper');
    var Resource = require('dw/web/Resource');

    var params = req.querystring;
    var product = ProductFactory.get(params);
    var addToCartUrl = URLUtils.url('Cart-AddProduct');
    var template = product.productType === 'set'
        ? 'product/setQuickView.isml'
        : 'product/quickView.isml';

    var context = {
        product: product,
        addToCartUrl: addToCartUrl,
        resources: productHelper.getResources(),
        quickViewFullDetailMsg: Resource.msg('link.quickview.viewdetails', 'product', null),
        closeButtonText: Resource.msg('link.quickview.close', 'product', null),
        enterDialogMessage: Resource.msg('msg.enter.quickview', 'product', null),
        template: template
    };

    res.setViewData(context);

    this.on('route:BeforeComplete', function (req, res) { // eslint-disable-line no-shadow
        var viewData = res.getViewData();
        var renderedTemplate = renderTemplateHelper.getRenderedHtml(viewData, viewData.template);

        res.json({
            renderedTemplate: renderedTemplate,
            productUrl: URLUtils.url('Product-Show', 'pid', viewData.product.id).relative().toString()
        });
    });

    next();
});
```

[**back to labs**](../README.md) | [next](../lab-10/readme.md)
