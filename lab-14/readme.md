# Lab-14: Server replace

    If you want to completely replace a route, rather than append it, use module.superModule to inherit the functionality of the controller and route you want to replace. Then register the functions you want the route to use.

    Example: replacing the Product-Varation route

    In your custom cartridge, create a Product.js file in the same location as the Product.js file in the base cartridge. Use the following code to import the functionality of Product.js and redefine it.
    
![](assets/img/Screenshot_29.png)

[**back to labs**](../README.md) 
