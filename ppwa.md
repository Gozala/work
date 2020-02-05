# Native talk

I have being exploring an idea of **progressive [peer-to-peer][] web applications (PPWA)** that would not requiring browsers to ship underlying networking protocol. In this document I will describe an implementation strategy and existing challenges.

### Access Point

General idea is to have a web site that acts as an **access point** to the [p2p](https://en.wikipedia.org/wiki/Peer-to-peer ) network. This site can be static, as its primary job will be installation of a [service_worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API ), that will represent a peer _(corresponding to the active user agent a.k.a browser)_ in the [p2p](https://en.wikipedia.org/wiki/Peer-to-peer ) network.

This service worker will provide a [message channel](https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel )  based interface to PPWAs for accessing / publishing data of the [p2p](https://en.wikipedia.org/wiki/Peer-to-peer ) network. It is worth pointing out that this way PPWAs will be able to publish data even offline _(which will replicate once peer is reconnects)_ and also access data from network that is peer has available in the local cache.

Service worker itself will also attempt to connect to the p2p network by the best means available. In best case scenario native application will accept connection from it on [loopback address](https://en.wikipedia.org/wiki/Loopback ) relaying it to the p2p network. If that fails service worker can fall back to the public gateway or some other known server. If all options fail it still can expose what's in the cache and allow publishing new content.

Service worker can use [origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin ) of client PPWA for access management. It can also prompt user to consent by navigating to own origin & back. Interaction will be similar to how authentication through popular sites happen today.

### Native Application

Native application can be a system service that can use all the capabilities of the underlying OS for p2p networking. Additionally it will include a local [web socket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API ) or [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol ) server to accept connections from an access point.

Loopback addresses like `127.0.0.1` and `::1` are considered [Potentially Trustworthy](https://w3c.github.io/webappsec-secure-contexts/#is-origin-trustworthy ) and there for connections to them from [secure contexts](https://w3c.github.io/webappsec-secure-contexts/#secure-contexts ) is allowed. That is to say that access point site can be served from `https` URL but still be able to connect to the native application.

As of this writing sadly Chrome is only browser that work as expected. In Firefox [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch ) request works from the document context but not from the service worker. Web socket connections also do not work  _(Tracked by [Bug 1376309](https://bugzilla.mozilla.org/show_bug.cgi?id=1376309 ))_. Safari does not seems to treat loopback addresses as potentially trustworthy.

As [explained by Let's Encrypt](https://letsencrypt.org/docs/certificates-for-localhost/ ) there is no good way for native application to serve over HTTPS. Only viable alternative I think is to generate a private key and self-signed certificate in the native application and add it to the locally trusted roots of the system _(do not know how to do later programmatically)_.

> It appears that [dropbox](http://dropbox.com/ ) desktop application exposes extra capabilities to it's website via web socket server on `127.0.0.1`. It is also uses [flash](https://en.wikipedia.org/wiki/Adobe_Flash ) based work around on Firefox  & I'd guess Safari.

### Progressive [peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer ) web applications

Applications will have a setup very similar to access point. They can be fully static with a sole goal of installing [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API ) and connecting it to a access point. That would require embedding a access point document via [iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe ) and obtaining a [MessagePort](https://developer.mozilla.org/en-US/docs/Web/API/MessagePort ) from the access point service worker and passing it over to an own service worker. Once channel between service workers is set up PPWA becomes capable of leveraging p2p network for loading it's own resources or publishing new data.

It is worth pointing out that PWAA could in fact be hosted by the underlying p2p network. Access point site could also provide access to it through own subdomain without requiring any assistance of an http host. Finally once loaded PWAA can stay fully functional even while offline.

### Conclusion

What excites me the most about this idea is that provides a way for us to get fully decentralized collaborative applications like [pushpin](https://inkandswitch.github.io/pushpin/ ) or [pixelpusher](https://medium.com/@pvh/pixelpusher-real-time-peer-to-peer-collaboration-with-react-7c7bc8ecbf74 ) in the **stock browser** and without a need to reach through the cloud in native companion is available. And when it is not access point will take care of degrading to other transport options.

I think it also might be interesting to explore access point as an account. That could allow connecting it to alternative peers like dedicated replication service you pay for or maybe be a box in the closet that your phone offloads all the work to.

[peer-to-peer]:https://en.wikipedia.org/wiki/Peer-to-peer