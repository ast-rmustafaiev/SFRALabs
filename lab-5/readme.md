## Lab-5: Templates

1. Create ISML template lab4/product.isml which shows the name of the Product. For this we use name property of the Product object, passed to the template by the controller:
      ```javascript
    <html>
        <head>
            <title>${ Resource.msg('Hello') }</title>
        </head>
        <body>
            <h1>${JSON.stringify(pdict.Product)}</h1>
        </body>
    </html>
    ```

2. Create ISML template lab4/productnotfound.isml with some simple text describing that the system can't find such product.

    ```javascript
    <html>
        <head>
            <title>${ Resource.msg('Hello') }</title>
        </head>
        <body>
            <h1>${pdict.Log}</h1>
        </body>
    </html>
    ```
3. Request the ShowProduct-Start controller appending a URL query string containing the product id: ShowProduct-Start?product=54399.
4. Debug and verify.
        ![](../assets/img/Screenshot_27.png)
5. Commit and Push to new branch, create Pull Request

[**back to labs**](../README.md) | [next](../lab-6/readme.md)
