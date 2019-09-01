---
layout: post
author: junowa
---

Comment optimiser les performances de NGINX pour délivrer le contenu.

## sendfile

Par defaut, pour transmettre un fichier, NGINX copie ce fichier dans un buffer avant de l'envoyer. La directive **sendfile** permet de copier directement les données d'un file descriptor vers un autre en élimant l'opération coûteuse de copie en mémoire.

Afin de limiter la charge du process worker, il est possible de limiter la taille des données transmises par appel à sendfile() avec la directive **sendfile_max_chunk**.
 
```
location /mp3 {
    sendfile           on;
    sendfile_max_chunk 1m;
    #...
}
```

**sendfile** est un appel système. Il est donc exécuté en kernel space, évitant le changement de contexte, ce qui permet de gagner en performances. Il se substitue donc aux appels **read/write** pour permet de faire du zero-copy, c-a-d. d'écrire directement depuis le buffer kernel vers le block device de la carte réseau grâce au DMA.

Si en revanche, vous n’utilisez nginx qu’en guise de reverse proxy devant un ou plusieurs serveurs d’application, sendfile ne vous servira absolument à rien, sauf, une fois encore, si vous mettez en place du micro caching...

## tcp_nopush

Utiliser conjointement avec **sendfile**, **tcp_nopush** force l'envoie de l'ensemble des données de la réponse HTTP en un seul paquet.

**sendfile** est utilisé pour traiter spécifiquement de la transmission des fichiers.

**tcp_nopush** (qui correspond à **tcp_cork** sous linux, _man tcp_), est une option de la pile TCP pour n'envoyer que des paquets remplis.

Dans NGINX, **tcp_nopush**  ne peut être utilisé sans **sendfile**.

```
location /mp3 {
    sendfile   on;
    tcp_nopush on;
    #...
}
```

## tcp_nodelay

L'algorithme de Nagle regroupe un certain nombre de petits paquets en un paquet plus gros, puis envoie le paquet avec un delais de 200ms (delay ack).

Nagle algorithm:
```
if [ data > MSS ]
    send(data)
else
    wait until ACK for previously sent data and accumulate data in send buffer (data)
    And after receiving the ACK send(data)
```

Utile pour résoudre les problémes de performances pour des petits paquets sur des réseaux à faible bande passante, mais plus vraiment d'actualité. En plus, le delais peut affecter un certain nombre d'applications (ssh, jeux, trading,...).

Par défault dans NGINX, **tcp_nodelay** est donc activé (Nagle désactivé).

```
location /mp3  {
    tcp_nodelay       on;
    keepalive_timeout 65;
    #...
}
```

Nginx utilise l’option TCP_NODELAY sur les connexions keepalive HTTP, c’est à dire des sockets qui restent ouvertes un certain temps après avoir terminé d’envoyer du contenu. Le keepalive évite d’ouvrir une nouvelle connexion et de rejouer un 3 ways handshake chaque fois qu’une requête HTTP est terminée. Cela permet de gagner du temps et d’économiser des sockets, puisqu’elles ne passent pas en FIN_WAIT à la fin d’un transfert. Connection: Keep-alive est une option en HTTP 1.0, et le comportement par défaut d’HTTP 1.1.

Quelle différence alors entre l'algorithme de Nagle et **tcp_nopush**. Les deux techniques accumulent des données pour réduire l'overhead réseau. Mais à la différence de Nagle qui attend le ACK delay (200ms), pour envoyer les données, **tcp_nopush** envoie les données dès que son buffer est rempli.


## Optimiser la Backlog Queue

Lorsqu'une connexion est établie, elle est insérée dans la "listen queue" de la socket en écoute. Lorsque la charge est nominale, la queue est presque vide voire vide. Lorsque la charge augmente, la taille de la queue augemente et peut entrainer des problèmes de performance, de latence, de perte de paquets.

Pour afficher la listen queue:
```
$ ss -lt
```

```
State      Recv-Q Send-Q       Local Address:Port     Peer Address:Port                                
LISTEN     0      128          *:http                 *:*                    
LISTEN     0      128          *:ssh                  *:*                    
LISTEN     0      128          *:https                *:*
```
Si la socket est dans l'état _listening_, Recv-Q est la taille courante de la queue, et Send-Q est la taille configurée du backlog.

Il est possible de modifier la taille de la backlog queue via le paramètre kernel:
```
$ sudo sysctl -w net.core.somaxconn=4096
```

Dans ce cas, il fait aussi l'indiquer à NGINX:
```
server {
    listen 80 backlog=4096;
    # ...
}
```


## References
https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/

https://thoughts.t37.net/optimisations-nginx-bien-comprendre-sendfile-tcp-nodelay-et-tcp-nopush-2ab3f33432ca

https://stackoverflow.com/questions/3761276/when-should-i-use-tcp-nodelay-and-when-tcp-cork
