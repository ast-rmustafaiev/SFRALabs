## Lab-2: Finding an Error on a Controller

If you had any problems in [Lab-1](../lab-1/readme.md) you can figure it out in two ways: using the Request Log, the Problem View or using DWithEase in your browser we installed before.

**1. Use the Request Log:**
   1. Using the URL you bookmarked, invoke the Hello controller with a missing start node: *https://your-sandbox-name-dw.demandware.net/on/demandware.store/Sites-RefArch-Site/en_US/Hello-Show*
   2. Verify that an error occurs on the [storefront](https://documentation.b2c.commercecloud.salesforce.com/DOC1/index.jsp?topic=%2Fcom.demandware.dochelp%2FSiteDevelopment%2FViewStorefront.html&cp=0_1_6_8).
   3. Mouse-over the upper-left corner of the page to re-enable [the Storefront Toolkit](https://documentation.b2c.commercecloud.salesforce.com/DOC1/index.jsp?topic=%2Fcom.demandware.dochelp%2FStorefrontToolkit%2FStorefrontToolkit.html&resultof=%22Storefront%22%20%22storefront%22%20%22Toolkit%22%20%22toolkit%22%20).
   4. From the toolkit, select [Request Log](https://documentation.b2c.commercecloud.salesforce.com/DOC1/index.jsp?topic=%2Fcom.demandware.dochelp%2FSTReference%2FRequestlogtool.html&resultof=%22Request%22%20%22request%22%20%22Log%22%20%22log%22%20).
   5. Re-login to Business Manager if required and reopen the request log.
   6. Read the request log to find the error message.
   7. Correct the controller invocation (Hello-World) and refresh the page.
   8. In the browser, retest the Hello-World Controller.
   9. Use the Request Log to determine what error occurs.
   10. Correct the error and retest.


**2. Use the Problem View**
   1. Remove the interaction node from the controller.
   2. Look in the Problems view in the Controller Editor.
   3. Fix the error.
   4. Retest the Hello-World controller to make sure it works.


[**back to labs**](../README.md) | [next](../lab-3/readme.md)
