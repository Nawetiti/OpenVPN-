<img src="assets/OpenVPN-logo.png" alt="Logo" width="150" style="vertical-align: middle; margin-right: 15px;"> TP-HAPROXY

<div style="border-left: 4px solid orange; padding-left: 12px;">
  <table>
    <tr>
      <td style="vertical-align: middle;">
        Iwaniec<br>
        Hugo<br>
        BTS SIO
      </td>
      <td style="vertical-align: middle; text-align: right;">
        <img src="assets/file.png" alt="Logo SIO" width="80">
      </td>
    </tr>
  </table>
</div>




---

**OpenVPN** sur pfSense est un service VPN SSL/TLS intégré qui permet de créer un tunnel chiffré entre des clients distants (PC, smartphones, sites distants) et le pare-feu. 

Grâce à ce tunnel, les utilisateurs peuvent accéder au réseau local derrière pfSense comme s’ils étaient sur place, ce qui est idéal pour le télétravail ou l’administration distante. 

OpenVPN fonctionne avec une autorité de certification et des certificats serveur/client.

> [!summary] Sommaire
> 1. Mise en place des serveurs WEB
> 2. Mise en place du serveur HAProxy
> 3. Phase de test

---
## Mise en place sur PFSENSE


> [!example] Introduction 
> Depuis un PFSENSE on se retrouve avec 2 interfaces : 
> - WAN : 192.168.20.227
> - LAN : 192.168.90.35
>   
>   
> Nous allons configurer un accès VPN nous permettant l'accès via un tunnel VPN à notre LAN depuis le WAN. 
> 
> <img src="assets/OpenVPN-schéma.png" alt="OpenVPN-schéma">
> 

## Création des certificats :

On crée notre autorité de certification depuis System/Certificate/Authorities :

<img src="assets/OpenVPN-Autorité de certif.png" alt="OpenVPN-Autorité de certif">

À l'aide de cette autorité on crée un certificat serveur :

<img src="assets/OpenVPN-certif-serveur.png" alt="OpenVPN-certif-serveur">

Enfin on crée un utilisateur pour se connecter au VPN depuis System/User Manager / Users :

<img src="assets/OpenVPN-User.png" alt="OpenVPN-User">

> [!important]
> Il faut penser à cocher la case certificate 
> 
> <img src="assets/OpenVPN-Certifié user.png" alt="OpenVPN-Certifié user">
> 
> 
> <img src="assets/OpenVPN-certif-user.png" alt="OpenVPN-certif-user">
> 
> Cela permet de créer un certificat pour l'utilisateur. 
> 


## Mise en place d’OpenVPN : 

Depuis VPN/OpenVPN/Servers, on ajoute un nouveau serveur : 


> [!todo] Configuration 
> Pour la configuration :
> 
> On met en place un chiffrement SSL/TLS et une authentification de l'utilisateur. 
> 
> <img src="assets/OpenVPN-serveurmode.png" alt="OpenVPN-serveurmode">
> 
> Depuis l'interface WAN, on modifie le port par défaut. 
> 
> <img src="assets/OpenVPN-Interface port.png" alt="OpenVPN-Interface port">
> 
> On ajoute notre autorité de certification et notre certificat **Serveur**.
> 
> <img src="assets/OpenVPN-config-Certif.png" alt="OpenVPN-config-Certif">
> 
> On configure l'adresse de notre tunnel. 
> 
> <img src="assets/OpenVPN-config-addr-tunnel.png" alt="OpenVPN-config-addr-tunnel">
> 
> L'option permet de forcer l'intégralité du trafic à passer par le VPN, si décochée les flux passent à la fois sur le VPN mais aussi sur le réseau du client.
> 
> <img src="assets/OpenVPN-config-forcetunnel.png" alt="OpenVPN-config-forcetunnel">
> 
> On configure l'adresse de notre LAN, on peut ajouter plusieurs réseaux à l'aide d'une **,**
> IP1**,** IP2
> 
> <img src="assets/OpenVPN-config-addr-lan.png" alt="OpenVPN-config-addr-lan">
> 
> En cochant cette case, on autorise le client à rester connecté même si son IP change pendant l'utilisation du VPN (utile si l'utilisateur se déplace).
> 
> <img src="assets/OpenVPN-config-dynamic IP.png" alt="OpenVPN-config-dynamic IP">
> 
> Cette option crée des réseaux isolés par utilisateur, afin de limiter les échanges entre les utilisateurs connectés simultanément au VPN par la mise en place d'adresses en /30 par utilisateur.
> L'inconvénient est qu'elle divise donc par 4 les adresses disponibles sur notre tunnel.
> 
> <img src="assets/OpenVPN-config ip30.png" alt="OpenVPN-config ip30">
> 
> 
> 
> 
> 

Depuis Package Manager on installe le paquet OpenVPN :

Depuis OpenVPN une nouvelle option a été ajoutée **Client Export***.

<img src="assets/OpenVPN-Clientexport.png" alt="OpenVPN-Clientexport">

On se rend dessus et on le configure : 

On sélectionne la manière dont on se connecte au VPN, par l'adresse publique ou par le nom de domaine... 

<img src="assets/OpenVPN-export-Hostnameresolution.png" alt="OpenVPN-export-Hostnameresolution">

Enfin en bas de la configuration, on retrouve nos accès VPN à télécharger, ils sont à transférer à nos clients.

<img src="assets/OpenVPN-Client-install.png" alt="OpenVPN-Client-install">



## Règle du Firewall 

On crée une règle WAN afin d'autoriser la connexion depuis le VPN.

On sélectionne le protocole UDP.  

<img src="assets/OpenVPN-rules-udp.png" alt="OpenVPN-rules-udp">

La source, on ne la connaît pas, par contre on ajoute notre destination, donc l'adresse de notre WAN ainsi que le port qu'on a préalablement configuré. 

<img src="assets/OpenVPN-rules-source-destination.png" alt="OpenVPN-rules-source-destination">



## Test de connexion 

Depuis le site d’OpenVPN, on télécharge le client GUI et on l'installe : [https://openvpn.net/community/](https://openvpn.net/community/)  
On récupère nos accès.

On importe le fichier en .ovpn dans la configuration.

<img src="assets/OpenVPN-GUI importer.png" alt="OpenVPN-GUI importer">

On peut maintenant se connecter au VPN avec succès.

<img src="assets/OpenVPN-connexion.png" alt="OpenVPN-connexion">

On retrouve bien dans nos cartes réseau l'adresse de notre tunnel. 

<img src="assets/OpenVPN-IPConfig.png" alt="OpenVPN-IPConfig">

Depuis l'onglet Rules/OpenVPN, on ajoute des règles afin de limiter les accès depuis le VPN.

<img src="assets/OpenVPN-Regle VPN.png" alt="OpenVPN-Regle VPN">

Cela permet de choisir ce que l'on autorise comme accès depuis le VPN, étant en labo j'ai tout laissé disponible pour tester. 

> [!check] Vérification
> Je cherche donc à accéder à mon VPN depuis mon PC dans le WAN.
> 
> <img src="assets/OpenVPN-accès.png" alt="OpenVPN-accès">
> 
> On accède bien au pfSense ainsi qu'à l'ensemble des services présents dans le LAN.
> 
> <img src="assets/OpenVPN-Accès VPN.png" alt="OpenVPN-Accès VPN">
