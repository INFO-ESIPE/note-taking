[[Network Services]]
****
**Table of Contents**
```table-of-contents
```

****
## Refresher

Basics of HTTP and W3 are explained [[07 - World Wide Web#HTTP|here]].


***
## URI

### Concept

Resources are identified by a string called **Uniform Resource Identifier** (RFC 3986).
It can be used for both **locating** and **naming** a resource:
- **Uniform Resource Location (URL)** is a type of URI that not only identifies the resource but also provides a means to locate it on the network.

> [!example]- URL Example
> `http://www.u-pem.fr` identifies the university's old website and allows it to be located since the naming is based on the DNS system.

- **Uniform Resource Name (URN)** is a type of URI that provides a globally unique and persistent name for a resource, regardless of its location. This is useful even if the resource no longer exists or is unavailable.

> [!example]- URN Example
> `URN:ISBN:2-7117-8689-7` is the unique identifier of a book

URIs follow a specific syntax and semantics (which are designed to be evolutive to adapt to new protocols and use cases).


### Hierarchical format

Hierarchical URIs are the most common type of URI and are used to locate resources on the internet. They follow this structure: `[<scheme>:][//<authority>][<path>][?<query>][#<fragment>]`

![[uri-structure.png|750]]

1. **Scheme**: The scheme identifies the protocol or purpose of the URI. It begins with a letter and can include letters, numbers, or specific symbols (`+`, `-`, or `.`).
    Usually represents a protocol (`http`, `telnet`, `ftp`), but not necessarily (`urn`, `about`...)
	    *All registered URI schemes are listed by IANA: [URI Schemes](https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml).*

2. **Authority**: Specifies the entity responsible for the resource. It begins with `//` and typically includes:
	`[userinfo@]<host>[:port]`
    - `userinfo`: Optional credentials (e.g., `username:password`).
    - `host`: The domain name or IP address of the server.
    - `port`: Optional port number (defaults to the scheme's standard port, e.g., 80 for HTTP, 443 for HTTPS...).

3. **Path**: Specifies the location of the resource within the authority. It consists of segments separated by `/`. Each segment can include parameters separated by `;`.
   
4. **Query**: Provides additional information for the resource, often in the form of key-value pairs (e.g., `?key1=value1&key2=value2`). 
   The query is optional and begins with `?`.
   
5. **Fragment**: Points to a secondary resource within the primary resource (e.g., a specific section of a webpage). It begins with `#` and is optional.

> [!example]- Hierarchical URI Example
> ```bash
> https: //www.oui.sncf /proposition/outward/train ?wishId=637b88d9-c88c-4416-805d #section2
> ↑_____↑ ↑___________↑ ↑________________________↑ ↑_____________________________↑ ↑_______↑
> Scheme   Authority           Path                        Query                   Fragment
> ```

> [!tip] Good practice
> An URI's structure should be imposed by its scheme, not by the application using it.
	>*The `http` scheme is defined by RFC 1738, the authority (domain name) is managed by the organisation having authority on the domain*


### Percent-Encoding

Certain characters have special meanings (e.g., `/`, `?`, `#`, `&`, `=`) and are used to delimit different parts of the URI. 
If these characters are needed as part of the data (e.g., in the **path**, **query**, or **fragment**), they must be encoded using **percent-encoding** (`%XX`), where `XX` is the hexadecimal value of the character.
- `space` → `%20`
- `/` → `%2F`
- `?` → `%3F`
- `#` → `%23`
- ...

> [!example]- Percent-encoded URI
> Original: `https://www.oui.sncf/proposition/outward/train?wishId=637b88d9-c88c-4416-805d#section2`
> Encoded: `https://www.oui.sncf/proposition%2Foutward%2Ftrain?wishId=637b88d9-c88c-4416-805d#section2` 

Modern web browsers and tools automatically handle percent-encoding for user input, so you rarely need to manually encode URIs manually.


### URI References

A **Base URI** contains at least one scheme. 
An **URI Relative Reference** has an incomplete writing that lacks a scheme.
	*Relative references are used in markup languages like HTML and protocols like HTTP to reference a resource on the same server*

A **Target URI** is the result of resolving a relative reference. 
An **Absolute URI** is a complete URI without any fragment.

> [!example]- Reference examples
> 
> | **URI**                                           | **Name**                |
> | ------------------------------------------------- | ----------------------- |
> | https://tools.ietf.org/                           | Base URI & Absolute URI |
> | https://tools.ietf.org/html/rfc3986               | Base URI & Absolute URI |
> | https://tools.ietf.org/html/rfc3986#section-3.2.1 | Base URI                |
> | //tools.ietf.org/html/rfc3986#section-3.2.1       | Network-path reference  |
> | /html/rfc3986#section-3.2.1                       | Absolute-path reference |
> | rfc3986#section-3.2.1                             | Relative-path reference |
> 

As the resource is identified by resolving the relative reference, it requires prior knowledge of the base URI.

> [!example]- HTTP Resolve example 
> The protocol can describe the resolve procedure. HTTP proceeds like so:
> ```bash
> GET /document.pdf HTTP/1/1 # "/document.pdf" is the absolute path reference
> host: www.monsite.mlv
> ```
> - The scheme `http:` is deduced (since it's the HTTP protocol). 
> - The authority `//www.monsite.mlv` is provided by the `host:` field.
> - The path `/document.pdf` is provided in the `GET` request.
> The URI is resolved as: `http://www.monsite.mlv/document.pdf`
> 


### Schemes

Let's take a look at schemes that are common in the wilderness.

#### HTTP & HTTPS

They follows the same syntax: `http(s)://<host>:<port>/<path>?<query>`
- **Host**: FQDN or IP of the machine
- **Port**: 80/443 by default
- **Path**: Can be interpreted as a path in a file system, **but not necessarily**
- **Query**: Additional parameters to perform a specific request

#### FTP
*To transfer a file or list a directory's content*

Follows this syntax: `ftp://<user>:<passwd>@<machine>:<port>/<r1>/<r2>/.../name;type=<typecode>`
- **User/Password**: If not specified, the connection is anonymous.
- **Port**: Defaults to 21.
- **Path**: Represents the relative path starting from the user's directory.
- **Type**: Optional parameter (`d` for directory listing, `i` for binary, `a` for ASCII).

> [!example]-
> `ftp://ftp.fr.debian.org/debian/doc/FAQ/`: Anonymous connection on the `/debian/doc/FAQ` directory.
> `ftp://alice:pass@igm.univ-mlv.fr/W3/img.gif;type=i`: Authenticated connection (login=alice, password=pass) and downloads the `img.gif` binary file contained in the `~/W3` directory.

#### Mailto
*Represents the email address of an individual*

Follows this syntax: `mailto:<adr1>,<adr2>?<header1>=<value1>&<header2>=<value2>`

> [!example]-
> `mailto:example@example.com?subject=Hello&body=Hi%20there`: Opens an email client with the specified subject and body.

#### File
*Browse a local file system*

Follows this syntax: `file://<machine>/<path>`

> [!example]-
> `file:///home/sunless/ESIPE/class/networking.pdf`: Accesses a local file on a Unix-like system.
> `file:///C:/Program%20Files/`: Accesses a local directory on a Windows system


### Well-known URI

Well-known URIs are standardised paths used by applications to access specific resources or metadata on a website.
- `/favicon.ico`: Queried by web browsers to display the website's icon in the browser tab or bookmark list
- `/robots.txt`: Queried by search engine crawlers to determine which parts of the website should not be indexed.
```bash
User-agent: *
Disallow: /private/
Disallow: /tmp/
```

Nowadays, the `/.well-known` directory is a recommended location to place all well-known URIs (RFC 8615): 
- **`/.well-known/acme-challenge/`**: Used by the ACME protocol (e.g., Let's Encrypt) to authenticate a client requesting a TLS/SSL certificate. The client places a token in this directory, and the certificate authority verifies it by accessing the token via HTTP.
- **`/.well-known/security.txt`**: Provides security-related information, such as contact details for reporting vulnerabilities.
- **`/.well-known/change-password`**: Redirects users to a password change page, as recommended by RFC 8615.

*In reality, this standard isn't yet applied by most websites. It will come with time...*


***
## HTTP

### Fundamentals

Stateless protocol through a TCP connection, based on a request/response paradigm.

All lines ends with a CRLF (`\r\n`).
The first line of a request is known as the **Request-Line**, while called **Status-Line** for a response.
	*In both cases, it is mandatory.*

Other header fields:
- Are optional
- Allows to provide parameters
- Are on a single line
- Doesn't follow a specific order
- Can be of four types:
	1. General-header: Field common to both requests and responses (chunk usage, date, cache, keep-alive...)
	2. Request-header
	3. Response-header
	4. Entity-header: Body metadata (length, language, encoding...)

> [!example]- General-header field examples
> | **Method**        | **Role**                                                                                                  |
> | ----------------- | --------------------------------------------------------------------------------------------------------- |
> | Cache-Control     | Directives given to the cache-managing algorithm (e.g., what must and mustn't be cached, for how long...) |
> | Connection        | Indicate if persistant connections are supported                                                          |
> | Date              | Datetime of when the message was created                                                                  |
> | Pragma            | Directives all equipment receiving the message must respect (e.g., no-cache)                              |
> | Transfer-Encoding | Possible transformation experienced by the message during transfer (e.g., chunk)                          |
> | Via               | List intermediates (proxys) traversed by the message                                                      |

> [!example]- Request-header field examples
> | **Method**        | **Role**                                                                                                                                                                                                                      |
> | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | Accept            | MIME Type of medias accepted in the response, followed by<br>a `q` value going from 0 to 1, the later being the favourite and default value.<br>    *e.g., Accept: text/plain; q=0.5, text/html, text/x-dvi; q=0.8, text/x-c* |
> | Accept-charset    | Charset accepted in the response<br>*e.g., Accept-Charset: iso-8859-5, unicode-1-1;q=0.8*                                                                                                                                     |
> | Accept-Encoding   | Similar to Accept but only about the encoding<br>*e.g., Accept-Encoding: compress, gzip*                                                                                                                                      |
> | Host              | URL (Host & port) of the targeted resource                                                                                                                                                                                    |
> | If-Modified-Since | For GET; Only returns the resource if it was modified since date given in parameter<br>*Mostly for cache-purposes*                                                                                                            |
> | If-Match          | For PUT; Contains the tag (hash) of the resource to modify.<br>Server should only apply this request if the given tag doesn't match the resource's tag                                                                            |
> | Range             | Interval of data the client wishes to collect<br>*Mostly after an interruption*                                                                                                                                               |
> | Referer           | Address of the resource from which the request URI was obtained                                                                                                                                                               |
> | User-agent        | Information on the client's web browser (or robot indexation)                                                                                                                                                                 |

> [!example]- Answer-header field examples
> | **Method**    | **Role**                                                                                                                                                                                                                            |
> | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | Accept-ranges | Interval units accepted by the server. Allows the client to ask for an interval via the `range` field.<br>*Accept-ranges: bytes -> Intervals accepted, must be expressed in bytes<br>Accept-ranges: none -> Intervals not accepted* |
> | Age           | Time spent in the answering proxy's cache                                                                                                                                                                                           |
> | Location      | In case of a redirection, URL to which the request is redirected                                                                                                                                                                    |
> | Server        | Information on the software that generated the response                                                                                                                                                                             |

> [!example]- Entity-header field examples
> | **Method**       | **Role**                                                                             |
> | ---------------- | ------------------------------------------------------------------------------------ |
> | Allow            | List supported methods on the resource                                               |
> | Content-Encoding | Additional encoding applied to the response (compression)                            |
> | Content-Language | Language(s) the response's recipient should understand                               |
> | Content-Length   | Size (in bytes) of the body                                                          |
> | Content-Location | Location of the resource<br>*Useful if the resource is present at several locations* |
> | Content-Type     | Type/Sub-type of a message                                                           |
> | ETag             | Tag (hash) of the resource                                                           |
> | Expires          | Datetime at which the cached resource should be considered out of date               |
> | Last-Modified    | Datetime of last modification on the resource by the server                          |


End of the header is represented by an empty line. 
Body is optional. In general, it contains parameters when in a request, and the asked resource/document when in a response.

HTTP Methods are known as **safe** if they don't modify the resource on which it is executed (no side-effect).
	*`GET /fichier.pdf HTTP/1.1` does not alter the resource on the server.*
HTTP Methods are known as **idempotent** if they always have the same behaviour, no matter how many times we execute them.

> [!info]- Classifications for each method
> 
> | **Method** | **Safe** | **Idempotent** |
> | ---------- | -------- | -------------- |
> | GET        | y        | y              |
> | POST       | n        | n              |
> | PUT        | n        | y              |
> | DELETE     | n        | y              |
> | PATCH      | n        | n              |
> | CONNECT    | n        | n (debatable)             |
> | TRACE      | n        | n              |
> | OPTIONS    | y        | y              |
> | HEAD       | y        | y              |

*Note: Header fields about proxy forwarding are mentioned [[01 - Filtering#"Forwarded" extension|in this lecture]].
	   Deeper look at the CONNECT method is provided [[01 - Filtering#HTTP CONNECT|here]].*


### HSTS

HTTP Strict Transport Security (HSTS) allows websites to force the usage of HTTPS.
The response will contain a `Strict-Transport-Security` field, which impose usage of TLS over HTTP for a `max-age` duration.
	*e.g., Strict-Transport-Security: max-age=3600*

![[hsts.png|600]]


### Non-persistant connections

The concept is: TCP connection for a single request-response pair.
	*Client opens connection and send request; Server sends response and close connection.
	This is how HTTP/0.9 and HTTP/1.0 worked. 
	Can be used in HTTP/1.1 with the `Connection: close` field.*

Of course, this is problematic if the client has several requests to make: creates huge bottlenecks on the network, waste a lot of CPU time and memory for both ends. 
On the other hand, it avoids wasting resources on the server if the client has only one request to make.

![[non-persistant.png|300]]


### Persistant connections

Also known as **Keep-Alive**, a same TCP connection can carry several request-response pairs.
	*Applied by default since HTTP/1.1*

- When a server receives a request, he starts a timer. When it expires, the server closes the connection.
- Client can now do **pipelining** (i.e. submit several requests in a row without waiting the response for the previous)
- Client can ask for the connection to be closed by using the `Connection: close` field.
- Responses must necessarily contain the `Content-length` field announcing the body's size.

![[persistant.png|300]]


### Chunked transfer encoding

This feature is used when the resource's size isn't known before the beginning of the transfer (e.g., dynamically-generated content).
The content is split into **chunks** which are transmitted independently in several responses.

Responses will contain a `Transfer-encoding: chunked` field (and no `Content-length`).
Each chunk will have this simple structure:
1. First line: chunk size in bytes, encoded in hexadecimal. Ends with a CRLF.
2. Content of the chunk. Ends with a CRLF.

The last chunk always has a length of 0, marking the end of the chunked transfer.

> [!example]- Chunked transfer of a simple message
> This transfers: 
> Content of first chunk, next part, Bye.
> ```bash
> HTTP/1.1 200 OK
> Content-Type: text/plain
> Transfer-Encoding: chunked
> 
> # Chunk 1
> 16 # 22 bytes (because all are ASCII) -> 0x16
> Content of first chunk
> 
> # Chunk 2
> B
> , next part
> 
> # Chunk 3
> 6
> , Bye.
> 
> # Chunk 4: ends transmission
> 0
> ```


### Authentication

Framework to access protected resources on a server, relies on a challenge/response mechanism.
	*"Basic" and "Digest" procedures where the first implementations (login/password).
	All registered schemes are listed here: [HTTP Authentication Scheme Registry](https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml)*

It leverages the two following fields:
- `WWW-Authenticate: <type> realm=<realm>` from the server
- `Authorization: <type> <credentials>` from the client
*The `realm` parameter allows the user to identify the resource's protection domain, and deduce which credentials to use. (e.g., a realm for teacher-reserved resources, for admin-reserved resources...*

It usually happens like this
- The request does not contain an `Authorization` field (first connection)
- The server returns a 401, containing a `WWW-Authenticate` field specifying which scheme to use.
- The client re-sends a request, this time containing an `Authorization` field which specifies both the scheme to use, and the according credentials
- If credentials are valid for the given scheme, the server returns the resource

> [!example]- Authentication procedure example
> ```bash
> GET /Protected HTTP/1.1  # First request: Does not contain anything auth-related
> Host: www.monsite.mlv
> 
> HTTP/1.1 401 Unauthorized # First response: Provide insight on how auth works
> Date: Mon, 04 Nov 2019 15:42:26 GMT
> Server: Apache/2.4.10 (Debian)
> WWW-Authenticate: Basic realm="teacher-reserved" # Use the "basic" scheme, and this realm
> Content-Length: 462
> Content-Type: text/html; charset=iso-8859-1
> 
> GET /Protected HTTP/1.1 # Second request: Contains auth data
> Host: www.monsite.mlv
> Authorization: Basic cXVpZGVsbGV1cjplc2lwZQ== # Uses the "Basic" scheme
> 											  # and provide credentials
> HTTP/1.1 200 OK           # Second response: server successfully authenticated the client
> Date: Mon, 04 Nov 2019 15:42:33 GMT
> Server: Apache/2.4.10 (Debian)
> Last-Modified: Mon, 04 Nov 2019 14:53:19 GMT
> # Etc... body containing the resource
> ```

#### Basic

Client authenticates by providing a login/password defined for the realm the resource is binded to.
The `Authorization` field will contain the base64-encoded version of this pattern: `<login>:<password>`

> [!info]- About base64
> Defined in RFC 4648, aims to represent a sequence of bytes via 65 characters: uppercase/lowercase letters, digits, and a complementary `=` sign (for padding).
> We split the input into 24-bit blocks. Since a letter will replace 6 bits, each block will be encoded on four values.
> If the last block contains less than 24 bits, we fill the incomplete 6-bit block with 0s and replace the missing blocks with the `=` sign.
> ![[base64.png|500]]

> [!caution] Vulnerabilities
> Login and password are in cleartext, since base64 is not a cipher! If basic scheme is used, it must be encapsulated with HTTPS.
> Furthermore, client can be victim of "web spoofing", a malicious server pretending to be the legitimate one in order to steal the credentials

#### Digest

With this scheme, the password doesn't appear in cleartext anymore.
1. The server includes a `WWW-Authenticate` specifying:
	- A **nonce** (different for each response)
	- Algorithms to use for each calculation
	- The realm
2. The client makes mathematical operations (concat, hash...) on his login/password, but also on the nonce and realm provided by the server. The final result is then hashed and placed in the `Authorization` field.
3. The server retrieves the user's password in database. 
   It computes the same calculations on its side. If it matches the client's answer, the authentication is successful and a response containing a `Authentication-Info` field is forwarded.

> [!example]- Digest procedure example
> ```bash
> GET /Teacher HTTP/1.1 # First request: Does not contain anything auth-related
> host: www.website.mlv
> 
> HTTP/1.1 401 Unauthorized # First response: Provide insight on how auth works
> WWW-Authenticate: Digest  # Uses digest scheme
> realm="teacher-reserved",
> algorithm=SHA-256,
> nonce="7ypf/xlj9XXwfDPEoM4URrv/xwf94BcCAzFZH4GiTo0v",
> opaque="Fqhe/qaU925kfnzjCev0ciny7QMkPqMAFRtzCUYo5tdS"
> 
> GET /Teacher HTTP/1.1 # Second request: Contains auth data
> host: www.website.mlv
> Authorization: Digest username="sunless",
> realm="teacher-reserved",
> uri="/Teacher",
> algorithm=SHA-256,
> nonce="7ypf/xlj9XXwfDPEoM4URrv/xwf94BcCAzFZH4GiTo0v",
> nc=00000001,
> cnonce="f2/wE4q74E6zIJEtWaHKaf5wv/H5QzzpXusqGemxURZJ",
> response="753927fa0e85d155564e2e272a28d1802ca10daf4496794697cf8db5856cb6c1",
> opaque="Fqhe/qaU925kfnzjCev0ciny7QMkPqMAFRtzCUYo5tdS"
> ```

> [!caution] Vulnerabilities
> Usage of a nonce protects against replay attacks, but that's all.
> User's password is still stored in a file, and can be weak.
> This scheme doesn't protect against Man In The Middle attacks (can replace algorithm with a weaker one, can even replace the digest method by a basic one) 
> 
> Overall, digest is not secure either, and must also be used with HTTPS...


### Submit data

GET requests provide the data directly in the URL via **query string**. Its convenient for several situations, but it should never be used to transport a login or a password (as history and logs usually keep track of the URLs).
	*Be careful not to cross the maximum URL accepted size*

Methods such as POST or PUT contains the data directly in the body. The conceptual logic is the following:
- POST: The data is **intended for a resource identified by the URL**, which should normally interpret the data
- PUT: The data **is the resource to be placed in the location specified by the URL**.
If resource already exists, replace + returns 200 OK.
If resource doesn't exist, creation + returns 201 CREATED.

*PUT and DELETE requires access rights*


### Sessions

Now, we know that HTTP is stateless: server executes requests independently from the others, and do not memorise any information between them.
There are a lot of scenarios where this doesn't come handy. In modern applications, a request is often a sequel of another, requiring a context to be preserved (even offline).

Three solutions:
1. Insert information in the GET request
The server associates each user with a session id, allocates it a dedicated memory space, and dynamically modify all the website's URL to attach the correct session id. 
This way, each time the user visits a link, the GET method inserts the session id.

> [!example]
> If the user's session id is 1234, his requests will look like:
>```bash
>GET /novel/achat?id=1234&article=B567 HTTP/1.1
>host: www.library.mlv
>```

> [!fail] Not good
> This is overall a poor design choice.
> URLs are not intuitive, might exceed the size limit, and forces the user to authenticate at each connection.
> The id is lost if the user disconnects. 

2. Insert information in a form and submit it via a POST request
We can hide a session id in a hidden field of a form.
Pages will contain a hidden input, pre-filled by the website with the session id:
```html
<input type="hidden" name="client" value="a12345678901">
```

> [!fail] Not good
> Feels like this solution is built on sticks...
> We got rid of the URL length issues, but we still lose information when the client disconnects.

3. Usage of **cookies**

Cookies (RFC 6265) works like this: The server emits a response containing `Set-Cookie` field(s) carrying various information that **the client must store locally** (usually in a simple text file), and the client must bind those information in a `Cookie` field in each request. 

> [!success] Better
> Cookies are persistant, even if the user disconnects.
> Some problems persists: Cookies are exchanged in cleartext and are vulnerable to attacks (usage of HTTPS recommended). 
> The client can also disable the usage of cookies by his web browser. 

> [!example]- Simple cookie exchange example
> ```bash
> GET / HTTP/1.1
> host: www.library.mlv
> 
> HTTP/1.1 200 OK # Create two cookies, first one having an expiracy date
> Date: Fri, 01 Nov 2019 18:41:34 GMT
> Server: Apache/2.4.10 (Debian)
> Last-Modified: Fri, 01 Nov 2019 09:46:22 GMT
> ETag: "1f-59645d6b08a3d"
> Accept-Ranges: bytes
> Content-Length: 312
> Content-Type: text/html
> Set-Cookie : SID=12345; Path=/; Domain=librairie.mlv ; Expires=Wed, 09 Jun 2021 14:13:15 GMT
> Set-Cookie : lang=fr
> # (message body)
> 
> GET /livres/index.html HTTP/1.1
> host:www.librairie.mlv
> Cookie: SID=12345 ; lang=fr # Sends the two cookies
> 
> HTTP/1.1 200 OK
> Date: Fri, 01 Nov 2019 18:41:37 GMT
> Server: Apache/2.4.10 (Debian)
> # Etc.
> ```

Other methods are available, going beyond cookies:
![[cookies.png|600]]


### Cache

A storage space for HTTP responses, can either be "private" (web browser local cache), or "shared" (proxies).
The following responses can be cached:
- A "cacheable" method (e.g., GET)
- Not requiring any kind of authentication
- Header does not contain fields dismissing cache (e.g., `cache-control: no-store`)
- Header must contain a field indicating how old the answer is (e.g., `Expires`)

> [!example]- Cache management field examples
> | **Method**               | **Role**                                                                                                                                                                                    | **For?** |
> | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
> | Age                      | Time spent in the answering proxy's cache                                                                                                                                                   | Request  |
> | Cache-Control            | Directives given to the cache-managing algorithm                                                                                                                                            |          |
> | - private (resp. public) | The resource cannot be stored in a proxy's cache (public specifies the opposite)                                                                                                            | Response |
> | - max-age                | In a request, the maximum age of the response that the client accepts to receive<br>In a response, the age at which the resource should be considered out of date                           | Both     |
> | - max-stale              | Client accepts a response whose expiry date does not exceed the parameter value                                                                                                             | Request  |
> | - no-cache               | in a request, indicates that the response sent must not come from a cache<br>in a response, indicates that the response must not be sent to a client without prior validation by the server | Both     |
> | - no-store               | No information about the request nor the response can be stored in cache                                                                                                                    | Both     |
> | Expires                  | Datetime at which the cached resource should be considered out of date                                                                                                                      | Response |
> | Vary                     | Contains fields that might influence the response<br>*e.g., `Vary: Accept-langage` -> Depending on the `Accept-Language` of the request, the server will not return the same content.*      | Response |

> [!info]- About proxies
> A machine acting as an intermediate between client and server.
> It registers cacheable resources clients asks.
