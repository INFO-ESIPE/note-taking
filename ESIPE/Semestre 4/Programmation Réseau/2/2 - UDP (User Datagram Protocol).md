[[Programmation Réseau]]
***

Le protocole UDP ne prend pas en considération le principe de connection, ni, par conséquent, de vérification de l'intégrité des données. Les données peuvent arriver dans le désordre, partiellement, voire pas du tout. 

Exemple: Si on envoie P1 puis P2 à une même adresse, on peut recevoir P1 puis P2, ou P2 puis P1, ou P1 seulement ou P2 seulement ou bien rien du tout.

****
## IP (Internet Protocol)

L'accès au protocole IP est très limité en Java, avec uniquement l'accès à l'adresse et au socket de l'adresse. 
	IPv4 sur **4 octets** -> Classe `Inet4Address`
	IPv6 sur **16 octets** -> Classe `Inet6Address`

On travaillera cependant systématiquement avec la classe englobante, InetAddress.
**InetAddress n'est pas une interface car le langage est guez et a mal calculé la popularité à venir de l'IPv6**. 

L'instanciation passe par la method factory `InetAddress.getByName(String host)` car il n'y a pas de constructeur public :
![[interaddress.png]]

On obtient (noter résolution DNS dans le deuxième cas)


**Adresse de socket = Adresse IP + port** 
On représente une adresse de socket via la classe `InetSocketAddress`. 
La classe possède plusieurs constructeurs :
![[socket.png]]

Ce qui donne :
![[socket_res.png]]

****
## DatagramChannel

Les sockets UDP sont représentés par cette classe. On la créé avec la method factory `DatagramChannel.open()`. 

Cependant, une fois créé, le channel n'est pas encore **attaché à une adresse de socket.** 
On attache **(bind)** une adresse avec la méthode d'instance `bind()`.
**Note**: Passer `null` en argument de la méthode **bind()** permet de s'attacher à un port libre aléatoire sur le système (supérieur aux 1024 premiers qui sont réservés)


La création avec `open()` ainsi que l'attachement avec `bind()` **réservent des ressources systèmes** (descripteur et buffers **SND_BUF/RCV_BUF** associés au port réservé). Il faut libérer ces ressources ainsi : 
- `dc.close();` 
- Soit en ouvrant le dc dans un try-with-resources :
![[try.png]]


Pour envoyer un paquet, il faut : 
- Les données (ByteBuffer) 
- La socket adresse du destinataire (InetSocketAddress)

On passe ces valeurs en argument de la méthode `send(Bytebuffer buffer, SocketAddress dest)` du channel.


**Pour recevoir un paquet, il faut Un ByteBuffer avec un espace nécessaire suffisant (il faut donc le prévoir.**

L'échange se passe ainsi:
![[exchange.png]]

