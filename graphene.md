# Graphene

In the early days of the project we developed a concept of a web runtime that was designed to be thinnest possible layer for the web. Runtime that you could pass a URL to an HTML and it would load (and cache) it as it's user interface. 



![image-20200201154707763](image-20200201154707763.png)



Empowering users to (re)shape their User Agent was the vision. But it also supercharged browser concept development and we kept it even as we switched web engine from [gecko][] to [servo][]. This also meant that deploying update was just `git push` away (as demonstrated in video below) and applying update did not required restarts in fact all even tabs were not reloaded.



<video autoplay="true" loop="true" controls src="browser-update.mp4"></video>



There was also a setting in `about:config` to configure URL for the UI. However our update mechanism assumed UI was hosted on github.

[gecko]:https://en.wikipedia.org/wiki/Gecko_(software)
[servo]:https://servo.org/

