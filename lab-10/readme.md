## Lab-10: Creating Social Networks Links

 In this lab you need to create and add a social networks links to the product template.
To extend template we need to create _\_ext_ cartridge folder

- app_storefront_base_ext/cartridge/templates/default/product/components/socialIcons.isml

Add this code to the list in the file, go to the browser and check how it works

 ```html
<li>
    <a href="https://www.google.com/search?q=${product.productName}" 
        data-share="google"
        title="${Resource.msgf('label.social.google', 'product', null, product.productName)}"
        aria-label="${Resource.msgf('label.social.google', 'product', null, product.productName)}"
        class="share-icons"
        target="_blank" >
        <i class="fa fa-google"></i>
    </a>
</li>

Do not forget to update cartridge path!
 ```


[**back to labs**](../README.md) | [next](../lab-11/readme.md)
