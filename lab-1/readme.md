## Lab-1: Creating the Hello Controller

**1. Create Hello.js controller**

Navigate to your custom cartridge (in my case it's "*app_custom_storefront*") and create *Hello.js* controller.

- *your_cartridge_name/cartridge/controllers/Hello.js*

```javascript
'use strict';

var server = require('server');


server.get('Show', function (req, res, next) {
    var testObject = {
        name: 'testname',
        email: 'testemail@astoundcommerce.com',
        type: 'object'
    }
    res.render('hello/helloTemplate', {testObject: testObject});
    next();
});

module.exports = server.exports();
```



**2. Create template helloTemplate.isml**

Navigate to *your_cartridge_name/cartridge/templates/dafault/hello* folder and create helloTemplate.isml file.

- *your_cartridge_name/cartridge/templates/dafault/hello/helloTemplate*

```html

<iscontent type="text/html" charset="UTF-8" compact="true" />

<html>
    <head>
        <title>${ Resource.msg('title') }</title>
    </head>
    <body>
        <span> ${JSON.stringify(pdict.testObject)} </span>
    </body>
</html>
```

**3. Check your endpoint in browser**

Go to *https://your-sandbox-name-dw.demandware.net/on/demandware.store/Sites-RefArch-Site/en_US/Hello-Show* in your browser. And check or it works.
**Do not forget to change url with your sandbox name**

**4. If all works commit and push it to your branch**

[**back to labs**](../README.md) | [next](../lab-2/readme.md)
