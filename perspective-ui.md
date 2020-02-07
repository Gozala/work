



# Perspectieve UI

Introduction of [3D touch][] inspired us to explore a new dimension. We completely redesigned user interface, stepped back from [cards](./web-cards) concept to more familiar tabs model as a default UI for [servo][] nightly builds. 



![browserhtml](browserhtml.gif)



In this iteration we opted into more minimal progress bar and spend a lot of time to improving load-time perception it created. Turns out same load time can be perceived as very slow or fast based on how it progresses. 



![image-20200201163441623](image-20200201163441623.png)



We made browser chrome some more minimal that it became indistinguishable from the content, so we made chrome fade-in  when you'd reach for it.



![image-20200201163643365](image-20200201163643365.png)



New tab button was added to the right bottom corner overlaying the content. And on the left bottom corner we placed a back button only if there was a page to go back to.



![image-20200201163828316](image-20200201163828316.png)

Clicking address bar transitions you UI to the edit mode.

![image-20200201163719434](image-20200201163719434.png)



Tabs were revealed on [3D touch][], when switching between tabs with `control tab` or via little hamburger menu button in the top right corner.



![image-20200201153940273](image-20200201153940273.png)



We also made it possible to pin tabs as a small sidebar so your tabs would always be at your fingertips.



![image-20200201154206720](image-20200201154206720.png)



[servo]:https://servo.org/
[3D touch]:https://developer.apple.com/design/human-interface-guidelines/ios/user-interaction/3d-touch/