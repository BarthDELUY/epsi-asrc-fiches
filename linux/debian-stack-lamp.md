# Installer une stack LAMP sous Debian Linux

## Introduction

La stack LAMP, pour Linux/Apache/MariaDB/PHP est l'ensemble de technologies nécessaires pour faire fonctionner la majorité des logiciels SaaS open source disposant d'un accès Web.

Cette fiche vous présente l'installation et la configuration d'une stack LAMP basique, ainsi que des pointeurs vers des ressources pour aller plus loin.

## Théorie sur la stack LAMP

Pour traiter une requête web de faon dynamique, le serveur doit pouvoir réaliser un traitement à l'aide d'un langage de programmation côté serveur.

Le langage PHP représente aujourd'hui le langage le plus utilisé dans ce rôle, loin devant Java et Javascript avec nodeJS).

Lorsque la requête arrive sur le serveur, elle doit être traitée par un serveur web (ici Apache), qui la relaie à un serveur d'application (PHP), qui lui-même peut se connecter à une base de données (MariaDB).

On utilise le mode FPM de PHP car il permet d'isoler les processus Apache et PHP, pour leur allouer des droits et des ressources différents. Il permet également de créer plusieurs groupes de processus PHP (appelés des "pools"), chacun avec ses propres droits et ses propres ressources. Par exemple, pour Wordpress, on pourra affecter un pool avec beaucoup de très petits processus pour la partie publique du site, et un pool avec de processus mais plus de ressources pour le panneau d'administration. Le pool "public" aura également les droits en lecture sur les répertoires, tandis que le pool 'admin' aura lui des droits plus larges.

Utiliser le mode FPM permet également de faire fonctionner PHP sur un serveur différent de celui qui héberge Apache.

## Installation d'apache

Rien de plus simple, `apt install apache2` !

Apache crée ses fichiers de configuration dans le dossier `/etc/apache2/`, et va lire son site par défaut dans le dossier `/var/www/html/` . Par défaut, Apache fonctionne avec l'utilisateur `www-data:www-data` (uid 33) sous Debian et ses dérivées.

Tout administrateur Linux qui se respecte se doit de connaître la notion de VirtualHost sous Apache (https://httpd.apache.org/docs/2.4/vhosts/examples.html) ou de server block sous nginx (https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/) (ces deux noms différents recouvrent le même concept, celui d'associer une URL à un répertoire en particulier sur le disque).  
En parallèle, tout administrateur Windows qui se respecte se doit de connaître la notion de Site et de Répertoire Virtuel sous IIS (https://learn.microsoft.com/fr-fr/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis)

## Installation de MariaDB

MariaDB est un fork libre de MySQL, réalisé par les développeurs originaux de MySQL, lors du rachat de Sun par Oracle en 2008.

Pour l'installer, `apt install mariadb-server`

Il est possible après l'installation de lancer un script automatique pour la sécuriser, avec la commande `mysql_secure_installation`.

Par défaut, mariadb installe ses fichiers de configuration dans le dossier `/etc/mysql/`, et héberge ses données dans le dossier `/var/lib/data/mysql/` ( notez la parenté entre mariadb et mysql, mariadb a gardé les mêmes commandes et les mêmes chemins que sont aïeul).

Plus d'informations sur la configuration de MariaDB : https://mariadb.com/kb/en/configuring-mariadb-with-option-files/ , https://mariadb.com/get-started-with-mariadb/

## Installation et paramétragee de PHP

Debian est une distribution extrêmement stable : une fois qu'ils ont rejoint la distribution, les paquets ne reçoivent que les mises à jour de sécurité ; les nouvelles versions doivent attendre la version suivante de la distribution.  
Il est toutefois possible d'avoir des paquets à jour en utilisant le système de `backports`, spécifique à Debian.

Dans le cadre de cette fiche, nous allons utiliser une méthode alternative, qui est également utilisée pour installer quantité d'autres paquets (Edge, Chrome, VS Code, etc) : nous allons ajouter des dépôts supplémentaires à la liste des dépôts de notre distribution, puis installer les paquets que nous voulons à partir de ces déôts.

Dans le cadre de PHP, il faut utiliser les dépôts Sury, car ils sont maintenus par un des développeurs du langage PHP.

### Paramétrage des dépôts Sury

Pour faire l'installation, suivre les commandes dans https://packages.sury.org/php/README.txt

Il est même possible d'exécuter directement ce script :

```
apt install wget
cd /tmp
wget -O php.sh https://packages.sury.org/php/README.txt
chmod +x php.sh
./php.sh
```

La commande wget permet de télécharger un fichier depuis Internet, et son flag `-O` permet d'indiquer sous quel nom l'enregistrer (s'il n'est pas précisé, wget garde le nom d'origine, mais là, ce serait dommage d'avoir un script d'installation qui s'appelle Readme.txt...)

### Installation de PHP

Pour installer PHP une fois que les dépôts Sury sont activés, il suffit de lancer la commande `apt install php8.3-fpm` pour installer la version 8.3 de php

Attention : une fois installé, PHP nous informe qu'il faut lancer deux commandes afin de permettre à Apache de communiquer avec lui :

`a2enmod proxy_fcgi setenvif` pour activer 2 modules pour Apache : le module proxy_fcgi pour qu'Apache puisse rediriger les requêtes vers php-fpm, et setenvif pour lui transmettre la configuration d'environnement.

`a2enconf php8.3-fpm` pour activer la configuration de PHP au sein d'Apache (la configuration du renvoi des requêtes depuis Apache vers PHP)

Une fois que les deux commandes ont été lancées, il faut redémarrer apache par `systemctl restart apache2` (il faut redémaarrer le service apache2 lorsqu'on ajoute ou retire des modules apache, tandis qu'on peut simplement le recharger lorsqu'on modifie sa configuration).

Par défaut sous Debian, PHP s'installe dans /etc/php/8.3/fpm/ (pour la version 8.3 en FPM).

### Test de PHP

Créez le fichier `/var/www/html/info.php` avec le contenu suivant :

```php
<?php phpinfo();
```

Lorsque vous essayez d'accéder à la page depuis votre navigateur, vous devez voir une page d'informations contenant toute la configuration de PHP sur le serveur.

Le fichier de configuration de PHP : https://www.php.net/manual/fr/configuration.file.php

La configuration des pools FPM : https://www.php.net/manual/en/install.fpm.configuration.php

### Fonctionnement de PHP sous Debian

Sous Debian, le package PHP est découpé par fonctionnalités, que l'on appalle *extensions*. Pour se connecter à une base mariadb, il faut par exemple installer le paquet php8.3-mysql.

Lorsque vous aurez besoin d'installer un logiciel en PHP (comme Wordpress ou GLPI par exemple), la documentation de ce logiciel vous donnera la liste des extensions nécessaires à son bon fonctionnement. Il faudra les installer pour le faire fonctionner.

# Voilà, vous disposez d'uns stack lamp fonctionnelle !
