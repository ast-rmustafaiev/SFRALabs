Open questions:

General
Custom cartridge setup has not been provided
(There is could be extra section for this, we can employ sgfm-scripts createCartridge
command)

Lab-4
There is an error in the product model, not sure whether 
it my setup were wrong or an error in general ...

Lab-5, 6
As per  (lab-4)  product.isml template there were
  <body>
      <h1>${JSON.stringify(pdict.Product)}</h1>
  </body>
 Product log out, not sure that is necessary 
 lets keep simply name & description.
 
Lab-7
https://astound05.alliance-prtnr-eu03.dw.demandware.net/on/demandware.store/Sites-RefArch-Site/en_US/ShowProduct-Start?pid=21736758M
an error occurs productTile.js
Uncaught ReferenceError: $ is not defined
    at Object.70 (productTile.js:1)
    at o (productTile.js:1)
    at productTile.js:1
    at productTile.js:1
1. It cant find jQuerry.
2. perhaps compile scripts doesn't accumulate bundle right way
3. It is required to adjust webpack scripts in order to show/allow source maps.
Currently sources is minified out of the box
4. Change heading/text in the lab ...
Decorator Templates in SFRA
   
   There are only two decorator templates:
   page.isml: Contains navigation information
   checkout.isml: Doesn't contain navigation information. Removing navigation information has been shown to improve the percentage of cart abandonment.
   These templates are located in app_storefront_base/cartridge/templates/default/common/layout/.
   Note: All of the pt_..._VARS and pt_..._UI templates that existed in previous versions of SiteGenesis were removed.

Lab-8
The section 
**Overriding Instead of Replacing a Step in the Middleware Chain**
Could be devided by sub-tasks, as its unclear what to do and what to commit.

Lab-9
The lab lucking a practical example, so newcomer could 
try something yourself.

Lab-10
No comments

Lab-11
The lab should reflect SFRA recommendation
Caching is no longer should be present or handled by templates!
The aim is to use controllers instead, as per layer which should control it

Lab-12
WIP

Lab-13
WIP

Lab-14
Incomplete lab, duplicates that has done Lab-8 **replace route**


