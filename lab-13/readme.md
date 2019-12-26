## Lab-13: ClickStream

    Represents the click stream in the session. A maximum number of 50 clicks is recorded per session. After the maximum is reached, each time the customer clicks on a new link, the oldest click stream entry is purged. The ClickStream always remembers the first click.
    The click stream is consulted when the GetLastVisitedProducts pipelet is called to retrieve the products that the customer has recently visited.

    This class does not have a constructor, so you cannot create it directly.


![](assets/img/image_2019-11-11_15-21-21.png)

![](assets/img/Screenshot_30.png)
![](assets/img/Screenshot_31.png)


[**back to labs**](../README.md) | [next](../lab-14/readme.md)
