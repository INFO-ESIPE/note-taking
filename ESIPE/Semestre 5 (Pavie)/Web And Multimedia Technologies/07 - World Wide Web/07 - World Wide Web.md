[[Web And Multimedia Technologies]]
28/10/2024
****

Internet is a network of networks. It is based on the ARPANET project and follows the TCP/IP protocols
	*The web is only a fraction of it that appeared a decade later (1989 at CERN, CH), it also includes VOIP, SSH/Telnet, FTP ...*

The web is simply a service — on the internet — that exploits hypertext
technology to distribute documents.


An **Hypertext** is a text that contains links to other documents. It is called an **Hypermedia** if it links to multimedia documents (images, videos)
	*This provides non-sequential browsing, which is very useful for web browser's crawlers (and for Google Dorks/Google Hacking)*

Resources are located via their unique **Uniform Resource Locator (URL)**


****
## Client-Server architecture

The **server** is the hardware that provides a service. It is constantly waiting for incoming connections, and then serves them until the connection is interrupted
	*Each service is associated to a port. For HTTP, it is 80; For HTTPS (TLS over HTTP), it is 443. This allows a server to host various services and clearly separate each.
	For instance, a web server will — most of the time — supports both HTTP and HTTPS*

The **client** is the hardware that contacts the remote service

Web is based on this basic architecture
	*Client requests a service (a web page, a script, a document) to the server
	The server returns what he asks (document, hypertext ...)*
