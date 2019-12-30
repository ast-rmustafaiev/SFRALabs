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

Navigate to *your_cartridge_name/cartridge/templates/dafault/lab1* folder and create helloTemplate.isml file.

- *your_cartridge_name/cartridge/templates/dafault/lab1/helloTemplate*

```html

<iscontent type="text/html" charset="UTF-8" compact="true" />

<html>
<head>
    <title>${ Resource.msg('title') }</title>
</head>
<body>
    <isprint value="${JSON.stringify(pdict.testObject)}" />
</body>
</html>
```

**3. Check your endpoint in browser**

Go to *https://your-sandbox-name-dw.demandware.net/on/demandware.store/Sites-RefArch-Site/en_US/Hello-Show* in your browser. And check or it works.
**Do not forget to change url with your sandbox name**

**4. If all works commit and push it to your branch**


**ISML**

Salesforce B2C Commerce uses ISML templates to generate dynamic storefront pages. 
Templates combine HTML with a proprietary language extension referred to as Internet Store Markup Language (ISML). 
ISML templates are similar to Java Server Pages in that both are used to present dynamically generated web content.

[Refer documentation...](https://documentation.b2c.commercecloud.salesforce.com/DOC1/index.jsp?topic=%2Fcom.demandware.dochelp%2FDWAPI%2Fscriptapi%2Fhtml%2Fapi%2Fclass_dw_web_HttpParameterMap.html)

[**back to labs**](../README.md) | [next](../lab-2/readme.md)
