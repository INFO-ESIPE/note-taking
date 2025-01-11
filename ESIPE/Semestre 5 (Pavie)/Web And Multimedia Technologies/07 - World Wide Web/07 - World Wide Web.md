[[Web And Multimedia Technologies]]
28/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Internet

Internet is a network of networks. It is based on the ARPANET project and follows the TCP/IP protocols
	*The web is only a fraction of it that appeared a decade later (1989 at CERN, CH), it also includes VOIP, SSH/Telnet, FTP ...*

The web is simply a service—on the internet—that exploits hypertext technology to distribute documents

Machines are identified on the internet by an IP address


An **Hypertext** is a text that contains links to other documents. It is called an **Hypermedia** if it links to multimedia documents (images, videos)
	*This provides non-sequential browsing, which is very useful for web browser's crawlers (and for Google Dorks/Google Hacking)*

Resources are located via their unique **Uniform Resource Locator (URL)**


****
## Client-Server architecture

The **server** is the hardware that provides a service. It is constantly waiting for incoming connections, and then serves them until the connection is interrupted.

> [!info]
> Each service is associated to a port. For HTTP, it is 80; For HTTPS (TLS over HTTP), it is 443. This allows a server to host various services and clearly separate each.
> For instance, a web server will—most of the time—supports both HTTP and HTTPS

The **client** is the hardware that contacts the remote service

Web is based on this basic architecture
	*Client requests a service (a web page, a script, a document) to the server
	The server returns what he asks (document, hypertext ...)*


****
## HTTP

HyperText Transfer Protocol (HTTP) is a protocol that allows document and multimedia exchange over TCP.

Example of an HTTP request (asking for a resource) :
```http
GET / HTTP/1.0
Host: http://igm.univ-eiffel.fr/
```

`GET` is an **HTTP method**. Those are specifying what action we are aiming at :

| Method     | Meaning                                                                   |
| ---------- | ------------------------------------------------------------------------- |
| **GET**    | We want to retrieve data                                                  |
| **POST**   | We want to submit a set of data to the server                             |
| **PUT**    | We want the server to update all occurrences of our data if they exists   |
| **DELETE** | We want the server to delete the specified data                           |
| HEAD       | Same as GET, but without the response's body (only metadata is retrieved) |
| CONNECT    | Establishes a tunnel to the server identified by the target resource.     |
| OPTIONS    | Describes the communication options for the target resource.              |
> [!info] There are two others we don't care about. The most important are the bold ones


HTTP Response (from the server)  :
```http
HTTP/1.1 302 Found
Date: Mon, 21 Dec 2015 17:15:35 GMT
Server: Apache
Location: http://igm.u-pem.fr/
Content-Length: 269
Connection: close
Content-Type: text/html; charset=iso-8859-1
```

An HTTP response has a **code** (302 in the example above) indicating how the operation went :

| Code | Meaning                                      | Common codes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1XX  | Informational response (rare, not so useful) | N.A.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 2XX  | Success                                      | **200: OK**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 3XX  | Redirection                                  | **301: Moved Permanently**<br>   *Specified URL changed, a new one is provided by the server and the user will be redirected to it*<br>**302: Found**<br>   *Same as 301 but temporary*                                                                                                                                                                                                                                                                                                                                         |
| 4XX  | Client-side error                            | **400: Bad Request**<br>   *Server can not process the request as it is malformed, has invalid syntax...*<br>**403: Forbidden**<br>   *The user is not authorised to access the resource*<br>**404: Not Found**<br>   *The requested resource could not be found by the server, the resource does not exist*<br>**405: Method Not Allowed**<br>   *The endpoint does not accept the HTTP method (e.g. We execute a POST method, but the endpoint is for a DELETE method*<br>**429: Too Many Requests**<br>   *Self explanatory* |
| 5XX  | Server-side error                            | **500: Internal Server error**<br>   *Generic error indicating the server has encountered an error while processing the request*<br>**502: Bad Gateway**<br>   *The server, while working as a gateway to get a response needed to handle the request, got an invalid response*<br>**503: Service Unavailable**<br>   *Server can not currently handle the request (usually because the server is down)*                                                                                                                        |


***
## 3D for Web

Several solutions exists:
- **Quick Time Virtual Reality (QVRT)**: Created by apple, allows to build "3D panoramas" from photos.
- **WebGL**: Common solution. JavaScript API for rendering interactive 2D and 3D graphics within any compatible web browser without the need for a plugin (it uses the `canvas` tag).


***
## Hidden Services

We usually divide the web into three big chunks:
- **Surface web**: Pages that are indexed by search engines.
	*Wikipedia pages, BBC articles, Amazon articles...*
- **Deep web**: Any page that is not directly indexed by a search engine (dynamic pages, password protected content...)
	*Your email inbox, Banking portal, Your private discord channels...*
- **Dark web**: A subset of the deep web that is intentionally hidden and designed to be accessed only through specialised tools (e.g., TOR browser).
	*Whistleblower platforms, Illicit marketplaces or forums, privacy-focused services*


